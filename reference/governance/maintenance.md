> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。

# 維護協議

> 讀者：日常 session 的主對話模型。定義誰能改什麼、教訓寫到哪、什麼時候精簡、多久稽核一次。

## 1) 檔案權限分級

| 區 | 規則 | 涵蓋 |
|---|---|---|
| **綠**：直接改，不需報備 | 正常工作的一部分 | memory 目錄（含 MEMORY.md）、`journal/`（工作日誌）、`NEXT_SESSION.md`、`vault_neo/` 內容與 `_PageIndex`、`_Insights/`、`workspace/`、governance 各檔的「教訓紀錄」append-only 段 |
| **黃**：可改，但改後在工作日誌記一行（改了什麼＋為什麼） | 影響未來 session 行為 | `governance/refs/*`、`scripts/`、skill 本體（`.claude/commands/*.md`）、`governance/delegation-templates.md` 的範本措辭、`governance/diagnosis.md`、`governance/letter-to-future.md`、`governance/model-registry.md` 的映射表查證值、`governance/onboarding.md` |
| **紅**：**先問使用者**，批准後先備份再改 | 制度骨架 | 兩層 `CLAUDE.md` 本體、`AGENTS.md`、`governance/core-rules.md`、`model-dispatch.md`／`judgment-rubrics.md`／`maintenance.md` 的**規則本文**（教訓段除外）、`model-registry.md` 的 tier 定義、`.claude/settings*.json`、launchd／cron |

紅區備份規範：改前 `cp <檔> governance/refs/archive/<檔名>.<YYYY-MM-DD>.bak.md`，再動手。

## 2) 教訓回寫（踩雷後怎麼記）

1. **一律**照既有 memory 機制寫一個 memory 檔（一檔一事實，frontmatter＋Why＋How to apply），
   並在 MEMORY.md 加一行索引。這是詳細版。
2. **若教訓改變「未來該怎麼調度／驗證」**（不只是單一工具的坑），另外在對應 governance 檔尾的
   「教訓紀錄（append-only）」加**一行**：`- YYYY-MM-DD 一句話結論（memory: 條目名）`。
   一行是索引不是內文；細節留在 memory 檔。
3. 教訓與現行規則**矛盾**時：教訓先記下，規則本文是紅區——提請使用者裁決後才改規則。

## 3) 行數預算與精簡觸發

| 檔 | 上限 | 超限處理 |
|---|---|---|
| `~/.claude/CLAUDE.md` | 80 行 | 內容移往專案 governance/，全域只留通則 |
| 專案 `CLAUDE.md` | 150 行 | 長內容抽到 `governance/refs/` 新檔，本體只留索引行 |
| `governance/core-rules.md` | 100 行 | 降頻內容移回 model-dispatch／judgment-rubrics 本文 |
| `MEMORY.md` | 70 行 | 見下方瘦身程序 |
| `NEXT_SESSION.md` | 80 行 | 舊內容整檔移 `governance/refs/archive/NEXT_SESSION.<日期>.bak.md`，重寫摘要版 |
| `AGENTS.md` | 60 行 | 只留憲法核心＋路由；細節移 `governance/onboarding.md` |
| `governance/model-registry.md` | 80 行 | 停用 provider 列移 archive |
| `governance/onboarding.md` | 100 行 | 細節抽到 `governance/refs/` 新檔 |
| 各 governance 檔「教訓紀錄」 | 15 條 | 合併同類、過時者刪（先確認對應 memory 檔仍在） |

行數上限同時換算為字元上限（行數上限 × 100 字元）：用長行把內容塞進同樣行數，等同超限。

**重寫後必查下游 writer**（2026-07-19 教訓）：受管文件常被腳本自動寫入，而錨點多半釘在標題、表格、
`---` 等結構元素上；歸檔重寫會靜默打斷它們——不報錯、exit 0，只有產物壞掉。跑完上表任一「超限處理」後，
先 `grep -rln "<檔名>" scripts/ .claude/` 清查所有 writer，逐一確認錨點仍命中。
（memory: `project_doc_rewrite_breaks_downstream_writers`）

