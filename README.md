 # 架构

整个项目分为以下模块

- public agents
- private agents
- redirector
- secretaries
- strategies
- secretary center
- monitor
- texchange
- tecretaries

## public agents

每个交易所有一个 public agent 程序，每个程序运行一个实例。用于获取该交易所中多个市场的行情数据，并转换为统一接口。

目前只实现 websocket 接口。

## private agents

每个交易所有一个 private agent 程序，每一个账户运行一个实例。用于发送交易指令，并提供统一接口。

之所以把 private agents 设计为 daemon 而不是 bindings，是为了让 private agent 可以跨语言开发。

## redirector

每个需要被别人连接的实例，在启动时将自己提供的多个服务的名称和地址注册到 redirector 用于别人连接时重定向。这样就不需要给每个被连接实例一个固定 URL。

之所以不用中介是因为 private agents 连接中介时中介是 server，从 server 向作为 client 的 private agents 发请求很麻烦

- http 不能从 server 向 client 发请求
- websocket 不支持 req/res pattern
- 中心化动态注册的 websocket 子协议 WAMP 协议不够主流太小众跨语言支持不多。
- socket.io 的 res 是以 callback 的形式而不是专用的 req/res pattern，如果不 res 不会自动报错
- 非中心化的消息队列 zeromq 不支持 server 向 client 发请求
- dubbo/grpc/hprose/thrift/tars 等主流 rpc 框架似乎均不支持 server 向 client 发请求
- 支持 server 向 client 发请求的 rpc 框架 dnode 不能跨语言

## secretaries

秘书程序有多个，每种语言一个，每个秘书实例本身就是交易程序主进程，以策略模式调用策略。一个秘书可能连接多个交易市场，这是为了支持套利策略。

秘书将记录交易数据并主动提供给秘书中心。秘书与秘书中心使用 websocket 持久连接。

## strategies

每一个策略都是一个不能独立运行的程序模块，以策略模式被秘书调用。

## secretary center

秘书中心是一个单例 daemon，整合各个秘书的交易信息持久化并被动提供给控制台。

秘书中心负责管理秘书们的运行状态，虽然秘书和策略可以是各种语言写的。

删除一个秘书的数据时需要到秘书中心数据库中手动删除。

## monitor

监控台是一个 webapp，主动从秘书中心获取

- 交易信息
- 账户信息
- 历史信息

还可以控制通过秘书中心秘书运行。

## texchange

texchange 是一个回测服务器，同时担任 public agent 和 private agent。

## tecretaries

tecretaries 是回测专用秘书，每种语言一个，对策略暴露的接口与 secretaries 完全相同。

tecretary 进程与 techange 进程直接连接用于同步回测进度。

为了能对轮询式策略进行回测，tecretaries 向策略提供一个定制版的 setTimeout()，而 secretaries 提供一个普通版的 setTimeout()。
