---
title: 這台電腦的 harness engineering 架構（2026-09-22 實測）
type: note
category: 分享課
domain: Projects
last_updated: 2026-10-05
source_ref: "[[急診醫師的AI使用指南_2026]]"
audience: 分享課備課用／技術附錄
tags: [harness engineering, 架構圖, AI使用, 教學分享, 知識管理]
---

> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。

# 這台電腦的 harness engineering 架構

> 全部數字為 2026-09-22 實測（`find`／`ls` 計數），非估計。
> 本檔是《急診醫師的 AI 使用指南》第 18–20 張的技術底稿，**不直接上投影片**——
> 投影片用最末節的精簡版。

---

## 一、為什麼不畫成目錄樹

最直覺的畫法是檔案總管式的目錄樹：這台電腦的實體結構就長那樣，一目了然、可逐行對照。

但目錄樹表達的是**儲存位置**，而 harness 的本體是**誰在約束誰、什麼觸發什麼**——
這兩件事在目錄樹上不可見：

- `governance/core-rules.md` 躺在 repo 裡，但它真正的作用是被 `@import` 注入每一輪對話的 context；
- launchd 的 `.plist` 在 `~/Library/LaunchAgents/`，驅動的卻是 repo 內的腳本；
- reviewer 與 writer 的檔案就放在同一個 `.claude/commands/`，但關鍵差異在於**兩者 context 互相隔離**。

所以主軸用**分層控制流**，每層標註實體路徑，兼顧可對照性。

---

## 二、主圖：六層控制流

