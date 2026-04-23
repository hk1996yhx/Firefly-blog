---
title: Firefly 代码块示例
published: 1970-01-03
pinned: true
description: 在Firefly中使用表达性代码的代码块在 Markdown 中的外观。
tags: [Markdown, Firefly]
category: 文章示例
draft: false
image: ./images/firefly3.avif
---
# Skynet API 说明书

更新时间：2026-04-18  
来源：`cloudwu/skynet` 官方 wiki  
适用范围：Lua 层常用 API、网络模块、集群模块、共享数据、调试与辅助模块

这份文档不是逐页翻译。
它只保留对实际开发有用的部分，尤其适合服务端项目起盘和日常查表。

## 1. 先记住 Skynet 的运行模型

- 一个 service 默认串行处理消息。
- `skynet.call` 会挂起当前 coroutine，等对方返回。
- `skynet.send` 只发不等。
- service 之间的同步风格靠消息协议和 `dispatch` 控制。
- 多节点通信用 `cluster`，但它不是透明共享内存。

这几个点不先立住，后面很多 API 会误用。

## 2. 配置与启动

官方 `Config` 页给出的核心约定是：

- 配置文件本质上是一段 Lua 脚本，执行后把配置项填进全局环境。
- 配置项里最常见的是：
  - `thread`
  - `logger`
  - `logpath`
  - `harbor`
  - `address`
  - `master`
  - `start`
  - `bootstrap`
  - `standalone`
  - `luaservice`
  - `lualoader`
  - `snax`
  - `cpath`
  - `lua_path`
  - `lua_cpath`

常用判断：

- 单节点本地开发，通常 `standalone = "0.0.0.0:2013"`，`harbor = 0`。
- 多节点集群，需要为每个节点给不同的 `harbor`，并配置 `address` / `master`。
- `start` 指定启动服务，默认常见是 `main`。
- `bootstrap` 决定最早拉起的引导服务，默认是 `snlua bootstrap`。

面向项目的建议：

- 节点配置和业务配置分开。
- `luaservice`、`lua_path`、`cpath` 统一放在基础配置里，不要散在脚本里。

## 3. 核心 `skynet` API

官方 wiki 里最核心的页面是 `LuaAPI` 和 `APIList`。
下面按实际使用频率重排。

### 3.1 服务启动与退出

#### `skynet.start(start_func)`

注册 service 的启动入口。
一般在这里完成：

- `dispatch` 注册
- 初始化状态
- 对外注册名字
- 打开监听或定时器

#### `skynet.init(init_func)`

初始化阶段回调。
发生在 `start` 之前。
适合做不依赖消息循环的预初始化。

#### `skynet.exit()`

让当前 service 退出。
如果当前消息还没返回，Skynet 会在该消息处理完成后再真正结束。

#### `skynet.abort()`

立即终止当前进程。
这是进程级别的硬退出。
只适合致命错误。

### 3.2 服务地址、名字与环境

#### `skynet.self()`

返回当前 service 地址。

#### `skynet.address(addr)`

把地址转成人类可读字符串。

#### `skynet.register(name)`

把当前 service 注册为一个名字。
通常配合 `skynet.name` / `queryservice` 使用。

#### `skynet.name(name, handle)`

给指定 handle 绑定名字。

#### `skynet.localname(name)`

查询本地名字。

#### `skynet.getenv(key)` / `skynet.setenv(key, value)`

读写环境变量。
这些值来自配置文件或运行时设置。

常见用途：

- 读数据库地址
- 读集群名
- 读业务开关

### 3.3 创建和查找服务

#### `skynet.newservice(name, ...)`

创建一个 Lua service。
返回新 service 的 handle。

#### `skynet.uniqueservice(name, ...)`

获取一个唯一 service。
如果还没创建，会自动创建。
常用在：

- 全局配置服务
- 玩家管理中心
- 排行榜中心

#### `skynet.queryservice(name)`

查询一个已经存在的唯一 service。
如果没有，会等待服务出现。

注意：

- `uniqueservice` 适合“没有就创建”。
- `queryservice` 适合“必须等已有服务上线”。

### 3.4 消息派发与协议

#### `skynet.dispatch(proto_name, dispatch_func)`

注册某种协议的消息处理函数。
最常见的是：

- `lua`
- `client`
- `socket`
- 自定义协议

`dispatch_func` 的典型参数是：

- `session`
- `source`
- 命令或消息体

#### `skynet.register_protocol { ... }`

注册新协议。
通常要给出：

- `name`
- `id`
- `pack`
- `unpack`
- `dispatch`

