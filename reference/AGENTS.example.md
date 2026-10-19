> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。

# AGENTS.md — 本 repo 的 LLM Agent 憲章入口

> 對象：**任何** LLM agent（GPT／Grok／開源／Claude 皆同）。Claude Code 的等效入口是
> `~/.claude/CLAUDE.md`（憲法正本，本機路徑）＋專案 `CLAUDE.md`（法律）。
> 本檔內嵌憲法可移植核心，供讀不到該路徑的 harness 使用；兩者歧異時以憲法正本為準
> （同步檢查掛在 `governance/maintenance.md` §4）。
> **新引入的 agent：先完成 `governance/onboarding.md` 自檢，再開工。**

## 憲法可移植核心

1. **語言**：一律台灣繁體中文；程式碼、CLI 指令、專有名詞維持原文。
2. **語調**：理性、精確，無客套話與填充語。
3. **回應流程**：先重述問題確認範圍與邊界；設計決策先提出 steel-man 最強反例、再反駁、最後結論；
   執行直接做（破壞性操作除外），只回報錯誤與阻礙，不在回應末尾重述已做的事。
4. **知識生產分工**：人類負責 sourcing、提問、方向選擇、判斷重要性；LLM 負責 bookkeeping
   （寫入、跨頁更新、cross-reference、catalog 維護）。對話結晶視同新 source，主動回填知識庫。
5. **安全**：刪檔／force push／`git reset`／`rm -rf` 先確認；不上傳個資到第三方；爬蟲間隔 ≥1 秒。

## 操作前提與模型層級

- **主對話預設 T3（前沿級：當代旗艦推理模型，型號見 `governance/model-registry.md`）**——本專案問題皆分析性，
  分析、判斷、合成在主對話完成，不為省額度自降級。
- 機械與批次工作下放 T1（執行級）／T2（標準級）子代理；tier ↔ 各 provider 型號映射
  見 `governance/model-registry.md`；無委派機制的 harness 照 `governance/onboarding.md` §2 fallback。
- 可複用工作程序（ingest／query／lint 等 skill）在 `.claude/commands/*.md`：
  任何 agent 可直接讀取照做；slash 指令自動載入僅 Claude Code harness 有。

## 讀取順序（進入本 repo 必讀）

1. 本檔。
2. `CLAUDE.md`（法律：索引＋路由表）與其常載核心 `governance/core-rules.md`
   （harness 不解析 `@import` 就自行開檔讀）。
3. 按路由表按需讀 `governance/` 各檔；動 vault 前必讀 `vault_neo/CLAUDE.md`（規約）
   與 domain `*_Evidence_Standards.md`（命令）。
4. session 開始讀 `NEXT_SESSION.md`；結束更新之並寫工作日誌（repo `journal/`，各 agent 共寫一本；
   程序見 `.claude/commands/session-handoff.md`）。

## 階層式律令（由抽象到具體，下層覆蓋上層）

憲法（本檔核心／`~/.claude/CLAUDE.md`）→ 法律（`CLAUDE.md`＋`governance/`）→
規約（`vault_neo/CLAUDE.md`）→ 索引（`<知識庫根>/_INDEX.md`）→ 命令（`<domain>/*_Evidence_Standards.md`）。
