# 五層規則對照

作者的系統把給 AI 的規則分成五層，由抽象到具體，**下層覆蓋上層**。
每層只放該層的事；越常載入的檔越要短，因為每次對話都要付一次閱讀成本。

| 層 | 管什麼 | 本 repo 檔案 | 何時載入 |
|---|---|---|---|
| 憲法 | 跨所有專案的通則：語言、語調、推理流程、人機分工、安全 | `reference/CLAUDE.global.example.md` | 每次啟動自動載入 |
| 法律 | 單一專案的索引與路由；規則本文拆在 `governance/` | `reference/CLAUDE.project.example.md`、`reference/governance/` | 進專案自動載入；`core-rules.md` 以 `@import` 常載，其餘照路由表按需讀 |
| 規約 | 知識庫怎麼操作：閉迴路、catalog、回填 | `reference/vault_neo/CLAUDE.example.md`、`reference/vault_neo/_Operations.md` | 讀寫知識庫時 |
| 索引 | 知識庫地圖：有哪些領域、各放什麼 | `reference/vault_neo/_INDEX.md` | 查詢前先讀 |
| 命令 | 單一領域寫筆記的格式、命名、檢核 | `reference/vault_neo/02_EM/EM_Evidence_Standards.md` | 寫該領域的筆記時 |

命令層的 `EM_Evidence_Standards.md` 綁著急診與作者的資料夾結構；給一般醫療工作者的通用改寫版在 `templates/note-standards.md`。

旁掛（不屬於階層，但同樣會進 context）：

- `reference/AGENTS.example.md`：給非 Claude agent（GPT、Grok、開源模型）的入口，內嵌憲法核心。
- `reference/skills-example/`：操作程序。原系統還有十多個 skill，這裡只收 ingest／query／lint／reconcile 四個；文中提到的其他 skill（deepnote、reviewer、MDer 等）沒有附。
- `reference/web-project-default-logic.txt`：五層濃縮成單檔，給只用網頁版、沒有本機 agent 的情境。
- `reference/Harness_Architecture_2026.md`：整套系統的全圖，包括規則以外的工具、資料與自動化層。

## 檔名為什麼改過

Claude Code 與 Codex 會自動載入路徑上的 `CLAUDE.md`／`AGENTS.md`，`.claude/commands/` 也會被註冊成指令。
為了不讓你的 AI 把作者的規則當成自己的，下列檔案改了名或換了位置：

| 原路徑 | 本 repo 路徑 |
|---|---|
| `~/.claude/CLAUDE.md` | `reference/CLAUDE.global.example.md` |
| `<專案根>/CLAUDE.md` | `reference/CLAUDE.project.example.md` |
| `<專案根>/AGENTS.md` | `reference/AGENTS.example.md` |
| `vault_neo/CLAUDE.md` | `reference/vault_neo/CLAUDE.example.md` |
| `.claude/commands/*.md` | `reference/skills-example/*.md` |
| `governance/web-project-default-logic.txt` | `reference/web-project-default-logic.txt` |
| `vault_neo/01_Projects/02_AI_Sharing/Harness_Architecture_2026.md` | `reference/Harness_Architecture_2026.md` |

其餘檔案維持原相對路徑（以 `reference/` 為根）。文中寫 `governance/core-rules.md`，對應的就是 `reference/governance/core-rules.md`。

## 和 LLM Wiki 的對應

原文：[LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)（Andrej Karpathy；本 repo 不附原文）。

| LLM Wiki 的概念 | 作者系統的實作 |
|---|---|
| Raw sources：不可改的原始素材 | `workspace/md/` 與各子庫的 `02_*_md/`（OCR 或爬取的原文 MD） |
| The wiki：LLM 寫的頁面 | `vault_neo/` 的 deep note 與 entity page |
| The schema：CLAUDE.md／AGENTS.md | 本頁的五層規則 |
| Ingest | `/ingest`（內含 `/reconcile` 跨頁更新） |
| Query：好答案回填成新頁 | `/query`、`/entity` |
| Lint | `/lint` |
| index.md | 各子庫的 `_PageIndex.md`（catalog）＋ `_INDEX.md`（地圖） |
| log.md | `vault_neo/_EventLog.md` |

作者在 LLM Wiki 之外另外加的：

- **回溯原文**：數字、劑量、診斷標準必須回 raw 原文核對才寫（`governance/core-rules.md` §2）。
- **產出者 ≠ 驗證者**：寫筆記的 agent 和檢查的 agent 分開（`governance/model-dispatch.md` §6）。
- **多模型分工**：雜事交給便宜的模型，判斷留給大模型（`governance/model-dispatch.md`）。
- **writeback**：對話中零散的洞察融回既有筆記（`vault_neo/CLAUDE.example.md`）。

## 建議閱讀順序

1. [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)（Andrej Karpathy）
2. 本頁
3. `reference/governance/diagnosis.md`：規則為什麼存在
4. `reference/governance/core-rules.md`：每次對話都載入的核心
5. 其餘按需要讀

## 閱讀提示

- `governance/delegation-templates.md` 的 T1–T5 是範本編號，和模型層級 T1–T3（`governance/model-registry.md`）無關。
- 各檔是同一天的快照，但寫於不同時間，偶有數字或時刻前後不一（例如排程時間、skill 數量）；這是原稿各自的寫作時點不同，不影響規則本身。

## 沒有公開的部分

下列檔案在規則檔中被提到，但沒有收進這個 repo：

- memory 條目（`feedback_*`、`project_*`）、工作日誌（`journal_*`）、`NEXT_SESSION.md`
- `scripts/`、`logs/`、`governance/refs/archive/`（舊版備份）
- 知識庫筆記本體（唯一例外是架構說明 `Harness_Architecture_2026.md`），以及 `_PageIndex.md`、`_EventLog.md`
- `EM_EBM_Reference.md`、`03_EM_Blogs/README.md`，以及 Endurance／Philosophy／EBM 三個領域的 Evidence Standards
- `vault_neo/llm-wiki.md`（文中的 `[[llm-wiki]]`）：Karpathy〈LLM Wiki〉原文的轉存；請讀上方「和 LLM Wiki 的對應」連結的原始出處

原文偶有指向這些檔或已改版段落的參照（例如 `EM_Evidence_Standards.md` 的「Part B 防幻覺條款」），屬原稿的懸空參照。看到這些名稱，把它們當成「作者那邊有一份對應紀錄」即可。
