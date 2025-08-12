# vue结合springboot通过websocket连接实现前端展示数据实时更新

### websocket入门

1. 引入websocket相关依赖

   ```xml
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-websocket</artifactId>
           </dependency>
           <dependency>
               <groupId>cn.hutool</groupId>
               <artifactId>hutool-all</artifactId>
               <version>5.1.0</version>
           </dependency>
           <dependency>
               <groupId>com.alibaba</groupId>
               <artifactId>fastjson</artifactId>
               <version>2.0.22</version>
           </dependency>
   ```

   

2. 新建websocket配置文件

   ```java
   
   @Configuration
   public class WebSocketConfig {
       @Bean
       public ServerEndpointExporter serverEndpointExporter(){
           return new ServerEndpointExporter();
       }
   }
   ```

3. 新建websocket服务类

   ```java
   /**
    * @ClassName: 开启WebSocket支持
    */
   @ServerEndpoint("/websocket/{userId}")
   @Component
   public class WebSocketServer {
   
       static Log log = LogFactory.get(WebSocketServer.class);
       /**
        * 静态变量，用来记录当前在线连接数。应该把它设计成线程安全的。
        */
       private static int onlineCount = 0;
       /**
        * concurrent包的线程安全Set，用来存放每个客户端对应的MyWebSocket对象。
        */
       private static ConcurrentHashMap<String, WebSocketServer> webSocketMap = new ConcurrentHashMap<>();
       /**
        * 与某个客户端的连接会话，需要通过它来给客户端发送数据
        */
       private Session session;
       /**
        * 接收userId
        */
       private String userId = "";
   
       /**
        * 连接建立成功调用的方法
        */
       @OnOpen
       public void onOpen(Session session, @PathParam("userId") String userId) {
           this.session = session;
           this.userId = userId;
           if (webSocketMap.containsKey(userId)) {
               webSocketMap.remove(userId);
               webSocketMap.put(userId, this);
               //加入set中
           } else {
               webSocketMap.put(userId, this);
               //加入set中
               addOnlineCount();
               //在线数加1
           }
   
           log.info("用户连接:" + userId + ",当前在线人数为:" + getOnlineCount());
   
           try {
               sendMessage("连接成功");
           } catch (IOException e) {
               log.error("用户:" + userId + ",网络异常!!!!!!");
           }
       }
   
       /**
        * 连接关闭调用的方法
        */
       @OnClose
       public void onClose() {
           if (webSocketMap.containsKey(userId)) {
               webSocketMap.remove(userId);
               //从set中删除
               subOnlineCount();
           }
           log.info("用户退出:" + userId + ",当前在线人数为:" + getOnlineCount());
       }
   
       /**
        * 收到客户端消息后调用的方法
        *
        * @param message 客户端发送过来的消息
        */
       @OnMessage
       public void onMessage(String message, Session session) {
           log.info("用户消息:" + userId + ",报文:" + message);
           //可以群发消息
           //消息保存到数据库、redis
   
           if (! StringUtils.isEmpty(message)) {
               try {
                   //解析发送的报文
                   JSONObject jsonObject = JSON.parseObject(message);
   
               } catch (Exception e) {
                   e.printStackTrace();
               }
           }
       }
   
       /**
        * @param session
        * @param error
        */
       @OnError
       public void onError(Session session, Throwable error) {
           log.error("用户错误:" + this.userId + ",原因:" + error.getMessage());
           error.printStackTrace();
       }
   
       /**
        * 实现服务器主动推送
        */
       public void sendMessage(String message) throws IOException {
           this.session.getBasicRemote().sendText(message);
       }
   
   
       /**
        * 实现服务器主动推送
        */
       public static void sendAllMessage(String message) throws IOException {
           ConcurrentHashMap.KeySetView<String, WebSocketServer> userIds = (ConcurrentHashMap.KeySetView<String, WebSocketServer>) webSocketMap.keySet();
           for (String userId : userIds) {
               WebSocketServer webSocketServer = webSocketMap.get(userId);
               webSocketServer.session.getBasicRemote().sendText(message);
               System.out.println("webSocket实现服务器主动推送成功userIds====" + userIds);
           }
       }
   
       /**
        * 发送自定义消息
        */
       public static void sendInfo(String message, @PathParam("userId") String userId) throws IOException {
           log.info("发送消息到:" + userId + "，报文:" + message);
           if (!StringUtils.isEmpty(message) && webSocketMap.containsKey(userId)) {
               webSocketMap.get(userId).sendMessage(message);
           } else {
               log.error("用户" + userId + ",不在线！");
           }
       }
   
       public static synchronized int getOnlineCount() {
           return onlineCount;
       }
   
       public static synchronized void addOnlineCount() {
           WebSocketServer.onlineCount++;
       }
   
       public static synchronized void subOnlineCount() {
           WebSocketServer.onlineCount--;
       }
   }
   
   
   ```

4. 新建websocket测试

   ```java
   @RestController
   @RequestMapping("/test")
   public class Test {
   
       //设置定时十秒一次
       @Scheduled(cron = "0/10 * * * * ?")
       @PostMapping("/send")
       public String sendMessage() throws Exception {
           Map<String,Object> map = new HashMap<>();
   
           // 获取当前日期和时间
           LocalDateTime nowDateTime = LocalDateTime.now();
           DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
           System.out.println(dateTimeFormatter.format(nowDateTime));
           map.put("server_time",dateTimeFormatter.format(nowDateTime));
           map.put("server_code","200");
           map.put("server_message","这是服务器推送到客户端的消息哦！！");
           JSONObject jsonObject =  new JSONObject(map);
           WebSocketServer.sendAllMessage(jsonObject.toString());
           return jsonObject.toString();
       }
   
   }
   ```