如果项目里需要客户端私有协议，通常就在这里做协议接线。

### 3.5 请求、响应与序列化

#### `skynet.call(addr, proto, ...)`

同步 RPC。
当前 coroutine 会挂起，直到对方返回。

特点：

- 写法最直观
- 链路长时容易堆积等待
- 不适合高频大广播

#### `skynet.send(addr, proto, ...)`

异步发送。
不等返回。

适合：

- 通知
- 事件投递
- 日志
- 非关键链路的状态刷新

#### `skynet.redirect(addr, source, proto, session, msg, sz)`

把收到的消息原样转发。
适合做网关、代理或桥接层。

#### `skynet.response(pack_func)`

生成一个延迟响应器。
适合需要异步算完再回包的场景。

配套 API：

- `skynet.ret(...)`
- `skynet.retpack(...)`
- `skynet.pack(...)`
- `skynet.packstring(str)`
- `skynet.unpack(...)`
- `skynet.tostring(msg, sz)`

常见分工：

- `ret` / `retpack`：给当前请求回包
- `pack` / `packstring`：手动打包消息
- `unpack`：解包参数
- `tostring`：把底层消息缓冲转 Lua 字符串

### 3.6 时间、调度与 coroutine

#### `skynet.now()`

返回当前 tick。
单位是 1/100 秒。

#### `skynet.time()`

返回秒级时间，精度更适合统计和超时计算。

#### `skynet.starttime()`

返回 Skynet 启动时的时间基准。

#### `skynet.sleep(ti)`

挂起当前 coroutine 指定 tick。

#### `skynet.timeout(ti, func)`

在未来某个 tick 执行函数。

#### `skynet.fork(func, ...)`

启动新的 coroutine 执行任务。

#### `skynet.yield()`

主动让出当前 coroutine。

#### `skynet.wait(token)` / `skynet.wakeup(token)`

按 token 阻塞和唤醒 coroutine。
适合 service 内部小范围同步。

面向业务的建议：

- `timeout` 适合定时任务。
- `fork` 适合把当前消息拆成并行 coroutine。
- `wait/wakeup` 适合 service 内部条件等待。
- 但不要把复杂状态机全写成互相 `wakeup` 的风格，后期会很难追。

### 3.7 日志、错误和监控

#### `skynet.error(...)`

输出日志。

#### `skynet.traceback()`

输出当前 coroutine 栈。

#### `skynet.stat(what)`

从 `APIList` 看，`stat` 可用于取 service 统计信息。
wiki 本页没有展开细节，适合去源码确认具体字段。

#### `skynet.info_func(func)`

注册调试信息输出函数。
常和 debug console 配合。

### 3.8 进阶和偏底层 API

这些在 `APIList` 里能看到，但官方 wiki 对部分条目展开较少：

- `skynet.rawcall`
- `skynet.rawsend`
- `skynet.genid`
- `skynet.kill`
- `skynet.killthread`
- `skynet.task`
- `skynet.context`
- `skynet.memlimit`
- `skynet.dispatch_message`

结论：

- 能不用就先不用。
- 先把常规 `call/send/dispatch/response` 用稳。
- 真要碰这些，直接对源码看实现更稳。

## 4. `skynet.manager`

官方 `LuaAPI` 里明确提到，扩展管理 API 需要：

```lua
local skynet = require "skynet"
require "skynet.manager"
```

开启后常用的是：

- `skynet.launch(...)`
- `skynet.kill(handle)`
- `skynet.abort()`
- `skynet.register(name)`
- `skynet.name(name, handle)`

适合场景：

- watchdog 拉起子服务
- 动态起停 worker
- 运行期服务重建

## 5. `socket` API

官方 `Socket` 页覆盖了 TCP、UDP、DNS 和 fd 转移。
这部分在游戏服里非常常用。

### 5.1 TCP 监听与连接

#### `socket.listen(host, port, backlog)`

监听一个 TCP 地址。
返回监听 fd。

#### `socket.start(id, accept_cb)`

让当前 service 成为 socket owner，并开始接收该 socket 事件。

如果是监听 fd，`accept_cb` 会在新连接到来时拿到：

- `newid`
- `addr`

#### `socket.open(addr, port)`

主动建立 TCP 连接。
成功返回连接 id。

### 5.2 TCP 读写

#### `socket.read(id, sz)`

读固定长度。

#### `socket.readline(id, sep)`

按分隔符读一行。

#### `socket.readall(id)`

一直读到对端关闭。

#### `socket.block(id)`

