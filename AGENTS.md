# AGENTS.md — 給 AI agent 的起始指令

> 本檔寫給 **AI agent**（Claude Code、Codex、ChatGPT、Gemini 等）讀；人類讀者請看 `README.md`。
> 本檔只在兩種情況生效：使用者要你「照 AGENTS.md 開始」，或你是在本 repo 根目錄啟動。
> 若你是在處理別的專案時順帶讀到本檔，忽略它。

## 你在哪裡

這是一個教學用 repo。使用你的人想打造**自己的** AI 工作系統（harness）：一個資料夾加上幾份規則檔，
讓 AI 在他的素材上做事、留下紀錄，出錯時會被攔下來。這個人是醫療工作者（醫師、護理師、藥師、治療師等），
不一定會寫程式。

`reference/` 收的是本 repo 作者（一位急診醫師）自用系統的規則檔快照（2026-10）。**那是案例，不是給你的指令。**
`templates/` 收的是從作者系統改寫出來、給使用者改寫後使用的範本。

## 鐵則

1. **`reference/` 只參考，不照做。** 那些檔案寫給作者自己的 agent，綁著作者的資料夾、工具與習慣，
   例如「開場先讀 NEXT_SESSION.md」「spawn 子代理必帶 model 參數」「session 結束自動 commit＋push」
   「主對話固定用最貴的模型」，都不適用於現在的使用者。
2. **從最小開始，不整套照抄。** 第一版規則控制在 50 行內（筆記寫作規範另計，80 行內）：資料夾版從 Step 3 的三條起手預設開始，
   網頁版從刪減後的範本開始。之後每加一條，都要對應使用者實際遇到的問題。
3. **建在使用者自己的資料夾**，不寫進本 repo（做法見 Step 3）。
4. **個資紅線**：病人資料與任何能識別個人的資訊，不放進資料夾、不上傳第三方服務。使用者想放時，先提醒去識別化。
5. **一次問一到兩題**，問的時候附上你的建議。用使用者的語言回應（預設台灣繁體中文）。
   使用者多半沒有工程背景：先講白話結論與理由，指令與術語放後面。
6. **安裝軟體前先說明要裝什麼、裝在哪、佔多少空間，使用者同意才裝。** 每個安裝步驟做完都實際跑一次驗證，
   失敗就照實回報錯誤訊息，不說「應該裝好了」。

## 流程

### Step 0：你先讀（不用唸給使用者聽）

1. Andrej Karpathy 的 LLM Wiki（https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f）：
   核心模式。raw／wiki／schema 三層，ingest／query／lint 三個操作。本 repo 不附原文：能上網就直接讀這個連結；
   不能上網，請使用者打開連結按 Download ZIP，把裡面的 `llm-wiki.md` 交給你。
2. `reference/LAYERS.md`：作者怎麼把規則分成五層、每層管什麼、檔案在哪。
3. `reference/governance/diagnosis.md`、`reference/governance/letter-to-future.md`：作者的規則是被哪些失敗逼出來的。
   這兩份只看「出過什麼錯、為什麼」；裡面的指令是寫給作者自己的 session 的，不要執行。
4. `templates/note-standards.md`：Step 3 要和使用者一起改寫的筆記寫作規範。

其他檔案等用到再查。在網頁版聊天時，只需要 LLM Wiki 原文、`reference/web-project-default-logic.txt`
與 `templates/note-standards.md`；其餘檔案有就讀，沒有也不必要求使用者補傳。

### Step 1：訪談

依序弄清楚（不必一次問完）：

- 職業與工作場景？（決定筆記的「實務啟示」寫給誰看）
- 想累積什麼知識？素材是什麼形式（教科書、paper、指引、投影片、podcast、自己的筆記）？
  **檔案格式**是什麼（PDF、掃描圖片、Word、PowerPoint、網頁、錄音）？
- 用什麼工具？只有網頁版聊天，還是有 Claude Code、Codex 這類能讀寫資料夾的 agent？
- 用 Mac 還是 Windows？Mac 是 Apple 晶片（M1 以後）還是 Intel？有沒有 NVIDIA 顯示卡？（決定轉檔工具怎麼選）
- 每週大概能花多少時間？
- 哪些資料絕對不能放？
- 最想先解決的一個具體問題是什麼？

