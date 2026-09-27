# 家中常开主机：新建 HY2 服务

本页以 **macOS Mac mini** 为具体参考；若主机是 Linux，执行 Agent 改用 [Hysteria 官方 Linux 安装与 systemd 服务](https://v2.hysteria.network/docs/getting-started/Server-Installation-Script/)，保留同样的配置、验收和回退关口。任何平台都先检查现有服务，**新建独立实例**，不改旧实例；已有默认实例时，不直接运行可能覆盖默认配置路径或服务名的安装脚本。

## Windows 主机边界

Windows 可由执行 Agent 参照 [官方安装页](https://v2.hysteria.network/docs/getting-started/Installation/)核对当前可用构建及 CPU 架构，用独立 `hysteria.exe server -c <本次配置绝对路径>` 做前台测试。真实配置与密钥放仓库外的受限目录，用 Windows 文件权限限制运行账户和管理员访问；Windows 防火墙只新增本次程序/UDP 端口及必要来源规则，不关闭整机防火墙。

前台与异网验收通过后，选择独立 Windows 服务或“系统启动时、用户未登录也运行”的计划任务，明确运行账户、工作目录、程序/配置绝对路径和日志权限，再实测重启、休眠及断电恢复。不能照搬下方的 launchd/systemd 操作，也不能用开机启动文件夹冒充无人登录可用。**本仓库尚未做 Windows 实机验收，也未提供特定 Windows 版本的逐屏服务配置手册**；执行 Agent 必须按实际系统补齐并交付本次启动/撤销步骤，不得仅凭这些说明宣布支持已验证。

## 1. 识别主机与安装官方程序

在主机上读取 OS、CPU 架构、已有 `hysteria` 版本、UDP 监听、系统防火墙和可用存储。macOS 可用 `uname -m` 确认 Apple Silicon (`arm64`) 或 Intel (`x86_64`)，从 [官方安装页](https://v2.hysteria.network/docs/getting-started/Installation/)下载对应 `hysteria-darwin-*` 可执行文件；核对来源、版本与架构后放进主机授权的程序目录。不要把第三方二进制提交到本仓库。能使用已有、来源明确的相同版本时不重复安装。

本次服务文件放在主机授权的受限目录，**不在 Git 仓库内**。执行 Agent 先选择一个空闲的高位内部 UDP 端口，检查它不与已有进程、映射及其他用户服务冲突。端口不能直接照抄示例。

## 2. 准备密码、证书和配置

用系统安全随机源生成新的长密码，例如在授权的私密终端用 `openssl rand -base64 48`。不要将命令结果写进聊天、终端共享记录、Git、工单或公开报告。真实 YAML、私钥和密码文件只让运行账户及管理员读取。

证书必须与客户端校验名称匹配。可选方式：

- **公网 iPhone 路径：**优先使用用户控制域名和可正常验证的公开 CA 证书，并在离境前验证续期。若没有域名或取得证书的权限，先说明该阻塞，不以“跳过验证”继续。
- **Tailnet 私网路径：**可使用客户端已信任的私有 CA 证书并匹配名称；若选择 Tailscale 的 `tailscale cert`，先向用户说明主机名会进入公开 Certificate Transparency 记录，证书文件需自行续期，再按 [Tailscale 官方说明](https://tailscale.com/docs/how-to/set-up-https-certificates)操作。

从 [服务端模板](../examples/server.example.yaml)制作真实配置，替换监听地址、内部端口、证书路径和密码。私网路径绑定主机的 Tailnet 地址；公网 IPv4 映射路径绑定保留的 LAN 地址，IPv6 用方括号包住地址，并单独限制入站。不要默认省略地址从而监听所有网卡。先检查证书 SAN 包含客户端使用的名称、私钥权限及证书有效期。不要使用模板中的占位值启动服务。

使用公开证书时，域名解析与证书签发是两个步骤。优先复用已验证的证书流程，或用 [ACME DNS-01](https://letsencrypt.org/docs/challenge-types/) 验证你控制的域名，DNS API 凭据按最小权限保存在私密位置；这不需要新增 TCP 入站。若改用 HTTP-01 / TLS-ALPN-01，需要分别验证其 TCP 入站条件，不能认为仅有 HY2 的 UDP 映射就能自动签发。先完成一次续期演练，记录续期负责人/机制和过期告警；使用文件证书时按官方说明保留可读权限和完整链。

保留模板中的 [服务端 ACL](https://v2.hysteria.network/docs/advanced/ACL/)，阻止代理请求访问本机、私有 LAN、链路本地及 Tailnet 地址；此处限制的是代理访问的目标，不影响客户端经 Tailnet 连接服务。另在私密配置中禁止回访本家庭公网地址及分配给家庭的全球 IPv6 前缀，避免经公网回环绕过内网限制；地址/前缀改变时同步维护。通用 ACL 不是家庭网络隔离的替代，建议让服务账户权限最小、保持主机防火墙。验收时请求一个专门的无敏感测试目标，确认内网目标被拒绝，不扫描家庭设备。

如果主机原有代理/DNS 返回 fake-IP（例如基准测试保留网段），服务端 ACL 会拒绝这类结果。先检查真实域名解析；需要时为本次 HY2 实例选择独立、受信任且在当地可达的解析器，按 [官方 resolver 配置](https://v2.hysteria.network/docs/advanced/Full-Server-Config/#resolver)验证。不删除内网拒绝规则来绕过解析问题，也不擅自修改整机 DNS。

## 3. 前台启动并检查

在服务目录运行 `hysteria server -c /absolute/path/to/server.yaml`（将 `hysteria` 换成实际已验证的可执行文件绝对路径）。看到官方文档中的成功启动日志后，从另一终端查看进程与本次 UDP 端口是否监听，例如 `lsof -nP -iUDP:<本次内部端口>`；再用匹配证书的客户端实际认证并发起网页请求。仅有监听记录不算链路验收。

如果服务启动失败，保留脱敏错误和配置备份，只修复本次新实例。前台验收通过前，不建立路由器映射之外的其他开放，也不设开机启动。

## 4. 通过链路验收后设开机启动

macOS 上以**独立的 launchd 服务标签**创建启动项，不复用或覆盖已有标签。需要系统启动后、无人登录时可用，就使用系统 LaunchDaemon，而不是仅在用户登录后启动的 LaunchAgent。启动项必须使用程序和配置的绝对路径，限定运行账户、配置/密钥读取权限和日志位置；先手动加载并读回进程状态，再重启主机验证。FileVault 冷启动可能需要本地解锁才能启动 macOS；改放配置目录无法消除这一条件。不要为了方便擅自关闭 FileVault。离境前确认停电后恢复方式、有人协助解锁或其他已验证方案；没有则明确列为无人值守限制。部分新系统支持特定的远程解锁方式，须按 [Apple 当前说明](https://support.apple.com/en-ca/guide/security/sec8447f5049/web)核对；不能假设同机 Tailscale 在启动解锁前已经在线，也不因此新增公网 SSH。另检查系统睡眠、断电后恢复和磁盘挂载条件，不能把“登录后服务自动启动”写成“停电后一定自动恢复”。

Linux 按官方说明使用独立 systemd unit，检查 `enabled`、`active` 和重启后状态。健康探测应检查真实请求或有效认证，不能只检查 PID；故障期间保留时间和脱敏日志，不因一次自动重启就宣称根因已修复。

## 撤销本次主机改动

先让客户端停用本次节点，再卸载**本次** launchd/systemd 启动项，停止本次 HY2 进程，移除仅属于本次的防火墙放行与受限配置；如果程序本来存在，不删除共用二进制。路由器映射和 DHCP 保留另按 [验收与撤销](validation.md)处理。最后验证原有服务、家中联网和远程管理仍正常。
