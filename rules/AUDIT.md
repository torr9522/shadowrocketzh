# Shadowrocket Rule Architecture Audit

Audit date: 2026-08-20

## Executive Summary

- `shadowrocketzh.conf` contains 47 external `RULE-SET` references and 0 external `DOMAIN-SET` references.
- 43 references come from `blackmatrix7/ios_rule_script`, 1 from `iab0x00/ProxyRules`, and 3 from the local `shadowrocketzh` repository.
- All 43 blackmatrix references use the `rule/Shadowrocket/` path. No `QuantumultX`, Clash or Surge path was found in the current configuration.
- All 47 URLs returned HTTP 200 during this audit.
- Current file-level freshness is `ACTIVE 3`, `SLOW 4`, `STALE 37`, `UNKNOWN 3`. This is file-level status, not repository-level status.
- Several providers intentionally split their generated output: the main `RULE-SET` file contains IP/USER-AGENT/keyword rules while `_Domain.list` contains bare domains. For example, Tencent is 25 main-file rules plus 2,498 domain-set entries = 2,523; Alibaba is 57 + 1,263 = 1,320; Global is 198 + 34,852 = 35,050; China is 63 + 3,689 = 3,752. The important finding is that the current configuration loads only the main file for these providers, not the domain companion.
- The current ordering is intentionally whitelist-first for domestic DIRECT and exception-first for Apple: domestic rules, international rules, Apple proxy, Apple direct, Global PROXY, China DIRECT, Lan, GEOIP and FINAL.

## Current Architecture

The effective rule groups, in order, are:

1. Manual `DOMAIN-SUFFIX,...,PROXY` exceptions.
2. Domestic application/platform DIRECT providers.
3. The local `ch-extra-direct` provider.
4. International service PROXY providers and the HBO `Max` provider.
5. Local Apple proxy exceptions before local Apple direct rules.
6. Global PROXY.
7. China DIRECT and Lan DIRECT.
8. Local IP rules, `GEOIP,CN,DIRECT`, then `FINAL,PROXY`.

Rules are first-match based. Therefore the placement of `AI` before Apple is significant: AI can claim Apple Intelligence/Siri-related hosts before the later Apple providers.

## Current External Dependencies

The complete row-by-row registry, URL, policy, position, counts, file SHA256, commit metadata, companions and migration rating is in [SOURCES.md](SOURCES.md). Summary:

| Source | References | Policies | Format |
|---|---:|---|---|
| blackmatrix7/ios_rule_script | 43 | DIRECT, PROXY, Max | Shadowrocket RULE-SET paths |
| iab0x00/ProxyRules | 1 | PROXY | Custom file, common rule syntax; native platform declaration not found |
| torr9522/shadowrocketzh | 3 | DIRECT, PROXY | Locally maintained Shadowrocket RULE-SET |

Migration rating counts: `KEEP_UPSTREAM` 39; `MIRROR_AND_TRACK` 0; `CONVERT_AND_MAINTAIN` 2; `MERGE_INTO_CUSTOM` 1; `REMOVE_OR_REDUNDANT` 3; `MANUAL_REVIEW` 2.

## Compatibility Findings

### Positive findings

- No `DOMAIN-SET` is currently referenced from `shadowrocketzh.conf`.
- No QuantumultX path is currently referenced.
- The blackmatrix files use supported rule lines such as `DOMAIN`, `DOMAIN-SUFFIX`, `DOMAIN-KEYWORD`, `USER-AGENT`, `IP-CIDR`, `IP-CIDR6`/IPv6 CIDR, and `IP-ASN`.
- No downloaded current provider included a trailing policy field such as `,DIRECT` or `,PROXY`.
- The local Apple provider has already converted the upstream bare-domain set into Shadowrocket rule syntax.

### Compatibility risks

- `Apple_Domain.list` is a bare-domain set upstream and must not be referenced as `RULE-SET` without conversion. The local `apple-direct.list` is the existing conversion.
- `iab0x00/ProxyRules/Rule/AI.txt` has no Shadowrocket-specific path or generated format declaration. Its individual lines are syntactically compatible with the current parser, but platform semantics remain `MANUAL_REVIEW`.
- `Amazon.list` contains a `URL-REGEX` rule and its upstream header explicitly recommends MITM. This is not a syntax failure, but it is a behavior dependency that should be tested separately.
- Companion `_Domain.list` files are bare-domain sets and companion `_Resolve.list` files are rule-form variants. The current config uses only the main rule-form files. Whether a companion improves matching depends on Shadowrocket DNS mode and must be tested, not assumed.
- For split providers, the header `TOTAL` is authoritative only when the main file and its documented companion are considered together. The companion should be audited before claiming full domain coverage.

## Overlap Findings

The overlap calculation only counts `DOMAIN` and `DOMAIN-SUFFIX` semantics. It treats `DOMAIN-SUFFIX,x` as covering `DOMAIN,y.x` and `DOMAIN-SUFFIX,y.x`; it does not infer coverage from `DOMAIN-KEYWORD`, USER-AGENT, IP, ASN or URL-REGEX. Those categories are reported separately and are not included in the percentages.

