> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。

# vault_neo — LLM Agent 操作規範

本檔規範 LLM agent（任何 provider；非 Claude agent 的入口見專案根 `AGENTS.md`）
將 `vault_neo/` 視為**知識 corpus** 的互動方式。
位階：承接 `<PROJECT_ROOT>/CLAUDE.md`（法律），補充 vault 層的閉迴路知識生產規則。
領域內細節仍以各 `<domain>/*_Evidence_Standards.md`（命令層）為準。

導覽地圖見 [[_INDEX]]。
四操作對照見 [[_Operations]]。

---

## 知識閉迴路四操作

vault 不只是被讀；對 vault 的提問與體檢產出的結果應**回填**為新知識，與爬取來的素材同等公民。
四操作各有獨立 skill 入口（按 LLM Wiki Pattern 三類核心操作 Ingest / Query / Lint 組織）：

| 操作 | skill | 動作 | 觸發 | 寫入 |
|---|---|---|---|---|
| **Ingest** | `/ingest`、`/reconcile` | 新 source → 寫 deep note + 更新涉及既有 page | 使用者 / launchd / Telegram | 新 deep note + 既有 page bump |
| **Query** | `/query`、`/entity` | 向 vault 提問 → 合成 → 回填新 page | 使用者提問 | 新 page（**y/N 確認**後寫入） |
| **Lint** | `/lint` | 對 vault 體檢，產出問題清單 + 後續建議 | 定期 / 使用者觸發 | `vault_neo/_Insights/_LintReport_YYYY-MM-DD.md` |
| **Writeback** | `/writeback` | 把 `_Insights/` 中未處理的洞察融回對應 note | 使用者觸發 / dialogue-catcher 提示 | 直接更新 target note + bump version |

---

### `/query`（提問 → 合成 → 新 page）

**使用情境**：對 vault 提任何形式的問題 —「比較 X 與 Y」、「整理 Z 領域近年共識」、「畫一張 A 的決策樹」。

**檢索策略**（多階段裁剪，不靠 embedding）：
1. **Stage A — 結構過濾**：依 domain、frontmatter `tags`、`type`、wikilink 圖找候選 page
2. **Stage B — 標題/H2 掃描**：只讀 frontmatter + 大綱，淘汰無關
3. **Stage C — full read**：留下 5–15 篇進行完整閱讀
4. **合成**：產出答案，**所有事實宣稱必須附 wikilink 引用來源 page**

**輸出形式**：
- `md`（預設）：markdown page
- `table`：比較表
- `marp`：投影片（寫到 `workspace/marp/`，非 vault）
- `chart`：matplotlib 圖（PNG 寫到對應 note 的 assets/）

**回填規則**：
1. 合成完，LLM **propose**：建議檔名 / `type` / `tags` / `cross_domain` / `derived_from`
2. 印給使用者看，等 `y/N`
3. `y` → 寫入並 `git add`；`N` → 印到 stdout 不寫
4. **寫完必接 `/reconcile <new-page>`**：給來源 page 加 back-link（vault 長新邊）

**新 page 的 frontmatter 必含欄位**：
```yaml
type: note          # query 結果一般為 note；entity aggregator 用 wiki + kind: entity
created_by: query   # /entity 則為 entity
created_date: YYYY-MM-DD
question: "<原始提問>"  # /entity 此欄改為 topic
derived_from:
  - "[[source-page-1]]"
  - "[[source-page-2]]"
```

**禁止**：
- 用訓練資料補 vault 沒覆蓋的內容（沿用憲法「vault 為唯一知識來源」原則）
- 引用未實際讀過的 page（先 read 再 cite）

---

### `/entity`（強制產 entity / concept aggregator page）

`/query` 的特化版。差別：
- `/query`：被動，使用者問什麼答什麼
- `/entity <topic>`：主動，指定主題就強制產跨 source 大全頁

frontmatter 多 `kind: entity`、`sources: [...]`，type 為 `wiki`。
存放位置：`vault_neo/<domain>/01_<Domain>_Notes/<slug>_Entity.md`。

skill 細節見 `.claude/commands/entity.md`。

---

### `/lint`（體檢，不改 vault）

**使用情境**：定期 vault 體檢，找問題與下一步可探索方向。

**兩層**：

**Layer 1（規則層，零 LLM）**：
| 檢查 | 方法 |
|---|---|
| `orphans`        | 圖計算：無 inbound wikilink 的 page |
| `stale`          | 規則：`last_updated` > N 個月，或來源 mtime 比 note 新 |
| `missing-page`   | 文中提及的概念無對應 page（grep candidate）|
| `source-drift`   | 比對 frontmatter `source_hash` vs raw source 檔當前 sha256（獨立腳本 `check_source_drift.py`，週排程） |

**Layer 2（LLM 層，`--llm` 旗標啟用）**：
| 檢查 | 方法 |
|---|---|
| `xref` 推薦      | 對 orphan page spawn T2（本 harness Sonnet）找應加 inbound link 的 target |
| `contradictions` | 對 high-traffic page 抽查事實宣稱衝突 |
| `gaps`           | 對 missing-page 候選評估值得開新 page、建議第一個 query |

**作用域**（`--scope`）：預設整個 `vault_neo/`，可指定子路徑。

