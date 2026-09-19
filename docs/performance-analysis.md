# 性能分析（标注制：Measured=本地实测 / Estimated=模型推算 / Inference=架构推断）

## 探测负载口径

节点级探测负载 = Σ(自动组成员数 × 600s/interval)，单位"次/10min"。
两次模型误差来源（Inference）：mihomo 对"自动组引用自动组"是重探全部下层成员还是只探子组当前节点，
两种实现下总负载相差可达 2×；本报告所有对比都在**同一口径**（直接正则成员计数）下进行，结论不受影响。

## 前后对比（同一节点集 = 现役订阅 172 节点，Measured）

| 文件 | 组数 | 自动组 | 探测负载/10min | 被探测节点 | 峰值/节点 |
|---|---|---|---|---|---|
| clash-rule.ini | 90 | 30 | 915 → **732** (−20%) | 162 → 143 | 21 → 21 |
| clash-rule-black.ini | 90 | 30 | 915 → **732** (−20%) | 162 → 143 | 21 → 21 |
| clash-rule-general.ini | 82 | 31 | 592 → **397** (−33%) | 128 → 126 | 7 → 5 |
| clash-rule-manual.ini（现网） | 81 | 16 | 503 → **491** (−2%) | 140 → 140 | 21 → 21 |
| clash-rule-black-manual.ini | 81 | 16 | 503 → **491** (−2%) | 140 → 140 | 21 → 21 |
| clash-rule-manual-test.ini | 81 | 16 | 503 → **491** (−2%) | 140 → 140 | 21 → 21 |
| clash-rule-gocn.ini | 32 | 8 | 349 → **349** (0%) | 34 → 34 | 24 → 24 |

注意：会话早前审计（219 节点订阅）测得 main=1215/10min；订阅在会话期间被机场更新（北场D 9→4、N转直 29→11、LF 111→86），
为公平对比上表 before/after 均按现 172 节点集重算。历史 1215 值仅作参考（Measured，节点集不同）。

## 收益来源分解（main.ini，Estimated）

| 来源 | 节省/10min | 说明 |
|---|---|---|
| 灾备 300→600（6 组） | ~115 | 普通/高质灾备成员减半频率 |
| 大流量灾备 300→900 | ~53 | 89→64 节点 + 频率 1/3 |
| 灾备/过境低延去自建、去 XX-L | ~45 | N转直×11、北场D×4、北场L×21 退出多个商业池 |
| 合计 | ~213 | 与实测差值 183 吻合（成员集重叠抵消） |

## 未动的热区（刻意保留，Inference）

- 回家专用灾备 30s（9 节点，180/10min）+ WHS 灾备 30s（6 节点，120/10min）：价值型高频探测，用户确认保留。
- 低延简选 600s（115 节点，115/10min）：主要代理默认路径，功能核心。
- 峰值/节点 21（回家 GOCN）= 30s fallback + 600s 低延双层，用户确认"回家和 WHS 相关不要改"。
- gocn 0%：其负载主体是 回家 GOCN 池（刻意），本次只修悬空引用。

## 非 client 端收益（仓库/CI 侧，Measured）

- 43 个孤儿 clash-classic yaml（13.1MB）不再被任何 ruleset 引用，其中 *.MANUAL 为生成中间产物；
  本次未删除（只读范围外），建议后续在 convertlist2yaml 中跳过 MANUAL 输出。
- 规则风暴：生成配置内 AND(DST-PORT/SRC-PORT) 规则 1767 条 = 131 域名×13 端口，可折叠为 131 条
  （子规则内嵌 OR），属 rule-update.yaml 生成逻辑改动，本阶段未动（Estimated −90% AND 规则量）。

## base/ClashBaseRule.yml 建议清单（本阶段未动，均为建议项）

| 项 | 现状 | 建议 | 风险 |
|---|---|---|---|
| tcp-concurrent | false | true（现网已被客户端改 true） | 无 |
| tun.stack | gvisor（gso 只对 system 生效，当前空设） | 高吞吐设备可换 system | 需按设备实测 |
| keep-alive-interval/idle | 15/180 | 30/300 | 无 |
| prefer-h3 | true | false（DoH3 兼容性参差） | 无 |
| sniffer QUIC 端口 | 443,8443,853,465,587,993,995 | 去掉 853/465/587/993/995 | 无 |
| hosts mtalk×8 | 固定 FCM IP | 复核时效 | FCM IP 轮换会失效 |
| find-process-mode | always（现网） | strict | 无 |
| log-level | debug（现网） | info | 无 |
