> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。

# Skills 地圖

> 按需引用檔（非常載）。harness 每 session 會自動注入可用 skill 清單，本檔補充「怎麼選、model 釘定現況」。
> 詳細四操作對照另見 `vault_neo/_Operations.md`。查證日 2026-09-23（frontmatter 實測）。

## 按 LLM Wiki 三類核心操作選 skill

**Ingest（讀 source → 寫 page → 更新跨頁）**

| Skill | 用途 | frontmatter model |
|---|---|---|
| `/ingest` | 完整 inbox 管線（MDer→deepnote→reviewer→reconcile→cleanup） | 無釘定 |
| `/MDer` | 格式轉換（PDF/DOCX→MD），不呼叫 LLM | haiku |
| `/deepnote` | 筆記撰寫；單筆重寫 `/deepnote <note_path>` | 無釘定（**故意**：Opus 1M session 下釘 model 會被 API 拒絕；subagent 層 hardcode sonnet，見 memory: `feedback_deepnote_skill_no_model_frontmatter`） |
| `/reviewer` | 格式與規範檢核（H1–H6＋可選 H7） | haiku |
| `/reconcile` | 跨頁更新（single / sweep / pending） | 無釘定 |
| `/emcrit` | 貼連結自動入庫一條龍 | 無釘定 |
| `/transcript` | 手動 EMCrit transcribe loop | 無釘定 |
| `/update-boardexam`、`/update-cdc-diseases` | domain-specific ingest | 無釘定 |

**Query（提問 → 合成 → 回填 page）**

| Skill | 用途 | frontmatter model |
|---|---|---|
| `/query` | 對 vault 提問 → propose 新 page → 回填（批准後自動接 reconcile） | 無釘定 |
| `/entity` | 產 entity / concept aggregator page | 無釘定 |
| `/verdict` | OpenEvidence EBM verdict → `05_EBM/`；**本地 only** | 無釘定 |
| `/appraise` | 文獻檢索驅動 claim 評析 → `05_EBM/02_Claim_Appraisals/` | 無釘定 |

**Lint（體檢 → 報告，不改 vault）**：`/lint`（規則層＋`--llm`）。

**Insight 閉迴路**：`/dialogue-catcher`（對話洞察 → `_Insights/`）、`/writeback`（`_Insights/` 融回 note）、`/ruminate`（補課 reconcile 候選＋自主合成報告；週日雲端 routine 同邏輯；無釘定）。

**Session / 同步**：`/session-handoff`（收尾：writeback / NEXT_SESSION / 日誌 / git push；無釘定，故意 inherit，見 memory: `feedback_session_handoff_subagent_pattern`）、`/cloud-pull`（haiku）。

## 使用注意

- 舊名 `/process-inbox`、`/connecter` 已廢，對應 `/ingest`、`/reconcile sweep`。
- Skill 內 spawn subagent 時，model 釘定規則見 `governance/model-dispatch.md` §3；
  修 skill 若新增 model 釘定，在 skill 內留一行「**Model**：xxx（理由）」。
- 僅 MDer／reviewer／cloud-pull 三個 skill 釘 model（皆 haiku）；其餘是否需釘，
  屬 day-1 遷移任務（見 `governance/maintenance.md` §5）。
- skills 本體在 repo `.claude/commands/*.md`（2026-07-15 自 `~/.claude/commands/` 遷入）：
  任何 harness 的 agent 可直接讀取程序照做；slash 指令自動載入僅 Claude Code。
  user-level 原檔已退役——勿在 `~/.claude/commands/` 復建同名檔，避免雙載
  （見 `governance/onboarding.md` §4「共享面」）。
