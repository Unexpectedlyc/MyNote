# 流式传输DEMO

## JAVA版

```java
public DeferredResult<ResponseBodyEmitter> knowledgeBaseChat(@RequestBody String paramJsonStringify) throws IOException {
    //String url = chatServerUrl + "/chat/kb_chat";

    DeferredResult<ResponseBodyEmitter> deferredResult = new DeferredResult<>();
    // 创建 URL
    URL url = new URL(chatServerUrl + "/chat/kb_chat");

    // 打开连接
    HttpURLConnection connection = (HttpURLConnection) url.openConnection();
    connection.setRequestMethod("POST");
    connection.setRequestProperty("Content-Type", "application/json; charset=utf-8");
    connection.setDoOutput(true); // 设置为 POST 请求
    connection.setDoInput(true); // 允许读取响应
    // 构建请求体
    try (OutputStream os = connection.getOutputStream()) {
        os.write(paramJsonStringify.getBytes(StandardCharsets.UTF_8)); // 写入请求体
        os.flush();
    }
    // 检查响应状态码
    int responseCode = connection.getResponseCode();
    if (responseCode == HttpURLConnection.HTTP_OK) {
        ResponseBodyEmitter emitter = new ResponseBodyEmitter();
        InputStream inputStream = connection.getInputStream();

        Thread thread = new Thread(() -> {
            byte[] buffer = new byte[4096]; // 缓冲区大小
            int length;

            try {
                while ((length = inputStream.read(buffer)) != -1) {
                    // 只发送实际读取的数据长度
                    byte[] dataToSend = new byte[length];
                    System.arraycopy(buffer, 0, dataToSend, 0, length);
                    emitter.send(dataToSend);
                }
                inputStream.close();
                emitter.complete(); // 完成发送
            } catch (IOException e) {
                log.error("错误信息为： " + e);
            }
        });

        thread.start();

        deferredResult.setResult(emitter);
    } else {
        // 如果第三方服务返回错误状态码
        deferredResult.setErrorResult(new RuntimeException("Third-party API returned an error: " + responseCode));
    }

    return deferredResult;
}
```

JS版本

```javascript
fetch(url, {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(data)
})
    .then(response => {
    if (!response.ok) {
        that.updateItemText(
            that.currentConversations.contents[that.currentConversations.contents.length - 1],
            '网络错误，请检查服务是否正常'
        )
        that.curInputIcon = that.inputIcon.unallowsend
        that.isDialoging = false
        // 如果HTTP状态码不是200-299，则抛出错误
        return response.text().then(err => Promise.reject(new Error(err)))
    }

    const reader = response.body.getReader()
    const decoder = new TextDecoder('utf-8')
    let isFirstChunk = true
    function readStream() {
        reader
            .read()
            .then(({ done, value }) => {
            if (!that.isDialoging) {
                that.curInputIcon = that.inputIcon.unallowsend
                return
            }
            if (done) {
                that.updateConversation(that.currentConversations)
                that.curInputIcon = that.inputIcon.unallowsend
                that.isDialoging = false
                that.streamText.replace('\n', '\n\n')
                // console.log('Stream complete.')
                return
            }
            const chunk = decoder.decode(value, { stream: true })
            let streamData = chunk.replace('data:', '')
            if (isFirstChunk) {
                streamData = JSON.parse(streamData)
                for (const item of streamData.docs) {
                    that.streamText = that.streamText + item
                    that.updateItemText(
                        that.currentConversations.contents[that.currentConversations.contents.length - 1],
                        that.streamText
                    )
                }
                isFirstChunk = false
            }
            // 实时输出接收到的文本块

            try {
                streamData = JSON.parse(streamData)
                that.streamText = that.streamText + streamData.choices[0].delta.content || ''
                that.updateItemText(
                    that.currentConversations.contents[that.currentConversations.contents.length - 1],
                    that.streamText
                )
            } catch (err) {
                console.log(streamData)
            }
            // 继续读取下一个块
            readStream()
        })
            .catch(error => {
            console.error('Error reading stream:', error)
        })
    }

    readStream() // 开始读取流

    return 'success'
})
    .catch(error => {
    console.error('There was an error!', error)
    throw error // 确保错误被传递给任何外部catch块
})
```

