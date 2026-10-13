---
summary: vault_neo 知識庫根目錄索引。純導覽用，不含操作規則。操作規則見各 domain 的 Evidence Standards。
type: wiki
updated: 2026-04-17
tags: [index, navigation, agent-reference]
---

> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。

# vault_neo 知識庫索引

此知識庫採 **Domain-first** 組織：依知識所屬領域分類，而非依格式或來源。  
每個 domain 有獨立的 Evidence Standards 文件，定義治理規則、pipeline、命名、frontmatter、檢核。**Agent 進入任一 domain 前，優先讀取該 domain 的 Evidence Standards。**

跨 domain 與全專案規則見 `<PROJECT_ROOT>/CLAUDE.md`（法律層）。

---

## 目錄結構

```
vault_neo/
├── _INDEX.md                  ← 本文件
├── 00_Archive/                ← 01_Projects 完結後的歸檔
├── 01_Projects/               ← 短期進行中的知識專案
│   └── 01_BoardExam/          ← 急診專科國考備考
├── 02_EM/                     ← 急診醫學知識庫
│   ├── EM_Evidence_Standards.md   ← 治理規則
│   ├── EM_EBM_Reference.md        ← EBM 方法論（被筆記引用）
│   ├── 01_EM_Notes/
│   ├── 02_EM_Textbook/
│   ├── 03_EM_Blogs/
│   ├── 04_EM_Study/
│   ├── 05_EM_Case/
│   └── 06_EM_Review/
├── 03_Endurance/              ← 耐力運動知識庫
│   ├── Endurance_Evidence_Standards.md    ← 治理規則
│   ├── Endurance_Sport_Science_Reference.md ← 運動科學方法論
│   ├── 01_Endurance_Notes/
│   ├── 02_Endurance_Textbook/
│   ├── 03_Endurance_Blogs/
│   ├── 04_Endurance_Study/
│   ├── 05_Endurance_Case/
│   └── 06_Endurance_Review/
└── 04_Philosophy/             ← 心法 / 哲學知識庫（跨領域應用）
    ├── Philosophy_Evidence_Standards.md   ← 治理規則
    ├── 04_Philosophy_Notes/                ← 人類心法札記（可選）
    ├── 01_inbox/                           ← domain 級散篇 inbox
    ├── 02_Philosophy_md/                   ← 散篇轉換 MD
    ├── 03_Philosophy_appendix/             ← 散篇圖片 / 附錄
    └── <書名>/                             ← 特定書籍子資料夾（如 The EM Mind）
```

---

## Domain 導覽

### 00_Archive — 完結歸檔

| 用途 | 來源 |
|---|---|
| 01_Projects 完結後移入 | 人工搬移 |

無 Evidence Standards。內容靜態封存。

---

### 01_Projects — 短期知識專案

| 子資料夾 | 主題 | 狀態 |
|---|---|---|
| [[01_Projects/01_BoardExam/\|01_BoardExam]] | 急診專科醫師國考題庫整合 | 進行中 |

主題多元、短期、完結後整體移至 `00_Archive/`。無統一 Evidence Standards，各專案自訂。

---

### 02_EM — 急診醫學

**治理規則**：[[02_EM/EM_Evidence_Standards]]  
**方法論參考**：[[02_EM/EM_EBM_Reference]]

| 子資料夾 | 內容類型 | 證據力 |
|---|---|---|
| [[02_EM/01_EM_Notes/\|01_EM_Notes]] | 人類手寫筆記、片段思考 | 最低 |
| [[02_EM/02_EM_Textbook/\|02_EM_Textbook]] | 急診教科書（如 Tintinalli 9E） | 高（保守） |
| [[02_EM/03_EM_Blogs/\|03_EM_Blogs]] | 急診 blog、評論（EMCrit 等） | 中 |
| [[02_EM/04_EM_Study/\|04_EM_Study]] | 醫學文獻（RCT/cohort/MA）| 高 |
| [[02_EM/05_EM_Case/\|05_EM_Case]] | Case / Image study | 低–中 |
| [[02_EM/06_EM_Review/\|06_EM_Review]] | Review / Guideline | 高（保守） |

