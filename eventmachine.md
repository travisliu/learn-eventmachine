# 關於EventMachine
## 什麼是 EventMachine
EventMachine 是一套事件驅動(event-driven IO) 的框架，基於Reactor Pattern 達到輕量化的併發處理，對於線性程式常因IO Blocking造成效能問題，EventMaching 會是一套很好的解決方案，以下是三種是常見又容易被忽略的效能瓶頸產生的原因：

* 資料庫存取
* 檔案存取
* HTTP API 存取

## 其他語言的Reactor 框架

| 語言 | 框架 |
| ---  | --- |
|JavaScript | Nodejs |
|Java JBoss | netty |
|Python | twisted |
|PHP |N/A|

## 可以用 EventMachine 來做些什麼？
Goliath https://github.com/postrank-labs/goliath 是 建立在EventMachine上的一套web server，可以建置出比一般web server 更高吞吐量的Web Service，您不妨可以從較簡單的Restful API Server 開始試用

使用EventMachine 建制server 有個相較于傳統的http server 有個優點，就是在一個ruby process能同時處理 5-10k 同時連線，礙於過去的Web Server connection 數的限制， 如果有要從server side 主動發訊息到client 的需求，往往都是透過pulling 或 long pulling 的方式，而在EventMachine下你可以從client 建立一個connection 到server，達到及時雙向溝通的效果

如果你有用ruby 自行開發Server的需求，EventMachine 就會是一個很好的選擇，有許多Project 就是使用EventMachine 建置server
像是..

* [FAYE](http://faye.jcoglan.com/ruby.html) - pub/sub messaging
* [Vine](http://www.getvines.org/) - XMPP server


另外pusher.com 也是使用EventMachine 底下的套件em-websocket 建制出來的pub/sub 服務

當然你也能用在處理各總網路IO 的問題
mobile app push server ( APN or GCM )
Email Server
客製的TCP  proxy

甚至是自行架一個小型的聊天室或者Telnet BBS 站台
