---
summary: 02_EM 急診醫學知識庫的治理規則。Agent 處理任何 02_EM 下的檔案前必讀。EBM 方法論參考見 EM_EBM_Reference.md。
type: wiki
updated: 2026-05-04
tags: [evidence-standards, emergency-medicine, agent-reference]
---

> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。
> 文中的刪檔、git commit／push、排程、Telegram 通知與 `scripts/` 呼叫都依賴作者的環境，改寫時不要沿用。

# 02_EM 急診醫學 — Evidence Standards（治理規則）

Agent 讀此文件以取得 02_EM 的**治理規則**。所有在 02_EM 下寫入、修改、搬移檔案的行為，均以本文件為最高準則。

**方法論參考**：EBM 與統計學知識見 [[EM_EBM_Reference]]（筆記引用用，非治理規則）。

---

## Part A：分類定義

### 屬於 02_EM 的內容

- 急診醫學臨床實務相關內容（評估、診斷、處置、程序）
- 急診科常見疾病與症狀的文獻、教科書、guideline
- 急診相關的 case study、image study
- 由急診醫師/急診教育者撰寫、以 ED 為場景的 blog 與評論

### 不屬於 02_EM 的內容

- 純粹 ICU/住院科病房管理（除非在 ED 中啟動）
- 耐力運動生理學（→ `03_Endurance/`）
- 台灣急診專科國考備考資料（→ `01_Projects/01_BoardExam/`）
- 法定傳染病監測（→ 另建分類）

### 跨領域處理原則

人類決定歸入哪個 domain（人類投入哪個 inbox 就屬於哪個 domain）。  
機器遇到跨領域主題時，在筆記 frontmatter 中加入 `cross_domain:` 欄位並於文末以 wikilink 交叉引用。

---

## Part B：子資料夾架構

```
02_EM/
├── EM_Evidence_Standards.md    ← 本治理規則
├── EM_EBM_Reference.md         ← 方法論參考（被筆記引用）
├── 01_EM_Notes/                ← 人類手寫筆記（最低證據力）
├── 02_EM_Textbook/             ← 教科書（每本書有獨立子資料夾）
│   └── <書名>/
│       ├── 01_inbox/           ← 人類放 PDF（處理後刪除）
│       ├── 02_<書名>_md/       ← marker OCR 原始 MD
│       ├── 03_<書名>_appen/    ← marker 截圖 / 附錄圖片
│       └── <書名>_<Chapter>.md ← LLM 整理之章節筆記
├── 03_EM_Blogs/                ← Blog 與評論
│   ├── 01_inbox/               ← 不特定來源（處理後刪除）
│   ├── 02_Blog_md/
│   ├── 03_Blog_appendix/
│   ← 不特定來源筆記放根目錄    ← FirstAuthorYear_Keyword.md
│   └── <特定來源>/             ← 有定期爬取的來源（如 TEMB）
│       ├── 01_<來源>_inbox/
│       ├── 02_<來源>_md/
│       ├── 03_<來源>_appendix/
│       └── <筆記>.md
├── 04_EM_Study/                ← 醫學文獻（RCT/cohort/MA 等）
│   ├── 01_inbox/
│   ├── 02_Journal_md/
│   ├── 03_Journal_appendix/
│   └── <FirstAuthorYear_Keyword.md>
├── 05_EM_Case/                 ← Case / Image study
│   ├── 01_inbox/
│   ├── 02_Journal_md/
│   ├── 03_Journal_appendix/
│   └── <筆記>.md
└── 06_EM_Review/               ← Review article / Guideline
    ├── 01_inbox/
    ├── 02_Journal_md/
    ├── 03_Journal_appendix/
    └── <筆記>.md
```

**01_EM_Notes 無 inbox/md/appendix**：人類直接在資料夾內建立 MD 檔，機器不主動整理，但可作為被引用的知識來源。

**教科書命名例外**：每本書的 MD 子資料夾用書名縮寫命名（如 `02_Tintinalli_md`），不用通用的 `02_md`，因為同一教科書下有多個章節 section 各自對應不同 MD 集合。