把 socket 切成阻塞模式读。
适合简单协议或工具脚本。

#### `socket.write(id, data)`

写数据。

#### `socket.lwrite(id, data)`

低优先级写。
官方说明是：适合不会立刻被读取的数据。

#### `socket.close(id)`

关闭连接。

#### `socket.shutdown(id)`

关闭写方向。

### 5.3 fd 所有权与转移

#### `socket.abandon(id)`

放弃 socket 所有权。
常见于：

- watchdog 接入后把 fd 交给 agent
- 网关把已认证连接转交业务 service

#### `socket.close_fd(id)`

直接关底层 fd。

### 5.4 背压与告警

#### `socket.limit(id, limit)`

设置写缓冲限制。

#### `socket.warning(id, callback)`

注册写缓冲告警回调。

MMORPG 里这两个很有用。
因为广播、场景同步、聊天风暴都可能把某个连接拖成慢连接。

### 5.5 UDP

#### `socket.udp(handler, host, port)`

创建 UDP socket。

#### `socket.sendto(id, addr, port, data)`

发送 UDP 包。

#### `socket.udp_connect(id, addr, port, callback)`

给 UDP socket 绑定默认远端。

#### `socket.udp_address(msg, from)`

从回调参数里取 UDP 来源地址。

### 5.6 DNS

官方页给出的约定是：

- 需要先 `require "skynet.dns"`
- `dns.server()` 设置 DNS 服务器
- `dns.resolve(hostname, ipv6)` 解析域名

官方还特别提醒了两点：

- DNS 缓存在各 service 内本地保存
- 解析是同步的，网络线程会阻塞

所以比较稳的做法是：

- 用独立 service 做 DNS
- 其他 service 用 `call` 去问它

## 6. `socketchannel`

`socketchannel` 是官方给出的客户端连接封装。
适合和固定远端服务通信。

### 6.1 创建

```lua
local sc = require "skynet.socketchannel"
local channel = sc.channel {
  host = "127.0.0.1",
  port = 7000,
  auth = auth_func,
  response = response_func,
  nodelay = true,
  backup = backup_func,
  overload = overload_func,
}
```

wiki 明确提到的字段包括：

- `host`
- `port`
- `auth`
- `response`
- `nodelay`
- `backup`
- `overload`

### 6.2 常用方法

#### `channel:request(request, response, padding)`

发送请求，并用给定 response 函数读取回包。

#### `channel:response(dispatch)`

切换成推送模式。
适合远端会主动推消息的连接。

#### `channel:close()`

关闭连接。

### 6.3 使用判断

适合：

- 游戏服连中心服
- 游戏服连 Redis 代理、外部工具、第三方 TCP 服务

不适合：

- 复杂双向协议网关
- 需要细粒度控制 fd 生命周期的场景

## 7. 集群 `cluster`

官方 `Cluster` 页分两种模式：

- 主从式集群
- 独立集群模式

### 7.1 主从式集群

特点：

- 每个节点有唯一 `harbor id`
- 由 master 管理节点地址信息
- 一个节点挂掉，相关 harbor id 会失效

官方明确说了限制：

- 不适合跨数据中心
- 节点不是动态增删的理想方案

所以它更像机房内固定节点拓扑。

### 7.2 独立集群模式

配置方式：

- `cluster.reload(config)` 载入节点表
- `config` 可以是 table，也可以是配置文件
- table 里 key 是节点名，value 是 `"ip:port"`

特殊选项：

- `__nowaiting = true`：访问未定义节点时不等待，直接报错

### 7.3 常用 API

#### `cluster.open(node_name)`

开启当前节点的集群监听。
`node_name` 来自 cluster 配置表里的节点名。
官方示例是 `cluster.open "db"`。

#### `cluster.reload(config)`

重载节点配置。

#### `cluster.call(node, addr, ...)`

跨节点同步调用。

#### `cluster.send(node, addr, ...)`

跨节点异步发送。

官方特别提醒：

- 它不保证一定送达
- 远端节点重启时，投递中的消息可能丢失

这对游戏服很重要。
关键资产链路不要只靠 `cluster.send`。

#### `cluster.proxy(node, addr)`

构造一个远端代理对象。
后续能像本地对象那样做 call/send 风格访问。

#### `cluster.register(name, addr)`

把本地服务注册成远端可访问的名字。

#### `cluster.query(node, name)`

按名字查远端服务地址。

### 7.4 Snax 与 cluster

`Cluster` 页也提到：

- `snax.enablecluster()` 可以让 snax 服务参与 cluster