```
┌─ L0 人類入口 ── 三條通道，同一個 git repo ───────────────────────────────────

   ① Terminal（Claude Code TUI）     ② Telegram Bot            ③ claude.ai web／mobile
      model = opus                      ~/.local/bin/              ＋ GitHub connector
      effortLevel = high                claude-telegram-host       （外出／雲端無人值守）
      ~/.claude/settings.json           （tmux 常駐 session）      plugin: telegram@0.0.7

            └──────────────┬──────────────┴──────────────┬──────────────┘
                           ▼                             ▼
              github.com/<user>/<private-repo> ◄── 唯一同步通道（git push／pull）
                           │
                           ▼
┌─ L1 約束層 ── 階層式律令，開場即注入 context，下層覆蓋上層 ──────────────────

   憲法  ~/.claude/CLAUDE.md ················ 身份／語言／推理流程／調度鐵律／安全
     │                                        （跨專案，所有 repo 都吃）
     ▼
   法律  <repo>/CLAUDE.md ··················· 索引與路由表；@import ↓
     │   └─ governance/core-rules.md ········ 常載核心（硬上限 100 行）
     │      governance/ 另 8 檔 ·············· 按需讀：model-dispatch／judgment-rubrics
     ▼                                              ／delegation-templates／maintenance …
   規約  vault_neo/CLAUDE.md ················ 知識庫操作：閉迴路／catalog／回填規則
     │
     ▼
   索引  vault_neo/_INDEX.md ················ 知識庫地圖
     │
     ▼
   命令  <domain>/*_Evidence_Standards.md ··· 4 份：EM／Endurance／Philosophy／EBM
                                              各 domain 的格式檢核條文（H1–H6）

   旁掛（非階層，但同樣自動進 context）：
     · memory/  157 個單一事實檔 ＋ MEMORY.md 索引（踩過的雷、使用者偏好、管線地雷）
     · NEXT_SESSION.md  交班單（≤80 行，含「停車場」）— session 開場第一件事就是讀它
     · AGENTS.md  非 Claude agent（GPT／Grok／開源）的等效入口，內嵌憲法可移植核心

┌─ L2 執行層 ── 主對話當老闆，雜事分包 ───────────────────────────────────────

   主對話（T3 前沿級 · Opus）
     │  ← 分析、判斷、合成留在這裡，不為省額度自降級
     │
     ├──► T1 執行級（Haiku）    找檔／grep／清點／格式檢核／套用既定 pattern 批次改檔
     ├──► T2 標準級（Sonnet）   筆記撰寫／合成／需判斷的批次
     └──► T3 前沿級（Opus）     平行重推理／fresh-context 第二意見／難 debug

   交辦鐵律：每次 spawn 必帶 model 參數（省略＝ inherit Opus ＝用最貴的跑雜務）
             每次交辦必含三要素：目標與動機／驗收條件／回報格式
             回報合約：第一行 PASS／FAIL／PARTIAL，證據用「檔案:行號」

   19 個 skill（<repo>/.claude/commands/，放 repo 內 → 跨 agent 共享、隨 git 版控）
     入庫類  /ingest  /emcrit  /deepnote  /MDer  /transcript
     檢核類  /reviewer  /lint  /reconcile  /writeback
     查詢類  /query  /entity  /verdict  /appraise
     維運類  /session-handoff  /ruminate  /cloud-pull  /dialogue-catcher
     專案類  /update-boardexam  /update-cdc-diseases

┌─ L3 工具與供料層 ── 讓它當場查，不憑記憶答 ─────────────────────────────────

   MCP server（本機 node）     openevidence ···· 臨床問答＋文獻，global scope
   claude.ai 連結器            PubMed ·········· 檢索／全文／ID 轉換
                               Google Drive ···· 試算表、文件
                               Claude Docs ····· 共筆文件
                               Coros ··········· 訓練與生理資料（Endurance domain）
   瀏覽器                      claude-in-chrome  擴充，接管既有 Chrome session
   本機工具鏈                  marker-m2 ······· PDF／DOCX → MD（~/marker-m2/bin/marker）
                               whisper.cpp ····· podcast 語音 → 逐字稿（CPU-only）
                               pandoc／curl ····· 網頁與 JATS 全文
   自有腳本                    <repo>/scripts/ 49 支 .py／.sh（scraper／refs_fetcher／
                               linter／reconcile／page_index／token_report …）

┌─ L4 資料層 ── raw 與 deepnote 分離，是防幻覺的實體基礎 ─────────────────────

   inbox/ ──► workspace/originals/ ──► raw MD ──► deep note ──► catalog
   （投遞）    （964 個原始檔封存）    （全文）    （我的筆記）   （檢索面）

   vault_neo/  5,241 個 MD
     ├ 01_Projects/     分享課、教案（本檔所在）
     ├ 02_EM/           EM_Notes／Textbook／Blogs（EMCrit 680 篇 raw）／Study／Case／Review
     ├ 03_Endurance/    Textbook／Blogs／Study／Case／Review
     ├ 04_Philosophy/   The EM Mind 等
     ├ 05_EBM/          EBM_Verdicts／Claim_Appraisals
     ├ _Insights/       ruminate 報告、待批單（_PendingReconcile／_PendingLintReview）
     ├ _PageIndex.md    20 份分層 catalog（/query 的主檢索面）
     └ _EventLog.md     所有寫入事件的流水帳

   workspace/  渲染產物與中間檔（marp_output／gitbook／appraise／verdict）
               ← 刻意**不放進 vault**：大 MD 會卡死 Obsidian indexer
   logs/       manifest（emcrit／vault_neo／boardexam）＋各管線 log
               ← 「完成」的判定看 manifest 與產物檔，不看 wrapper log 沒報錯

   閱讀端：Obsidian 直接開 vault_neo/ ── LLM 寫、人類讀，同一批檔案

┌─ L5 自動化層 ── 該定期做的事，不靠我記得 ───────────────────────────────────

   每日 07:00  launchd  com.<user>.vault-neo-inbox
               → run_vault_neo_processor.sh → marker OCR（週日切 md_review 相位）
   每日 23:00  launchd  com.<user>.emcrit-transcribe
               → whisper.cpp 夜間批次轉錄，完成主動推 Telegram
   每週一 22:00 launchd com.<user>.vault-weekly-maintenance
               → 三階段：① 安全自動修復 ② 偵測（lint／漂移／缺 deepnote／用量）
                 ③ 單一 Telegram digest ＋ git commit & push
   每週日 22:00 雲端    routine: weekly-vault-ruminate（14:00 UTC）
               → additive-only 白名單：只新增報告與待批單，不改既有筆記
               → commit 身分 vault-ruminate <actions@anthropic.com>，本機 /cloud-pull 收回

┌─ L6 閘門層 ── 真正在擋錯的地方 ─────────────────────────────────────────────

   擋幻覺   fact-claim（數值／劑量／診斷標準／引句）必回溯 raw 原文，標檔名＋頁碼
            vault 沒有的 → 明說「vault 中尚無此內容」，不補、不寫
            產出者 ≠ 驗證者：reviewer 是另一個 agent，只拿驗收條件，不給它前一個的推理
   擋誤刪   刪檔／force push／git reset／rm -rf／覆寫非自建檔 → 一律停下等人點頭
            auto-commit 白名單：只有 vault_neo／scripts／governance／.claude/commands 等
   擋越界   macOS Seatbelt 沙箱：寫入路徑白名單（repo 內＋$TMPDIR），
            .claude/settings.json 等設定檔即使在白名單內也 deny
            網路走過濾 proxy，未列出的主機直接拒絕
   擋漂移   source_hash 偵測 raw 變動；entity_cross_link 偵測該連未連的頁；
            md_linter 檢 frontmatter／必填段落；check_missing_deepnote 抓有 raw 無筆記
```