### Highest measured overlap

| A | B | A domain rules covered by B | A domain rules | Coverage |
|---|---|---:|---:|---:|
| Nintendo | Game | 125 | 125 | 100.0% |
| Epic | Game | 15 | 15 | 100.0% |
| Steam | Game | 51 | 51 | 100.0% |
| SteamCN | Game | 17 | 17 | 100.0% |
| BiliBili | ChinaMedia | 115 | 115 | 100.0% |
| TencentVideo | ChinaMedia | 19 | 19 | 100.0% |
| NetEaseMusic | NetEase | 10 | 10 | 100.0% |
| Apple proxy | Apple direct | 3 | 3 | 100.0% exact-domain subset; proxy is earlier by design |
| DouYin | ByteDance | 12 | 13 | 92.3% |
| SteamCN | Steam | 12 | 17 | 70.6% |

These are not automatic deletion recommendations. For example, `Game` may be broader than a platform-specific provider, and `ChinaMedia` contains IP/USER-AGENT categories not represented by the domain-only percentage.

### Domestic overlap observations

- `BiliBili` is fully covered at the measured domain level by `ChinaMedia`; its separate provider may still carry IP and USER-AGENT rules.
- `TencentVideo` is fully covered at the measured domain level by `ChinaMedia`, but it also has 32 IP rules.
- `NetEaseMusic` is fully covered at the measured domain level by `NetEase`, while its 18 IP and 2 USER-AGENT rules are not represented by that result.
- `DouYin` is mostly covered by `ByteDance`; keeping both is explainable only if the smaller set is retained as an explicit app exception and tested for IP/host differences.
- `WeChat`/`Tencent` are not equivalent: the measured intersection is small and the rule types differ.

### International overlap observations

- `Nintendo`, `Epic`, `SteamCN` and `Steam` are subsets of `Game` at the measured domain level. Future deletion requires testing because `Game` is a broad aggregate and order can encode intent.
- `AI` overlaps `Google`, `Microsoft`, `GitHub`, `Twitter` and the local Apple direct provider. These overlaps are expected for service infrastructure but can alter policy because AI is earlier.
- `YouTube` overlaps `Google`; `Netflix` and `HBO` overlap a small number of Amazon/Microsoft/Disney entries.
- `Global` was compared by domain suffix semantics against the other downloaded domain rules. No direct Apple-China domain conflict was found by this limited domain-only calculation, but its keyword, IP and USER-AGENT rules were not treated as suffix coverage.

## Apple Findings

The local providers are:

```ini
RULE-SET,https://raw.githubusercontent.com/torr9522/shadowrocketzh/main/rules/apple-proxy.list,PROXY
RULE-SET,https://raw.githubusercontent.com/torr9522/shadowrocketzh/main/rules/apple-direct.list,DIRECT
```

The proxy provider contains exactly three Private Relay host rules. The direct provider contains 1,603 converted rules: 9 DOMAIN, 1,551 DOMAIN-SUFFIX, 7 DOMAIN-KEYWORD, 13 IP-CIDR and 23 USER-AGENT.

The current `AI` file contains these Apple-related entries:

| Entry | AI section | Local Apple direct relation | Assessment |
|---|---|---|---|
| `smoot.apple.com` | Apple Intelligence | Covered by local Apple direct suffix rules | Policy conflict/grey area; AI PROXY wins because AI is earlier |
| `apple-relay.apple.com` | Apple Intelligence | Covered by local Apple direct rules | Policy conflict/grey area; AI PROXY wins |
| `apple-relay.cloudflare.com` | Apple Intelligence | Not in local Apple providers | AI PROXY wins |
| `apple-relay.fastly-edge.com` | Apple Intelligence | Not in local Apple providers | AI PROXY wins |
| `cp4.cloudflare.com` | Apple Intelligence | Not in local Apple providers | AI PROXY wins |
| `gspe1-ssl.ls.apple.com` | Apple Intelligence | Covered by local Apple direct rules | Policy conflict/grey area; AI PROXY wins |
| `guzzoni.apple.com` | Apple Intelligence | Covered by local Apple direct suffix rules | Siri/Apple Intelligence boundary requires manual product testing; AI PROXY wins |

The evidence establishes ordering and overlap, not that every Apple Intelligence host should be DIRECT or PROXY. Do not resolve this by broadening `apple.com` or by deleting AI entries without a real device test.

Recommended future review: split Apple Intelligence from the broad AI provider or add a separately documented policy layer. This is a design review, not a change made in this audit.

## China / Global Findings

### Current files

- `Global.list`: 198 parsed lines in the current download, plus 34,852 entries in `Global_Domain.list`; the combined header total is 35,050. It also has `Global_Resolve.list`.
- `China.list`: 63 parsed lines in the current download, plus 3,689 entries in `China_Domain.list`; the combined header total is 3,752. It also has `China_Resolve.list`.
- `China_Domain.list` is a bare-domain set; `China_Resolve.list` is rule syntax. The same distinction exists for Global.
- The current configuration uses the main Shadowrocket `RULE-SET` files only. It does not use `DOMAIN-SET` companions.
- Current historical paths `rule/Shadowrocket/China/ChinaMax.list` and `ChinaMaxNoIP.list` returned 404, so those names should not be added based on old documentation.

