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

## Rule Development Policy

- No guessing
- Research first
- Prefer public/official evidence
- Cross-check with real Shadowrocket logs
- Apply minimum necessary rule changes
- Test in `shadowrocketzh` before production
- Evidence insufficient = no change

## `ch-extra-direct.lis`

该文件用于存放没有必要单独建立 Provider、但经过查证应使用 `DIRECT` 的中国大陆服务补充规则。
第一版仅包含向日葵精确主机：`DOMAIN,pubsub02.oray.net`。规则文件不包含策略字段，主配置通过 `RULE-SET` 指定 `DIRECT`。

本次采用最小范围：没有加入 `DOMAIN-SUFFIX,oray.net`、`DOMAIN-SUFFIX,oray.com` 或其他向日葵域名，因为官方资料只确认向日葵使用 UDP 3000，不能据此把整个 Oray 域名空间视为直连；其他域名继续由现有 `China` / `GEOIP,CN` 规则处理。

证据来源：

- 官方资料：[贝锐服务中心：登录向日葵需要开放哪些端口](https://service.oray.com/question/1131.html)，明确列出向日葵 PC 控制端上下线通知使用 UDP 3000。
- 官方产品：[向日葵官方下载](https://sunlogin.oray.com/download/)，确认向日葵客户端及控制端的官方产品归属。
- 公开实测：[OpenClash issue #2493](https://github.com/vernesong/OpenClash/issues/2493)，公开活动连接记录出现 `pubsub02.oray.net` 的 UDP 中国地址连接，并在同一设备记录中出现 `ws-std01.sunlogin.oray.com`。
- 本地 Shadowrocket 日志：当前工作环境没有可读取的 `.db` 或日志文件；本次关于该主机落入 `FINAL,PROXY` 的现象依据用户提供的实际日志观察，未将其表述为本地文件已复核。

因此本次只修复明确观察到的精确主机，不根据域名名称猜测或扩大直连范围。证据不足的其他域名不修改。
