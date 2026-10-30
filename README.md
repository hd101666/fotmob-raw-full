# FotMob 原始响应 完整版数据包 (Untruncated Raw Responses)

本仓库**独立于** `football-predictor`，仅存放 FotMob 原始响应的完整未截断版本，供 MUSE 复制核对。

## 文件

| 文件 | 大小 | 说明 |
|---|---|---|
| `fotmob_raw_responses_FULL.csv` | 14,656,358 字节（13.98 MiB） | 20 列，100 行，`raw_response` **未截断** |
| `README_FULL_PACKAGE.md` | 5,413 字节 | 完整说明：大小、行数、长度分布、截断规则、一致性结论 |

## 校验

```
SHA-256  19475f77eaaeaabdbc11c2010f4270ada312e285d1198807ad838b6a176dcd69
行数     100（不含表头）
列数     20
raw_response 长度：最大 1,389,457 / 最小 18,111 / 中位 39,133
```

## 与截断版的关系

仓库 `hd101666/football-predictor` 分支 `hit-system-export` 的
`hit-system/samples/fotmob_raw_responses.csv` 是**截断版**（118,136 字节）。

两版对应关系：
- 行序完全对应（同一 `LIMIT 100`，表默认插入顺序）
- 20 列元数据完全一致
- 除 `raw_response` 外 19 列逐单元格 **0 处差异**
- 截断版 `raw_response` = 本版前 600 字符 + `...[TRUNCATED N chars, full length M]`

核对脚本见 `README_FULL_PACKAGE.md` 第四节。

## 快速核对

```bash
git clone https://github.com/hd101666/fotmob-raw-full.git
cd fotmob-raw-full
python -c "
import csv,sys,hashlib,statistics
csv.field_size_limit(10**9)
p='fotmob_raw_responses_FULL.csv'
raw=open(p,'rb').read()
print('size   ', len(raw))
print('sha256 ', hashlib.sha256(raw).hexdigest())
rows=list(csv.DictReader(open(p,encoding='utf-8-sig')))
print('rows   ', len(rows), 'cols', len(rows[0]))
L=[len(r['raw_response']) for r in rows]
print('max/min/median', max(L), min(L), int(statistics.median(L)))
"
```

## 安全

- 无密钥、token、密码、cookie、`.env`
- 已扫描：真实凭据比对 0 命中；私钥/AWS/GitHub/Slack/Bearer 0 命中
- `observation_id`（SHA-1）与 `raw_sha256`（SHA-256）为标识符/内容哈希，非凭据
- 内容为 FotMob 公开赛事统计（球员、球队、比赛事件），不含私人信息

## 范围说明

数据库 `fotmob_raw_responses` 表当前共 2,844 行；本包为与截断版样本对应的**前 100 行**。
如需全表导出，请另行生成。
