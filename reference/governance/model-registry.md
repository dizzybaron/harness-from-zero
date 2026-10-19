> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。

# 模型層級註冊表（Model Registry）

> 讀者：任何 provider 的主對話模型與引入中的新 agent。tier（T1/T2/T3）是制度的通用語彙；
> 本檔是 tier ↔ 各 provider 具體型號與價格的**唯一映射面**（規則本文引用 tier，不引用型號）。
> 憑記憶填型號或價格＝違規（同 model-dispatch 頭則）；每格必附查證日。行數上限 80。

## 1) Tier 定義（provider-neutral）

| Tier | 名稱 | 承擔工作 | 引入新 provider 時的判定準則 |
|---|---|---|---|
| **T1** | 執行級 | 檢索、提取、格式檢核、套用既定 pattern 的機械批次 | 該 provider 最便宜的堪用模型；錯了成本低、可重派 |
| **T2** | 標準級 | 筆記撰寫、合成、需要判斷的批次、中等推理 | 主力平衡型；品質接近旗艦、單價明顯低於旗艦 |
| **T3** | 前沿級 | 主對話、架構決策、難 debug、跨 domain 合成、第二意見 | 該 provider 當代旗艦推理模型 |

- **主對話預設 T3**（2026-07-15 使用者裁決：本專案問題皆分析性，主對話不為省額度自降級）。
- tier 歸屬以本 repo 任務實測為準，不以行銷名稱；一個 provider 可以只有部分 tier，缺格不硬湊。

## 2) Provider 映射

### Anthropic — Claude Code harness（查證 2026-09-23，來源 claude-api skill＋`/model` 輸出）

| Tier | `model` 參數 | model ID | $/MTok in/out | 備註 |
|---|---|---|---|---|
| T1 | `"haiku"` | claude-haiku-4-5 | $1 / $5 | 不支援 effort；context 200K，其餘皆 1M |
| T2 | `"sonnet"` | claude-sonnet-5 | 未確認（2026-07-04 牌價 $3/$15；促銷 $2/$10 已於 2026-08-31 到期） | 官方定位 near-Opus |
| T3 | `"opus"` | claude-opus-5-5 | $4 / $20 | 主對話預設；API effort 預設 medium；thinking 恆開不可關；fast mode 可用 |
| T3 | `"fable"` | claude-fable-5-1 | $10 / $50 | 單價高於 opus；使用者指定才用 |

### xAI — Grok Build（查證 2026-10-15，https://docs.x.ai/developers/pricing）

| Tier | model ID | $/MTok in/out（<200k） | 備註 |
|---|---|---|---|
| T1 | （未引入） | — | spawn 無 model 參數；較低價型號未做本 repo 冒煙，不預填 |
| T2 | （未引入） | — | 同上 |
| T3 | `grok-4.7` | $2 / $6 | 官方標 flagship。≥200k：$4 / $12。Fast（僅 Cursor／Grok Build）為 2×，本 session 是否走 Fast 未確認 |

### 其他 provider（尚未引入；引入時照 §3 填寫，禁止憑記憶預填）

| Tier | OpenAI | 開源（自架／託管） |
|---|---|---|
| T1 | （未引入） | （未引入） |
| T2 | （未引入） | （未引入） |
| T3 | （未引入） | （未引入） |

## 3) 新 provider 填表程序

1. 查該 provider **當日官方** pricing／model docs，選定 T1–T3 候選型號與單價。
2. 用本 repo 真實任務各跑一件冒煙（T1：一次檢索定位；T2：一篇 deepnote 級寫作；T3：一則判斷題），
   並通過 `governance/onboarding.md` §3 conformance 驗收。
3. 填入 §2 表：model ID＋單價＋查證日；工作日誌記一行（黃區程序）。
4. 查證日超過一季的格 → 季度稽核時重查證或標 `stale`（見 `governance/maintenance.md` §4）。

## 教訓紀錄（append-only，一行一條）

- 2026-07-15 立檔：自 model-dispatch §0 抽出型號價格表並抽象為 tier 語彙。
- 2026-09-23 prompt-audit：主對話改 Opus 5.5；Agent tool 已接受 `fable`（Fable 5.1）；Sonnet 促銷到期，T2 現價待查證。
- 2026-10-15 grok 首次登記 T3＝`grok-4.7`（牌價當日查 docs.x.ai：$2/$6，≥200k $4/$12）。T1/T2 留空：Grok Build 的 spawn 不能釘型號，未冒煙不預填。
