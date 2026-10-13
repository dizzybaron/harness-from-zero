> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。
> 文中的刪檔、git commit／push、排程、Telegram 通知與 `scripts/` 呼叫都依賴作者的環境，改寫時不要沿用。

# /reconcile — 跨頁更新（單篇 + sweep 模式）

**觸發方式**：
- `/reconcile <new-note-path>` — **single 模式**：把新 note 整合進 vault 既有 page
- `/reconcile sweep [folder]` — **sweep 模式**：對既有 page 群批次補 wikilinks
- `/reconcile --pending` — 處理 `_Insights/_PendingReconcile_*.md`（靜默 ingest 留下的待批單）

**閉迴路位置**：見 `vault_neo/_Operations.md`（reconcile 是「新→舊」這條路 — 讓既有 page 知道有新 note；sweep 模式則是「對歷史 page 補圖譜密度」）。

承接憲法「知識生產分工」、法律「知識 compounding」第 1 條：
> 每次 ingest 必觸發跨頁更新：新 source 不只新增 deep note，
> 還要更新涉及的 entity / concept / 比較 page。

設計：Python 端做 anchor 抽取 + candidate 收集 + context preview；LLM 端提具體 diff 並自行裁決每個 target 是否套用——跨頁 bookkeeping 屬機器職責（`governance/judgment-rubrics.md` R3），不逐筆拋 y/N 給使用者。

---

## 模式 1：Single（新 note → 周圍既有 page）

### Step 1a：Classify 預覽（categorization first）

```bash
cd <PROJECT_ROOT>
python3 scripts/reconcile.py gather <new-note> --classify [--limit 25]
```

把 candidate 分四桶輸出：

| 桶 | 意義 | 處理路徑 |
|---|---|---|
| `NEW`           | 候選 0；vault 無此主題 | 不需 reconcile；考慮 `/entity <topic>` |
| `EXTEND_LIGHT`  | 1–3 候選 | 直接走 Step 2（提 diff）|
| `EXTEND_HEAVY`  | >3 候選 | 直接走 Step 2（提 diff）|
| `CONFLICT`      | 任一 candidate 命中 CONFLICT bucket（共享 H2 + quant/directive 不一致）| 先走 **Step 1b** 走 [m/r/k/a] 路徑，再進 Step 2 |

CONFLICT 偵測為 heuristic（template H2 如 Clinical Pearls / See Also 已排除）；false positive 可能，**最終由 LLM 判定**。

輸出若出現 `--- SUPPRESSED ---` 區段，表示新 note 中該 wikilink 處於「原文明示不連結」的語境
（如 `本頁**未**連結 [[X]]`），已不計為 outbound、也未併入 catalog keyword pool。
**預設尊重原文的判斷，不要替它建立該連結**；除非讀過上下文後判定為誤抑制，才以一般 outbound 對待。
偵測規則與邊界見 `scripts/test_reconcile_negated_links.py`（`--vault` 可列全庫抑制清單）。

### Step 1b（CONFLICT only）：對立裁決 — [m/r/k/a]

對每個 CONFLICT candidate，**逐筆**呈現給使用者：

```
⚠️ CONFLICT — vault_neo/<target_rel>
   共享 section: ## <h2>
   diff: <quants 對比 / directive 對立>
   new 段落: <new_excerpt 100 字>
   target 段落: <target_excerpt 100 字>

選項：
  [m]erge            — 在 target 加註「新 source 補充：…」，保留兩說
  [r]eplace          — 用 new 取代 target 該段
  [k]eep-old         — target 不動，僅在 See Also 加 back-link
  [a]nnotate-both    — 兩處都加 "⚠️ 與 [[<other>]] 有差異，見 [[<reconcile-note>]]" tag
  [s]kip             — 此 candidate 完全跳過，連 back-link 都不加
```

使用者選 `m/r/a` → 用 Edit tool 套用；`k` → 該 candidate 降級為普通 EXTEND 處理；`s` → 完全跳過。

裁決理由記到 `_EventLog.md`（meta: 一行）：
```
## [YYYY-MM-DD HH:MM] reconcile-conflict | <new-stem> vs [[<target>]] @ ## <h2>: <m/r/k/a>
```

### Step 2：LLM 篩選 + 提 diff

讀 stdout 的 candidate list，**自行**：

1. **挑選**：5–10 個確實該與新 note 互連 / 補完的 target
   - 高優先：outbound wikilink（新 note 已主動指出，target 應加 back-link）
     ⚠️ 被列入 `SUPPRESSED` 的 stem 不屬此類——原文明示不連結，預設略過
   - 中優先：anchor 多重匹配（共享多個 tag/concept）
   - 低優先：單一弱匹配（謹慎，可能 noise）

