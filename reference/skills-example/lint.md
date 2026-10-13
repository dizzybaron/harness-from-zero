> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。
> 文中的刪檔、git commit／push、排程、Telegram 通知與 `scripts/` 呼叫都依賴作者的環境，改寫時不要沿用。

# /lint — vault 體檢

**觸發方式**：`/lint [--scope <path>] [--checks ...] [--llm]`

**閉迴路位置**：見 `vault_neo/_Operations.md`（lint 是「找問題不修」這條路）。
是**唯一不改任何 note** 的操作 — 只產報告 + 給建議（建議形式為「下一個該跑的 query / reconcile / writeback」）。

---

## 兩層檢查

### Layer 1：規則層（零 LLM，預設執行）

```bash
cd <PROJECT_ROOT>
python3 scripts/ruminate.py lint [--scope <path>] [--checks orphans,stale,missing-page] [--dry-run]
```

| 檢查 | 方法 | 成本 |
|---|---|---|
| `orphans`        | 圖計算：無 inbound wikilink 的 page | 零 LLM |
| `stale`          | 規則：`last_updated` > N 個月 / 來源 mtime 比 note 新 | 零 LLM |
| `missing-page`   | 文中提及的概念無對應 page（先 grep candidate） | 零 LLM |

`extract_wikilinks` 降噪（2026-06-10）：code fence / inline code 內的 `[[...]]` 為 schema 範例或指令示意，不算真實邊；`![[assets/X.png]]` 等圖片/附件 embed 依副檔名跳過。missing baseline 經一次性 triage 後為 26（真實稀疏訊號：paper 互引未建 + EMCrit 集待 deepnote），週報 diff `> 26` 才值得看。

### Layer 2：LLM 層（`--llm` 旗標啟用，成本較高）

針對 Layer 1 結果做語義判斷與深度分析：

| 檢查 | 方法 | 成本 |
|---|---|---|
| `xref` 推薦      | 對 orphan page spawn Sonnet：找 vault 中應該但沒被連結的 target | 中 |
| `gaps`           | 對 missing-page 候選 spawn Sonnet：判斷是否真值得開新 page、建議第一個 source query | 中 |

LLM 層**不自動跑全 vault**（成本爆炸），預設只跑 Layer 1 結果中 top-N（預設 N=20）的優先項。

### Layer 1.5：contradictions 啟發式（零 LLM）

```bash
python3 scripts/ruminate.py lint --checks orphans,stale,missing-page,contradictions
```

對 inbound link ≥ 5 的 high-traffic page，挑前 N（預設 10）個 target，
與其前 K（預設 6）個 inbound source 跑 `reconcile.detect_conflicts()`（共享 H2 + quant/directive 不一致）。

純 Python，無 LLM call。找到的 contradiction 寫入 `_PendingLintReview` 走 [m/r/k/a] 裁決。

控制 flag：`--contradiction-targets N` / `--contradiction-sources K`。

### Layer 0：結構 safe-fix（`md_linter.py --auto-fix-safe`，預設不跑）

確定性結構修補，跟 Layer 1/2 分開（不需要先跑 lint 報告）：

```bash
python3 scripts/md_linter.py --all-neo --auto-fix-safe       # 全 vault_neo
python3 scripts/md_linter.py --auto-fix-safe <file.md>       # 單篇
python3 scripts/md_linter.py --all-neo --auto-fix-safe --dry-run   # 預覽
```

| 修補類型 | 條件 | 動作 |
|---|---|---|
| 補 `updated:` | frontmatter 完全缺欄位 | 加 `updated: <mtime date>` |
| 補 `## 相關文章｜See Also` | section 完全不存在 | 加骨架；若 body 有 wikilinks 則 lift 前 8 個進 See Also |

**不做**：`updated:` < mtime 的 bump（無法區分自動 touch vs 內容修；應由 reviewer / reconcile 顯式 bump）；
See Also 已有但內容不齊（由 `/reconcile sweep` 補連結）。

報告寫到 `vault_neo/_Insights/_LintAutoFix_<date>.md`（無修補則不寫）。

### 外掛工具門面（`--catalog` / `--xlink` / `--pearls`）

三支獨立 lint 工具統一從 `ruminate.py lint` dispatch，不必記它們的存在：

```bash
python3 scripts/ruminate.py lint --catalog --xlink            # 零 LLM，週維護預設帶
python3 scripts/ruminate.py lint --pearls --pearls-limit 3    # spawn Sonnet，手動才跑
```

| flag | 轉呼叫 | 內容 | 成本 |
|---|---|---|---|
| `--catalog` | `audit_page_index.py` | _PageIndex broken-link / summary-drift / stale-date | 零 LLM |
| `--xlink`   | `entity_cross_link.py --pending` | entity 頁間缺的交叉 See Also 候選 | 零 LLM |
| `--pearls`  | `lint_pearl_fidelity.py` | Pearl 對 raw 原文忠實度 spot-check | **LLM（Sonnet）** |

