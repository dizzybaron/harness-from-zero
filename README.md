# harness-from-zero

讓你的 AI 陪你從零打造自己的 harness：一個資料夾加上幾份規則檔，讓 AI 在你的素材上做事、留下紀錄，出錯時會被攔下來。

這個 repo 有兩樣東西：

1. **給 AI 的起始指令**（`AGENTS.md`）：交給你的 AI，它會先問你幾個問題，再陪你建出最小可用的版本。
2. **一套實際在用的系統的規則檔快照**（`reference/`，2026-10）：一位急診醫師自用的知識庫與 AI 工作規則，分成五層，當作案例參考。

不必複製這套系統。它是用了幾個月、踩過很多雷才長成這樣；你的版本應該從小開始，遇到問題才加規則。

## 怎麼用

**A. 用 Claude Code 或 Codex（能讀寫電腦資料夾的 AI）**

1. 下載本 repo：本頁上方 **Code → Download ZIP** 後解壓縮，或 `git clone https://github.com/dizzybaron/harness-from-zero.git`
2. 在下載下來的資料夾裡開 Claude Code 或 Codex
3. 對它說：「照 AGENTS.md 帶我開始」

**B. 只用網頁版 ChatGPT／Claude／Gemini**

1. 上傳本 repo 的兩個檔：`AGENTS.md`、`reference/web-project-default-logic.txt`
2. 打開 [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)，按 **Download ZIP** 下載並解壓，把裡面的 `llm-wiki.md` 也上傳（本 repo 不附原文）
3. 對它說：「照 AGENTS.md 帶我開始」

**C. 只想看看**：從 [`reference/LAYERS.md`](reference/LAYERS.md) 開始讀。

## 裡面有什麼

```
AGENTS.md                         給 AI 的起始指令（Claude Code 經由 CLAUDE.md 載入同一份）
reference/
├── LAYERS.md                     五層規則對照：每層管什麼、檔案在哪、和 LLM Wiki 的對應
├── web-project-default-logic.txt 五層規則濃縮成單檔，改寫後可貼進網頁版 Project 指令
├── Harness_Architecture_2026.md  整套系統的六層控制流全圖
├── CLAUDE.global.example.md      憲法：跨專案通則
├── CLAUDE.project.example.md     法律：專案索引與路由
├── AGENTS.example.md             給非 Claude agent 的入口
├── governance/                   法律本文：調度、判斷、交辦範本、維護、診斷
├── vault_neo/                    規約、索引、命令：知識庫操作規則（不含筆記內容）
└── skills-example/               ingest／query／lint／reconcile 四個操作程序
```

## 刻意沒放的東西

- **知識庫筆記與其他受版權保護的內容**：作者的知識庫裡有教科書筆記、文獻筆記與 podcast 逐字稿，這裡一概不收；唯一收錄的筆記是架構說明 `reference/Harness_Architecture_2026.md`。你的素材請自行合法取得。
- **LLM Wiki 原文**：這套系統的設計原型，著作權屬 Andrej Karpathy，請讀[原始出處](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)。
- **個人資料**：私人記憶檔、工作日誌、待辦清單。
- **腳本與排程設定**：規則檔提到的 `scripts/`、launchd 都沒附；需要時讓你的 AI 依需求重寫。

規則檔裡提到的 memory 條目、工作日誌、`NEXT_SESSION.md` 等是作者的私有檔案。`<PROJECT_ROOT>`、`<MEMORY_DIR>`、`<user>`、`<private-repo>` 是去識別化後的佔位符；`<repo>`、`<domain>` 這類寫法是原文本來就有的。

## 注意

- 這是 2026-10 的快照，照原樣提供，不提供支援。
- 模型名稱、價格與工具功能會過時；`reference/governance/model-registry.md` 的數字以表內查證日為準。
- 內容是知識管理與 AI 協作的方法，**不是醫療建議**。

## 延伸資源

- [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)（Andrej Karpathy）：這套系統的設計原型
- [textbook-to-note](https://github.com/drpwchen/textbook-to-note)（drpwchen）：教科書轉筆記的管線，不含教科書內容

## 授權

本 repo 的文件以 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 授權：可自由複製、修改、再散布，註明出處即可。詳見 [LICENSE](LICENSE)。
