## 浏览器前后端通信

### 1. HTTP 请求 / 响应

HTTP 是 Web 前后端通信最常用的应用层协议。

基本通信模型：

客户端发送请求：

Browser → HTTP Request → Server

服务器返回响应：

Server → HTTP Response → Browser

特点：

- 请求 / 响应模型
- 通常由客户端主动发起请求
- 服务端处理请求后返回响应
- 常用于 REST API、普通业务接口

---

### 2. AJAX

#### 2.1 AJAX 是什么

AJAX：

> Asynchronous JavaScript and XML

AJAX 不是某个具体的 API，也不是一种协议。

它是一种 Web 开发思想 / 技术模式：

> JavaScript 在不刷新整个页面的情况下，异步向服务器发送请求并获取数据。

例如：

传统方式：

用户操作  
↓  
提交表单  
↓  
服务器返回新页面  
↓  
整个页面重新加载

AJAX：

用户操作  
↓  
JavaScript 发送 HTTP 请求  
↓  
服务器返回数据  
↓  
JavaScript 更新局部页面

---

### 3. XMLHttpRequest（XHR）

XMLHttpRequest 是浏览器提供的原生 JavaScript API。

主要用于：

- 发送 HTTP 请求
- 接收服务器响应
- 实现 AJAX

关系：

AJAX
└── XMLHttpRequest
    └── AJAX 早期最主要的实现方式

示例思想：

JavaScript
↓
XMLHttpRequest
↓
HTTP Request
↓
Server

XHR API 较老，通常需要处理：

- readyState
- status
- callback
- onreadystatechange

因此现代项目中一般不会直接大量使用 XHR。

---

### 4. Fetch API

Fetch 是浏览器提供的现代 HTTP 请求 API。

关系：

AJAX
└── Fetch API
    └── 现代 AJAX 的主要实现方式之一

特点：

- 浏览器原生 API
- 基于 Promise
- API 更简洁
- 更适合 async / await

例如：

fetch(...)
↓
HTTP Request
↓
Server
↓
HTTP Response
↓
Promise

现代代码通常写成：

async / await
↓
fetch()
↓
HTTP Request

---

### 5. Axios

Axios 是第三方 JavaScript HTTP 客户端库。

它不是浏览器原生 API。

关系：

Axios
└── 对 HTTP 请求进行进一步封装
    ├── 浏览器环境
    │   └── 基于 XHR / Fetch 等适配器
    │
    └── Node.js 环境
        └── 使用对应的 HTTP Adapter

Axios 提供了更方便的功能：

- GET / POST / PUT / DELETE
- 请求参数处理
- JSON 自动处理
- 请求拦截器
- 响应拦截器
- 错误统一处理
- 请求配置
- 超时配置

典型使用场景：

Vue / React
↓
Axios
↓
HTTP
↓
Spring Boot REST API

例如：

Vite :5173
↓
Axios
↓
HTTP Request
↓
Spring Boot :8080

---

## HTTP 请求技术之间的关系

可以理解为：

AJAX
├── XMLHttpRequest
│   └── 浏览器早期 HTTP 请求 API
│
├── Fetch
│   └── 浏览器现代 HTTP 请求 API
│
└── Axios
    └── 第三方 HTTP 请求库
        └── 对底层请求能力进行进一步封装

需要特别注意：

AJAX ≠ XMLHttpRequest

AJAX 是开发思想。

XMLHttpRequest 是实现 AJAX 的一种 API。

同样：

AJAX ≠ Fetch

Fetch 也是实现 AJAX 的一种方式。

---

## 6. WebSocket

WebSocket 用于客户端和服务器之间的双向实时通信。

通信模型：

Client ⇄ Server

与普通 HTTP 最大的区别：

HTTP：

Client
↓ Request
Server
↓ Response
Client

通常由客户端主动发起请求。

WebSocket：

Client ⇄ Server

建立连接之后：

- 客户端可以主动发送消息
- 服务端也可以主动发送消息

常见场景：

- 在线聊天
- 即时消息
- 实时游戏
- 实时行情
- 实时协作
- 实时通知

---

## 7. SSE（Server-Sent Events）

SSE：

> Server-Sent Events

主要用于：

> 服务端向客户端持续进行单向实时推送。

通信方向：

Server → Client

客户端首先建立连接：

Client
↓
HTTP Request
↓
Server

连接建立后：

Server
↓
Event
↓
Client

Server
↓
Event
↓
Client

Server
↓
Event
↓
Client

特点：

- 服务端 → 客户端单向通信
- 基于 HTTP
- 浏览器原生支持 EventSource
- 比 WebSocket 更简单
- 适合只需要服务端推送的场景

常见场景：

