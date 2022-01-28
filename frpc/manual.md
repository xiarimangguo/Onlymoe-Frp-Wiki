# frpc 用户手册

SakuraFrp frpc 基于原版 frpc 进行了一些修改，下面是 SakuraFrp frpc 的用户手册，高级用户请直接查看 [此章节](#高级用户)

## 普通用户

### 使用 TUI 启动

在 `frpc.ini` 不存在的情况下，不带参数直接运行 frpc 会出现一个交互式 UI

输入 **访问密钥**，然后使用 `Tab` 键切换到 **Login** 按钮并按 `回车` 键登录 (若终端支持也可使用鼠标进行操作)

![](_images/tui-0.png)

登录成功后 TUI 会显示当前账户下的隧道列表，使用方向键选中想要启动的隧道，按空格标为绿色 (或使用鼠标直接点击隧道)

?> 可以一次性启用多个隧道，但是这些隧道必须位于同一节点下  
您也可以直接选中节点来启用该节点下的所有隧道

![](_images/tui-1.png)

选择完毕后，按 `Ctrl-C` 即可启动隧道，相关启动参数会被保存到配置文件 `frpc.ini` 中，下次不带参数直接运行 frpc 时不再显示 TUI 而是直接启动隧道

![](_images/tui-2.png)

### 从命令行启动

!> 如果您没有按照 [Linux 使用教程](/frpc/usage/linux) 安装 frpc，只是下载了文件    
或是使用 Windows 系统，启动时 frpc 要换成下载到的的文件名  
如 `frpc_windows_386.exe` 、 `frpc_linux_amd64` 等

![](_images/manual-0.png)

假设您的 Token 为 `wdnmdtoken6666666`

![](_images/manual-1.png)

您的隧道列表如下图所示

![](_images/manual-2.png)

假设当前运行的系统为 32 位的 Windows 系统，因此您下载到的 frpc 文件名是 `frpc_windows_386.exe` 。

1. 启动图中的第一条隧道：
```cmd
frpc_windows_386.exe -f wdnmdtoken666666:85823
```

1. 启动 **#6 镇江双线** 节点下的所有隧道，可以不输入隧道 ID：
```cmd
frpc_windows_386.exe -f wdnmdtoken666666:n6
```

1. 第二条命令也可以替换为手动输入两个隧道 ID，效果是相同的：
```cmd
frpc_windows_386.exe -f wdnmdtoken666666:85823,94617
```

## 高级用户

下面的文档详细解释了 Sakura Frp 提供的 frpc 与原开源版本的差异

### 新增启动参数

1. `-f, --fetch_config`
   - 从 Sakura Frp 服务器自动拉取配置文件
   - 参数列表 1: `<Token>:<TunnelID>[,<TunnelID>[,<TunnelID>...]]`
   - 参数列表 2: `<Token>:n<NodeID>`
1. `-w, --write_config`
   - 拉取配置文件成功后将配置文件写入 `./frpc.ini` 中
1. `--update`
   - 进行自动更新，如果不设置该选项默认只进行更新检查而不自动更新
1. `-n, --no_check_update`
   - 启动时不检查更新
1. `--watch`
   - 监控指定 PID 并在进程退出时退出 frpc，该参数用于避免启动器崩溃造成的进程残留
   - 参数列表: `<PID>`

### 新增配置文件选项

##### [common]
1. `sakura_mode = <Boolean>`
   - 启用 Sakura Frp 自有的各类特性，设置为 `false` 将 **禁用所有** Sakura Frp 相关特性，默认值为 `false` 
1. `use_recover = <Boolean>`
   - 启用重连功能，默认值为 `false`
1. `persist_runid = <Boolean>`
   - 该选项启用后 RunID 将不再从服务器拉取而是根据本机特征 & 隧道信息生成，默认值为 `true`
1. `remote_control = <String>`
   - 配置 Sakura Frp 远程管理端对端加密密码，留空则禁用远程管理相关功能，默认值为空
   - 请参阅 [frpc 远程管理](/frpc/remote) 获取更多信息

##### [tcp_proxy]
1. `concat_packet = <Int>`
   - 配置合并封包功能的最小字节数，有助于减少小包并降低服务器网卡 PPS，设置为 `-1` 将禁用此功能，默认值为 `-1`
1. `auto_https = <String>`
   - 当您需要通过 TCP 隧道转发 HTTP 流量时，该流量可能会被机房拦截
   - 通过该配置项可将流量自动加上 TLS 层 (转为 HTTPS) 来避免拦截，取值如下:
     - 留空 **[默认值]**: 禁用自动 HTTPS 功能
     - `auto`: frpc 将使用`server_name`作为证书 Common Name
     - 自定义域名:  
       frpc 将尝试加载 `<自定义域名>.crt` 和 `<自定义域名>.key` 两个证书文件  
       若文件不存在则使用 `自定义域名` 作为 *CommonName* 生成一份自签名证书并保存到上述文件中  
       *注: 若文件已存在，`自定义域名` 就作为一个单纯的文件名进行处理，不会对证书产生影响*
1. `auth_pass = <String>`
   - 配置 SakuraFrp 访问认证功能的密码，留空则禁用访问认证相关功能，默认值为空
   - 请参阅 [安全指南-frpc 访问认证](/bestpractice/security#frpc-访问认证) 获取更多信息
1. `auth_time = <String>`
   - 配置访问认证功能在没有勾选「记住」时的过期时间，默认为 `2h`
   - 接受的后缀为`h`/`m`/`s`，请从大到小排列，如 `1h3m10s`, `20m`
1. `auth_mode = <String>`
   - 配置 SakuraFrp 访问认证功能的认证模式，取值如下:
     - `online` **[默认值]**: 允许通过密码认证或通过 SakuraFrp 面板进行授权
     - `standalone`: 仅允许通过密码认证, 忽略 frps 下发的 IP 授权信息

##### [https_proxy]
1. `force_https = <Int>`
   - 配置 frps 自动重定向 HTTP 请求到 HTTPS 的功能，有助于减少隧道占用，取值如下:
     - `0` **[默认值]**: 禁用自动重定向功能
     - 其他数字: 启用重定向功能并在重定向时返回该数字作为状态码，推荐使用 `301` 或 `302`

### 新增特性

1. 日志输出会对用户 Token 进行打码，防止 Token 泄漏
1. 连接成功后会输出一段提示信息，提示用户当前隧道的连接方式
   - 该提示信息不会匹配日志格式。目的是兼容旧版启动器对旧版本 frpc 日志解析的逻辑
1. 与服务器连接断开后会尝试自动进行重连，客户端将尝试直接恢复 MUX 连接，因此短暂的断线 (10 秒内) 能实现用户无感知重连
1. 根据本机特征 & 隧道信息生成 `RunID`
   - 这有助于服务端快速辨识掉线的 `frpc` 并进行重连作业。生成的 `RunID` 为一串 `Hash`，不会包含敏感信息
1. 启动时会从 API 服务器的 `/client/get_version` 获取最新版本信息, 并提示用户进行更新或进行自动更新
1. 内建 TUI，方便用户在无参数启动时进行配置
1. 增加封包合并功能以减少小包
1. 下发客户端限速，提升连接体验并有助于解决部分应用断线问题
   - 服务端读取限速存在上行速度在 **跑满本地带宽** 和 **0 Byte/s** 之间反复跳动的问题，在客户端也进行限制即可获得稳定的最大速度
1. 增加自动 TLS 配置功能以简化通过 TCP 隧道调试 HTTP 应用的配置流程
1. 增加自动 HTTPS 重定向功能以减少隧道占用
1. 增加访问认证功能以提升隧道安全性
