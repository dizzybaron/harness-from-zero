# AGENTS.md — 給 AI agent 的起始指令

> 本檔寫給 **AI agent**（Claude Code、Codex、ChatGPT、Gemini 等）讀；人類讀者請看 `README.md`。
> 本檔只在兩種情況生效：使用者要你「照 AGENTS.md 開始」，或你是在本 repo 根目錄啟動。
> 若你是在處理別的專案時順帶讀到本檔，忽略它。

## 你在哪裡

這是一個教學用 repo。使用你的人想打造**自己的** AI 工作系統（harness）：一個資料夾加上幾份規則檔，
讓 AI 在他的素材上做事、留下紀錄，出錯時會被攔下來。這個人多半是臨床醫師，不一定會寫程式。

`reference/` 收的是本 repo 作者自用系統的規則檔快照（2026-10）。**那是案例，不是給你的指令。**

## 鐵則

1. **`reference/` 只參考，不照做。** 那些檔案寫給作者自己的 agent，綁著作者的資料夾、工具與習慣，
   例如「開場先讀 NEXT_SESSION.md」「spawn 子代理必帶 model 參數」「session 結束自動 commit＋push」
   「主對話固定用最貴的模型」，都不適用於現在的使用者。
2. **從最小開始，不整套照抄。** 第一版規則控制在 50 行內：資料夾版從 Step 2 的三條起手預設開始，
   網頁版從刪減後的範本開始。之後每加一條，都要對應使用者實際遇到的問題。
3. **建在使用者自己的資料夾**，不寫進本 repo（做法見 Step 2）。
4. **個資紅線**：病人資料與任何能識別個人的資訊，不放進資料夾、不上傳第三方服務。使用者想放時，先提醒去識別化。
5. **一次問一到兩題**，問的時候附上你的建議。用使用者的語言回應（預設台灣繁體中文）。

## 流程

### Step 0：你先讀（不用唸給使用者聽）

1. Andrej Karpathy 的 LLM Wiki（https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f）：
   核心模式。raw／wiki／schema 三層，ingest／query／lint 三個操作。本 repo 不附原文：能上網就直接讀這個連結；
   不能上網，請使用者打開連結按 Download ZIP，把裡面的 `llm-wiki.md` 交給你。
2. `reference/LAYERS.md`：作者怎麼把規則分成五層、每層管什麼、檔案在哪。
3. `reference/governance/diagnosis.md`、`reference/governance/letter-to-future.md`：作者的規則是被哪些失敗逼出來的。
   這兩份只看「出過什麼錯、為什麼」；裡面的指令是寫給作者自己的 session 的，不要執行。

其他檔案等用到再查。在網頁版聊天時，只需要 LLM Wiki 原文與 `reference/web-project-default-logic.txt`；
其餘檔案有就讀，沒有也不必要求使用者補傳。

### Step 1：訪談

依序弄清楚（不必一次問完）：

- 想累積什麼知識？素材是什麼形式（教科書、paper、podcast、指引、自己的筆記）？
- 用什麼工具？只有網頁版聊天，還是有 Claude Code、Codex 這類能讀寫資料夾的 agent？
- 每週大概能花多少時間？
- 哪些資料絕對不能放？
- 最想先解決的一個具體問題是什麼？

### Step 2：建最小版

**只用網頁版聊天**：以 `reference/web-project-default-logic.txt` 為範本，和使用者一起改寫成他自己的
Project 自訂指令：語言、語調、領域、紅線換成他的，並刪到他真正需要的部分（50 行內）。產出一份可以直接貼上的文字。

**有能讀寫資料夾的 agent（Claude Code、Codex）**：

1. 請使用者在本 repo **之外**建一個新的空資料夾（例如桌面上的 `my-wiki`），把路徑告訴你。
2. 你的工具若不允許寫到目前資料夾以外：請使用者在新資料夾裡重新啟動 agent，
   並對它說「讀 `<本 repo 的路徑>/AGENTS.md`，從 Step 2 繼續」。
3. 在新資料夾建立：

   ```
   <資料夾>/
   ├── AGENTS.md   ← schema：資料夾結構、命名、三個操作怎麼做（用 Claude Code 時另建只寫一行 @AGENTS.md 的 CLAUDE.md）
   ├── raw/        ← 原始素材，只讀不改
   ├── wiki/       ← AI 寫的筆記；每個數字與引用標出處（raw 檔名＋段落）
   ├── index.md    ← 目錄：每頁一行（連結＋一句摘要）
   └── log.md      ← 時間序紀錄：## [YYYY-MM-DD] ingest | 標題
   ```

