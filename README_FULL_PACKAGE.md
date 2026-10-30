# FotMob 原始响应 完整版数据包 说明

生成时间：2026-10-30
数据源表：`sports_data.db` → `fotmob_raw_responses`
用途：提供 `raw_response` 列**未截断**的完整版，供核对截断版样本

---

## 一、包内文件

| 文件 | 说明 |
|---|---|
| `fotmob_raw_responses_FULL.csv` | 完整版原始响应，20 列，`raw_response` 不截断 |
| `README_FULL_PACKAGE.md` | 本说明 |

**本包不含**：任何密钥、token、密码、cookie、`.env` 文件。
**本包未推送** GitHub，未改动 `hit-system-export` 分支，未覆盖仓库内截断版样本。

---

## 二、文件大小与行数

| 指标 | 完整版（本包） | 截断版（仓库样本） |
|---|---|---|
| 文件大小 | **14,656,358 字节（13.98 MiB）** | 约 120 KB |
| 行数（不含表头） | **100** | 100 |
| 列数 | **20** | 20 |

完整版 SHA-256：`19475f77eaaeaabdbc11c2010f4270ada312e285d1198807ad838b6a176dcd69`

> 注：14.66 MB 是按 MB(10^6) 计的 14,656,358 字节；按 MiB 计为 13.98 MiB。与最初导出时的 14.66MB 一致。

---

## 三、raw_response 列长度分布（100 行）

| 统计量 | 字符数 |
|---|---|
| **最大** | **1,389,457** |
| **最小** | **18,111** |
| **中位数** | **39,133** |
| 均值 | 126,786 |
| 总和 | 12,678,605 |

分布特征：极度长尾。中位数仅 3.9 万字符，但最大值 138.9 万字符，单行即占全文约 9.5%。
这正是截断的必要性所在——少数超大 JSON 撑起了 21.6MB 的原始样本体积。

raw_response 长度最大的 10 行（降序）：
```
1389457, 1289128, 839816, 822747, 779403,
730074, 704370, 688666, 636311, 629027
```
仅这 10 行就占全部 raw_response 字符总量的 67.1%（8,508,999 / 12,678,605）。逐行完整长度见 CSV 的 raw_response 列。

---

## 四、截断版与完整版的对应关系

**截断规则（仓库样本使用的）**：

```
raw_response = 原值前 600 字符 + "...[TRUNCATED N chars, full length M]"
其中 N = M - 600，M = 原始 raw_response 长度
```

**对应关系**：
- 行序号完全对应：完整版第 i 行 ↔ 截断版第 i 行（均为无 ORDER BY 的 `LIMIT 100`，即表默认插入顺序）
- 列名与列顺序完全一致：20 列一一对应
- `raw_response` 截断版的值 = 完整版该列前 600 字符 + 标注后缀

**核对方法**（助手可用）：
```python
import csv, sys
csv.field_size_limit(10**9)
full  = list(csv.DictReader(open('fotmob_raw_responses_FULL.csv', encoding='utf-8-sig')))
trunc = list(csv.DictReader(open('fotmob_raw_responses.csv',      encoding='utf-8-sig')))

assert len(full) == len(trunc) == 100
for i, (f, t) in enumerate(zip(full, trunc)):
    orig = f['raw_response']
    # 校验：截断版以完整版前600字符开头，且标注的长度等于原长
    assert t['raw_response'].startswith(orig[:600]), i
    assert f'full length {len(orig)}' in t['raw_response'], i
print('100 行全部对应')
```

---

## 五、一致性核验结论

| 核验项 | 结果 |
|---|---|
| 20 列元数据是否与仓库样本一致 | ✅ **完全一致**（列名、列顺序、20 列全部相同） |
| 除 `raw_response` 外其他 19 列是否与仓库样本一致 | ✅ **完全一致**（100 行 × 19 列，逐单元格比对 0 处差异） |
| 行数是否一致 | ✅ 100 行 vs 100 行 |
| `raw_response` 前 600 字符是否与截断版吻合 | ✅ 100/100 行吻合 |
| 截断版标注的原长度是否等于本包实际长度 | ✅ 100/100 行吻合 |

**结论：除 `raw_response` 列被截断外，本包与仓库截断版样本完全一致，可放心对照。**

---

## 六、20 列清单（与仓库样本一一对应）

| # | 列名 |
|---|---|
| 1 | observation_id |
| 2 | provider |
| 3 | endpoint |
| 4 | source_match_id |
| 5 | source_league_id |
| 6 | request_url |
| 7 | request_started_at_utc |
| 8 | observed_at_utc |
| 9 | http_status |
| 10 | etag |
| 11 | last_modified |
| 12 | content_type |
| 13 | source_version |
| 14 | source_version_note |
| 15 | raw_sha256 |
| 16 | response_bytes |
| 17 | raw_response ← **本包为完整版，仓库为截断版** |
| 18 | first_seen_utc |
| 19 | last_seen_utc |
| 20 | observation_count |

---

## 七、安全声明

已对本包执行以下扫描，结果均为 0 命中：

- 真实凭据比对：`.env` 中全部 KEY/TOKEN/SECRET 值 + 3 个未纳管 Odds API key → **0 命中**
- 私钥 / AWS AKIA / GitHub token / Slack token / Bearer → **0 命中**
- `cookie` / `authorization` / `api_key` / `password` / `secret` / `bearer` 在 raw_response JSON 内 → **0 命中**
- `session` 关键词命中 10 行，经核实均为足球统计指标 `Possession won final 3rd` 等词中的 "session" 子串，**非会话凭据**

以下为非敏感标识符，已确认可安全保留：
- `observation_id`：40 位 SHA-1 观测标识（100 个）
- `raw_sha256`：64 位 SHA-256 内容哈希（100 个）
- `etag`：FotMob 弱 ETag，形如 `W/"t91ub22ga5191i"`

数据内容为 FotMob 公开赛事统计（球员姓名、球队、比赛事件等公开体育数据），不含私人信息。

---

## 八、注意

本包为**数据快照**，与数据库当前状态可能不同步——数据库该表现有 2,844 行，
本包取的是与仓库样本对应的**前 100 行**（`LIMIT 100`，表默认插入顺序，无排序）。
若需全表 2,844 行的完整导出，需另行生成。
