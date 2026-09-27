# 把海外家宽变成私人出口：Agent 部署指南

目标：离开海外住处前，先留一条能远程管理家中常开主机的救援通道，再让一台自己的设备按规则通过家庭宽带出网。救援通道采用 Tailscale、获授权的远程登录和可选的图形远控；日常分流采用 Hysteria 2（HY2）及兼容的规则客户端。适用于你拥有或获授权管理的家庭网络。仓库不提供订阅、多人账号或收费功能。

**验证状态：尚未在空白家庭网络完成端到端验收。** 示例已通过本地格式检查，Hysteria 服务端与客户端模板用合成凭据完成过回环连接；这些检查不能替代路由器登录、手机导入或真实的异网测试。Windows 与 Android 分支也尚未实机验收。

## 从这里开始

先确认自己具备：获授权的海外家宽、能持续开机的主机、主机管理员权限，以及需要时配合登录路由器/手机的本人。Agent 能安装配置软件，但不能替你取得这些条件。只有普通对话能力、无法操作设备的 AI 只能指导，不能直接部署。

1. 将 [Agent 执行说明](docs/agent-deploy.md) 交给能操作你的主力设备与家中常开主机的 Agent，并提供需要走海外出口的网站。它必须先盘点设备，不得照搬示例端口。
2. 先按 [救援通道](docs/remote-admin.md)验证离家后仍能管理主机。若需要整机临时走家宽，另外配置并验收 [Exit Node](docs/exit-node.md)。
3. 为 HY2 判断入口：有可用公网 UDP 时按 [家庭公网入口](docs/home-nat.md)设置单条映射；若没有，评估 [Tailscale 私网 HY2](docs/private-path.md)。私网路径在 Mac 等可并行运行两类客户端的设备上才可能保留规则分流；普通 iPhone 单 VPN 场景不能把它当成已完成的分流方案。
4. 按 [常开主机部署](docs/server-host.md)安装服务，再用 [服务端](examples/server.example.yaml) 与 [Mihomo 分流](examples/mihomo.example.yaml)示例生成**仓库外**的真实配置。[Hysteria 客户端示例](examples/client.example.yaml)仅供连接烟测，不负责域名分流。所有占位字段必须由本次部署的实测值替换。
5. 按 [验收与回退](docs/validation.md)先从家庭外网络测试，到中国后再用当地网络复测。没有中国侧测试设备时，完成可做的验收，把中国网络质量标为待验证。

客户端分别参照 [Mac / Clash Verge](docs/mac-clash.md)、[iPhone / Shadowrocket](docs/iphone-shadowrocket.md) 或 [Android / 规则客户端](docs/android-client.md)。先完成其中**一台**，不要把多台设备的验收混成一个结果。Windows 可作为服务端，但目前仅有[实施边界](docs/server-host.md#windows-主机边界)，尚无 Windows 实机端到端验收。

```text
单台设备 ── HY2 / UDP ── 家中常开主机 ── 海外家庭宽带 ── 指定网站
    └─────────────── DIRECT ──────────────────────── 其他网站
```

## 成功的含义

**Exit Node：**设备离开家庭 Wi-Fi 后，启用时出口匹配家宽，关闭时恢复当前网络；重连及主机重启后仍能恢复。它没有按网站分流验收。

**HY2 分流：**需要同时观察到：客户端规则命中、服务端认证与双向数据、指定网站呈现家庭出口、直连网站不绕行、设备离开家庭 Wi-Fi 后仍可连接，以及主机重启后服务恢复。家里连通、单次节点测速或路由器页面显示“映射成功”都不足以替代这些检查。

## 风险单独看

“自己的住宅出口”不等于平台认证的“纯净 IP”，也不保证解锁服务或免风控。权限、凭据、家庭内网暴露、跨境可达性和无人值守限制，集中见 [风险与限制](docs/risks.md)。

## 范围与边界

- 先实现单设备自用。若客户端系统不允许 Tailscale 与规则客户端并行，需要经过实测的公网路径才能完成 HY2 分流；没有可达公网入口时要明确记录未完成，不要强行双开整机 VPN。
- Tailscale 先用于远程管理。若还需要 Exit Node 整机保底，须单独配置、获得 Tailnet 管理端批准并异网验收；使用 HY2 分流时不启用 Exit Node，切换前先关闭会接管整机路由的客户端模式。
- 无公网入站时，手动映射不能解决运营商 NAT。停止公网分支并向使用者说明可选路径。
- 任何中国网络的可用性和晚高峰质量都需要中国侧实际测试；无测试端时不得承诺。
- 本仓库工作区只保留占位配置、文档和通用保留网段，没有生产凭据、第三方安装包或路由器自动化脚本。复制到个人环境的配置不要提交回仓库。

官方依据：[Hysteria 安装](https://v2.hysteria.network/docs/getting-started/Installation/)、[服务端](https://v2.hysteria.network/docs/getting-started/Server/)、[客户端](https://v2.hysteria.network/docs/getting-started/Client/)、[Mihomo HY2](https://wiki.metacubex.one/config/proxies/hysteria2/)、[Mihomo 规则](https://wiki.metacubex.one/en/config/rules/)、[Tailscale Exit Node](https://tailscale.com/docs/features/exit-nodes)。部署前由执行 Agent 核对当前文档。

文档与占位示例使用 [MIT License](LICENSE)。公开使用前请自行核对设备、当地网络和所选客户端；这里的占位配置不可直接导入。
