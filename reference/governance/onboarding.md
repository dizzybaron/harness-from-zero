> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。

# 新 Agent 引入協定（Onboarding）

> 適用：任何被引入本 repo 協作的 LLM agent——不限 provider（GPT／Grok／開源皆同）、
> 不限 harness（CLI／IDE／雲端）。Claude Code 視為已在籍。行數上限 100。
> 引入的第一個指令固定是：「讀專案根 `AGENTS.md`，照 `governance/onboarding.md` 完成自檢後再開工」。

## 1) 讀取順序（不可跳讀）

1. `AGENTS.md`（憲法可移植核心＋路由入口）。
2. `CLAUDE.md`（法律：索引與路由）＋其常載核心 `governance/core-rules.md`
   （harness 不解析 `@import` 就自行開檔讀）。
3. 按路由表按需讀 `governance/` 各檔；動 vault 前必讀 `vault_neo/CLAUDE.md`（規約）
   與對應 domain `*_Evidence_Standards.md`（命令）。
4. session 開始讀 `NEXT_SESSION.md`；結束更新之並寫工作日誌。

## 2) 能力自檢（capability mapping）

開工前逐項宣告有／無；缺項照 fallback 執行，不得靜默假裝有：

| 能力 | 有 → | 無 → fallback |
|---|---|---|
| 檔案讀寫 | 正常工作 | 不可引入（本制度以檔案為唯一共享面） |
| shell | 可跑 `scripts/` 驗證 | 驗收改 read-back 型；回報標明「未實跑」 |
| 子代理委派 | 照 core-rules §1 調度；tier 映射查 `governance/model-registry.md` | 全部工作 inline 以自身 tier 執行；回報合約與驗證規則不變（產出與驗證分兩輪、fresh 視角自查） |
| 網路 | 可查證外部來源 | 禁用訓練資料補外部 fact，一律標「未確認」 |
| git | 照 core-rules §5 | 改動清單交使用者提交 |

## 3) Registry 登記與 conformance 驗收

1. 在 `governance/model-registry.md` §2 登記 provider／model ID／tier／查證日（照 registry §3 程序）。
2. Conformance 三題，**全 PASS 才開工**，結果記工作日誌：
   - [ ] 複誦 vault 鐵律兩條核心（知識只來自 vault；fact-claim 回溯 raw），
         並示範一次「vault 中尚無此內容」的正確應答。
   - [ ] 用回報合約格式（core-rules §1 第 5 條）回報一個 mock 任務
         （第一行 PASS/FAIL＋一句話；證據 `檔案:行號`）。
   - [ ] 對 vault 任一 deepnote 的一條 fact-claim，實際演示沿 `source_md`／`source_ref` 回溯 raw 的路徑。

## 4) 身份與協作紀律

- **身份標記**：commit 訊息 `[auto] YYYY-MM-DD <agent-id> session: <簡述>`
  （agent-id 例：`gpt`、`grok`、`qwen`；Claude Code 可省略 agent-id，維持既有格式）。
  工作日誌與 NEXT_SESSION 條目同樣註明 agent-id。
- **單一 writer**：同一時間只有一個 agent 對 repo 寫入；並行僅限 read-only。
  動 repo 前先 `git status`；發現非自己的未提交變更 → 停手回報，不覆寫。
- **共享面只有 repo 內檔案**：`~/.claude/` 下的 memory 為 Claude Code 私有，其他 agent 不得假設可讀；
  需跨 agent 共享的教訓 → 寫對應 governance 檔「教訓紀錄」一行。
  skills 本體在 repo `.claude/commands/*.md`（2026-07-15 遷入）——markdown 程序文件，
  任何 agent 可直接讀取照做；slash 指令自動載入僅 Claude Code harness 有。
- **交接面**：跨 agent 的工作交接只透過 `NEXT_SESSION.md`（含停車場）與 governance 教訓段，
  不透過任何 agent 的私有記憶。

## 5) 對所有 agent 等效的鐵律（違反任一＝不合格，不分 provider）

1. 知識來源 vault 鐵律與 fact-claim 回溯 raw（core-rules §2）。
2. 完成的定義與誠實：驗收逐條＋artifact 證據；沒把握標「未確認」，不編造（core-rules §4）。
3. 破壞性操作先問；auto-commit 只限白名單（core-rules §5）。
4. 安全守則：個資不上第三方、爬蟲間隔 ≥1 秒（AGENTS.md 憲法核心）。
5. 交辦三要素與回報合約（core-rules §1）；無委派能力者至少遵守回報合約部分。

## 教訓紀錄（append-only，一行一條）

- 2026-07-15 立檔。尚無非 Claude agent 實際引入；首次引入時本協定需實測校準。