5. 启动类中配置websocket注解

   ```java
   @ServletComponentScan //webSocket
   public class WorkReviewApplication {
       public static void main(String[] args) throws FileNotFoundException, IOException {
           SpringApplication.run(WorkReviewApplication.class, args);
       }
   }
   ```

   

6. 测试websocket

   测试网站 [http://coolaf.com/tool/chattest]

   ![image.png](https://segmentfault.com/img/bVdb4To)

   websocket端口默认为spring boot启动端口，接口路由为websocketServer中@ServerEndpoint("/websocket/{userId}")所配置

7. 前端vue对接（websocket重连策略配置）

   ```js
   // 实现WebSocket连接失败后3分钟内尝试重连3次的功能，可以自行设置重连策略，
   // 包括重连的间隔时间、尝试次数以及总时间限制。
   
   /**
    * @param {string} url  Url to connect
    * @param {number} maxReconnectAttempts Maximum number of times
    * @param {number} reconnect Timeout
    * @param {number} reconnectTimeout Timeout
    *
    */
   class WebSocketReconnect {
     constructor (url, maxReconnectAttempts = 3, reconnectInterval = 20000, maxReconnectTime = 180000) {
       this.url = url
       this.maxReconnectAttempts = maxReconnectAttempts
       this.reconnectInterval = reconnectInterval
       this.maxReconnectTime = maxReconnectTime
       this.reconnectCount = 0
       this.reconnectTimeout = null
       this.startTime = null
       this.socket = null
   
       this.connect()
     }
   
     // 连接操作
     connect () {
       console.log('connecting...')
       this.socket = new WebSocket(this.url)
   
       // 连接成功建立的回调方法
       this.socket.onopen = () => {
         console.log('WebSocket Connection Opened!')
         this.clearReconnectTimeout()
         this.reconnectCount = 0
       }
       // 连接关闭的回调方法
       this.socket.onclose = (event) => {
         console.log('WebSocket Connection Closed:', event)
         this.handleClose()
       }
       // 连接发生错误的回调方法
       this.socket.onerror = (error) => {
         console.error('WebSocket Connection Error:', error)
         this.handleClose() // 重连
       }
     }
   
     // 断线重连操作
     handleClose () {
       if (this.reconnectCount < this.maxReconnectAttempts && (this.startTime === null ||
         Date.now() - this.startTime < this.maxReconnectTime)) {
         this.reconnectCount++
         console.log(`正在尝试重连 (${this.reconnectCount}/${this.maxReconnectAttempts})次...`)
         this.reconnectTimeout = setTimeout(() => {
           this.connect()
         }, this.reconnectInterval)
   
         if (this.startTime === null) {
           this.startTime = Date.now()
         }
       } else {
         console.log('超过最大重连次数或重连时间超时，已放弃连接！Max reconnect attempts reached or exceeded max reconnect time. Giving up.')
         this.reconnectCount = 0 // 重置连接次数0
         this.startTime = null // 重置开始时间
       }
     }
   
     // 清除重连定时器
     clearReconnectTimeout () {
       if (this.reconnectTimeout) {
         clearTimeout(this.reconnectTimeout)
         this.reconnectTimeout = null
       }
     }
   
     // 关闭连接
     close () {
       if (this.socket && this.socket.readyState === WebSocket.OPEN) {
         this.socket.close()
       }
       this.clearReconnectTimeout()
       this.reconnectCount = 0
       this.startTime = null
     }
   }
   
   // WebSocketReconnect 类封装了WebSocket的连接、重连逻辑。
   // maxReconnectAttempts 是最大重连尝试次数。
   // reconnectInterval 是每次重连尝试之间的间隔时间。
   // maxReconnectTime 是总的重连时间限制，超过这个时间后不再尝试重连。
   // reconnectCount 用于记录已经尝试的重连次数。
   // startTime 用于记录开始重连的时间。
   // connect 方法用于建立WebSocket连接，并设置相应的事件监听器。
   // handleClose 方法在WebSocket连接关闭或发生错误时被调用，根据条件决定是否尝试重连。
   // clearReconnectTimeout 方法用于清除之前设置的重连定时器。
   // close 方法用于关闭WebSocket连接，并清除重连相关的状态。
   
   // 使用示例
   // const webSocketReconnect = new WebSocketReconnect('ws://your-websocket-url')
   // 当不再需要WebSocket连接时，可以调用close方法
   // webSocketReconnect.close();
   
   export default WebSocketReconnect
   
   ```

   

8. 前端vue对接（任意vue界面初始化websocket连接）

   ```vue
     mounted () {
       if ('WebSocket' in window) {
         // 连接WebSocket节点
         this.websocket = new WebSocketReconnect('ws://127.0.0.1:8002' + '/websocket/123')
       } else {
         alert('浏览器不支持webSocket')
       }
   
       // 接收到消息的回调方法
       this.websocket.socket.onmessage = function (event) {
         const data = event.data
         console.log('后端传递的数据:' + data)
         // 将后端传递的数据渲染至页面
         // textarea1.value = textarea1.value + data + '\n' + '【消息】---->'
       }
       // 监听窗口关闭事件，当窗口关闭时，主动去关闭websocket连接，防止连接还没断开就关闭窗口，server端会抛异常。
       window.onbeforeunload = function () {
         this.websocket.close()
       }
        // 关闭连接
        function closeWebSocket () {
          this.websocket.close()
        }
        // 发送消息
        function send () {
          this.websocket.socket.send({ kk: 123 })
        }
       .    .    .
   }
   ```

   

9. 效果展示

   

   ![image.png](https://segmentfault.com/img/bVdb41r)