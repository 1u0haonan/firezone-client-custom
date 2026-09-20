# Firezone 自托管客户端

这是 1u0haonan 维护的非官方定制版本，不是 Firezone 官方发行包。

## 源码基线

- 上游： https://github.com/firezone/firezone
- 固定提交： `bbd97e44b72fd405a738202a3a0674f866569cdf`，与本次调研环境的 Portal 源码提交对应。
- 定制分支： `self-hosted-clients`。不要直接跟随上游 main 升级。
- 基线内版本号：Apple 1.5.20、Windows/Linux GUI 1.5.17、Android 1.5.14、Headless 1.5.12。这些是此提交内的版本号，不意味着等同于同名正式 release/tag。
- 保留上游许可证、版权声明及依赖锁文件。此仓库不包含部署密码、Token 或签名证书。

## 定制内容

- macOS、iOS、Windows、Linux、Android 和 Headless 的默认认证地址为 `https://portal.apps.ct-hs-1.xtj.zone`。
- 默认 API 地址为 `wss://api.apps.ct-hs-1.xtj.zone`（部分 URL 类型会规范化为带末尾斜杠）。
- Debug 和 Release 均使用上述地址，仍允许显式配置覆盖。
- macOS 和 Windows/Linux GUI 在代码层禁用官方版本检查，不请求官方 release 接口，也不显示由该检查产生的 “Update available” 提醒；MDM 或调试开关不会重新开启它。
- Android/iOS 的应用商店更新提示不属于这段应用内版本检查；本流程不会发布至应用商店。
- 定制构建关闭上游遥测，但仍会与自托管 Portal/API 通信。

已安装客户端的保存配置、MDM、命令行参数或环境变量可能覆盖新默认值。升级后请检查实际 URL；不要为切换默认值删除整个系统钥匙串或清空无关配置。

禁用更新提醒不代表不需要安全更新。维护者应定期审查上游安全修复，并手动验证客户端与服务端兼容性。

## GitHub Actions 使用

仓库 Actions → **Custom self-hosted clients** → **Run workflow**：

1. 选择 `self-hosted-clients` 分支。
2. `platform` 默认 `macos`；也可选择 `ios`、`linux`、`windows`、`android` 或 `all`。
3. 等待任务完成，在该次运行页面的 Artifacts 下载。产物保留 14 天，不自动创建公开 Release，也不提交到商店。
4. Apple 构建失败时查看上传的构建日志；其他平台查看任务日志。绿灯仅代表对应构建/测试通过，不代表 VPN 端到端联网已验证。

此工作流仅手动触发，令牌只有 contents:read，不使用上游的发布、Sentry、Azure 或应用商店凭证。仓库中保留的上游工作流不用于本定制发行，并在此 fork 的 Actions 设置中禁用。

## 各平台产物和限制

| 平台 | 初始构建产物 | 使用限制 |
| --- | --- | --- |
| macOS | 未签名 Firezone.app ZIP，Release 通用架构 | **仅编译验证**，没有开发者签名、Network Extension provisioning profile 或公证，不能承诺可启动 VPN |
| iOS | 未签名真机 Firezone.app ZIP | **仅编译验证**，不是可直接安装的 IPA/TestFlight 包 |
| Windows x86_64 | 未签名 MSI、Headless EXE | SmartScreen/签名限制；此版本 GUI 使用 sparse MSIX 身份，未签名包可能无法注册或运行，不能当正式可用安装包 |
| Linux x86_64 | DEB/RPM、Headless 二进制 | 需管理员安装及相应系统依赖，需实际联网验证 |
| Android | Debug 签名 APK | 调研用途；CI 临时 Debug 密钥跨运行可能变化，不可保证覆盖安装，不兼容官方包签名 |

“全平台”指上述操作系统，不代表所有 CPU 架构、所有商店分发方式或所有签名配置均已验证。

## macOS / iOS 正式可用包的前提

当前没有 Apple Developer 账号，因此先做未签名编译。要得到能正常使用 Network Extension 的专属客户端，需要：

1. 自有 Apple Developer Program 账号及团队 ID。
2. 自有 App 和 Network Extension Bundle ID、App Groups 及相应 entitlement。
3. 匹配的签名证书、私钥和 provisioning profiles。macOS 独立分发还应配置 Developer ID 签名及公证；iOS 配置相应开发/分发渠道。
4. 更新 `swift/apple/Firezone/xcconfig/config.xcconfig` 和关联签名/扩展配置，并在 GitHub Actions Secrets 安全配置签名资料，再增加签名分发流程。

目前仍保留上游的应用标识以最小化源码变更，**不意味着拥有官方团队的签名权限，也不要与官方客户端并排安装**。正式定制标识需要连同回调 URL、App Groups、钥匙串访问组及扩展一并验证。

不要在聊天或仓库中提交证书密码、私钥或部署 Token；不要关闭 SIP、Gatekeeper 或系统扩展安全检查来绕过签名要求。

## 验证与回退

- Apple 工作流执行 ConfigurationTests，桌面工作流执行定制更新检查回归测试和 Headless CLI 测试。
- 构建后仍需验证：认证跳转域名、OIDC 回调、连接自托管 API、策略实时同步、直连/Relay 和目标资源访问。
- 回退可安装原官方客户端并重新配置自托管地址；先退出正在运行的定制客户端，注意保存配置与签名兼容性。
- 默认 URL 修改不会改变服务端或 Gateway/Relay 部署，也不会自动迁移已有用户配置。