**輸出**：寫到 `vault_neo/_Insights/_LintReport_YYYY-MM-DD.md`，每項附「**建議的下一步 query / reconcile / writeback**」。

Lint 報告本身視為 Insight；使用者讀完手動觸發後續操作。

---

### `/writeback`（散→主，融回主線）

**使用情境**：`_Insights/` 中累積了 dialogue-catcher 或 lint 產出的洞察，需融回對應原始 note。

- 掃描 `vault_neo/_Insights/*.md`，找 `status: unprocessed`
- 解析 `extends: [[note-stem]]` 找 target note
- LLM 合成：將洞察融入 target，bump version，更新 `last_updated`
- 已處理的 Insight 移至 `vault_neo/_Insights/Archive/`

預覽：`--dry-run`；單檔：`--file <path>`。

---

### Source Provenance — `source_hash` 漂移偵測

每篇 deep note 若有 `source_md` 指向 raw source（PDF OCR、blog raw、podcast transcript），
frontmatter 應同時帶：

```yaml
source_md: "[[02_<sub>_md/<stem>]]"   # wikilink 形式
source_hash: "<sha256 前 16 字>"        # 寫 note 時 raw 檔的 hash
source_hash_date: YYYY-MM-DD           # hash 寫入日期
```

- **新 deep note**：`build_page_index.py --append` 會自動解析 `source_md`、算 hash、寫回 frontmatter。
- **既有 deep note 補欄**：`scripts/check_source_drift.py --backfill`
- **漂移偵測**：`scripts/check_source_drift.py`（週三 22:30 launchd 自動跑；有 drift 才推 Telegram）
- **報告**：`vault_neo/_Insights/_SourceDrift_<date>.md`（只在有 drift 或 missing 時寫）

drift 出現時的處理建議：
- 比對 raw 變動實質（不只排版）→ `/deepnote <note_path>` 重寫
- 純排版修（marker 升級、字距修補）→ `check_source_drift.py --backfill` 重寫 hash

---

### Raw 反向連結 — raw 檔尾的 deep note 指標

`source_md` 是 deep note → raw 的單向指標；從 raw 讀起時看不出「這篇寫成 note 了沒、寫在哪」。
`scripts/backlink_raw_to_deepnote.py` 掃全 vault deep note 的
`source_md` / `source_md_alt` / `transcript_md`，建反向索引寫進 raw 檔尾：

```markdown
<!-- deepnote-backlink:start -->

---

## 📝 對應 Deep Note

- [[SomeNote|標題]]（本檔為逐字稿 `transcript_md`）

<!-- 自動生成：scripts/backlink_raw_to_deepnote.py｜勿手改，重跑即更新 -->
<!-- deepnote-backlink:end -->
```

規則：

- **只有 sentinel 之間是機器區**，手改必被下次重跑覆蓋；raw 正文本身永遠不動。
- 區塊**直接接在原文末尾、不插入分隔位元組**，故 `split(START)[0]` 位元組級還原原文。
  `source_hash.compute_sha256()` 據此在算 hash 前剝除區塊 —— 尾註不計入漂移偵測，
  加尾註不會讓既有 `source_hash` 失效。同理 `ruminate.py` 算 inbound/outbound 前也剝除，
  自動反向連結不會把 deep note 從 orphan 名單洗白。
- 無對應 deep note 的 raw **不加尾註**（該缺口由 `check_missing_deepnote.py` 追蹤）。
- scraper 重抓 raw 會蓋掉尾註，屬預期；每週維護（`vault_weekly_maintenance.sh` §2c-2）重掃補回。
- 手動重掃 `python3 scripts/backlink_raw_to_deepnote.py`；
  `--check` 只驗陳舊（exit 1）、`--dry-run` 只報告、`--strip` 全庫回退。

---

### `/reconcile`（新→舊，跨頁更新）

**使用情境**：新 deep note 寫完後，把它整合進周圍既有 page。三模式：

| 模式 | 觸發 | 用途 |
|---|---|---|
| `single` | `/reconcile <new-note>` | 對單一新 deep note 提 diff |
| `sweep` | `/reconcile sweep [folder]` | 批次補 wikilinks |
| `pending` | `/reconcile --pending` | 處理 `/ingest` 在靜默場景留下的 `_PendingReconcile_*.md` |

skill 細節見 `.claude/commands/reconcile.md`。

---

## 四操作如何串成閉迴路

```
                       /ingest（含 reconcile）
                              │
                              ↓
                       新 deep note 進 vault
                              │
                              ↓
                          vault 長大
                       ┌──────┼──────┐
                       ↓             ↓
                    /query        /lint
                  (提問)        (體檢)
                       │             │
              新 page 回填       _LintReport.md
                       │             │
                       │      ─→ _Insights/ ←─ /dialogue-catcher
                       │             │
                       │             ↓
                       │       /writeback
                       │       (融回主線)
                       │             │
                       └──→ /reconcile（給新 page / 更新後 page 加 back-link）
```

**核心原則**：每次 ingest 都更新多頁；每次 query 都讓 vault 多一頁；每次 lint 都暴露 vault 弱點並建議下一個 query；每次 writeback 把零散洞察沉澱成主線 note。**探索與維護都會結晶**，不流失到對話歷史中。
