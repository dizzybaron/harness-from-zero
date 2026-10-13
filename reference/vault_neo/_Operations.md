---
title: "Vault 閉迴路四操作對照"
type: reference
created_date: 2026-05-03
last_updated: 2026-05-03
tags: [meta, cheatsheet, query, lint, writeback, reconcile, closed-loop]
summary: "/reconcile、/query、/lint、/writeback 四操作的差異對照與場景指南；/ingest 與 /entity 為延伸入口。"
---

> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。

# Vault 閉迴路四操作對照

## 一句話分

| 操作 | 一句話 |
|---|---|
| **`/reconcile`** | 新 note 已寫好 → **更新它周圍的既有 page**（讓既有 page 知道有新 note）；含 `single` / `sweep` / `--pending` 三模式 |
| **`/query`** | 對 vault **提問** → 合成答案 → 可寫成新 page |
| **`/lint`** | 對 vault **體檢** → 找問題清單 → 寫到 LintReport（不改 vault 任何 note） |
| **`/writeback`** | `_Insights/` 累積的零散洞察 → **融回對應原 note**（bump version） |

延伸入口：

- **`/ingest`** — 完整 inbox → vault 管線（MDer → deepnote → reviewer → reconcile → cleanup），內嵌 `/reconcile single`
- **`/entity <topic>`** — `/query` 的特化版，強制產跨 source 的 entity / concept aggregator page

---

## 對照表

| 維度 | `/reconcile` | `/query` | `/lint` | `/writeback` |
|---|---|---|---|---|
| **觸發時機** | 新 note 剛寫完 | 使用者有問題要問 vault | 定期 / 體檢時 | `_Insights/` 有未處理 Insight 時 |
| **輸入** | 一篇新 deep note 路徑（或 `sweep` 整個資料夾、`--pending` 待批單） | 一個自然語言問題 | 整個 vault（或子 scope） | `_Insights/` 中的 Insight 檔 |
| **改誰** | **既有 page**（多個 target 的 See Also / 內文） | 通常**新建**一篇 page | **不改 vault**，只寫報告 | **既有 page**（target note 主文 + bump version） |
| **新 artifact** | 無（純 cross-reference 補強） | 一篇新合成 page（在 vault） | `_LintReport_YYYY-MM-DD.md` | 無（Insight 移至 `_Insights/Archive/`） |
| **方向** | 新 → 舊（讓舊認識新） | 問 → 新（合成補空缺） | 找問題（不修） | 散 → 主（零散沉澱進主線） |
| **互動** | per-target 由 Claude 裁決（R3） | y/N 是否寫新 page | 直接 commit 報告 | y/N 是否執行整批 |
| **Log 寫到** | `_EventLog.md` reconcile 條（含 detail bullets） | `_EventLog.md` query 條 | `_EventLog.md` lint 條 | `_EventLog.md` writeback 條 |

---

## 閉迴路位置

```
                       【/ingest】 新 source → 管線
                              │
                              ↓
                       /deepnote 寫單頁
                              │
                              ↓
                       【/reconcile】 ← 更新既有 page，讓圖譜長新邊
                              │
                              ↓
                          vault 長大
                       ┌──────┼──────┐
                       ↓             ↓
                  【/query】       【/lint】
                  【/entity】    體檢→產 LintReport
              提問→合成 page          │
                       │             │
                       │  /dialogue-catcher → _Insights/
                       │             │
                       ↓             ↓
                            _Insights/ 累積
                                  │
                                  ↓
                         【/writeback】 ← 融回主線 note
```

---

## 場景對照

| 你想做的事                                                                 | 用哪個                                                                             |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| 「inbox/ 有新 PDF / DOCX，跑完整管線到 vault」                                   | **`/ingest`**                                                                   |
| 「我剛寫好 EMCrit399_XXX，去把相關 Tintinalli 章節加上 back-link」                   | **`/reconcile <new-note>`**（single 模式）                                          |
| 「整個資料夾批次補 wikilinks」                                                  | **`/reconcile sweep [folder]`**                                                 |
| 「比較 LBBB 和 STEMI 的處置邏輯」                                               | **`/query`**                                                                    |
| 「我想要一篇 PE & RV failure 大全，把所有 source 整合」                              | **`/entity PE`**（強制 entity aggregator）；之後跑 **`/reconcile`** 給新 page 加 back-link |
| 「vault 裡有哪些 page 沒人引用？哪些概念被提到但沒對應 page？」                              | **`/lint`**                                                                     |
| 「對話中討論出 PE 處置的補充重點，已經被 dialogue-catcher 抓進 _Insights/，把它整合進原 PE note」 | **`/writeback`**                                                                |

---

## 兩組常被混淆的 pair

### `reconcile` vs `query` / `entity`

- 都會「動 vault」，但**動的方向相反**
- `query` / `entity` = **新建** page（vault 長新節點）
- `reconcile` = **更新既有** page（vault 長新邊）
- 完整流程：`query` / `entity` 寫完新 page 後**應接** `reconcile`，給新 page 加 back-link

### `reconcile` vs `writeback`

- 都會「更新既有 page」，但**素材來源不同**
- `reconcile` 素材 = **完整成形的 deep note**（單篇 source-bound）
- `writeback` 素材 = **零散 Insight**（對話結晶 / lint 提案 / 片段補充）
- 動 target 的方式：`reconcile` 多在 See Also 加 wikilink；`writeback` 直接改主文段落並 bump version

### `lint` vs 其他三者

- `lint` 是**唯一不改任何 note** 的操作
- 它只**產報告 + 給建議**，建議形式是「下一個該跑的 query / reconcile / writeback」
- 角色像 vault 的「健檢醫師」：診斷不開刀

---

## 相關文件

- 設計與實作規範：`vault_neo/CLAUDE.md`（vault 規約層）
- LLM Wiki Pattern 原文：[[llm-wiki]]
- skill 檔：`.claude/commands/{ingest,query,entity,lint,writeback,reconcile}.md`（repo 內）
- 統一事件 log：`vault_neo/_EventLog.md`（reconcile / query / lint / writeback / ingest 同寫）