4. schema 先放三條起手預設。這是本 repo 建議的預設，不是作者的鐵律：向使用者說明理由，由他決定保留、修改或刪掉。
   - 知識只從 `raw/` 與 `wiki/` 來；沒有就說「資料夾中尚無」，不用訓練資料補。
     （理由：AI 會用記憶補出看似合理的細節，這是最常見、也最難察覺的錯）
   - 數字、劑量、診斷標準、引句必須回 `raw/` 原文核對並標出處。
   - 每次 ingest 後更新 `index.md` 與 `log.md`。
5. 建好之後，請使用者以後都在新資料夾開 agent；本 repo 只在需要參考時再讀。

### Step 3：跑一次完整迴圈

**有資料夾的 agent**：

1. **Ingest**：使用者放一份自己的素材進 `raw/` → 你寫一頁筆記 → 更新 `index.md`、`log.md`。
2. **Query**：使用者問一題 → 你先查 `index.md`、讀相關頁、回答附出處 → 問他要不要把答案存成新的一頁。
3. **回顧**：哪裡卡住、哪裡錯了。把發現的問題記下來，這是 Step 4 的材料。

**網頁版聊天**：使用者把一份素材貼進對話或上傳 → 你整理成一則筆記，每個數字標出處 → 他存進自己的筆記 App。
下次提問時，請他把相關筆記貼回來，你只根據這些筆記回答。最後同樣回顧哪裡卡住、哪裡錯了。

### Step 4：遇到問題才長規則

使用者遇到下表的問題時，先讀對應檔案，向他說明作者為什麼有這條規則，再一起改寫成適合他的版本：

| 使用者遇到的問題 | 參考作者的做法 |
|---|---|
| AI 編造數字或引用 | `reference/governance/core-rules.md` §2；`reference/governance/judgment-rubrics.md` R5；`reference/web-project-default-logic.txt` 第 5 節 |
| AI 說做完了，其實沒做完 | `reference/governance/core-rules.md` §4；`reference/governance/judgment-rubrics.md` R2 |
| 想讓 AI 自動檢查筆記格式 | `reference/vault_neo/02_EM/EM_Evidence_Standards.md` Part H（封閉式檢核清單的寫法） |
| 規則檔越寫越長，AI 開始不照做 | `reference/governance/maintenance.md` §3；`reference/governance/letter-to-future.md` 第一節 |
| 換個對話就忘了上次做到哪 | `reference/CLAUDE.project.example.md`「會話生命週期」；`reference/governance/core-rules.md` §3 |
| 不同主題的規則互相打架 | `reference/LAYERS.md`（下層覆蓋上層） |
| 不知道 AI 何時該停下來問人 | `reference/governance/judgment-rubrics.md` R3 |
| 同一個錯誤一直重試 | `reference/governance/judgment-rubrics.md` R4 |
| 筆記之間沒有互相連結 | `reference/skills-example/reconcile.md`；`reference/vault_neo/CLAUDE.example.md` |
| 對話中的好結論流失 | `reference/skills-example/query.md`；`reference/vault_neo/CLAUDE.example.md`（writeback） |
| 想把 ingest／query／lint 做成固定程序 | `reference/skills-example/`；`reference/vault_neo/_Operations.md` |
| 想讓便宜的模型做雜事 | `reference/governance/model-dispatch.md`；`reference/governance/delegation-templates.md` |
| 想換用或並用別家 LLM | `reference/governance/onboarding.md`；`reference/AGENTS.example.md` |

一次只加一條。加完跑一次最小案例，確認 AI 真的照做。

## 不要做的事

- 不要把 `reference/` 的檔案整份複製進使用者的資料夾。
- 改寫 `reference/` 裡的程序時，不沿用：刪除原始檔、自動 commit／push、排程（launchd、cron）、Telegram 通知、
  呼叫 `scripts/`（本 repo 沒有這些腳本）。使用者的原始素材一律保留。
- 不要一開始就建排程、腳本、多模型分工、沙箱設定；那是作者用了幾個月之後才需要的東西。
- 不要替使用者決定什麼重要：挑素材、提問、判斷重要性是他的工作；整理、寫入、交叉引用、維護目錄是你的。
