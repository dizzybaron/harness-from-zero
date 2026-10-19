> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。

# AI Playground 專案法律（索引與路由）

本檔為 `<PROJECT_ROOT>/` 的法律層，承接 `~/.claude/CLAUDE.md`（憲法）。
**本檔只當索引**：規則本文在下方 @import 的常載核心與 `governance/` 各檔。
非 Claude 的 LLM agent 入口為專案根 `AGENTS.md`（引入協定 `governance/onboarding.md`）；tier 映射見 `governance/model-registry.md`。

@governance/core-rules.md

## 路由表：要做什麼 → 先讀什麼

| 情境 | 先讀 |
|---|---|
| spawn Agent／交辦任務 | 上方 core-rules §1 速查卡；範本 `governance/delegation-templates.md`；細則 `governance/model-dispatch.md` |
| 引入其他 LLM agent／查 tier↔型號映射 | `governance/onboarding.md`；`governance/model-registry.md` |
| 判斷升級／完成／該不該問人／換路 | `governance/judgment-rubrics.md` |
| 入庫（PDF/DOCX/圖片）、雲端本地分工 | `governance/refs/ops-pipelines.md` |
| 選 skill／查 skill model 釘定 | `governance/refs/skills-map.md`；vault 四操作見 `vault_neo/_Operations.md` |
| 改制度檔、寫教訓、精簡、季度稽核 | `governance/maintenance.md` |
| vault 具體操作（閉迴路、catalog、回填） | `vault_neo/CLAUDE.md`（規約層），再到各 domain `*_Evidence_Standards.md` |
| 踩到工具報錯／sandbox 限制 | 先 grep memory 目錄的 MEMORY.md，多半踩過 |

## 知識 compounding 三原則

vault 是 compounding artifact。細節在各 skill 與 `vault_neo/CLAUDE.md`，這裡只立原則：

1. **每次 ingest 必觸發跨頁更新**：新 source 不只新增 deep note，還更新涉及的 entity／concept／比較 page
   （主入口 `/ingest` 內嵌 reconcile；靜默場景寫 `_PendingReconcile` 事後 `/reconcile --pending`）。
2. **每次 query 結晶必回填**：有保存價值的合成走 `/query` 回填成 page；通用主題強制 `/entity <topic>`。
3. **每篇 deep note 在 catalog 留條目**：`vault_neo/<collection>/_PageIndex.md` 是 `/query` 主檢索面。
   全量重建 `python3 scripts/build_page_index.py`；增量 `--append <new-note>`。
   新書入庫需手動加 DEEP_NOTE_COLLECTIONS（memory: `project_buildpageindex_hardcoded_book_targets`）。

MD frontmatter 必含 `type: wiki` 或 `type: note`（分類定義見 `governance/refs/ops-pipelines.md`）。

## 會話生命週期

- **開始**：讀 `NEXT_SESSION.md`（上限 80 行；含「## 停車場」）。
- **結束**：呼叫 `/session-handoff`（dialog writeback → 更新 NEXT_SESSION.md → 工作日誌（`journal/`）→ git commit+push）。
  系統改動（腳本／設定／流程）一律入日誌，不需使用者提醒。
- Git auto-commit 白名單與破壞性操作規則見 core-rules §5；不入 git 清單見 `governance/refs/ops-pipelines.md`。

## 重要路徑

| 用途 | 路徑 |
|---|---|
| 知識庫 | `vault_neo/` |
| 制度檔 | `governance/`（本檔的規則本文所在） |
| 腳本 | `scripts/` |
| 新檔投遞 | `inbox/` |
| OCR／爬取 MD | `workspace/md/` |
| 原始檔封存 | `workspace/originals/` |
| marker 工具 | `~/marker-m2/bin/marker` |
| Skill 指令 | `.claude/commands/`（repo 內，跨 agent 共享） |
| 工作日誌（各 agent 共寫，入 git） | `journal/`（索引 `journal/journal-index.md`） |
| Claude Code memory（私有，其他 agent 不寫） | `<MEMORY_DIR>/` |

## governance/ 檔案索引

| 檔案 | 內容 | 讀取時機 |
|---|---|---|
| `core-rules.md` | 常載核心（已 @import 在上方） | 常載 |
| `diagnosis.md` | 本 harness 三大耗損模式診斷 | 想知道規則為什麼存在時 |
| `model-dispatch.md` | 模型調度守則全文（tier 語彙／交辦／回報合約／升降級／驗證） | spawn 前不確定時 |
| `model-registry.md` | tier 定義＋各 provider 型號價格唯一映射面 | spawn 選型／季度稽核／引入新 provider |
| `onboarding.md` | 新 LLM agent 引入協定（讀取順序／能力自檢／conformance／協作紀律） | 引入非 Claude agent 時 |
| `judgment-rubrics.md` | 判斷力 rubric（升級／完成／問人／換路／品質底線） | 卡在判斷時 |
| `delegation-templates.md` | 五種任務交辦 prompt 範本 | 每次交辦，照抄填空 |
| `maintenance.md` | 維護協議（權限分級／教訓回寫／精簡／季度稽核） | 改制度檔或踩雷後 |
| `letter-to-future.md` | 給未來 session 的信（退化模式／低信心清單） | 新 session 想理解全局時 |
| `refs/` | 按需參照（ops-pipelines／skills-map）與 archive/ 備份 | 路由表指到時 |