2. **對每個挑中的 target，提具體 diff**：常見三類
   - **加 back-link**：在 target 的 `## See Also` / `## 相關文章` / `## Cross-References` 段加 `- [[new-note-stem]] — <一句說明關聯>`
   - **補 finding**：target 既有段落提及該主題但未引用新 note 的具體數據/結論，補一句 + wikilink
   - **新增 cross-ref 段**：target 沒有 See Also 段，新增 `## See Also` 段並加首條

3. **逐 target 裁決並印紀錄**（供事後稽核，不停下等使用者）：
   ```
   === target N: vault_neo/<path> ===
   位置：## See Also（既有段）
   diff：
   + - [[<new-note-stem>]] — <一句說明>
   裁決：套用 / skip（理由一句）
   ```
   - 套用 → Edit tool；bump frontmatter `version`、更新 `last_updated`、`last_updated_by: reconcile`
   - 有疑慮 → skip，不拋問
   - 例外（只 propose，等使用者裁決）：刪除或改寫既有段落、改寫主文核心宣稱、跨多 page 的大規模重構

### Step 3：套用後回報

```
=== /reconcile 完成 ===
新 note：[[<stem>]]
觸動 pages：M (套用) / N (提案)
跳過：K
受影響檔案：
  - vault_neo/.../A.md (version 1.0 → 2.0)
  ...
```

### Step 4：Log 事件 + EventLog

```bash
python3 scripts/reconcile.py log <new-note> <touched-page-1> <touched-page-2> ...
echo "## [$(date '+%Y-%m-%d %H:%M')] reconcile | <new-stem> → M pages" >> vault_neo/_EventLog.md
```

寫到 `vault_neo/_EventLog.md`（單一統一 log，oneline header + indented detail bullets）。
（歷史 `_Insights/_ReconcileLog.md` 已歸檔到 `_Insights/Archive/_ReconcileLog_pre-merge_2026-05-03.md`）

### Step 5：Git 提交

```bash
git add vault_neo/
git commit -m "[reconcile] YYYY-MM-DD <new-stem>: 跨頁更新 N pages"
```

push 留給 `/session-handoff`。

---

## 模式 2：Sweep（批次補 wikilinks）

**觸發**：
- `/reconcile sweep` — 掃整個 `vault_neo/`，跨資料夾建關聯
- `/reconcile sweep <folder>` — 僅編輯該資料夾內筆記，連結目標仍跨整個 vault_neo
- `/reconcile sweep <folder> --within` — 僅在資料夾內部建關聯（不跨資料夾）

### 評分（`scripts/reconcile.py sweep` 計算）

四類訊號：同 container、有序 container（有 `book:`）內 position 相鄰、tag Dice（交集 ≥2 才計）、同 entity（`guest`／`first_author`，不含 `host`）；
收斂為相對門檻 `max(3, top × 0.6)`＋cap 10。腳本輸出的 reasons 欄就是這些訊號；推導細節以腳本為準，本檔不重抄。

### Sweep 執行步驟

建索引、評分、提案都由 `scripts/reconcile.py sweep` 產出；LLM 不自己掃檔，拿腳本的 dry-run 提案逐 target 決定是否寫入。

```bash
# 整個 vault
python3 scripts/reconcile.py sweep

# 限定資料夾（目標 = folder，候選池 = 整個 vault）
python3 scripts/reconcile.py sweep vault_neo/02_EM/03_EM_Blogs/EMCrit

# 嚴格 within（目標 = 候選池 = folder）
python3 scripts/reconcile.py sweep vault_neo/02_EM/03_EM_Blogs/EMCrit --within

# JSON 輸出供腳本化
python3 scripts/reconcile.py sweep --json > /tmp/sweep.json

# 調整門檻
python3 scripts/reconcile.py sweep --top 5 --floor 3 --top-ratio 0.6
```

Output 含：scope / target_count / candidate_count / 預計補入連結數 / 跨資料夾條數 / 每 target 提案明細（含 score + reasons：container/Dice/entity）。耗時 vault 全量 ~4s。

腳本輸出後的 LLM 流程：

#### Step 1–3：跑腳本取提案

範圍語義：無參數＝全 `vault_neo/`；`<folder>`＝只編輯該資料夾、候選池為全庫；`<folder> --within`＝兩者皆限該資料夾。
排除規則與評分都在腳本內（見 `scripts/reconcile.py`）；目標數為 0 則結束。

#### Step 4：Dry-run 預覽

```
=== /reconcile sweep 預覽（尚未寫入）===
範圍：<scope_description>
目標筆記：{N} 篇
有候選：{M} 篇
預計補入連結：{K} 條（跨資料夾：{X} 條）

明細（前 10 篇）：
- vault_neo/…/NoteA.md：+ [[NoteX]] (score=4), [[NoteY]] (score=3)
- ...
```

