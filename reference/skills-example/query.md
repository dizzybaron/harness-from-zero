> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。
> 文中的刪檔、git commit／push、排程、Telegram 通知與 `scripts/` 呼叫都依賴作者的環境，改寫時不要沿用。

# /query — 對 vault 提問 → 合成 → 可選回填

**觸發方式**：`/query <自然語言問題>` 或 `/query`（互動詢問）

承接憲法「知識生產分工」與法律「知識 compounding」第 2 條：
> 每次 query 結晶必回填：對 vault 的提問若產出有保存價值的合成，
> 走 propose + y/N 回填路徑寫成新 page。

**閉迴路位置**：見 `vault_neo/_Operations.md`（query 是「問→新節點」這條路）。

---

## 流程（Stage A → B → C → propose → write）

### Step 1：Stage A — 候選收集（Python，零 LLM）

```bash
cd <PROJECT_ROOT>
python3 scripts/ruminate.py query gather "<question>" [--scope <path>] [--limit 30]
```

腳本會：
- 從 question 抽 keyword（英文 token + 中文連續字串 + 2-gram；過濾虛詞）
- 兩路 grep：`_PageIndex.md` 條目（高優先 +2 score）+ 全文（廣覆蓋）
- 合併 score，回傳 top N candidate（含 frontmatter slim + 240 chars body preview）

### Step 2：Stage B / C — LLM ranking + full-read

從 stdout 讀 candidate list，**自行**：

1. 依 question 與 candidate frontmatter+preview 排序，挑 5–15 篇真正相關
2. 對 top N 用 `Read` tool 讀全文（Stage C）
3. 合成答案：**所有事實宣稱必須附 `[[wikilink]]` 引用實際讀過的 page**
   - 禁止引用未讀過的 page（破壞 vault-as-knowledge-source 原則）
   - 禁止用訓練資料補 vault 沒覆蓋的內容（直接說「vault 中尚無」）

### Step 3：先檢查既有 entity page（避免重複新建）

合成完成後、propose 寫檔之前，**必跑**這步：

```bash
# 用合成主題的 keyword 再做一次 catalog grep
python3 scripts/ruminate.py query gather "<合成主題的 1–3 keyword>" --limit 5
```

若回傳清單中有 **明顯同主題的既有 entity page**（`type: wiki` + `kind: entity`），
則 propose 走 **update 模式**（用 `/writeback` 或 `/reconcile` 路徑融回該 page），
而不是新建。否則進 Step 4。

### Step 4：Propose 回填給使用者

格式：
```
=== 提案回填 ===
模式：新建 page  /  更新既有 page [[<existing>]]
建議檔名：vault_neo/<collection>/<slug>.md
建議 type：note  /  wiki (kind: entity)
建議 tags：[...]
derived_from：
  - [[source-1]]
  - [[source-2]]
  ...
----
<完整 page 內容 preview>
----
寫入？(y/N)
```

`y` / 「寫入」/ 「好」 → 進 Step 5；其他 → 印答案到對話、不寫檔。

### Step 5：寫檔 + catalog append + 觸發 reconcile

新 page 的 frontmatter 必含：
```yaml
type: note          # 通常為 note；entity aggregator 用 wiki + kind: entity
created_by: query
created_date: YYYY-MM-DD
question: "<原始 question>"
derived_from:
  - "[[source-1]]"
  - "[[source-2]]"
```

寫完後：

```bash
# catalog 維護
python3 scripts/ruminate.py query write <new-page-path>
```

寫完後**直接接** `/reconcile <new-page-path>` single 模式，給所有 derived_from 的 source page 加 back-link（vault 長新邊）；
per-target 套用由 Claude 裁決，不停下等使用者再下指令。

### Step 6：Git 提交

```bash
git add vault_neo/
git commit -m "[query] YYYY-MM-DD: <slug> — <短描述>"
```

push 留給 `/session-handoff`。

---

## 手動執行

```bash
cd <PROJECT_ROOT>
python3 scripts/ruminate.py query gather "<question>"          # Stage A
python3 scripts/ruminate.py query write <new-page> [--dry-run] # 回填後 catalog 維護
```

---

## 與其他 skill 的關係

| skill | 關係 |
|---|---|
| `/reconcile` | **下游必接**：query 寫完新 page 後跑，補 back-link |
| `/lint` | **上游觸發**：lint 報告中的 missing-page 建議可直接貼回 `/query` |
| `/entity` | **特化版**：`/entity <topic>` 是 query 的「強制產 entity page」變體 |
| `/writeback` | **互補**：query 是新建 page；writeback 是融回既有 page |

---

## 相關路徑

| 用途 | 路徑 |
|---|---|
| 設計規範 | `vault_neo/CLAUDE.md`、`vault_neo/_Operations.md` |
| 執行腳本 | `scripts/ruminate.py`（query 子命令）|
| 事件 log | `vault_neo/_EventLog.md`（type: query）|