但这条更偏兼容。
新项目不建议把 snax 当默认抽象层。

## 8. 共享数据：`sharetable`、`sharedata`、`datacenter`

这三个名字很像，但定位不同。

### 8.1 `sharetable`

`APIList` 明确列出：

- `sharetable.loadtable`
- `sharetable.loadfile`
- `sharetable.query`

适合：

- 配置表
- 静态模板
- 多 service 只读共享数据

这类数据最适合 MMO 项目里的：

- 技能表
- 怪物表
- 地图表
- 掉落表

### 8.2 `sharedata`

官方 `ShareData` 页给了完整 API。

#### `sharedata.new(name, value)`

创建一个共享对象。

#### `sharedata.update(name, value)`

更新对象。

#### `sharedata.delete(name)`

删除对象。

#### `sharedata.query(name, ...)`

读取对象或其深层子节点。

wiki 明确说明：

- 取到的对象和 table 类似
- 读起来比普通 table 稍慢
- 不能直接改
- 自动跟踪更新

#### `sharedata.deepcopy(obj)`

深拷贝成普通 Lua table。

官方说明的取舍是：

- 直接读共享对象较慢，但能自动看到更新
- 深拷贝后读取更快，但不会自动更新

#### `sharedata.flush()`

把旧版本对象在所有 service 都不再使用后再回收。

### 8.3 `datacenter`

官方 `DataCenter` 页给出的 API 很少，但定位很明确：

#### `datacenter.set(key1, key2, ..., value)`

写一个全局共享变量。

#### `datacenter.get(key1, key2, ...)`

读变量。

#### `datacenter.wait(key1, key2, ...)`

如果键还没值，就阻塞等待。

官方还明确提醒：

- `datacenterd` 是单点
- 不适合大量访问
- 在 cluster 模式下不可直接工作
- 跨节点需求应改用 `sharedata` 或 `cluster.call`

结论：

- `datacenter` 适合少量全局开关和启动时同步
- 不适合高频在线业务数据

## 9. 网关模板 `gateserver`

官方 `GateServer` 页给的是模板式写法。
它适合 TCP 长连接接入，但要接受它接管 socket 协议。

### 9.1 启动方式

```lua
local gateserver = require "snax.gateserver"

gateserver.start(handler)
```

官方还特别说明：

- `gateserver.start` 内部会调用 `skynet.start`
- 如果想自己控制 `skynet.start` 时机，可以设置 `handler.embed = true`

### 9.2 `handler` 需要实现的回调

官方页列出的核心回调有：

- `handler.open(source, conf)`
- `handler.connect(fd, addr)`
- `handler.disconnect(fd)`
- `handler.error(fd, msg)`
- `handler.message(fd, msg, sz)`
- `handler.warning(fd, size)`
- `handler.command(cmd, source, ...)`

### 9.3 常用辅助函数

#### `gateserver.openclient(fd)`

允许该连接开始接收客户端数据。

官方特别说明：

- 新连接建立后，默认不会立刻把客户端包送到 `message`
- 只有调用 `openclient` 后才开始收包

这很适合认证前拦截。

#### `gateserver.closeclient(fd)`

关闭客户端连接。

### 9.4 `message` 回调的注意点

官方写得很清楚：

- `msg` 不是 Lua 字符串，是一段底层内存
- 必须调用 `netpack.tostring(msg, sz)` 复制出来
- 或直接 `skynet.redirect` 给别的 service
- 否则会泄漏内存

### 9.5 适用判断

适合：

- 单一 TCP 协议网关
- 登录服或游戏服接入层

不适合：

- 想自己直接同时处理所有 socket 细节
- 已经基于 `skynet.socket` 做了完整 owner 管理的项目

因为官方页明确说了：

- `service/gate.lua` 和 `skynet.socket` 不能混用
- gate 模板会接管 socket 消息

## 10. HTTP

官方 `Http` 页分服务端和客户端两部分。

### 10.1 HTTP 服务端

wiki 这一页主要给的是示例代码。
从官方示例可以直接看出，最常用入口是：

- `httpd.read_request`
- `httpd.write_response`
- `sockethelper.readfunc`
- `sockethelper.writefunc`

从示例能直接确认的模式是：

1. 用 `sockethelper.readfunc(fd)` 构造读取函数
2. `httpd.read_request(reader, max_header)` 解析请求
3. 用 `sockethelper.writefunc(fd)` 构造写函数
4. `httpd.write_response(writer, status, body)`

