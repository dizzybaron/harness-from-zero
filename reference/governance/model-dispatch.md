> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。

# 模型調度守則

> 讀者：主對話模型（預設 T3 前沿級；任何 provider 皆適用）。速查卡在 `governance/core-rules.md` §1（常載）；
> 本檔是規則本文與理由，以 tier 語彙書寫；型號與價格的唯一映射面是 `governance/model-registry.md`。
> 憑記憶改型號或價格＝違規；更新程序見 §9。

## 0) 層級語彙與預設

- 制度語彙用 **tier**：T1 執行級／T2 標準級／T3 前沿級。定義、各 provider 型號與價格見
  `governance/model-registry.md`（本 harness 現值：haiku=T1、sonnet=T2、opus 與 fable=T3）。
- **主對話預設 T3**：本專案問題皆分析性，主對話品質不讓給額度；判斷輕的純批次 session 可手動 `/model` 降 T2。
- **計費框架**：訂閱制用量視窗（Max）。貴模型消耗視窗額度的倍率≈單價比——管理的是視窗內相對消耗，
  不是每 token 帳單；倍率邏輯與 API 計費相同，規則跨 provider 通用。
- 省略 `model` 即 inherit 主對話模型——那就是預設陷阱本身；合法值以當下 Agent tool schema 為準。

## 1) 指揮官不下場

主對話只做六件事：拆解任務、寫驗收條件、裁決 subagent 回報、與使用者溝通、
分析／設計／跨 domain 合成的本體（主對話就是 T3，分析不外包也不自降級）、
以及「換便宜模型就掉品質」的判斷本身。其餘交辦。

**交辦時機**（工作量大到值得 spawn 的固定開銷——冷 context、再探索、回報後主對話再讀）：
- 全 repo／多目錄 fan-out 搜尋，且只需要結論、不需要原文
- 同一 pattern 套到大量檔案的機械批次
- 網路抓取／爬蟲／文件查證
- 彼此獨立、可平行的長工作

**主對話直接做**：幾次讀檔、幾處編輯、單一搜尋；以及主對話即將**親自編輯或裁決**的文本——摘要不可替代原文，直讀。
判準：spawn 的固定開銷比動作本身貴、或委派會損失分析所需的原文保真時，直接做。並行 subagent 能一個就不用多個。
門檻暫不寫數字，待 weekly token report 能量測調度遵從後再校準（maintenance §4 第 4 項）。

## 2) 任務交辦三要素

每次交辦必含（範本照抄 `governance/delegation-templates.md`，不要即興）：
1. **目標與動機**——為什麼做。讓 agent 遇到範本沒寫的微決策時不偏航。
2. **驗收條件**——客觀可查核，逐條列。寫不出驗收條件＝任務還沒想清楚，先想清楚再交辦。
3. **回報格式**——固定欄位＋長度上限（見 §4）。

## 3) 顯式指定 model 與 effort ［本節為 Claude Code harness 專屬；其他 harness 見 governance/onboarding.md §2］

- 每次 `Agent` 呼叫必帶 `model`。選型照 registry 映射表；拿不準時：先試低一級 tier，靠 §5 升級路徑補救——
  升級一次的成本遠低於預設用貴模型。
- **Agent tool 呼叫本身沒有 per-call effort 參數**（查證 2026-09-23）。要控 effort 有三個位置：
  - session 層：`/effort` 或 `/model`，檔位 `low/medium/high/xhigh/max`，可存為新 session 預設（含 max）；
    headless `claude -p` 排程呼叫一律顯式帶 `--model`／`--effort`，不依賴互動預設。Haiku 4.5 不支援 effort。
  - 自訂 agent 定義：`.claude/agents/*.md` frontmatter（`name`／`description` 必填；`model`、`effort`、`tools` 選填；
    `effort` 合法值同上）。目前本專案無自訂 agent；若某類交辦高頻出現，可建一個把 model＋effort 釘死。
  - skill frontmatter：同樣支援 `model` 與 `effort` 欄位。
- 內建 subagent（交辦會用到的五種）：`Explore`（read-only 檢索）、`Plan`（read-only 規劃）、
  `general-purpose`（全工具）、`claude`（全工具 catch-all）、`claude-code-guide`（查 Claude Code／API 官方文件）；
  另有 `statusline-setup`（僅設定狀態列用，非交辦對象）。
  內建 agent 的預設 model 未查證——**一律顯式帶 model** 繞過此未知。
  Explore／Plan 不能 Edit/Write；要改檔用 `general-purpose`。

## 4) 回報合約

subagent 的回報必須：
- 第一行：`PASS`／`FAIL`／`PARTIAL` ＋ 一句話結論。
- 主體 ≤30 行；每個宣稱附證據 `檔案:行號` 或指令輸出摘句。
- 長產物寫到指定路徑，回傳路徑；**不貼全文回主對話**。
- `FAIL`／`PARTIAL` 時附：試了什麼（依序）、卡在哪、原始錯誤訊息原文。
- 禁止：貼整檔內容、無證據的「已完成」、把 partial 說成 success、為湊 PASS 放寬驗收條件。

