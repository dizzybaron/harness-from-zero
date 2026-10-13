> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。
> 文中的刪檔、git commit／push、排程、Telegram 通知與 `scripts/` 呼叫都依賴作者的環境，改寫時不要沿用。

# /ingest — 完整 inbox 處理管線（含 reconcile）

**觸發方式**：`/ingest`

**閉迴路位置**：見 `vault_neo/_Operations.md`（ingest 是「新 source → vault 多頁更新」的主入口）。

承接憲法「知識生產分工」與法律「知識 compounding」第 1 條：
> 每次 ingest 必觸發跨頁更新：新 source 不只新增 deep note，
> 還要更新涉及的 entity / concept / 比較 page。

此 skill 為 orchestrator，依序執行：
1. **MDer** — 格式轉換
2. **deepnote** — 筆記撰寫
3. **reviewer** — 格式檢核
4. **reconcile**（內嵌）— 跨頁更新（per-target 由 Claude 裁決套用）
5. **cleanup** — 刪除原始檔 + git commit

各步驟細節邏輯見對應 skill 文件（`MDer.md`、`deepnote.md`、`reviewer.md`、`reconcile.md`）。

---

## 執行步驟

### Step 1：確認 inbox 內容（MDer Step 1）

```bash
find <PROJECT_ROOT>/vault_neo -type d -name "*inbox*" | sort
```

對每個 inbox 目錄列出檔案。若所有 inbox 均為空，告知並結束。

**Step 1.5：批次大小評估（context 預算）**

清點 inbox 總檔數 N：

- N ≤ 5 → 正常單批 /ingest
- 5 < N ≤ 15 → 仍單批跑，但於 Step 8 結尾**必然提示** `/clear`（context 已累積至 50–100k+）
- N > 15（整書場景）→ **主動建議使用者分批**：

  ```
  ⚠️  inbox 共 N 個檔，整批跑完 context 預估 > 150k（cached 仍偏貴）。
  建議：分 ceil(N/8) 批跑，每批 ≤ 8 個檔，每批之間 /clear。
  繼續單批跑 / 中止改分批？(c/s)
  ```

  使用者選 `c` 才繼續；選 `s` 則中止，告知使用者手動把 inbox 拆分後分批 /ingest（或暫存到 inbox/_backlog/）。

Token 經濟依據：`governance/core-rules.md` §3（Session 衛生）— 連續處理 5–8 個 unit 後主動建議 /clear。

### Step 2：MD 化（MDer Step 2）

```bash
cd <PROJECT_ROOT>
python3 scripts/vault_neo_processor.py --phase md
```

完成後讀 `logs/vault_neo_jobs.json`。空陣列則結束。

### Step 3：寫筆記（deepnote）

對 `logs/vault_neo_jobs.json` 中每個 entry，spawn 一個 Agent 寫 deep note。
Prompt 細節見 `deepnote.md`。收集 DONE / FAIL 結果。

### Step 4：檢核（reviewer）

對 Step 3 中所有 DONE 的筆記，spawn Agent(model="haiku")。
Prompt 細節見 `reviewer.md`（H1–H6，可選 H7 語義一致性）。
收集 PASS / FAIL 結果。

### Step 5：FAIL 處理（reviewer Step 4-5 邏輯）

對每個 FAIL 筆記直接修正（Edit 工具），重跑 Haiku；仍 FAIL 則寫 `<原始檔名>.review_error.md` 標「需人工介入」。

### Step 6：Reconcile propose（內嵌）

對 Step 4-5 中所有 PASS 的 deep note，**集中**做 reconcile 提案：

**Step 6.1**：對每篇 PASS 跑 classify（先看影響全景再提 diff）

```bash
for note in $PASS_NOTES; do
  python3 scripts/reconcile.py gather "$note" --classify --limit 15
done
```

每篇輸出 `global_class: NEW / EXTEND_LIGHT / EXTEND_HEAVY / CONFLICT` 與分桶 candidate 列表。

**處理優先序**：
1. **CONFLICT** 桶 — 先走 `/reconcile` Step 1b 的 `[m/r/k/a]` 裁決路徑（共享 H2 + quant/directive 不一致；可能 false positive 由 LLM/使用者裁決）
2. **EXTEND_STRONG**（score ≥3 或 ≥2 anchor 命中）— 直接進 Step 6.2 提 diff
3. **EXTEND_WEAK**（score 1 單一 anchor）— LLM 自選；多數略過

`global_class: NEW` 的 PASS note 跳過 Step 6.2（無 candidate；建議使用者考慮 `/entity <topic>` 建聚合頁，但不強制）。

**Step 6.2**：逐 target 裁決並印紀錄（同 `/reconcile` single 模式 Step 2；不停下等使用者）

```
=== Reconcile（共 N 個 PASS 篇 → M 個 target page） ===

[1/M] vault_neo/.../target-A.md
  關聯：[[new-note-X]] (anchor: tags=cardio,arrhythmia)
  位置：## 相關文章｜See Also（既有段）
  diff：
  + - [[new-note-X]] — <一句說明關聯>
  裁決：套用 / skip（理由一句）

[2/M] ...
```

**Step 6.3**：套用

