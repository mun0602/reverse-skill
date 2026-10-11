# 2026-10-07 AgileDotNet WPF 空 IL

## 场景分类
二进制分析 / .NET

## 目标概述
离线 dump 一个 WPF .NET Framework 4.7.2 样本。托管头真实，业务方法体被 AgileDotNet 虚拟化。

## Scope 摘要（脱敏）
- auth_basis: preset offline-sample
- network_profile: offline
- asset_types: [local pe]

## 角色
- lead_role: lead
- specialists: [dotnet-reverse]

## 完整执行链路
1. master-route.sh 命中 dotnet-reverse。
2. case-init.sh --preset offline-sample。
3. file + CLR DataDirectory[14] 确认托管。无 Confuser/Reactor 字符串，跳过 de4dot。
4. ilspycmd -p 产出签名工程。方法体为 ret / ldnull ret。
5. ilspycmd -il 全量 IL。2639 方法里仅 44 个大于 20 字节，最大者是调用 AgileDotNetRT.Initialize 的 .cctor。
6. 编译器生成的异步状态机名仍在元数据里，用作调用图，不当作实现。

## Evidence 链摘要（脱敏）
| E-id | severity | status | source_type | 可复用命令模式 | 关联 Finding |
|------|----------|--------|-------------|----------------|--------------|
| E-001 | info | validated | command | `file sample.exe` 含 Mono/.Net 且 CLR RVA 非 0 | F-001 |
| E-002 | info | validated | command | `ilspycmd -il sample.exe` 后统计 Code size | F-001 |

## Finding / Path 摘要
- top_finding: AgileDotNet 把业务 IL 换成桩，运行时用 ModuleHandle 代理恢复。静态 ilspycmd 只能留下签名和异步状态机名。
- path_type: callflow
- path_one_liner: 识别 CLR → 排除 de4dot 壳 → ilspycmd 工程 + 全量 IL → 用状态机名画流程，不把空方法当逻辑。

## 踩坑记录

| 问题 | 原因 | 解决方案 | 耗时 |
|------|------|---------|------|
| -t 过滤失败仍吐出整份 IL | 类型名被混淆，过滤器未命中就转储全部 | 把输出当全量 IL，改名 il-all.il | 短 |
| 资源列表只有 GUID | Agile 加密了资源，没有 BAML | 不再等 XAML | 短 |

## 工具链发现
macOS 上 ilspycmd 11.0 足够确认 Agile 桩。de4dot 解不了这种虚拟化。要见业务 IL 必须在 Windows .NET Framework 4.7.2 上做运行时 dump。

## 关键代码/命令

```
ilspycmd -p -o dump sample.exe
ilspycmd -il sample.exe
```

## 对本包的改进建议
dotnet-reverse 应把 AgileDotNet（方法体 ret + AgileDotNetRT.Initialize）写成与 Confuser 不同的分支：不要跑 de4dot，直接告诉用户静态 IL 为空。

## 可复用的模式/脚本片段
统计 `Code size:`。若绝大多数为 1 或 2，且存在 `<AgileDotNetRT>`，停止静态深挖。

## 进化动作
- [ ] 更新了路由矩阵
- [ ] 更新了 tool-index
- [ ] 更新了 bootstrap-manifest
- [x] 更新了子 skill 文档
- [ ] 新增了 pitfalls 记录
- [ ] 无需更新

## 环境信息
- OS: macOS
- 工具版本: ilspycmd 11.0.0.9375
- 目标平台/版本: .NET Framework 4.7.2 WPF