限定 folder 的 sweep 印完直接進 Step 5（只追加 See Also 連結，可逆的 bookkeeping）；無 folder 參數的全庫 sweep 屬大規模變更，先 propose 等使用者 `y/N`。

#### Step 5：執行寫入

對每個目標：
1. Read 整篇
2. 定位 `## 相關文章｜See Also` 段：
   - 存在 → 讀現有 wikilinks，去重後**只追加**新連結
   - 不存在 → 在檔尾或最後一個 `## ` section 之後新增
3. 格式：
   ```
   ## 相關文章｜See Also

   - [[existing_link]] — 原有說明（保留）
   - [[new_link_1]] — 自動補入（container={…}; Dice=0.32; ∩=vertigo,hints-exam,neurology）
   - [[new_link_2]] — 自動補入（entity=Kline）
   ```
4. 更新 frontmatter `updated:` 為今日
5. Edit 寫回

**禁止**：
- 移動或刪除現有 wikilinks
- 改寫現有說明文字
- 變更其他 frontmatter 欄位（除 `updated`）

#### Step 6：寫 EventLog + Git 提交

```bash
echo "## [$(date '+%Y-%m-%d %H:%M')] reconcile | sweep <scope> → {K} links / {N} pages" \
  >> vault_neo/_EventLog.md

git add vault_neo/
git commit -m "[reconcile] YYYY-MM-DD sweep: 補入 {K} 條 wikilinks 至 {N} 篇（跨資料夾 {X} 條）"
```

---

## 模式 3：Pending（處理靜默 ingest 留下的待批單）

**觸發**：`/reconcile --pending`

當 `/ingest` 在靜默場景（launchd / Telegram）跑時，會把 reconcile candidate 寫到 `vault_neo/_Insights/_PendingReconcile_YYYY-MM-DD.md` 而不直接套用。

此模式：
1. 讀最新 `_PendingReconcile_*.md`
2. 對每筆 (new_note, target, diff) 走 single 模式 Step 2 的裁決流程（machine-decides）
3. 處理完歸檔到 `vault_neo/_Insights/Archive/`

---

## 安全規則（所有模式共用）

1. **每個 diff 先印 preview 與裁決再套用**：套用與否由 Claude 判斷（R3）；刪除／改寫既有段落、改寫主文核心宣稱、跨多 page 大規模重構，先問使用者
2. **既有段落只 append，不刪改原文**：除非使用者明確同意 rewrite
3. **frontmatter bump**：每次套用必 bump `version` 並更新 `last_updated`
4. **target 為 textbook 章節**（如 `Tintinalli_Ch*`）：通常只在 See Also 加 wikilink，不改主文（textbook deep note 是 source-of-truth）
5. **連結必須指向存在的 page**：sweep 模式的 candidate basename 在 Step 2 已建索引；single 模式由 reconcile.py 保證

---

## 與其他 skill 的關係

| skill | 關係 |
|---|---|
| `/ingest` | **內嵌呼叫**：ingest Step 6 對所有 PASS 的 deep note 跑 single 模式 propose |
| `/query` | **下游觸發**：query 寫完新 page 後跑 single 模式，給來源 page 加 back-link |
| `/entity` | **下游必接**：entity 寫完跑 single 模式，給所有 source 加 back-link |
| `/lint` | **上游觸發**：lint 報告中 orphan 建議跑 single 模式；xref 推薦走 sweep 模式 |

---

## 手動執行

```bash
cd <PROJECT_ROOT>
python3 scripts/reconcile.py gather <new-note.md>                    # candidate 收集
python3 scripts/reconcile.py gather <new-note.md> --classify         # 分桶（NEW/EXTEND_*/CONFLICT）
python3 scripts/reconcile.py gather <new-note.md> --no-context       # 只列 path/score
python3 scripts/reconcile.py log <new-note.md> <touched-1> <touched-2>  # log 事件
```

---

## 相關路徑

| 用途 | 路徑 |
|---|---|
| 設計規範 | `vault_neo/CLAUDE.md`（compounding 原則第 1 條）、`vault_neo/_Operations.md` |
| 執行腳本 | `scripts/reconcile.py` |
| Log 檔 | `vault_neo/_EventLog.md`（reconcile 與其他事件統一在此；歷史 ReconcileLog 在 `_Insights/Archive/`） |
| Pending 待批單 | `vault_neo/_Insights/_PendingReconcile_YYYY-MM-DD.md` |
| 事件 log | `vault_neo/_EventLog.md`（type: reconcile）|
