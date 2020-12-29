# BitHub

## 架构

整个项目分为以下模块

- Public Agent
- Private Agent
- [Redirector](https://github.com/bithub-framework/redirector)
- Secretary
    - [JavaScript](https://github.com/bithub-framework/secretary-js)
- Strategy
- [Secretariat](https://github.com/bithub-framework/secretariat)
- Monitor
- [Tecretary](https://github.com/bithub-framework/tecretary)
- [Texchange](https://github.com/bithub-framework/texchange)

![architecture](./arch.png)

### Public Agents

每个交易所有一个 Public Agent 程序，每个程序运行一个实例。用于获取该交易所中多个市场的行情数据，并转换为统一接口。

目前只实现 websocket 接口。

### Private agents

每个交易所有一个 Private Agent 程序，每一个账户运行一个实例。用于发送交易指令，并提供统一接口。

之所以把 Private Agents 设计为 daemon 而不是 bindings，是为了让 Private Agent 可以跨语言开发。

### Redirector

每个需要被别人连接的实例，在启动时将自己提供的服务的名称和地址注册到 Redirector 用于别人连接时重定向。这样就不需要给每个被连接实例一个固定端口。

之所以不用中介是因为 Private Agent 连接中介时中介是 server，从 server 向作为 client 的 Private Agent 发请求很麻烦

- http 不能从 server 向 client 发请求
- websocket 不支持 req/res pattern
- 中心化动态注册的 websocket 子协议 WAMP 协议不够主流太小众跨语言支持不多。
- socket.io 的 res 是以 callback 的形式而不是专用的 req/res pattern，如果不 res 不会自动报错
- 非中心化的消息队列 zeromq 不支持 server 向 client 发请求
- dubbo/grpc/hprose/thrift/tars 等主流 rpc 框架似乎均不支持 server 向 client 发请求
- 支持 server 向 client 发请求的 rpc 框架 dnode 不能跨语言

### Secretary

秘书程序有多个，每种语言一个，以策略模式调用策略，向策略提供交互接口。一个秘书可能连接多个市场，这是为了支持套利策略。

秘书将交易状态信息主动提供给秘书处，秘书与秘书处使用 http 持久连接。

### Strategy

每一个策略都是一个不能独立运行的程序模块，以策略模式被秘书调用。

### Tecretary

Tecretary 是回测专用秘书，每种语言一个，对策略暴露的接口与 secretary 完全相同。

Tecretary 调用 Techange 作为回测服务器。

### Texchange

Texchange 是一个回测服务器，同时担任 Public Agent 和 Private Agent，是 Tecretary 的组件。

### Secretariat

秘书处是一个单例 daemon，整合各个秘书的交易信息持久化并被动提供给监控台。

秘书开始运行时向秘书处注册自己。

### Monitor

监控台是一个 webapp，主动从秘书处获取

- 交易信息
- 账户信息
- 历史信息