各工具仍寫自己的 `_Insights/` 報告；門面只統一進入點與摘要輸出。

### Layer 3：_PendingLintReview 隊列（`--pending` 旗標）

```bash
python3 scripts/ruminate.py lint --pending
```

把 orphans / missing / contradictions 全寫成 actionable 隊列檔 `_Insights/_PendingLintReview_<date>.md`，
每筆含 `suggested_action`（多為 `reconcile gather --classify` 或 `/entity`）。

設計同 `_PendingReconcile`：一次 lint 留下隊列，後續 `/lint --review` 走 y/N 處理（隊列模式不一次跑完）。

`--pending` 自動啟用 contradiction check（即使 --checks 未列）。

---

## 流程

### Step 1：跑 Layer 1（規則）

```bash
python3 scripts/ruminate.py lint
```

輸出寫到：`vault_neo/_Insights/_LintReport_YYYY-MM-DD.md`

報告含：
- orphans 清單（含 wikilink + 路徑）
- stale 清單
- missing-page 候選（含 candidate snippet）
- 每項末附「**建議的下一步 query**」（直接可貼回 `/query`）

### Step 2（可選）：跑 Layer 2（LLM）

若使用者下 `/lint --llm`：

對 Layer 1 結果中 top-20 優先項：
1. **orphan + xref**：spawn Sonnet subagent，prompt：「讀此 orphan page，從 catalog 找出 3-5 個應加 inbound link 的 target page，提案具體 diff」
2. **missing-page + gaps**：spawn Sonnet：「判斷此 missing-page 候選是否值得獨立成 page；若是，建議 entity page 的第一個 query 主題」
3. **contradictions**：對 high-traffic page（>5 inbound links）spawn Sonnet：「抽查 3 條事實宣稱，與其他 page 比對是否衝突」

LLM 層輸出 append 到同一份 `_LintReport`，分節：
- `## LLM 分析：xref 推薦`
- `## LLM 分析：missing-page 評估`
- `## LLM 分析：contradictions 抽查`

### Step 3：印簡短摘要 + git commit

```
✅ /lint 完成
規則層：orphans=N stale=M missing=K
LLM 層：xref 推薦 X 條 / missing-page 評估 Y 篇 / contradictions Z 條
報告：vault_neo/_Insights/_LintReport_YYYY-MM-DD.md

下一步建議：
  - 高優先 query：「<從報告挑 3 條>」
  - 高優先 reconcile：<orphan page>
```

直接 git commit 報告（本地 commit 可逆、`vault_neo/` 在 auto-commit 白名單；push 留給 `/session-handoff`）。

### Step 4：寫 EventLog

```bash
# 自動寫入（由 ruminate.py lint 內建）
echo "## [$(date '+%Y-%m-%d %H:%M')] lint | orphans=N stale=M missing=K [--llm: xref=X gaps=Y contra=Z]" \
  >> vault_neo/_EventLog.md
```

---

## 報告產出後的後續

Lint 報告本身是 Insight；使用者讀完可：
- 對 missing-page 候選 → `/query <主題>`
- 對 orphan → `/reconcile <orphan-path>`（給其加 back-link）或 `/reconcile sweep <folder>`
- 對 contradictions → 手動 review 兩 page，必要時 `/writeback` 或 `/query` 補一篇澄清

---

## 與其他 skill 的關係

| skill | 關係 |
|---|---|
| `/query` | **下游觸發**：lint 的 missing-page 建議直接貼回 `/query` |
| `/reconcile` | **下游觸發**：lint 的 orphan + xref 推薦直接貼回 `/reconcile` |
| `/writeback` | **下游觸發**：對 contradictions / Insights 結晶可走 writeback 融回 |

---

## 相關路徑

| 用途 | 路徑 |
|---|---|
| 設計規範 | `vault_neo/CLAUDE.md`、`vault_neo/_Operations.md` |
| 執行腳本 | `scripts/ruminate.py`（lint 子命令）|
| 報告輸出 | `vault_neo/_Insights/_LintReport_YYYY-MM-DD.md` |
| 事件 log | `vault_neo/_EventLog.md`（type: lint）|

---

## 自動化對接

每週一 22:00 `com.<user>.vault-weekly-maintenance`（launchd）跑
`scripts/vault_weekly_maintenance.sh`，其中 lint 段等效於：

```bash
python3 scripts/md_linter.py --all-neo --auto-fix-safe        # Layer 0
python3 scripts/ruminate.py lint --catalog --xlink --llm      # Layer 1 + 門面 + Layer 2
```

並自產 `_LintDiff_<date>.md`（vs 上週 state file `logs/.lint_prev_summary`）收進 Telegram digest。
手動 `/lint` 適用於週中即時體檢或限定 `--scope`。