---

## Part C：Pipeline

### C1. 教科書（02_EM_Textbook）

```
人類放 PDF → 01_inbox/
  → marker OCR → 02_<書名>_md/（MD）+ 03_<書名>_appen/（圖片）
  → LLM 分析 MD → <書名>_<Chapter>.md（章節筆記，放書籍根目錄）
  → 01_inbox/ 中的原始 PDF 刪除
```

### C2. Blog（03_EM_Blogs）

**不特定來源**（人類手動投遞）：
```
人類放 PDF/TXT/JPG/MD → 01_inbox/
  → marker OCR（若為 PDF）→ 02_Blog_md/
  → LLM 分析 → FirstAuthorYear_Keyword.md（放 03_EM_Blogs/ 根目錄）
  → 01_inbox/ 原始檔刪除
```

**特定來源**（定期自動爬取，如 TEMB）：
```
爬蟲抓取 → <來源>/01_<來源>_inbox/
  → 轉 MD → <來源>/02_<來源>_md/
  → LLM 分析 → <來源>/<筆記>.md
  → inbox 原始檔刪除
```

### C3. 文獻 / Case / Review（04, 05, 06）

```
人類放 PDF → 01_inbox/
  → marker OCR → 02_Journal_md/（MD）+ 03_Journal_appendix/（圖片）
  → LLM 分析 → FirstAuthorYear_Keyword.md（放分類根目錄）
  → 01_inbox/ 原始 PDF 刪除
```

### C4. 手寫筆記（01_EM_Notes）

無自動化 pipeline。人類直接建立 MD 檔。機器不觸碰，除非被明確要求提取知識。

---

## Part D：命名規範

### 文獻筆記（04/05/06）

```
FirstAuthorYear_Keyword1Keyword2.md
```

- `FirstAuthor`：第一作者姓氏（英文），首字大寫
- `Year`：發表年份（4碼）
- `Keyword1Keyword2`：2-3 個主題關鍵詞，駝峰式，無底線
- 範例：`Wang2023_tPAACS.md`、`Smith2021_POCUSPneumonia.md`

### 教科書 Section Note

```
<書名縮寫>_<SectionID>_Note.md
```

- 範例：`Tintinalli_S1_Note.md`

### 教科書 Chapter Note

```
<書名縮寫>_Ch{N}_<Keyword>.md
```

- `{N}`：原書章節整數編號，不補零
- `<Keyword>`：1–2 個英文主題詞，CamelCase，取自原書章節標題
- 範例：`Tintinalli_Ch1_EMS.md`、`Tintinalli_Ch164_HeadacheStroke.md`

### Blog 筆記（不特定來源）

```
FirstAuthorYear_Keyword.md
```

若無作者，用網站縮寫替代：`EMCases2024_AorticDissection.md`

### Blog 筆記（EMCrit 子庫）

EMCrit 有八類命名分支，依集數類型擇一（判定程式：`scripts/emcrit_prefix.py`，本表與程式同步改）：

#### (a) 正規編號集 — `EMCrit{EP}_{Keyword}.md`

- `{EP}`：episode 整數編號，無補零
- `{Keyword}`：1–3 個英文主題詞，CamelCase，可含結尾數字（如系列編號 `MembersQA003`）
- 範例：`EMCrit423_HBOCarbonMonoxide.md`、`EMCrit360_VasopressorUpdate.md`、`EMCrit419_MembersQA003.md`

#### (b) Wee 類（無編號短篇）— `EMCritWee_{Keyword}.md`

- Wee 為 EMCrit 系列中無 episode 整數編號的短篇（含 CV-EMCrit Wee）
- `{Keyword}`：CamelCase，可含年份補上集區隔（如 `AirwayDiscussion2026`）避免同主題不同年份撞名
- frontmatter `episode_number` 填字串 `"Wee"`，不填整數
- 範例：`EMCritWee_DizzinessVertigo.md`、`EMCritWee_EcgInPE.md`、`EMCritWee_ElectricalStormVaEcmo.md`

#### (c) Special 類（專題集）— `EMCritSpecial_{Keyword}.md`

