> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。

# 常載核心規則

> 本檔由專案 CLAUDE.md 以 @import 常載，行數上限 100（超過即精簡，見 governance/maintenance.md）。
> 規則細節與理由分別在 governance/model-dispatch.md 與 governance/judgment-rubrics.md，這裡只放每次都要用的部分。

## 1) 調度速查卡（每次 spawn Agent 前過一遍）

0. 前提：**主對話預設 T3**（前沿級，Opus 同級或以上智力）——分析、判斷、合成在主對話做，不自降級；
   下放的是機械與批次。tier ↔ 各 provider 型號映射見 `governance/model-registry.md`。
1. Agent 呼叫**必帶 `model` 參數**；缺 model＝bug。省略會 inherit 主對話模型（T3），等於用最貴的模型跑雜務。
2. 選型由低往高 tier，能用上面的就不用下面的（括號＝本 harness 型號）：
   - T1 找檔／grep symbol／清點目錄 → `subagent_type="Explore"` ＋ `model="haiku"`（read-only）
   - T1 讀多檔提取、摘要、格式檢核、套用既定 pattern 批次改檔 → `general-purpose` ＋ `model="haiku"`
   - T2 筆記撰寫、合成、需要判斷的批次、中等推理 → `model="sonnet"`
   - T3 平行重推理、fresh-context 第二意見、難 debug → `model="opus"`
     （分析本體留在主對話；spawn T3 是為平行或隔離，不是為「更聰明」）
   - `model="fable"`（單價高於 T3）不在預設選型內，使用者指定才用
3. 交辦時機：工作量要大到值得 spawn 的冷啟動開銷（重建 context、再探索、回報後主對話再讀）——全 repo／多目錄
   fan-out 搜尋、同一 pattern 套到大量檔案的機械批次、網路抓取、可平行的獨立長工作。幾次讀檔、幾處編輯、單一搜尋
   在主對話直接做；主對話即將**親自編輯或裁決**的文本（制度檔、待改稿）一律直讀——摘要不可替代原文。並行 subagent 能一個就不用多個。
4. 交辦必含三要素：**目標與動機／驗收條件／回報格式**。範本照抄 `governance/delegation-templates.md`。
5. 回報合約：第一行 PASS/FAIL/PARTIAL＋一句話；主體 ≤30 行；證據用 `檔案:行號`；長產物落檔回傳路徑；失敗明說，不硬掰。
6. 升降級：T1 同一子任務錯 1 次 → T2；T2 連錯 2 次 → T3（必附完整失敗軌跡）；
   同一件事最多重試兩輪，第三次必換策略或問人。細則見 `governance/model-dispatch.md` §5。

## 2) 知識來源（vault 鐵律）

1. 回答與筆記的知識**只能來自 vault**（`vault_neo/` deepnote＋`workspace/md/` raw）。
   訓練資料不得主動出現；vault 沒有的，不寫、不補，明說「vault 中尚無此內容」。
2. **fact-claim 必回溯 raw**：涉及數值（cutoff／劑量／比率）、診斷標準、藥名、引句、定義、判讀依據時，
   回應前沿 deepnote 的 `source_md`／`source_ref` 回原文核對，並標來源檔名＋段落／頁碼。
   結構性合成（跨 source 比較、鑑別框架、entity 比較、流程整合）以 deepnote 與 entity page 為主，無須回溯。
3. raw 不可達（未同步、已刪）時明示「raw 不可達，以下基於 deepnote」，不靜默退化。
4. 每篇 note 的 `source_ref`／`source_md` frontmatter 必須指向實際存在的 vault 檔案（wikilink）。

## 3) Session 衛生

- 每完成一個單元先落檔，再做下一個。對話不是儲存；session 隨時可能中斷，存檔的就是全部。
- 切換任務性質（ingest→query→其他）或連續處理 5–8 個單元 → 主動建議使用者 `/clear`。
- 互相獨立的 Agent 呼叫放同一則訊息並行；等背景工作靠完成通知，不為保溫 cache 空轉（TTL 以 harness 當下說明為準，本檔不寫死）。
- **停車場**：途中冒出的新任務／新發現，寫進 `NEXT_SESSION.md` 的「## 停車場」一行，不當場展開。

## 4) 完成的定義與誠實

- 「完成」＝驗收條件逐條打勾＋每條有 artifact 證據（`檔案:行號` 或指令輸出）＋下游登記完成（_PageIndex／catalog／manifest）。
- wrapper log 沒報錯 ≠ 完成；回報管線狀態前必須核實產物檔案與 manifest。
- 產出者 ≠ 驗證者：subagent 與管線產物由主對話或 fresh-context agent 對照驗收條件驗（不給產出者的推理），不採信自陳；主對話自己的產出在主迴圈 read-back／實跑驗證，不另派驗證 subagent。
- 沒把握就寫「未確認」＋建議查證路徑，不編造——特別是模型型號、價格、參數、引文。

## 5) Git 與破壞性操作

- 刪檔、force push、`git reset`、`rm -rf`、覆寫非自己建立的檔案 → 先向使用者確認。
- Session 結束 auto-commit（不需確認）白名單：`vault_neo/`、`scripts/`、`workspace/md/`、`CLAUDE.md`、
  `AGENTS.md`、`NEXT_SESSION.md`、`governance/`、`.claude/commands/`（memory 目錄在 repo 外，不入 git）。訊息格式：`[auto] YYYY-MM-DD session: <簡述>`；
  非 Claude Code agent 加身份標記 `[auto] YYYY-MM-DD <agent-id> session: <簡述>`（見 governance/onboarding.md §4）。
- 沙箱內 push 走 HTTPS＋gh credential helper（見 memory: `project_sandbox_git_push_https_gh`）。
