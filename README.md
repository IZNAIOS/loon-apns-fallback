# Loon APNs Push Fallback

这个 Loon 插件只接管 Apple Push Notification service（APNs）相关流量，用于改善开启 Loon 后 X、Telegram 等 App 的 iOS 推送。它不包含节点、订阅、脚本或 MITM，也不会代理整个 `apple.com`。

本仓库根据上传配置定制：该配置已有 `SUB_A`、`SUB_B` 两个私密远程订阅和 `PROXY` 策略，且 `ipv6=false`。仓库只引用标签名，不包含或复制任何订阅 URL、Token 或节点信息。

## 为什么需要额外的策略组

插件里的普通规则本身不能实现条件式回退。真正的 fallback 必须由主配置中的策略组完成：

`APNs → APNS-FALLBACK → SUB_A → SUB_B → DIRECT`

APNs 是 iOS 的共享系统连接，客户端无法根据通知来自 X、Telegram 还是国内 App 来分别路由。`DIRECT` 兜底可以降低代理节点失效时所有通知一起中断的风险，但 iOS 重连和节点健康检查仍可能带来短暂延迟，无法承诺绝对零延迟。

## 安装

1. 备份当前 Loon 主配置。
2. 把 [fallback-group.conf](./fallback-group.conf) 中的策略组加入主配置的 `[Proxy Group]`。不要把私密订阅 URL 放进本仓库。
3. 在 Loon 的插件页面导入以下 Raw 地址：

   `https://raw.githubusercontent.com/IZNAIOS/loon-apns-fallback/main/apns-push-fallback.plugin`

4. 导入或编辑插件时，把插件中的 `PROXY` 映射到主配置的 `APNS-FALLBACK`，然后启用。
5. 重新连接一次 Loon；必要时开关一次飞行模式，让 iOS 重建 APNs 长连接。

如果插件日志仍命中主配置中的 `DOMAIN-SUFFIX,apple.com,DIRECT`，请把本插件移动到更高优先级并再次检查策略映射。不要删除原配置的 Apple 直连规则；它们对其他 Apple 服务仍然有用。

## 规则范围

- `DOMAIN-SUFFIX,push.apple.com` 已覆盖 `push.apple.com`、`gateway.push.apple.com`、`api.push.apple.com` 和 `sandbox.push.apple.com`。
- IPv4/IPv6 网段来自 Apple 的 APNs 官方故障排查文档。
- 没有加入过宽的 `akadns.net`、`apple.com.edgekey.net` 或整个 `apple.com`，以减少影响 Handoff、通用剪贴板、AirDrop 及其他 Apple 服务的概率。
- 没有 MITM、重写或脚本；APNs TLS 不应被解密。

## 验证与回滚

锁屏后分别让海外 App 和国内 App 发送一条测试通知。随后暂时禁用 `SUB_A`、`SUB_B` 或选择不可用节点，确认 `APNS-FALLBACK` 会切到 `DIRECT`。恢复节点后等待下一次健康检查，策略会重新按顺序选择可用订阅节点。

如出现 Handoff、通用剪贴板、Apple ID 或其他异常，先停用本插件并重连网络。插件不修改证书和系统通知设置，停用后即可回滚。

## 隐私

仓库只包含公开的 Apple 域名和网段。不要在 Issue、截图或提交中粘贴订阅 URL、节点地址、Token、UUID、密码或完整主配置。

## 参考

- [Apple：APNs 所需端口和网段](https://support.apple.com/en-us/102266)
- [Apple：企业网络所需主机（`*.push.apple.com`）](https://support.apple.com/en-us/101555)
- [Loon 官方插件示例](https://github.com/Loon0x00/LoonExampleConfig/blob/master/Plugin/Plugin_Example2.plugin)
- [Loon 官方 fallback 配置示例](https://github.com/Loon0x00/LoonExampleConfig/blob/master/example.conf)
- [LINUX DO：APNs 处理思路](https://linux.do/t/topic/2453154/1)
- [LINUX DO：节点失效时需要 fallback 的提醒](https://linux.do/t/topic/2453154/11)
- [X：推送域名及 fallback 讨论](https://x.com/qkl2058/status/2051001181938602447)