主對話收到回報後的義務：對照驗收條件逐條裁決，不因為「看起來像完成了」就採信（見 judgment-rubrics R2）。

## 5) 升降級路徑

**升級**：
- T1 同一子任務**錯 1 次** → 換 T2 重派。
- T2 同一子任務**連錯 2 次** → 換 T3 重派，且必附**完整失敗軌跡**：
  原 prompt、兩次輸出、每次錯在哪。不附軌跡的升級會讓 T3 重犯同樣的錯，等於白升。
- T3 仍錯 → 停。這通常不是模型問題而是方向問題：走 judgment-rubrics R4（方向錯誤訊號），換路或問使用者；
  產出 load-bearing 且 registry 有其他 provider 的 T3 時，可先做一次跨 provider 第二意見再停。
- **同一件事最多重試兩輪**；第三次動手前必須換策略、升級或問人，不得原樣重試。

**先排除三種假升級題**（升級救不了，見 judgment-rubrics R1 反例）：
指示不清（先修 prompt）、缺資料（先補資料）、幻覺（接地治——給原文、要求引用，不是換大模型）。

**降級**：一旦解出 pattern（例：確立某類格式的修法、某類錯誤的判式），把解法寫成明確步驟
（輸入→步驟→預期輸出），降回 T1/T2 批次套用。解題用 T3、量產用低 tier。

**fast mode ≠ 降級**：`/fast` 是同模型加速輸出、單價更高（支援型號見 `governance/model-registry.md`），只在互動延遲敏感時用。

## 6) 驗證不自驗

- **產出者 ≠ 驗證者**。subagent／管線產物由主對話或 fresh-context agent 驗：只給驗收條件與產物路徑，
  **不給產出者的推理**（避免錨定）。主對話自己的產出在主迴圈 read-back／實跑驗證，不另派驗證 subagent。
- 驗證手段按產物類型：
  - 檔案／文件 → **read-back**：重開檔案，逐條對驗收清單，回報 `檔案:行號` 級證據。
  - 程式碼 → **實跑或測試**：一個 happy path＋一個 edge case；「看起來對」不算。
  - 高風險判斷（醫療 fact、對外發布、不可逆操作）→ **第二意見**：不同模型重答，或 3 樣本＋評審擇優。
- **fact-claim 類驗證不打折**（回溯 raw 是鐵律，見 core-rules §2）。
- 驗證的 model 選擇：checklist 式 read-back 用 T1；語義一致性／第二意見用 T2 以上
  （第二意見可跨 provider，registry 有他家 T3 時優先考慮——不同訓練譜系的盲點重疊最小）。

## 7) 併發與 cache 經濟

- 互相獨立的 Agent 呼叫放**同一則訊息**並行送出；有依賴才排序。
- prompt cache TTL 以 harness 當下說明為準（本檔不寫死數字）；長等待就接受可能的 cache miss，不空轉保溫。
- subagent 是背景執行、完成自動通知——不要輪詢、不要 sleep 等待。
- subagent 的 context 用完即棄：可複用的結論落檔（vault／governance／memory），不留在對話裡。

## 8) 特殊事項與未確認

- Fable 5 有安全導流機制（部分請求自動轉 Opus 4.8 處理）；其**額度歸屬未確認**（官方公告與 docs 皆未載明，
  查證 2026-07-04）→ 需要時到 usage 儀表板實測。
- 訂閱視窗的實際額度數字帳戶面才看得到，session 內不可見；接近上限的訊號是 API 回應變慢或被限流。

## 9) 查證與更新

- 型號與價格的查證值一律改在 `governance/model-registry.md`（本檔不再持有型號表）。更新程序：
  1. Anthropic 列：invoke `Skill "claude-api"`（本地權威參照，離線可用）；
     必要時派 `claude-code-guide` agent 查官方 docs（sub-agents／model-config／pricing 頁）。
  2. 其他 provider 列：查各家**當日官方** pricing／docs（照 registry §3，禁止憑記憶）。
  3. 改 registry 表與查證日，並在其教訓段記一行。
- 觸發時機：每季一次、模型換代新聞出現時、或 registry 備註中的促銷到期日。
  排程掛在 `governance/maintenance.md` §4。

## 10) 教訓紀錄（append-only，一行一條）

- 2026-07-04 立檔。門檻數字（§1）與升降級次數（§5）為啟發式初值，未經本地量測校準。
- 2026-07-15 使用者裁決：主對話預設升 T3（Opus 級或同等智力，理由：問題皆分析性）；tier 語彙上線，
  型號價格表移 `governance/model-registry.md`；provider-neutral 化（AGENTS.md＋onboarding.md）。
- 2026-09-23 prompt-audit（主對話改 Opus 5.5）：§1 交辦門檻改為工作量判準、§6 主對話自身產出不另派驗證 subagent（Opus 5 起傾向過度委派與過度自驗）；§0／§3／§5／§7／§9 的 harness 事實更新。依據：`workspace/prompt_audit/2026-09-23/report.md` F12／F15／F16。
