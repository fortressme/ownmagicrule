# clash-rule 项目说明

个人自用 Clash（mihomo）规则与节点分组配置仓库。通过 [subconverter](https://github.com/tindy2013/subconverter) 把 `clash-rule*.ini`（分组/规则模板）与 `rules/`（规则集源）合成为可直接导入客户端的 Clash 配置。

规则来源：[Loyalsoldier/clash-rules](https://github.com/Loyalsoldier/clash-rules)、[ACL4SSR/ACL4SSR](https://github.com/ACL4SSR/ACL4SSR)、[blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script) 等，并按个人情况调整。

## 目录结构

| 路径 | 作用 |
|---|---|
| `clash-rule*.ini` | 7 个分组/规则模板（subconverter `[custom]` 格式），决定代理组拓扑与规则→组的映射 |
| `rules/*.list` | 规则集源（text 格式）。`*.MANUAL.list` 为手动维护项，`*.SERVER.list` 由 CI 生成 |
| `clash-classic/**/*.yaml` | 由 `rules/*.list` 转换出的 Clash classic ruleset，经 raw GitHub 被 ini 的 `ruleset=` 引用 |
| `base/` | 客户端基线配置（`ClashBaseRule.yml`、`ClashBaseRuleGoCN.yml`、`SingboxBaseRule.json`）|
| `.github/workflows/` | CI：规则更新与 list→yaml 转换 |
| `docs/` | 本文档 |

## ini 变体

| 文件 | 定位 |
|---|---|
| `clash-rule.ini` | 主线：白名单模式，CN 分流，漏网之鱼默认走代理，含自建落地 |
| `clash-rule-black.ini` | 黑名单模式，漏网之鱼默认直连 |
| `clash-rule-general.ini` | 公共服务版：无自建落地、无特定服务分流，弱化自建 |
| `clash-rule-manual.ini` | 手动为主，现网 Sparkle 使用中 |
| `clash-rule-black-manual.ini` | 黑名单 + 手动为主 |
| `clash-rule-manual-test.ini` | 手动变体的测试用（`!!empty-fallback` 等）|
| `clash-rule-gocn.ini` | 回国 / 海外访问内网场景 |

变体之间的差异限于 `ruleset`、`🐟 漏网之鱼` 顺序、`CQGAS` 指令与自建相关正则；代理组命名与拓扑保持一致。

## 数据流

```
rules/*.list ──(convertlist2yaml.yaml)──► clash-classic/**/*.yaml ─┐
                                                                   ├─► subconverter ─► 客户端配置
clash-rule*.ini (ruleset= 引用 + custom_proxy_group 定义) ─────────┘
```

- `rule-update.yaml`：每 6 小时（及 push/PR）由 `rules/*.update.list` 刷新 `*.SERVER.list` 并提交。
- `convertlist2yaml.yaml`：在 rule-update 完成后触发，将 `rules/` 转为 `clash-classic/` 的 YAML 并提交。

## 生成订阅

订阅转换站点（如 id9.cc、suburl.v1.mk、acl4ssr-sub.github.io）：用生成的 config 地址替换转换 URL 中的 `config=` 参数。通用 config：

```
https%3A%2F%2Fraw.githubusercontent.com%2Ffortressme%2Fownmagicrule%2Fmain%2Fclash-rule-general.ini
```

示例：

```
https://sub.id9.cc/sub?target=clash&new_name=true&url=<订阅地址>&insert=false&config=<config 地址>&filename=TEST&emoji=true&list=false&tfo=false&scv=false&fdn=false&sort=false
```

## 仓库维护

自动更新提交频繁且每次重写大规则文件，历史会持续膨胀。当仓库超过 ~50 MiB 时，按 README 的"定期压缩 Git 历史"步骤压成单个提交（orphan 分支 + 强推）。

## 相关文档

- [topology.md](topology.md) —— 代理组拓扑：节点角色词典、质量阶梯、探测频率
- [performance.md](performance.md) —— 探测负载分析、热区、待优化项