- AI 流式输出
- 新闻推送
- 日志实时输出
- 系统通知
- 任务进度推送

例如：

Browser
↓
HTTP Request
↓
Spring Boot
↓
SSE Stream
↓
Browser


### 7.1 EventSource

`EventSource` 是浏览器提供的原生 Web API，用于建立 SSE（Server-Sent Events）连接。

它不是：

- HTML API
- CSS API
- Vue API

而是浏览器 Web Platform 提供给 JavaScript 使用的接口。

关系可以理解为：

```text
Vue
 ↓
JavaScript
 ↓
Browser Web API
 ↓
EventSource
 ↓
HTTP / SSE
 ↓
Server
```

也就是说：

- JavaScript 是编程语言
- EventSource 是浏览器提供给 JavaScript 调用的 Web API
- Vue 是基于 JavaScript 的前端框架
- Vue 中使用 SSE 时，本质上仍然是在调用浏览器原生的 `EventSource`

例如：

```javascript
const source = new EventSource('/api/events');

source.onmessage = function (event) {
    console.log(event.data);
};
```

其中：

```javascript
new EventSource('/api/events')
```

会让浏览器向服务器发起一个 HTTP 请求，并建立 SSE 连接。

浏览器通常会发送类似：

```text
GET /api/events HTTP/1.1
Accept: text/event-stream
```

服务器返回：

```text
HTTP/1.1 200 OK
Content-Type: text/event-stream
```

与普通 HTTP 请求不同，服务器不会立即结束 Response，而是保持连接并持续向客户端发送事件。

例如服务器发送：

```text
data: hello

data: world

```

浏览器收到事件后，会触发：

```javascript
source.onmessage = function (event) {
    console.log(event.data);
};
```

此时：

```text
event.data = "hello"
```

之后下一条事件到达时再次触发 `onmessage`。

完整通信过程：

```text
Browser
   │
   │ new EventSource('/api/events')
   │
   │ HTTP Request
   ▼
Server
   │
   │ Content-Type: text/event-stream
   │
   │ connection stays open
   │
   ├── data: message1 ───────→ Browser
   │                           ↓
   │                     onmessage(event)
   │
   ├── data: message2 ───────→ Browser
   │                           ↓
   │                     onmessage(event)
   │
   └── data: message3 ───────→ Browser
```

需要注意：

`EventSource` 只负责浏览器端的 SSE 客户端连接。服务端仍然需要提供一个返回 `text/event-stream` 的 HTTP 接口。

例如在 Spring Boot 中，可以使用 `SseEmitter` 或 WebFlux 持续向客户端推送数据。

因此可以总结为：

> EventSource = Browser Web API → JavaScript 调用 → 建立 HTTP SSE 长连接 → 接收服务器持续推送的数据。

---

---

## 8. HTTP、WebSocket、SSE 对比

| 技术      | 通信方式        | 方向                     | 典型场景               |
| --------- | --------------- | ------------------------ | ---------------------- |
| HTTP      | 请求 / 响应     | Client → Server → Client | REST API、普通业务接口 |
| AJAX      | Web 开发思想    | Client → Server → Client | 页面异步请求           |
| XHR       | 浏览器 HTTP API | Client → Server → Client | 传统 AJAX              |
| Fetch     | 浏览器 HTTP API | Client → Server → Client | 现代 AJAX              |
| Axios     | 第三方 HTTP 库  | Client → Server → Client | Vue / React 项目       |
| SSE       | 长连接          | Server → Client          | AI 流式输出、通知      |
| WebSocket | 长连接          | Client ⇄ Server          | 聊天、实时交互         |

---

## 9. 整体知识结构

浏览器前后端通信
│
├── HTTP
│   ├── Request
│   └── Response
│
├── AJAX
│   │
│   ├── XMLHttpRequest（XHR）
│   │   └── 浏览器早期 HTTP API
│   │
│   ├── Fetch API
│   │   └── 浏览器现代 HTTP API
│   │
│   └── Axios
│       └── 第三方 HTTP 客户端库
│
├── WebSocket
│   └── Client ⇄ Server
│       双向实时通信
│
└── SSE
    └── Server → Client
        单向实时推送

---

## 10. 一句话理解

HTTP：

> Web 通信协议。

AJAX：

> 不刷新整个页面，通过 JavaScript 异步请求服务器的开发思想。

XMLHttpRequest：

> 浏览器早期发送 HTTP 请求的 API。

Fetch：

> 浏览器现代发送 HTTP 请求的 API。

Axios：

> 对 HTTP 请求进行进一步封装的第三方 JavaScript 库。

WebSocket：

> 客户端和服务器双向实时通信。

SSE：

> 服务端向客户端单向实时推送。