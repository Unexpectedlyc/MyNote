# 流式传输DEMO

## JAVA版

```java
@PostMapping(value = "/chat/kb_chat",produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public SseEmitter knowledgeBaseChat(@RequestBody String paramJsonStringify,
                                    @RequestHeader String uuid) throws Exception {
    log.info("uuid为：{}", uuid);

    SseEmitter emitter = new SseEmitter();
    final AtomicBoolean completed = new AtomicBoolean(false); // 自定义完成标志

    Thread thread =
        new Thread(
        () -> {
            try {
                HttpURLConnection connection = getHttpURLConnection();

                try (OutputStream os = connection.getOutputStream()) {
                    os.write(paramJsonStringify.getBytes(StandardCharsets.UTF_8));
                    os.flush();
                }

                int responseCode = connection.getResponseCode();
                if (responseCode == HttpURLConnection.HTTP_OK) {
                    BufferedReader reader =
                        new BufferedReader(
                        new InputStreamReader(
                            connection.getInputStream(), StandardCharsets.UTF_8));
                    String line;

                    while ((line = reader.readLine()) != null && !completed.get()) {
                        if (line.isEmpty()) {
                            continue;
                        }
                        // 处理每一行的JSON数据,加入uuid字段
                        try {
                            JsonNode jsonNode = objectMapper.readTree(line.replaceAll("data: ", ""));
                            ((ObjectNode) jsonNode).put("uuid", uuid); // 添加uuid字段
                            line = objectMapper.writeValueAsString(jsonNode); // 将修改后的JSON对象转换回字符串
                        } catch (Exception e) {
                            log.error("Error processing JSON line: ", e);
                            //emitter.completeWithError(e);
                            //completed.set(true);
                            continue;
                        }
                        log.info("Stream data received: {}", line);
                        emitter.send(SseEmitter.event().data(line).id("").name("message"));
                    }
                    reader.close();
                    emitter.complete();
                    completed.set(true); // 设置完成标志
                } else {
                    emitter.completeWithError(
                        new RuntimeException("Third-party API returned an error: " + responseCode));
                    completed.set(true); // 设置完成标志
                }
            } catch (Exception e) {
                emitter.completeWithError(e);
                log.error("Error occurred during stream processing: ", e);
                completed.set(true); // 设置完成标志
            }
        });

    thread.start();

    return emitter;
}

private HttpURLConnection getHttpURLConnection() throws IOException {
    URL url = new URL(chatServerUrl + "/chat/kb_chat");
    HttpURLConnection connection = (HttpURLConnection) url.openConnection();
    connection.setRequestMethod("POST");
    connection.setRequestProperty("Content-Type", "application/json; charset=utf-8");
    connection.setRequestProperty("Accept", "text/event-stream");
    connection.setDoOutput(true);
    connection.setDoInput(true);
    connection.setChunkedStreamingMode(0); // 禁用缓冲
    return connection;
}
```

## JS版本

```javascript
fetch(url, {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        Authorization: 'Bearer ' + token,
        uuid: this.curEyeApp.appUuid
    },
    body: JSON.stringify(data)
})
    .then(response => {
    if (!response.ok) {
        that.updateItemText(
            that.currentConversations.contents[that.currentConversations.contents.length - 1],
            that.$t('appBulider.manager.neterror')
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
            if (!that.isDialoging || done) {
                that.curInputIcon = that.inputIcon.unallowsend
                that.isDialoging = false
                if (done) {
                    that.updateConversation(that.currentConversations)
                    that.streamText = that.streamText.replace('\n', '\n\n')
                }
                return
            }
            const chunk = decoder.decode(value, { stream: true })

            // 解析所有顶层JSON对象
            const dataList = parseAllTopLevelJSONObjects(chunk)

            dataList.forEach(streamData => {
                if (isFirstChunk) {
                    if (streamData.docs) {
                        streamData.docs.forEach(item => {
                            if (item.includes('span')) {
                                const index = that.currentConversations.contents.length - 1
                                that.currentConversations.contents[index].docs = item
                            } else {
                                that.streamText += item + '\n'
                                that.updateItemText(
                                    that.currentConversations.contents[that.currentConversations.contents.length - 1],
                                    that.streamText
                                )
                            }
                        })
                    }
                    isFirstChunk = false
                }

                try {
                    if (streamData.choices && streamData.choices[0] && streamData.choices[0].delta) {
                        that.streamText += streamData.choices[0].delta.content || ''
                        that.updateItemText(
                            that.currentConversations.contents[that.currentConversations.contents.length - 1],
                            that.streamText
                        )
                    }
                } catch (err) {
                    console.error('Error processing JSON data:', err)
                }
            })

            readStream() // 继续读取下一个块
        })
            .catch(error => {
            console.error('Error reading stream:', error)
            that.curInputIcon = that.inputIcon.unallowsend
            that.isDialoging = false
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

