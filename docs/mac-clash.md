# Mac 客户端：Clash Verge / Mihomo 单设备分流

本页针对一台 Mac。服务端可以是 [私网路径](private-path.md)或已验收的[公网路径](home-nat.md)；两者只有“可达地址与端口”不同。实际客户端版本、菜单和权限以设备上安装的版本为准。

1. 在 Mac 上安装并打开 Clash Verge，确认内核是支持 Hysteria 2 的 Mihomo。先保存现有配置和系统代理状态，不覆盖正在使用的订阅。
2. 从 [Mihomo 示例](../examples/mihomo.example.yaml)制作一份新的**本地配置**，保存在仓库外。填入本次地址、UDP 端口、强随机密码、证书名称及用户指定域名；保留 `skip-cert-verify: false` 与最后的 `MATCH,DIRECT`。私网路径用主机 Tailnet 地址；公网路径用从家庭外可达的入口。若使用私有 CA，先核对当前 Mihomo 版本的信任配置能力并做证书校验实测，不能以跳过校验解决。
3. 导入新配置并先用“规则模式 + 系统代理”，不要同时开启 Tailscale Exit Node。检查 Mac 能访问 Tailnet 主机时，测试 HY2 连接；再检查一条海外规则与一条 DIRECT 的日志命中。系统代理只覆盖遵循系统代理设置的应用；用户要覆盖的应用如果不遵循，必须单独验证，再决定是否开启 TUN。
4. 如确需 TUN，先确认从 Mac 到 HY2 主机的连接没有被自己的规则重新代理，也没有与 Tailscale 的默认路由竞争。逐个开启、验证并记录恢复方法；一旦断网，先关闭 Clash Verge 的 TUN/系统代理，再关闭 Tailscale Exit Node，回到正常直连网络。
5. 将 Mac 从家庭 Wi-Fi 切到独立网络，按 [验收表](validation.md)检查出口、持续使用、重连和主机重启。日常关闭分流时，在 Clash Verge 关闭本次代理模式；若另行配置并验收了 Tailscale Exit Node，切换前确保 Clash Verge 不再接管整机路由。

本示例是最小的“指定域名海外、其他直连”。一个网站可能使用多个域名和 CDN；出现部分内容不加载时，从客户端请求日志补充**确实需要**的域名，避免直接把所有流量改走海外。