---

## 三、一條龍資料流：一則 EMCrit 從連結到入庫

```
  貼連結
    │
    ▼
  /emcrit ──► scraper 抓 raw.md ＋ mp3 ──► refs_fetcher 解析 references
    │           （vault raw 區）            │  每個 DOI／PMID 建 paper note
    │                                       │  （撞號檢查：同姓同年不同論文不得覆寫）
    ▼                                       ▼
  whisper.cpp 轉錄（CPU-only，夜間或 session 內）
    │
    ▼
  T2 Sonnet subagent 寫 deep note ── 只准用 raw ＋ transcript，不准用訓練資料
    │
    ▼
  主對話逐條回溯核對 ── 每個數值／人名／引用 grep 回原文（單集曾核 168 項）
    │                    抓到的典型錯：ASR 人名誤植、跨筆記數字誤引、自行推導的加總
    ▼
  T1 Haiku reviewer（fresh context）── 對 Evidence_Standards H1–H6 逐條檢
    │                                  FAIL → 回頭改；但 reviewer 編造規範外規則者判幻覺
    ▼
  登記：manifest bump ／ _PageIndex append ／ raw 檔尾反向連結 ／ _EventLog
    │
    ▼
  _PendingReconcile 待批單 ──► /reconcile 跨頁更新（See Also、entity page、比較頁）
```

---

## 四、閘門對照表：每一條都對應一次真實事故

| 閘門 | 實作位置 | 擋掉過什麼 |
|---|---|---|
| 回溯 raw | 各 `*_Evidence_Standards.md` | deepnote 把「arrest→插管」寫成「到院→插管」 |
| 產出者 ≠ 驗證者 | `/reviewer` 獨立 agent | writer 自認合格的格式缺漏 |
| refs 出處錨點欄 | `/emcrit` prompt | refs=0 的集數編造 citation（強化後捏造數歸零） |
| 撞號檢查 | `emcrit_refs_fetcher.py` | 同集同姓同年不同論文**靜默覆寫** |
| 否定式 wikilink | `reconcile.py` | 「本頁**未**連結 [[X]]」被誤判為真連結 |
| PMC elink | `emcrit_scraper.py` | DOI 搜尋抓到「引用本文者」當成本文全文 |
| auto-commit 白名單 | `core-rules.md` §5 | 把渲染產物／個資檔一起推上 GitHub |
| 沙箱寫入 deny | Seatbelt 設定 | agent 改自己的權限設定檔 |

---

## 五、投影片精簡版（可直接貼，21 行）

```
                        人（三條通道：終端機／Telegram／手機網頁）
                                      │
        ┌─────────────────────────────┼─────────────────────────────┐
        ▼                             ▼                             ▼
    【規矩】                       【做事】                      【擋錯】
    員工手冊 CLAUDE.md            主對話當老闆               回溯原文才敢寫
    SOP     19 個 skill           雜事分包給小模型           產出者 ≠ 驗證者
    便利貼  157 則 memory         判斷留給大模型             刪檔前停下問我
    交班單  NEXT_SESSION.md       每次交辦附驗收條件         沙箱限制寫入範圍
    反射    4 條排程                                         
        └─────────────────────────────┬─────────────────────────────┘
                                      ▼
                            【語料】vault 5,241 篇
                            原文 raw ─────► 我的筆記 deep note ─────► 目錄 catalog
                            （可回溯）      （LLM 寫）                （可檢索）
                                      │
                                      ▼
                            Obsidian（人讀）／ git（同步）
```

> 講者一句話：**左邊三根柱子都不是寫程式，是把「怎麼做事」和「什麼時候要停」寫下來。**
> 底下那條 raw → deep note → catalog 才是防幻覺的實體基礎——
> 沒有可回溯的原文，上面三根柱子都是擺設。