### Completeness conclusion

The current `China`/`Global` main references are syntactically appropriate Shadowrocket RULE-SET files, but they are not the complete domain coverage represented by the upstream header. `China_Domain.list` and `Global_Domain.list` are bare-domain sets and would require `DOMAIN-SET` if loaded directly; `_Resolve` is rule-form and may duplicate the main file. Whether to load the domain companions must be tested under the exact Shadowrocket client mode.

No custom migration is recommended yet. The higher-value next step is a controlled UAT comparison of the current main-only references against the main-plus-domain-companion design, without changing production first.

## Stale / Slow Rules

| Status | Count | Meaning |
|---|---:|---|
| ACTIVE | 3 | Game, Global, China had file commits in the last 90 days |
| SLOW | 4 | AI, Nintendo, Google, Lan had file commits 90-180 days ago |
| STALE | 37 | Other upstream files had no file commit in over 180 days |
| UNKNOWN | 3 | Local providers do not have upstream file-level freshness semantics |

The repositories themselves are active: blackmatrix7 HEAD was 2026-08-19 and iab0x00 HEAD was 2026-08-19/20 depending on branch metadata. A stale individual rule file is therefore not evidence that its repository is abandoned.

## Recommended Future Architecture

1. Keep the current first-match order while collecting real Shadowrocket logs.
2. Add a machine-readable audit script outside the configuration workflow to re-download each URL, compare file SHA256, query file-level commits and detect header/actual count drift.
3. Treat `apple-proxy` before `apple-direct` as intentional precedence.
4. Keep China and Global upstream until their generated count mismatch is explained.
5. Review the AI Apple entries as a separate policy decision before changing any Apple rule.
6. Consider consolidating only after UAT confirms that a broad aggregate provider preserves IP/USER-AGENT behavior.

## Migration Priority

### P0

- Confirm the split-provider generation model and whether the current main-only references intentionally omit domain companions for Tencent, Alibaba, Global, China and other affected providers.
- Confirm the actual downloaded branch/ref and use the combined main-plus-companion counts when comparing source updates.
- Validate `AI.txt` platform semantics before relying on it for Apple Intelligence routing.

### P1

- Test Apple Intelligence/Siri hosts from the AI list against real Shadowrocket logs.
- Compare `China.list`/`Global.list` with their `_Domain` and `_Resolve` companions under the exact Shadowrocket client mode.
- Re-evaluate `BiliBili`, `TencentVideo`, `NetEaseMusic`, `DouYin`, Nintendo, Epic, SteamCN and Steam because their measured domain overlap is high.

### P2

- Decide whether to keep small app-specific providers when a platform aggregate covers their domain rules.
- Review Amazon's URL-REGEX + MITM dependency.
- Add source tracking for local providers with source commit SHAs, not only local commit hashes.

### P3

- Leave active native Shadowrocket files such as Game, Global and China upstream until a tested custom requirement exists.
- Do not mirror stable native files merely to reduce the number of URLs.

## Rules Not Recommended for Migration

- Do not convert or mirror all 43 blackmatrix native Shadowrocket files without a concrete compatibility, priority, pinning or merge requirement.
- Do not replace China/Global with broad custom copies while their generated count discrepancy is unresolved.
- Do not merge IP-heavy providers into a domain-only custom list; that would change semantics.
- Do not remove Nintendo, Epic, SteamCN, Steam or app-specific rules solely from domain overlap percentages.
- Do not treat the iab0x00 AI file as a proven native Shadowrocket provider without a format/source review.

## Proposed Provider Layout

This is a future design sketch only. No files were created or changed by this audit:

```text
rules/
  apple-direct.list
  apple-proxy.list
  ch-extra-direct.lis
  SOURCES.md
  AUDIT.md
```

No additional provider is recommended until the P0/P1 items are resolved.

## Evidence URLs

- Current test configuration: https://raw.githubusercontent.com/torr9522/shadowrocketzh/main/shadowrocketzh.conf
- blackmatrix7 repository: https://github.com/blackmatrix7/ios_rule_script
- iab0x00 repository: https://github.com/iab0x00/ProxyRules
- blackmatrix7 Shadowrocket China: https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Shadowrocket/China/China.list
- blackmatrix7 Shadowrocket China Domain: https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Shadowrocket/China/China_Domain.list
- blackmatrix7 Shadowrocket Global: https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Shadowrocket/Global/Global.list
- blackmatrix7 Shadowrocket Global Domain: https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Shadowrocket/Global/Global_Domain.list
- iab0x00 AI: https://raw.githubusercontent.com/iab0x00/ProxyRules/main/Rule/AI.txt
- Apple support entry point: https://support.apple.com/
