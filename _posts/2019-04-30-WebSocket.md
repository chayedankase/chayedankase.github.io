---
layout: post
title: springboot-webSocket
categories: [spring]
description: webSocket
keywords: webSocket
---



# WebSocket

> 当你想吃一碗牛肉面的时候，你可以通过app查到味道最棒的一家店，这家店通常不会正好有一个空位置在等着你，你最好可以通过电话提前了解是否有你可以坐的位置。很不巧，现在客满。你可以待会再来询问，当然你的肚子会不停的提醒你去不停的询问店家。换一种方式，你可以给店家留下你的联系方式，以便于有空位的时候店家能第一时间联系你。这样你就不用总是不停的去询问店家了，你将有更多的时间来做你自己的事情。
>
> 早期浏览器要实时获取服务器的数据，只能通过轮询的方式一遍又一遍的进行查询操作，以便于随时了解服务端的数据更新情况。websocket这项技术，可以使服务端有能力主动与浏览器进行通讯。
>
> 通过websocket，浏览器将和服务器建立起一个连接，在建立连接时，浏览器会告诉服务器它对那些数据感兴趣，那么当这些数据有变化时，服务器会给浏览器做出响应。

## 基于spring的websocket

> 在spring中使用较低层级的api来处理websocket消息，我们需要实现WebsocketHandler接口。

```java
package org.springframework.web.socket;

public interface WebSocketHandler {
    void afterConnectionEstablished(WebSocketSession var1) throws Exception;

    void handleMessage(WebSocketSession var1, WebSocketMessage<?> var2) throws Exception;

    void handleTransportError(WebSocketSession var1, Throwable var2) throws Exception;

    void afterConnectionClosed(WebSocketSession var1, CloseStatus var2) throws Exception;

    boolean supportsPartialMessages();
}
```

> 我们可以通过继承AbstractWebSocketHandler来减少实现方法的编写。
>
> 然后通过websockect配置类来对这个处理器进行映射配置

```java
package com.pansky.demowebsocket.demowebsocket;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

/**
 * 描述:
 *
 * @author Xue_Pan
 * @date 2019-05-05 10:10
 */
@Configuration
@EnableWebSocket
public class SocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry webSocketHandlerRegistry) {
        webSocketHandlerRegistry.addHandler("创建一个WebsocketHandler的实现类对象", "映射的路径");
    }
}
```

> 不使用配置类的话使用websocket命名空间

```XML
<websocket:handlers>
	<websocket:mapping handler="对象id"，path="映射路径" />
</websocket:handlers>
```

> 浏览器端代码

``` js
var url = 'ws://localhost:8080/simpledata'
//创建一个websocket
var sock = new WebSocket(url);
//打开一个websocket连接,连接成功后执行函数
sock.onopen = function(){
    console.log("连接成功了~~~")
    //给服务器发送消息
    sock.send("你好，我是浏览器")
}；
//服务器返回消息时执行
sock.onmessage = function(e){
    console.log(e.data)
}
//连接关闭时执行
sock.onclose = function(){
    console.log('closing')
}
```

> 这是原始版的websocket,有些浏览器并不支持websocket,我们可以使用sockjs来保证我们的websocket能够正常运行。只需要再注册执行器时增加  withSockJs()即可
>
> ```java
> webSocketHandlerRegistry.addHandler("创建一个WebsocketHandler的实现类对象", "映射的路径").withSockJs()
> ```

## STOMP

> 与http协议相似，websocket在交互中也有一个协议，就是stomp。

## 基于springboot+vue的websocket

### 导包

```XML
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-websocket</artifactId> 
</dependency> 

```



```java
package com.pansky.demowebsocket.demowebsocket;

import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;

/**
 * 描述:
 *
 * @author Xue_Pan
 * @date 2019-04-29 9:10
 */
@Configuration
@EnableScheduling
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    /**
     * 注册stomp节点，启用sockjs
     * @param registry
     */
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/simpledata").setAllowedOrigins("*").withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/topic", "/s");
        registry.setApplicationDestinationPrefixes("/app");
    }
}
```