- 套用 → Edit tool；bump frontmatter `version`、更新 `last_updated`、`last_updated_by: ingest-reconcile`
- 有疑慮 → skip；刪除／改寫既有段落、改寫主文核心宣稱這類高 blast-radius 變更只 propose，等使用者

**Step 6.4**：寫 ReconcileLog

```bash
for new_note in $RECONCILED; do
  python3 scripts/reconcile.py log "$new_note" $TOUCHED_PAGES
done
```

**為何 ingest 內嵌 reconcile**：
- LLM Wiki 第一原則「每次 ingest 觸發跨頁更新」要求 reconcile **每次都跑**
- 若是 launchd / Telegram **靜默場景**（無 stdin），自動跑 `reconcile.py gather` 但**不套用**，
  把候選 diff 寫到 `vault_neo/_Insights/_PendingReconcile_YYYY-MM-DD.md`，
  下個 interactive session 跑 `/reconcile --pending` 收割

### Step 7：Cleanup

對所有成功（PASS）的 job：
1. 刪除 inbox 原始檔
2. 更新 `logs/vault_neo_manifest.json`（加入 processed 記錄）
3. catalog 增量維護：

```bash
for note in $PASS_NOTES; do
  python3 scripts/build_page_index.py --append "$note"
done
```

4. 寫 EventLog：

```bash
echo "## [$(date '+%Y-%m-%d %H:%M')] ingest | {N} files → {M} notes (reconcile: {K} pages)" \
  >> vault_neo/_EventLog.md
```

5. Git commit + push：

```bash
git add vault_neo/ logs/vault_neo_manifest.json logs/vault_neo_jobs.json
git commit -m "[ingest] $(date +%Y-%m-%d): {N} 個筆記 + {K} pages reconciled"
git push
```

### Step 8：輸出摘要 + Telegram 通知

```
✅ /ingest 完成
MD化：{N} 個檔案
寫筆記：{M} 個
  - vault_neo/路徑/note名.md  ✅
Reconcile 觸動：{K} pages 套用 / {L} pages 提案
需人工介入：{P} 個
  - 原始檔案：vault_neo/路徑/原始檔名.ext
    問題：<從 review_error.md 取得>

待處理（非阻塞）：
  - 待 reconcile 的 pending：{若有} → vault_neo/_Insights/_PendingReconcile_<date>.md
```

**Telegram 通知規則**：
- 若本次 `/ingest` 是從 Telegram 觸發，**只在 Step 8（全部完成）才透過 `reply` 傳送最終摘要**
- Step 1–7 期間**不要**呼叫 `reply` 或 `edit_message`；一律靜默
- 若執行中發生致命錯誤，才傳一則錯誤訊息作結

**/clear 提示規則**（Step 1.5 判定的 context 預算）：

- N ≤ 5：摘要結尾不額外提示
- N > 5（含整書場景）：摘要末尾固定附一行：

  ```
  建議下一步：/clear（本次 /ingest 處理 {N} 檔 / {M} 筆記，主對話 context 已累積；
  狀態已落地 vault + git，後續任務不會 reuse 此 context）
  ```

- Telegram 觸發的靜默場景**不附**此提示（使用者不在 Claude Code 主對話）

---

## 拆分 skill 關係

```
/ingest（orchestrator）
  ├── MDer（Step 1–2）
  ├── deepnote（Step 3）
  ├── reviewer（Step 4–5）
  ├── reconcile（Step 6，內嵌 propose 模式）
  └── cleanup + catalog + EventLog + git（Step 7–8）
```

個別 skill 可單獨呼叫，**不執行 cleanup**。

---

## 注意

- `--phase md` 不刪原始檔；收尾在 Step 7 統一處理
- 已處理過的檔案記錄於 `logs/vault_neo_manifest.json`，不重複處理
- 自動排程（2026-05-05 改 batch 模式）：launchd `com.<user>.vault-neo-inbox.plist` 每日 08:00 觸發 wrapper；wrapper 內 DoW gate：
  - 平日（週一–週六）→ `--phase md`（純 OCR，免 LLM）
  - 週日 → `--phase md_review`（含 reviewer Haiku，LLM）
  - `VAULT_NEO_FORCE_REVIEW=1` 或 `VAULT_NEO_PHASE=md_review` 可手動補跑
- EMCrit nightly Stage D + E（deepnote / reviewer）同樣改週末批次：每晚跑 Stage C (whisper) + D.5 (refs)，僅週六 23:00→週日凌晨進 Stage D + E。`EMCRIT_FORCE_DAILY_PIPELINE=1` 可繞過。
- 支援格式：PDF / PPTX / DOCX / XLSX / HTML / EPUB / TXT / MD / JPG / PNG

---

## 相關路徑

| 用途 | 路徑 |
|---|---|
| 設計規範 | `vault_neo/CLAUDE.md`、`vault_neo/_Operations.md` |
| vault_neo 根 | `vault_neo/` |
| 所有 inbox | `vault_neo/**/01_*inbox*/` |
| 執行腳本 | `scripts/vault_neo_processor.py` |
| Jobs 清單 | `logs/vault_neo_jobs.json` |
| 處理記錄 | `logs/vault_neo_manifest.json` |
| Pending reconcile | `vault_neo/_Insights/_PendingReconcile_YYYY-MM-DD.md` |
| 事件 log | `vault_neo/_EventLog.md`（type: ingest）|