- Special 為 EMCrit 策劃的專題/特別集（Burnout、Trauma Resus、Damage Control 等）
- frontmatter `episode_number` 填字串 `"Special"`
- 範例：`EMCritSpecial_BurnoutLizCrowe.md`、`EMCritSpecial_DamageControlResuscitation.md`

#### (d) RACC-Lit Review 月刊 — `EMCritRACCLit{YYYYMM}_{Keyword}.md`

- `{YYYYMM}`：年月 6 位字串（如 `202604`）
- frontmatter `episode_number` 填字串 `"RACCLit{YYYYMM}"`
- 範例：`EMCritRACCLit202604_AprilLitReview.md`

#### (e) Shadowboxing 類（教學 case 重演）— `EMCritShadowboxing{N}_{Keyword}.md`（無集號者 `EMCritShadowboxing_{Keyword}.md`）

- Shadowboxing 為 EMCrit 持續性子系列，基於 Gary Klein naturalistic decision-making 框架的教學 case 重演（無 episode 整數編號）
- `{N}`：Shadowboxing 系列序號（整數，無補零）
- frontmatter `episode_number` 填字串 `"Shadowboxing{N}"`（如 `"Shadowboxing9"`）
- 範例：`EMCritShadowboxing9_SpinalCordInjury.md`

#### (f) Ghali Grills 類（Sam Ghali 提問子系列）— `EMCritGhaliGrills{N}_{Keyword}.md`

- Ghali Grills 為 Sam Ghali 與 Weingart 對談的提問式子系列（無 episode 整數編號）
- `{N}`：Ghali Grills 系列序號（整數，無補零）
- frontmatter `episode_number` 填字串 `"GhaliGrills{N}"`（如 `"GhaliGrills003"`，3 位補零以對齊系列原生標記）
- 範例：`EMCritGhaliGrills003_DisambiguatingPEA.md`

#### (g) FGEBM 類（A Field Guide to Evidence Based Medicine）— `EMCritFGEBM{N}_{Keyword}.md`

- FGEBM 為 Rory Spiegel 與 Jerry Hoffman 的 limited series（2026-07 起，掛在 emcrit.org/emnerd/ 下，slug 形如 `fgebm001`／`fgebm-002`／`fgebm-3`），逐篇讀近期文獻示範文獻判讀方法（無 episode 整數編號）
- `{N}`：FGEBM 系列序號（整數，無補零），取 title 的 `Episode N`；2026-09 起 title 改縮寫式 `FGEBM 003 – …`，取 `FGEBM` 後數字去補零
- frontmatter `episode_number` 填字串 `"FGEBM{N}"`（如 `"FGEBM1"`）
- host 為 `Rory Spiegel`（非 Weingart），`guest` 填 `Jerry Hoffman`
- 範例：`EMCritFGEBM1_ChlorideEBMMethodology.md`

#### (h) 純文字 blog post 類（EMNerd 等，無音檔）— `EMCritBlog_{Keyword}.md`

- 適用 `logs/emcrit_manifest.json` 該集 `content_type: blog_post`（無 mp3、無 transcript，raw.md 即原文全文）；多為 Rory Spiegel 的 EMNerd 系列與 emcrit.org 上的純文字貼文
- frontmatter `episode_number` 填字串 `"Blog"`；`author` 填原作者（EMNerd 系列為 `Rory Spiegel`），不填 `host`／`guest`
- 必填段落依 Part H5「blog — EMCrit 子庫額外必填」表之 `content_type: blog_post` 適用列（`## 內容摘要` 取代 `## Shownote 摘要`，豁免 `## Podcast 重點時間戳`）
- 範例：`EMCritBlog_CrystalloidPValueThreshold.md`、`EMCritBlog_BayesianPriors2.md`、`EMCritBlog_TTMEvidenceRetreat.md`

#### 共通規則

- `{Keyword}`：1–3 個英文主題詞，CamelCase，無連字號；可含結尾數字（系列編號、年份）
- 存放位置：`03_EM_Blogs/EMCrit/` 根目錄（與 `02_EMCrit_md/` 同層）
- `guest` frontmatter 欄位：無來賓時**整行省略**，不留空字串