**MEMORY.md 瘦身程序**（機械，haiku/sonnet 可執行）：
1. 備份 MEMORY.md 到 archive（同 §1 格式）。
2. 跑 `python3 scripts/build_journal_index.py` 重建 `journal/journal-index.md`（repo 內，type: reference，一檔一行）。
   腳本會保留既有 frontmatter，冪等可重跑。2026-10-15 起日誌在 repo `journal/`，不在 memory 目錄。
3. MEMORY.md 刪除**所有**「工作日誌」條目行，保留一行指標，以絕對路徑指向 repo 的 `journal/journal-index.md`。
4. 非日誌條目超限時：同一管線／skill 的叢集收進 `*-index.md` 二級索引（舊索引行原文照搬）；governance 以「grep MEMORY.md」引用的條目保留直接索引；其餘改寫為「情境→規則」短句，壓縮前先確認索引行的事實已在本體。

> 2026-07-19 修訂：原訂「只搬 60 天前的日誌」達不到 ≤70 目標——非日誌活規則本身就佔 63 行，
> 預算只剩 5 行。經使用者裁決改為日誌全搬，MEMORY.md 只留活規則。但這也意味著**非日誌條目缺退休機制**，
> 每次踩雷 +1 行，遲早再超標；下次超限時該稽核的是非日誌條目，不是再砍日誌。

## 4) 季度稽核（每季一次；或模型換代新聞時立即）

由使用者開一個稽核 session（或掛排程），依序：
1. **模型現況**：invoke `Skill "claude-api"` → 對照 `governance/model-registry.md` §2 映射表（Anthropic 列）
   → 有出入即更新＋改查證日；非 Anthropic 列查證日 >1 季 → 重查證或標 `stale`。
   已知到期日（促銷等）只記在 registry 對應列的備註，不在本檔另記。
2. **行數體檢**：`wc -lm` 本檔 §3 表列各檔（行數或字元數任一超限）→ 跑對應精簡程序。
3. **路徑健檢**：派 haiku agent 驗 governance 各檔提到的路徑／skill 名／工具名是否仍存在，
   報 findings（用 delegation-templates T5）。
4. **門檻校準**：若 weekly token report 已擴充到能量測調度遵從（見停車場），對照 dispatch §1 門檻數字調整。
5. **入口同步**：`AGENTS.md` 內嵌的憲法可移植核心與 `~/.claude/CLAUDE.md` 是否一致；
   歧異以憲法正本為準修 AGENTS.md。

## 5) Day-1 遷移任務（制度上線後第一批便宜 session 做）

按價值排序，每項獨立可做：
1. **T2 冒煙測試**：開一個 T2 session（手動 `/model` 降 sonnet），用 delegation-templates T1 實填範例真跑一次，
   驗證：速查卡在 T2 上可被遵守（spawn 帶了 model）、回報合約格式正確。結果記工作日誌。
2. **skills model 釘定稽核**：對 `governance/refs/skills-map.md` 的 15 個未釘定 skill，逐個判斷：
   純規則／Bash 協調 → 釘 haiku；合成寫作 → 釘 sonnet；有「故意不釘」註記者（deepnote、session-handoff）**跳過**。
   每釘一個，在 skill 內留一行「**Model**：xxx（理由）」。
3. **週報擴充**（可選）：`scripts/weekly_token_report.py` 加一項「本週 Agent 呼叫缺 model 參數次數」
   ——沒有量測面，調度遵從只能靠自律。
4. **usage 儀表板實測**（可選，僅未來還會用 Fable 時）：驗證安全導流的額度歸屬。

## 6) 本檔自身

本檔規則本文屬紅區。§5 的任務做完就把該行刪掉（黃區操作，日誌記一行）。