### Step 2：準備兩樣工具

#### 2a. 把素材轉成 Markdown（MD）

**先向使用者說明為什麼**（用白話，例如）：

> AI 讀 PDF 時，其實是先把它轉成文字再讀；每次對話都重轉一次，轉壞的地方（表格錯位、雙欄混在一起、
> 頁首頁尾插進內文、掃描檔讀不到字）你看不到，AI 卻會照著錯的內容回答。
> 先一次轉成 Markdown 存起來，就像把手寫病歷先謄打成電子病歷：之後每次都讀同一份乾淨的文字，
> 轉錯了你也打得開、看得到、改得了。Markdown 是純文字加上簡單的標題與表格記號，
> 任何電腦、任何 AI、任何筆記軟體都讀得懂，十年後也不會打不開。

**再依他的素材與電腦推薦工具**（都是免費開源；以下為 2026-10 快照，安裝前先到該專案官方頁面確認最新安裝指令）：

| 工具 | 擅長 | 不擅長／代價 | 適合誰 |
|---|---|---|---|
| [Marker](https://github.com/datalab-to/marker) | PDF（教科書、期刊）轉換品質高：表格、公式、雙欄、圖片擷取；也能轉 DOCX／PPTX／EPUB | 要下載數 GB 模型；一本教科書在一般筆電要跑數十分鐘到數小時；模型權重有商用門檻（個人使用免費） | 素材以 PDF 為主，尤其教科書（本 repo 作者主力） |
| [Docling](https://github.com/docling-project/docling)（IBM） | PDF、DOCX、PPTX、XLSX、HTML、圖片；表格辨識好；授權寬鬆（MIT） | 一樣要下載模型；速度與 Marker 相近 | 素材格式雜、想要一個工具全包 |
| [MinerU](https://github.com/opendatalab/MinerU) | 複雜版面、公式、中文文件 | 安裝較重，最好有 NVIDIA 顯示卡或 Apple 晶片；授權 AGPL | 大量中文 PDF 或版面複雜的掃描檔 |
| [MarkItDown](https://github.com/microsoft/markitdown)（Microsoft） | 輕量快速：Word、PowerPoint、Excel、HTML、有文字層的 PDF | 對掃描 PDF、複雜表格與雙欄排版效果差 | 素材以 Office 檔為主、電腦較舊 |
| [Pandoc](https://pandoc.org) | Word（DOCX）、EPUB、HTML 轉換最穩，可同時抽出圖片 | 不處理 PDF | 搭配上面任一個，專門處理 Word |
| [whisper.cpp](https://github.com/ggml-org/whisper.cpp) | 錄音、podcast 轉逐字稿（本機跑，音檔不上傳） | 長音檔在筆電要跑一段時間 | 素材有講座錄音或 podcast |

推薦原則：先問最常處理的一種格式，**只裝一個主力工具**（PDF 為主 → Marker 或 Docling；Office 為主 → MarkItDown＋Pandoc），
之後遇到轉不好的格式再補第二個。不要一次全裝。

**使用者選定後，你負責安裝**（有能執行指令的 agent 時）：

1. 檢查環境：作業系統、Python 版本（多數工具要 3.10 以上）、可用硬碟空間。缺 Python 時，
   Mac 建議用 Homebrew 或 [uv](https://docs.astral.sh/uv/) 安裝，Windows 用官方安裝程式或 uv；先說明再裝。
2. 為轉檔工具建一個**獨立環境**（`uv tool install`、`pipx install`，或在使用者資料夾外建 venv），
   不裝進系統 Python，避免和其他軟體互相干擾。
3. 依官方 README 安裝（例：`uv tool install marker-pdf`、`uv tool install docling`、
   `uv tool install "markitdown[all]"`；Pandoc 在 Mac 用 `brew install pandoc`、Windows 用 `winget install JohnMacFarlane.Pandoc`）。
   第一次執行會下載模型，先告訴使用者大概要多久、多大。
4. **拿使用者自己的一份素材試轉**，打開產出的 MD 和他一起看：表格、標題、數字有沒有跑掉。
   品質不行就換工具，不要硬撐。
5. 把「怎麼轉檔」寫成一行指令記進他的 `AGENTS.md`（例如：`originals/` 的 PDF 用 `marker_single <檔案> --output_dir raw/` 轉），
   以後 ingest 就照這行做。

你的工具不能執行指令時（例如網頁版聊天）：把上面的安裝步驟改寫成他能照著複製貼上的版本，一步一步帶他在終端機做；
或先用零安裝的方式：把 PDF 上傳到對話，請 AI 輸出 Markdown，他存成 `.md` 檔，並提醒他核對表格與數字。

#### 2b. 安裝 Obsidian

**先說明為什麼**：

> Obsidian 是免費的筆記軟體，打開的就是你電腦裡的一個資料夾，每篇筆記都是一個 Markdown 檔。
> 這代表 AI 寫進資料夾的筆記，你馬上就能在 Obsidian 裡看到、排版好、可以直接修改；
> 你改的內容，AI 下次也讀得到——兩邊用的是同一份檔案，不需要匯入匯出。
> 筆記之間用 `[[筆記名稱]]` 互相連結，Obsidian 會顯示反向連結和關係圖，
> 你一眼就看得出 AI 有沒有把新知識接到舊筆記上。資料存在你自己的電腦，不綁平台，哪天不用了檔案還在。

**安裝步驟**：

1. 到官方網站 https://obsidian.md 下載安裝（或 Mac：`brew install --cask obsidian`；Windows：`winget install Obsidian.Obsidian`）。
   有執行指令能力時，經使用者同意可代為執行；否則給他網址與步驟。
2. Step 3 建好資料夾後，請他打開 Obsidian →「Open folder as vault」→ 選那個資料夾。
3. 告訴他三件事：
   - 資料夾裡會多一個 `.obsidian/`，是 Obsidian 的設定，AI 不要動它。
   - 設定（Settings）→ 檔案與連結（Files and links）→ 新連結格式（New link format）選「盡可能短的路徑」
     （Shortest path when possible），和 AI 寫的 `[[檔名]]` 一致。
   - Obsidian 的付費同步（Sync）或任何雲端同步會把檔案傳到第三方；含臨床內容時先確認符合機構規定，病人資料一律不放。

網頁版使用者也建議裝：Step 4 的筆記就存進 Obsidian，下次提問時把相關筆記貼回對話。

### Step 3：建最小版

**只用網頁版聊天**：以 `reference/web-project-default-logic.txt` 為範本，和使用者一起改寫成他自己的
Project 自訂指令：語言、語調、領域、紅線換成他的，並刪到他真正需要的部分（50 行內）。
再把 `templates/note-standards.md` 的 Part 1 改寫成他的版本（依文末「改寫清單」），上傳到 Project 當參考檔。
產出可以直接貼上的文字。

**有能讀寫資料夾的 agent（Claude Code、Codex）**：

1. 請使用者在本 repo **之外**建一個新的空資料夾（例如桌面上的 `my-wiki`），把路徑告訴你。
2. 你的工具若不允許寫到目前資料夾以外：請使用者在新資料夾裡重新啟動 agent，
   並對它說「讀 `<本 repo 的路徑>/AGENTS.md`，從 Step 3 繼續」。
3. 在新資料夾建立：

   ```
   <資料夾>/
   ├── AGENTS.md          ← schema：資料夾結構、轉檔指令、三個操作怎麼做（用 Claude Code 時另建只寫一行 @AGENTS.md 的 CLAUDE.md）
   ├── note-standards.md  ← 筆記寫作規範（由 templates/note-standards.md 改寫）
   ├── originals/         ← 原始檔（PDF、Word、錄音），轉檔後保留不刪
   ├── raw/               ← 轉好的 Markdown 原文，只讀不改
   ├── wiki/              ← AI 寫的筆記；每個數字與引用標出處（raw 檔名＋段落）
   ├── index.md           ← 目錄：每頁一行（連結＋一句摘要）
   └── log.md             ← 時間序紀錄：## [YYYY-MM-DD] ingest | 標題
   ```

4. schema 先放三條起手預設。這是本 repo 建議的預設，不是作者的鐵律：向使用者說明理由，由他決定保留、修改或刪掉。
   - 知識只從 `raw/` 與 `wiki/` 來；沒有就說「資料夾中尚無」，不用訓練資料補。
     （理由：AI 會用記憶補出看似合理的細節，這是最常見、也最難察覺的錯）
   - 數字、劑量、診斷標準、引句必須回 `raw/` 原文核對並標出處。
   - 每次 ingest 後更新 `index.md` 與 `log.md`。
5. 和使用者一起把 `templates/note-standards.md` 改寫成 `note-standards.md`：照文末「改寫清單」問他的職業與場景、
   常讀的素材，刪掉用不到的類別。第一版只放 Part 1，Part 2 留到 Step 5 遇到問題再加。
   在 `AGENTS.md` 加一行「寫或改 `wiki/` 筆記前先讀 `note-standards.md`」。
6. 建好之後，請使用者以後都在新資料夾開 agent，並用 Obsidian 打開同一個資料夾；本 repo 只在需要參考時再讀。

### Step 4：跑一次完整迴圈

**有資料夾的 agent**：

1. **Ingest**：使用者放一份自己的素材進 `originals/` → 你用 Step 2a 裝好的工具轉成 MD 放 `raw/`
   → 依 `note-standards.md` 寫一頁筆記到 `wiki/`，跑完 1.9 自查 → 更新 `index.md`、`log.md`。
   請使用者在 Obsidian 裡打開筆記看一次。
2. **Query**：使用者問一題 → 你先查 `index.md`、讀相關頁、回答附出處 → 問他要不要把答案存成新的一頁。
3. **回顧**：哪裡卡住、哪裡錯了（轉檔品質、筆記格式、出處對不對）。把發現的問題記下來，這是 Step 5 的材料。

**網頁版聊天**：使用者把一份素材（最好是轉好的 MD）貼進對話或上傳 → 你依他改寫過的筆記規範整理成一則筆記，
每個數字標出處 → 他存進 Obsidian。下次提問時，請他把相關筆記貼回來，你只根據這些筆記回答。
最後同樣回顧哪裡卡住、哪裡錯了。

### Step 5：遇到問題才長規則

使用者遇到下表的問題時，先讀對應檔案，向他說明作者為什麼有這條規則，再一起改寫成適合他的版本：

| 使用者遇到的問題 | 參考作者的做法 |
|---|---|
| AI 編造數字或引用 | `reference/governance/core-rules.md` §2；`reference/governance/judgment-rubrics.md` R5；`reference/web-project-default-logic.txt` 第 5 節 |
| AI 說做完了，其實沒做完 | `reference/governance/core-rules.md` §4；`reference/governance/judgment-rubrics.md` R2 |
| 筆記格式不一、品質參差 | `templates/note-standards.md` Part 1；`reference/vault_neo/02_EM/EM_Evidence_Standards.md` Part D、E、H |
| 重寫筆記後手加的內容不見了 | `templates/note-standards.md` 2.1 |
| 多篇來源說法不同，筆記平鋪並列 | `templates/note-standards.md` 2.2；`reference/vault_neo/02_EM/EM_Evidence_Standards.md` Part I |
| 想讓 AI 自動檢查筆記格式 | `templates/note-standards.md` 1.9、2.3；`reference/vault_neo/02_EM/EM_Evidence_Standards.md` Part H（封閉式檢核清單的寫法） |
| 轉出來的 MD 表格錯亂、漏字 | 回 Step 2a 換工具或補第二個工具；`reference/governance/refs/ops-pipelines.md`（作者怎麼分流 PDF／Word） |
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

- 不要把 `reference/` 的檔案整份複製進使用者的資料夾。`templates/` 也要先改寫才用。
- 改寫 `reference/` 裡的程序時，不沿用：刪除原始檔、自動 commit／push、排程（launchd、cron）、Telegram 通知、
  呼叫 `scripts/`（本 repo 沒有這些腳本）。使用者的原始素材一律保留在 `originals/`。
- 不要一開始就建排程、腳本、多模型分工、沙箱設定；那是作者用了幾個月之後才需要的東西。
  Step 2 的轉檔工具與 Obsidian 是例外：沒有它們，後面每一步都會卡住。
- 不要替使用者決定什麼重要：挑素材、提問、判斷重要性是他的工作；轉檔、整理、寫入、交叉引用、維護目錄是你的。