**現有 Textbook 子資料夾**：[[02_EM/02_EM_Textbook/Tintinalli 9E/|Tintinalli 9E]]  
**現有 Blog 特定來源**：[[02_EM/03_EM_Blogs/EMCrit/|EMCrit]]、[[02_EM/03_EM_Blogs/EMDxHouse/|EMDxHouse（急症福爾摩斯偵探館，Wayback 存檔）]]

---

### 03_Endurance — 耐力運動

**治理規則**：[[03_Endurance/Endurance_Evidence_Standards]]  
**方法論參考**：[[03_Endurance/Endurance_Sport_Science_Reference]]

| 子資料夾 | 內容類型 | 證據力 |
|---|---|---|
| [[03_Endurance/01_Endurance_Notes/\|01_Endurance_Notes]] | 訓練日誌、比賽回顧、個人反思 | 最低 |
| [[03_Endurance/02_Endurance_Textbook/\|02_Endurance_Textbook]] | 耐力訓練教科書 | 高（保守） |
| [[03_Endurance/03_Endurance_Blogs/\|03_Endurance_Blogs]] | 耐力運動 blog、評論 | 中 |
| [[03_Endurance/04_Endurance_Study/\|04_Endurance_Study]] | 運動科學文獻 | 高 |
| [[03_Endurance/05_Endurance_Case/\|05_Endurance_Case]] | 個案研究 | 低–中 |
| [[03_Endurance/06_Endurance_Review/\|06_Endurance_Review]] | Review / 機構 guideline | 高（保守） |

---

### 04_Philosophy — 心法 / 哲學

**治理規則**：[[04_Philosophy/Philosophy_Evidence_Standards]]

收納人類處理資訊、面對情境的心法 / 思想框架 / 哲學詮釋。內容雖非實證，但會回流影響其他 domain（急診決策、耐力訓練、競技心理）—— 故每篇筆記強制有 `## 跨領域應用｜Cross-Domain Application` 段落。

| 子資料夾 | 內容類型 | 證據力 |
|---|---|---|
| [[04_Philosophy/04_Philosophy_Notes/\|04_Philosophy_Notes]] | 人類心法札記、實踐反思 | 最低 |
| [[04_Philosophy/\|04_Philosophy（根目錄）]] | 散篇 / blog 筆記（不特定來源） | 中 |
| `<書名>/`（如 `The EM Mind/`） | 哲學 / 心法書籍章節筆記 | 中–高 |

**現有書籍子資料夾**：[[04_Philosophy/The EM Mind/|The EM Mind]]（Dan Dworkis, 2021）

---

## Meta / Reference

vault 根層的非 domain 文件（操作層 cheatsheet 與設計原文）：

| 檔案 | 用途 |
|---|---|
| [[_Operations]] | 閉迴路四操作對照（`/reconcile` / `/ruminate query` / `/ruminate lint` / `/ruminate writeback`）— 場景指南與差異對照 |
| [[llm-wiki]] | LLM Wiki Pattern 設計原文（本系統 P0+P1 升級的依據） |
| [[_PageIndex]] | 全 vault deep note master catalog（`/ruminate query` Stage A 主檢索面） |
| [[_EventLog]] | vault 操作 chronological log（append-only） |

---

## 跨 Domain 規則

1. **分類由人類決定**：投入哪個 inbox，就屬於那個 domain
2. **機器標記關聯**：跨領域筆記於 frontmatter 加 `cross_domain: [EM]` 或 `cross_domain: [Endurance]`
3. **Wikilink 交叉連結**：在 See Also 中明確連向另一 domain 的相關筆記
4. **不自動搬移**：機器不因為主題跨領域而搬動檔案

詳細 pipeline、命名、frontmatter 規則見各 domain 的 Evidence Standards。

---

## 未來擴充

新增 domain 時：
1. 在 `vault_neo/` 下建立 `0N_DomainName/`
2. 建立 `DomainName_Evidence_Standards.md`（治理）與 `DomainName_*_Reference.md`（方法論，若適用）
3. 更新本 `_INDEX.md`

候選：
- `05_General_Medicine/`（一般醫學、內科）
- `06_Nutrition/`（獨立於 Endurance 的營養學）