### Blog 筆記（TEMB 子庫）

```
TEMB-{NNNN}_{Keyword}.md
```

- `{NNNN}`：TEMB 流水號，4 位補零
- 範例：`TEMB-0001_TraumaResuscitation.md`、`TEMB-0262_SepsisBundle.md`

### OCR 原始 MD

marker 自動產生，保留原始檔名，不改名。

---

## Part E：Frontmatter Schema

### 文獻筆記（04_EM_Study）

```yaml
---
title: <論文標題>
type: note
domain: EM
category: study
study_design: <RCT|cohort|case-control|cross-sectional|meta-analysis|case-series>
first_author: <姓>
year: <YYYY>
journal: <期刊名>
pmid: <PMID>
doi: <DOI>
source_md: "[[02_Journal_md/<檔名>]]"
cross_domain: []           # 若跨領域填入，如 [Endurance]
tags: []
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---
```

### 教科書章節筆記（02_EM_Textbook）

```yaml
---
title: <章節標題>
type: note
domain: EM
category: textbook
book: <書名>
edition: <版次>
section: <Section ID>
source_md: "[[02_<書名>_md/<檔名>]]"
cross_domain: []
tags: []
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---
```

### Blog 筆記（03_EM_Blogs）

**通用欄位**（所有 blog 筆記）：

```yaml
---
title: <文章標題>
type: note
domain: EM
category: blog
source: <TEMB|EMCrit|EMCases|手動|...>
source_url: <URL>
source_md: "[[02_Blog_md/<檔名>]]"     # 一般 blog；子庫用對應 02_<source>_md/
published: <YYYY-MM-DD>                  # 原文發表日；不確定時留空，不猜測
cross_domain: []
tags: []
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---
```

**子庫延伸欄位**：以下子庫**得在通用欄位之上**加入專屬欄位（H3 檢核允許這些延伸欄位存在）：

#### TEMB 延伸（`source: TEMB`）

```yaml
temb_id: "TEMB-NNNN"          # 4 位補零流水號
temb_series: "<系列名>"        # 若屬於某系列主題，否則省略
```

#### EMCrit 延伸（`source: EMCrit`）

EMCrit 為 podcast，三層檔案架構（raw / transcript / deepnote），deepnote frontmatter **必填**：

```yaml
source_md: "[[02_EMCrit_md/<slug>.raw]]"          # 指向 .raw.md（wikilink stem 含 .raw 為合法，見 H4）
transcript_md: "[[02_EMCrit_md/<slug>.transcript]]"  # 指向 .transcript.md
episode_id: <slug>                                 # URL 末段 slug
episode_number: <EP integer>                       # EMCrit 集數編號
host: "<主持人姓名>"
guest: "<來賓姓名與資歷簡述>"                       # 無來賓時省略
note_lang: zh-TW                                   # deepnote 主體語言
```

EMCrit deepnote **選填**（若曾跑過 ASR）：

```yaml
asr_engine: <whisper.cpp|emcrit-native>
asr_model: <ggml-large-v3-turbo|...>               # asr_engine 為 emcrit-native 時省略
asr_duration_sec: <音檔總秒數>
```

