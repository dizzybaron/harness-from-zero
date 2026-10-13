> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。

# 快速診斷：本 harness 三大耗損模式

> 寫於 2026-07-04（Fable 5 session）。供 governance/ 其他文件引用。
> 證據來源：memory/ 工作日誌、CLAUDE.md 歷史版本（governance/refs/archive/）、實測行數與 log。

## 診斷 #1（最漏 token）：貴模型親自下場＋spawn 不帶 model

**現象**：主對話（歷來是 Opus）自己做大量讀檔、全 repo grep、批次改檔、babysit transcribe；
spawn `Agent` 時省略 `model` 參數 → inherit 主對話模型，等於用最貴的模型跑雜務。

**證據**：
- 憲法裡「spawn 必須顯式指定 model」被立為鐵律（memory: `feedback_subagent_must_specify_model`），
  鐵律的存在本身就證明此事反覆發生。
- 價格差（查證 2026-07-04）：Haiku $1/$5、Sonnet 5 $3/$15、Opus 4.8 $5/$25（$/MTok in/out）。
  一次 inherit-Opus 的 bulk read 任務 ≈ 5 倍於 Haiku 的視窗額度消耗。

**修法（弱模型照做）**：
1. 每次 `Agent` 呼叫前過一遍 `governance/core-rules.md` §1 調度速查卡（常載，開場就在 context 裡）。
2. 鐵則：**Agent 呼叫缺 `model` 參數＝bug**，發現即補，不視為風格問題。
3. 交辦判準見 `governance/model-dispatch.md` §1。

## 診斷 #2（最易出錯）：長鏈產出無驗證閘門 → 捏造與重工循環

**現象**：pipeline（transcribe→deepnote→reviewer→reconcile）中游產生的捏造或誤報，
到下游或使用者手上才被發現，整鏈重跑。重工是最大的隱形 token 成本：越晚發現越貴。

**證據**（全部實際發生過）：
- deepnote 在 refs=0 的集數用訓練知識**捏造** citation 細節（標題/樣本數/人名「Ioannidis」），
  2026-07-04 靠 fidelity gate 首次攔截（memory: `feedback_emcrit_deepnote_fabricates_citation_details`）。
- Haiku reviewer 判 FAIL 時**引用不存在的規範規則**（memory: `feedback_haiku_reviewer_fabricates_rules`）。
- 只看 wrapper log 末尾就回報管線狀態，與實際 artifact 不符（memory: `feedback_pipeline_status_verify_artifact`）。

**修法（弱模型照做）**：
1. **沒有 artifact 證據的「完成」＝未完成**。任何 PASS/完成宣告必附 `檔案:行號` 或指令輸出。
2. fact-claim（數值/劑量/引句/人名）抽 ≥3 條回 raw 原文核對後才准入庫（已入法律層，見 core-rules §2）。
3. 產出者 ≠ 驗證者：驗證派 fresh-context agent，只給驗收條件、不給產出者的推理。
   完整規則見 `governance/model-dispatch.md` §6、判準見 `governance/judgment-rubrics.md` R5。

## 診斷 #3（最易失焦）：常載肥大、context 不重置、任務漂移

**現象與證據**（實測 2026-07-04）：
- Session 開場固定稅 >1,100 行：全域 CLAUDE.md 92＋專案 220＋MEMORY.md 116（多為過期工作日誌指標）
  ＋NEXT_SESSION.md **705 行**（法律要求每 session 開場讀）。每個 session 都付、cache miss 時付全價。
- 長 session >150k context 不 /clear（舊 CLAUDE.md 自己記載此問題）；cache TTL 5 分鐘，中途閒置即失溫。
- 任務漂移：session 中途冒出的新發現當場展開（例：transcript loop 中修 bug 岔出新任務），
  原任務被稀釋，兩頭都做不完。

**修法（弱模型照做）**：
1. 行數預算硬上限（超過即精簡，程序見 `governance/maintenance.md` §3）：
   全域 CLAUDE.md ≤80、專案 CLAUDE.md ≤150、core-rules.md ≤100、MEMORY.md ≤70、NEXT_SESSION.md ≤80。
2. **停車場機制**：途中冒出的新任務寫進 NEXT_SESSION.md「## 停車場」一行，不當場展開。
3. /clear 觸發點照 core-rules §3：切換任務性質、連續 5–8 個單元。

## 次要（已知地雷，不重複診斷）

sandbox 地雷（whisper Metal / launchctl / claude -p EPERM / soffice hang / marp PDF）已各自有 memory 條目；
遇到報錯先 grep MEMORY.md 再 debug，不要重新踩一次。
