# 探测负载与性能

clash-rule 的自动组（`url-test` / `fallback`）会周期性探测其成员节点。本页记录负载口径、现行频率分布、已知热区，以及待决策项。

## 口径

节点级探测负载 = Σ(自动组成员数 × 600s / interval)，单位「次/10min」。同一节点被多个自动组引用时会重复计入。

一个模型不确定点：mihomo 遇到「自动组引用自动组」时，是重探全部下层成员还是只探子组当前节点，两种实现下总负载可相差约 2×。因此跨文件对比应在**同一口径**（按正则直接成员计数）下进行。

文件组数（现行）：

| 文件 | 组总数 | 自动组 |
|---|---|---|
| `clash-rule.ini` / `clash-rule-black.ini` | 89 | 29 |
| `clash-rule-general.ini` | 81 | 30 |
| `clash-rule-manual.ini` / `-black-manual` / `-manual-test` | 80 | 15 |
| `clash-rule-gocn.ini` | 32 | 8 |

## 现行 interval 分布

| 组类别 | interval | 说明 |
|---|---|---|
| 普通地区`灾备` | 600 | 故障感知延迟较均衡 |
| `💖 高质灾备` | 600 | 高频切换成本高，宜慢 |
| `🔅 大流量灾备` | 900 | 成员多（商业大流量），降频收益大 |
| 过境`<地区>过境北场` / `<地区>过境` | 300 | 自建贵重，需较快感知 |
| `<地区>过境低延`（url-test）| 60 / 100 / 150 | 按地区，取最快 |
| `🏘 回家专用*` | 15 / 30 | 家内穿透，刻意高频 |
| `🌭 WHS*` | 15 / 30 | 设备穿透，刻意高频 |
| `general` 各国`低延`（url-test）| 100 | general 变体特有 |

## 热区（刻意保留）

- **回家 / WHS 穿透组**：15–30s，成员少但频率高；用户明确要求不改。
- **过境北场 / 过境**：300s 的自建优先段，是过境链的关键路径。
- **`general` 各国低延**：100s × 6 地区，是 general 变体负载的主体。

评估负载时以现役订阅的节点集为准：订阅会随机场更新变动，任一时刻的具体数值需按当时的节点集重新计算。

## 待决策项

### base/ClashBaseRule.yml

| 项 | 现状 | 建议 | 风险 |
|---|---|---|---|
| tcp-concurrent | false | true | 无 |
| tun.stack | gvisor | 高吞吐设备可换 system | 需按设备实测 |
| keep-alive-interval / idle | 15 / 180 | 30 / 300 | 无 |
| prefer-h3 | true | false（DoH3 兼容性参差）| 无 |
| sniffer QUIC 端口 | 443,8443,853,465,587,993,995 | 去掉 853/465/587/993/995 | 无 |
| hosts mtalk×8 | 固定 FCM IP | 复核时效 | FCM IP 轮换会失效 |
| find-process-mode | always | strict | 无 |
| log-level | debug | info | 无 |

### 仓库 / CI 侧

- `clash-classic/` 中的 `*.MANUAL.yaml` 是 list→yaml 转换的中间产物，无 ruleset 引用。可在 `convertlist2yaml.yaml` 中跳过 MANUAL 输出以减小仓库体积。
- 生成的配置中 `AND(DST-PORT/SRC-PORT)` 规则存在端口风暴（131 域名 × 13 端口展开成大量 AND 规则），可在 `rule-update.yaml` 生成逻辑中折叠为「子规则内嵌 OR」。