> 詳細 deepnote 章節結構規範見 [[03_EM_Blogs/README#EMCrit deepnote 結構規範（L3）|03_EM_Blogs/README]]。

### Review / Guideline（06_EM_Review）

```yaml
---
title: <標題>
type: note
domain: EM
category: review
review_type: <review-article|guideline|consensus-statement>
organization: <發布機構，如 ACEP>
year: <YYYY>
pmid: <PMID 若有>
source_md: "[[02_Journal_md/<檔名>]]"
cross_domain: []
tags: []
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---
```

### 手寫筆記（01_EM_Notes）

Part C4 定義的「人類直接建立 MD 檔」類型，無自動化 pipeline。
適用於 source-bound 思考性筆記（單篇或數篇 page 整合的衍生整理）。
Cross-source 主題彙整改走 entity page（`type: wiki`、`kind: entity`），規範見 `vault_neo/CLAUDE.md`。

```yaml
---
title: <筆記標題>
type: note
domain: EM
category: note
source_ref: "[[<page wikilink>]]"          # 必填；多個來源用 list（- "[[...]]" / - "[[...]]"）
source_ref_external: "<書目資訊>"          # 選填；source 已自 vault 移除時記錄書目，與 source_ref 擇一
source_md: "[[02_*_md/<檔名>]]"            # 選填
tags: []
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---
```

延伸欄位允許（與其他 category 一致）：query 衍生筆記得加 `created_by: query` / `question` / `derived_from`；
boardexam 整理筆記得加 `boardexam_refs: [...]`；reconcile 過得加 `last_updated_by` / `version`。

---

## Part F：Obsidian 連結規範

1. **筆記 → 原始 MD**：frontmatter `source_md` 欄位用 wikilink 指向 `02_*_md/` 的原始文件
2. **文末 See Also**：每篇筆記末尾加 `## 相關文章｜See Also`，列出 3-5 篇最相關筆記的 wikilink
3. **跨領域連結**：`cross_domain` 填入後，在 See Also 中也加入對應 domain 筆記的 wikilink
4. **最少 wikilink 數**：每篇筆記至少 2 個 wikilink 連向已存在的內容
5. **Concept Hub**（若存在）：在 See Also 下方加 `## 概念標籤｜Concepts`，連向對應 concept 頁
6. **方法論引用**：引述 EBM 方法論時用 wikilink 指向 [[EM_EBM_Reference]] 的對應 section

---

## Part H：檢核規則（Checker Agent 參考）

Checker Agent（T1 執行級；本 harness 為 Haiku，tier 映射見 `governance/model-registry.md`）
對每篇生成筆記執行以下檢查，回傳 `PASS` 或 `FAIL: [問題清單]`。

### H1. 檔案存放位置規則

筆記必須存放於下表「允許路徑」，禁止存放於 inbox、md、appen 子目錄。

| category | 允許路徑（相對於 vault_neo/02_EM/） |
|----------|-----------------------------------|
| textbook | `02_EM_Textbook/<書名>/`（書籍根目錄，非子目錄） |
| blog | `03_EM_Blogs/`（根目錄）或 `03_EM_Blogs/<特定來源>/`（根目錄） |
| study | `04_EM_Study/`（根目錄） |
| case | `05_EM_Case/`（根目錄） |
| review | `06_EM_Review/`（根目錄） |
| note | `01_EM_Notes/`（手寫筆記，無 inbox/md/appendix 子目錄） |

`type: wiki` + `kind: entity` 的 entity page 不適用本表，存放規範見 `vault_neo/CLAUDE.md`（`01_<Domain>_Notes/<slug>_Entity.md`）。

禁止存放於：`01_inbox/`、`02_*_md/`、`03_*_appen/` 下的任何路徑。

### H2. 命名格式

對照 Part D 規範：

- textbook：`<書名縮寫>_<SectionID>_Note.md`
- study / case / review：`FirstAuthorYear_Keyword.md`（FirstAuthor 首字大寫，Year 4碼）
- blog（一般）：`FirstAuthorYear_Keyword.md` 或 `網站縮寫Year_Keyword.md`
- blog（EMCrit 子庫）：依 Part D §Blog 筆記（EMCrit 子庫）**八分支** (a)–(h) 擇一，非只有整數編號式。
  正規編號集 `EMCrit{EP}_{Keyword}.md`（EP 整數，無補零）；其餘為 branch 前綴式：
  `EMCritWee_`、`EMCritSpecial_`、`EMCritRACCLit{YYYYMM}_`、`EMCritShadowboxing{N}_`、`EMCritGhaliGrills{N}_`、`EMCritFGEBM{N}_`、`EMCritBlog_`（適用 content_type: blog_post）。
  Keyword 一律 CamelCase。**新子系列出現時擴充 Part D 分支表，不得因不在既有分支就 FAIL 該筆記**。
- blog（TEMB 子庫）：`TEMB-{NNNN}_{Keyword}.md`（NNNN 4 位補零）

### H3. Frontmatter 語法

- `tags` 必須是 **inline array**，不帶引號：`tags: [tag1, tag2]`
  - FAIL 情境：`tags: "tag1"` 或 multi-line block（`- tag1` 形式）
- `type` 必須是 `note`
- `domain` 必須是 `EM`
- 必填欄位（見 Part E 對應 category）不得空缺或保留 `<placeholder>` 原文
- **延伸欄位允許**：Part E 列出之欄位為**最低必填集合**；筆記得加入額外欄位（如 EMCrit 子庫的 `episode_number`、`host`、`guest` 等，見 Part E §Blog 子庫延伸）。Checker **不應**因「多出 Part E 未列欄位」而 FAIL；只在必填欄位缺漏、tags 格式錯誤、或 type/domain/category 值不合時 FAIL

### H4. Obsidian 顯示語法

- Wikilink 格式：`[[檔名]]` 或 `[[檔名|顯示文字]]`，**不帶副檔名 `.md`**
  - **檔名 stem 得包含 dot**（如 `[[02_EMCrit_md/<slug>.raw]]`、`[[02_EMCrit_md/<slug>.transcript]]` 對應實際檔案 `<slug>.raw.md` / `<slug>.transcript.md`）；只要對應檔案實存即合法
  - 含 heading anchor 亦合法：`[[檔名#段落標題]]`
- 禁止：`[[]]`（空 wikilink）、裸 URL 當 wikilink
- 至少 2 個有效 wikilink
- Markdown 表格每行必須以 `|` 起始與結尾
- `---` 只用於 frontmatter 分隔，正文水平線請改用 `***`

### H5. 必要段落

依 category 與 note 類型確認以下 section 標題存在：

**textbook — Section Note**：

| 必要 section 標題 | 備註 |
|-----------------|------|
| `## 核心概念｜Core Concepts` | 必填 |
| `## 相關文章｜See Also` | 必填 |
| `## 章節筆記索引｜Chapter Notes` | 若該 section 下已有任何 chapter note 則必填 |

**textbook — Chapter Note**：

| 必要 section 標題 | 備註 |
|-----------------|------|
| `## 章節定位｜Chapter Context` | 必填 |
| `## 核心概念｜Core Concepts` | 必填 |
| `## 關鍵表格｜Key Tables` | 必填 |
| `## 關鍵圖表與演算法｜Key Figures & Algorithms` | 必填 |
| `## 臨床要點｜Clinical Pearls` | 必填 |
| `## 相關文章｜See Also` | 必填 |

**其他 category**：

| category | 必要 section 標題 |
|----------|-----------------|
| study | `## 研究背景｜Background`、`## 研究設計｜Methods`、`## 關鍵數據｜Key Results`、`## 相關文章｜See Also` |
| blog | `## 核心論點｜Main Argument`、`## 臨床啟示｜Clinical Takeaways`、`## 相關文章｜See Also` |
| case | `## 案例摘要｜Case Summary`、`## 鑑別診斷｜Differential Diagnosis`、`## 相關文章｜See Also` |
| review | `## 主要建議｜Key Recommendations`、`## 相關文章｜See Also` |
| note | `## 相關文章｜See Also`（其他段落結構由作者決定，手寫筆記性質本就靈活） |

`type: wiki` + `kind: entity` 的 entity page 不適用本表，段落結構視主題彈性安排，但仍須含 `## 相關文章｜See Also`（vault graph 完整性）。

**blog — EMCrit 子庫額外必填**（在通用 blog 必填基礎上加入）：

| 必要 section 標題 | 適用 | 備註 |
|-----------------|------|------|
| `## Shownote 摘要` | `content_type: podcast` | 中文化原始 shownote 重點 |
| `## Podcast 重點時間戳` | `content_type: podcast` | 表格形式：時段 / 主題 / 摘要 |
| `## 內容摘要` | `content_type: blog_post` | 依原文章節以 `###` 分節；取代 `## Shownote 摘要` |
| `## 引用文獻｜References` | 兩者皆必填 | 表格形式，每筆對應 04/05/06 子庫之 paper note wikilink |

> `content_type: blog_post`（EMNerd 等純文字貼文）無 mp3 即無時間戳，**明文豁免** `## Podcast 重點時間戳`；
> 補寫時間戳等同杜撰，違反 Part B 防幻覺條款。`content_type` 以 `logs/emcrit_manifest.json` 該集欄位為準，
> 不看標題或檔名推測（2026-09-08 使用者裁決；實作面對應 `.claude/commands/deepnote.md` EMCrit Step 4b）。

### H6. Chapter Note ↔ Section Note 雙向連結

Chapter Note 與 Section Note 之間的雙向連結為獨立檢核項目，PASS 條件：

**Chapter Note 端（逐篇檢核）**：
- `## 相關文章｜See Also` 的**第一條**必須是 `[[Tintinalli_S{N}_Note]]`（N 與 frontmatter `section` 欄位一致）
- frontmatter 必須有 `chapter:` 欄位（整數，無引號）

**Section Note 端（有 chapter note 存在時檢核）**：
- 若存在 chapter note 其 frontmatter `section` 欄位與本 Section Note 相符，則本 Section Note 必須有 `## 章節筆記索引｜Chapter Notes` 段落
- 判定 scope 以 **frontmatter `section` 欄位相符** 為準，非以資料夾共存為準（同書資料夾會同時存有多個 section 的 chapter note）
- 該段落中每個 section 相符的 chapter note 必須以 wikilink 列出（`[[Tintinalli_Ch{N}_{Keyword}]]`）
- 禁止列出**不存在**的 chapter note wikilink（空連結）

---

## Part I：跨 source 整合的時序與證據品質取捨

當一篇 entity / aggregator / cross-source page 同時引用多個 source 對「同一處置 / 同一臨床決策」提出建議，**不得平鋪並列**。必須依下列原則取捨並在內文標明層級。

### I1. 證據品質加權預設順序

由高至低（GRADE 對映見 [[EM_EBM_Reference#G5.3 GRADE System]]）：

1. **多中心 RCT、SR/MA**（GRADE A–B）
2. **多學會共識 / guideline**（背書層級越廣權重越高，如 ACEP+ACCP+ACC 聯合 > 單一學會）
3. **單一 expert update**（podcast / blog / 個別專家更新文）
4. **教科書段落**（依出版年計權；新版優於舊版，但教科書整體晚於即時 RCT）
5. **個人意見 / case-based 法律觀點 / 註釋型評論**

> 此順序為**預設**；當高品質證據族群極小（如 ≥75 歲 PE）、外推性差、或 RCT 設計缺陷顯著（open-label 偏差、停試驗早期），可降權，並在內文短註說明。

### I2. 時序原則

- 較新 RCT / 共識**直接打到**舊 expert update 的同一處置建議時，舊建議必須被標記為其中之一：
  - **「已被新證據相對取代」**（subsumed）
  - **「限定於特定情境」**（scope-narrowed，如「無機械取栓 / ECMO 場域才採用」）
  - **「歷史脈絡保留」**（historical anchor，如 PEITHO 2014 → HI-PEITHO 2026）
- **不得只列新 source 而刪除舊 source**：知識演進線是 vault 資產（pre-trial → trial → post-trial 立場變化是有教育價值的）。
- 時序判定**以證據發表年**為準，非以筆記入庫日為準。

### I3. 衝突處理 — 在 entity 內的呈現

- 設「**主推 / 備選 / 已取代或限情境**」三層；同層內可並列。
- 主推與備選之間以「**依據：`[[trial-page]]` (RCT) > `[[older-podcast]]` (expert update)**」短註明示加權邏輯。
- 「已取代或限情境」段落必須註明被誰取代（wikilink to 較新 source）與取代範圍（全替換 / 部分情境）。

### I4. 對單一 entity / cross-source page 的最低必填

當一篇 page 整合 **≥3 個 source 且涵蓋同主題的處置建議**時，必填：

- **時序註**：主處置段落末尾以 1–2 句說明證據演進（例：「Kline 2025 的 X 是 pre-Y trial 立場；Y trial 2026 後主流轉向 Z」）。
- **限制 / Pitfalls 段**：對主推 RCT 的外推性弱點明示（族群限制、open-label、無 head-to-head 等），不得只列正面結果。
- **Gaps 段保留 trial-in-progress** 名單（如 PE-TRACT / PEERLESS-II / PEITHO-3）— 因進行中 trial 結果落地時會直接觸發本 entity 重整。

### I5. 與 H 系列檢核的關係

Part H 為 reviewer agent 的格式 / 命名 / wikilink 檢核；Part I 為**內容整合品質**規範，由 `/reconcile`、`/writeback`、`/entity` skill 實施，不在 reviewer 自動檢核範圍。LLM 層 lint（`/lint --llm` 之 contradictions 檢查）為事後抽查路徑。

### I6. 適用範圍

本 Part 規範 02_EM 下的：

- entity page（`type: wiki` + `kind: entity`）
- query 衍生 cross-source 整合 page（`created_by: query` 且 `derived_from` 含 ≥3 篇 source）
- 任一 deep note 在「相關文章 / Pearls」段內陳述跨 source 處置取捨時

不規範：

- 單一 source 的 deep note（study / blog / textbook chapter）— source 自身的觀點忠實轉述為主
- 純粹結構性聚合（如 `_PageIndex`、README）

---

## Sources

- Governance rules: vault_neo architecture design, 2026-04-16（2026-04-17 拆分 EBM 方法論至 [[EM_EBM_Reference]]）
- 2026-04-20：加入 EMCrit 子庫支援——Part E §Blog 加入子庫延伸欄位 schema、Part D/H2 加入 EMCrit/TEMB 命名規則、H3 明確允許延伸欄位、H4 釐清 wikilink stem 可含 dot、H5 加入 EMCrit deepnote 額外必填段落
- 2026-04-29：Part D §EMCrit 子庫由四分支擴為五分支，新增 (e) Shadowboxing 類（`EMCritShadow{N}_{Keyword}.md`，`episode_number: "Shadow{N}"`）——對應 EMCritShadow9_SpinalCordInjury 進入 vault 後 reviewer 失敗修補
- 2026-05-04：新增 **Part I 跨 source 整合的時序與證據品質取捨**——對應 PE_RVFailure_Entity 整合 EMCrit424 + HI-PEITHO 原文時暴露的治理層缺口（entity 平鋪 Kline 2025 與 Weingart 2026 + RCT 證據，未依時序 / GRADE 取捨）
- 2026-04-29：Part D §EMCrit 子庫再擴為六分支，新增 (f) Ghali Grills 類（`EMCritGhaliGrills{N}_{Keyword}.md`，`episode_number: "GhaliGrills{N}"`）——對應 further-disambiguating-pea（Ghali Grills 003）進入 deepnote
- 2026-08-01：Part D §EMCrit 子庫擴為七分支，新增 (g) FGEBM 類（`EMCritFGEBM{N}_{Keyword}.md`，`episode_number: "FGEBM{N}"`，host 為 Rory Spiegel）——對應 fgebm001 入庫；同時修正 **H2 條文**：原文只列 (a) 整數編號式，Part D 歷次擴充未同步，導致 reviewer 對所有 branch 前綴式筆記（Wee／Shadow／GhaliGrills／FGEBM）都會誤判 H2 FAIL，現改為指向 Part D 分支表並明訂「新子系列擴充分支表，不得因不在既有分支就 FAIL」
- 2026-09-23：Part D (e)／H2 前綴由 `EMCritShadow{N}` 改為 `EMCritShadowboxing{N}`（`episode_number: "Shadowboxing{N}"`），對齊 writer 與庫內 6 個既有檔；EMCrit 分支判定改由 `scripts/emcrit_prefix.py` 執行，本表與程式同步改（prompt-audit F14／F19；備份 `governance/refs/archive/EM_Evidence_Standards.2026-09-23.bak.md`）