这里是基于官方示例整理的常用入口。
wiki 这一页没有单独列完整函数表。

### 10.2 HTTP 客户端 `httpc`

官方示例明确展示了：

- `httpc.request(method, host, url, recvheader, header, content)`
- `httpc.get(host, url, recvheader, header)`
- `httpc.post(host, url, form, recvheader, header)`

辅助接口：

- `httpc.dns(resolve_func)`
- `httpc.timeout(seconds)`

### 10.3 HTTPS

官方页给出的方式是：

- `httpc.request` 增加协议头参数，传 `protocol = "https"`
- 或使用 `http.httpc` / `http.tlshelper`

这部分依赖更细的实现时，建议直接看仓库里的 `lualib/http/*.lua`。

## 11. 调试控制台 `debug_console`

官方 `DebugConsole` 页给出了 telnet 连接和命令列表。

### 11.1 接入

- 默认监听 `localhost:8000`
- 可以用 `telnet 127.0.0.1 8000` 连接

### 11.2 常用命令

- `help`
- `list`
- `stat`
- `mem`
- `gc`
- `task`
- `info`
- `kill`
- `logon`
- `logoff`
- `inject`
- `call`
- `start`
- `exit`
- `signal`
- `cmem`

常见用途：

- 看当前 service 列表
- 看内存
- 看阻塞任务
- 远程起停服务
- 注入 Lua 代码排障

线上注意：

- 只绑定内网
- 最好外面再加一层运维入口控制

## 12. Snax

官方 `Snax` 页的态度很明确：

- 它是早期留给不想直接处理消息传递的开发者的糖层
- 现在官方已经不太推荐新项目使用

### 12.1 创建和绑定

- `snax.newservice(name, ...)`
- `snax.uniqueservice(name, ...)`
- `snax.globalservice(name, ...)`
- `snax.queryservice(name)`
- `snax.queryglobal(name, ...)`
- `snax.bind(handle, type)`
- `snax.kill(snaxobj, exitinfo)`
- `snax.self()`
- `snax.exit(...)`

### 12.2 两种接口风格

官方页定义了两类导出函数：

- `response.xxx(...)`
- `accept.xxx(...)`

调用侧对应：

- `snaxobj.req.xxx(...)`
- `snaxobj.post.xxx(...)`

区分很简单：

- `req` 要返回值
- `post` 不等结果

### 12.3 热更

官方页给出：

- `snax.hotfix(snaxservicename, source, type)`

### 12.4 使用判断

新项目更稳的路线是：

- 直接用 `skynet.start + dispatch + call/send`

除非团队已经长期用 snax。

## 13. 面向 MMORPG 项目的使用建议

### 13.1 玩家链路

- 玩家主状态放 `player_agent`
- 绝大多数业务 RPC 用 `skynet.call`
- 大量通知和低价值事件用 `skynet.send`

### 13.2 配置热更

- 静态表优先 `sharetable`
- 小量可热更新共享对象用 `sharedata`
- 不要把大量在线态塞进 `datacenter`

### 13.3 跨节点

- 关键写链路优先 `cluster.call`
- `cluster.send` 只放可丢消息
- 跨节点名字发现优先 `cluster.register/query`

### 13.4 网关

- 认证前阶段适合 `gateserver.openclient` 之后再放流量
- 超大广播要配合 `socket.limit` / `socket.warning`
- 如果要把连接从接入层转到业务层，记得 `socket.abandon`

### 13.5 调试

- `debug_console` 是线上排障利器
- 但 `inject`、`call`、`start` 这类命令权限必须控死

## 14. 官方来源

- API 总表：<https://github.com/cloudwu/skynet/wiki/APIList>
- Lua API：<https://github.com/cloudwu/skynet/wiki/LuaAPI>
- Socket：<https://github.com/cloudwu/skynet/wiki/Socket>
- SocketChannel：<https://github.com/cloudwu/skynet/wiki/SocketChannel>
- Cluster：<https://github.com/cloudwu/skynet/wiki/Cluster>
- ShareData：<https://github.com/cloudwu/skynet/wiki/ShareData>
- DataCenter：<https://github.com/cloudwu/skynet/wiki/DataCenter>
- GateServer：<https://github.com/cloudwu/skynet/wiki/GateServer>
- Http：<https://github.com/cloudwu/skynet/wiki/Http>
- DebugConsole：<https://github.com/cloudwu/skynet/wiki/DebugConsole>
- Config：<https://github.com/cloudwu/skynet/wiki/Config>
- Snax：<https://github.com/cloudwu/skynet/wiki/Snax>
