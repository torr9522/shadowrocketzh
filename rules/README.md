# Shadowrocket Rules Provider

这是本仓库自行维护的 Shadowrocket Rules Provider。第一阶段仅维护 Apple 分流规则，后续规则迁移将独立评估和实施。

## 文件

### `apple-direct.list`

来源：

- [blackmatrix7 Shadowrocket Apple.list](https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Shadowrocket/Apple/Apple.list)
- [blackmatrix7 Shadowrocket Apple_Domain.list](https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Shadowrocket/Apple/Apple_Domain.list)

`Apple.list` 中有效的 Shadowrocket 规则直接保留。`Apple_Domain.list` 中以 `.` 开头的域名转换为 `DOMAIN-SUFFIX`，无前导点的完整域名转换为 `DOMAIN`。合并后按完整规则去重。

### `apple-proxy.list`

来源：

- [blackmatrix7 AppleProxy](https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Shadowrocket/AppleProxy/AppleProxy.list)
- [blackmatrix7 iCloudPrivateRelay](https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Shadowrocket/iCloudPrivateRelay/iCloudPrivateRelay.list)

第一版只选取已明确属于 iCloud Private Relay、需要特殊处理的最小规则集合，不直接复制完整 AppleProxy 规则。

## 格式与引用

两个文件均使用 Shadowrocket 原生 `RULE-SET` 文件格式，文件内部不包含策略字段。主配置应按以下顺序引用：

```ini
RULE-SET,https://raw.githubusercontent.com/torr9522/shadowrocket-config-single-file/shadowrocket-rules/rules/apple-proxy.list,PROXY
RULE-SET,https://raw.githubusercontent.com/torr9522/shadowrocket-config-single-file/shadowrocket-rules/rules/apple-direct.list,DIRECT
```

`apple-proxy.list` 必须放在 `apple-direct.list` 前面，以便代理例外优先匹配。
