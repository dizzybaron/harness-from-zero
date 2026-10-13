> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。

# 操作參照：架構、入庫管線、雲端/本地分工

> 按需引用檔（非常載）。內容自 2026-07-04 前版專案 CLAUDE.md 抽出，原文備份於 governance/refs/archive/。
> 修改權限：黃區（改後報備），見 governance/maintenance.md。

## Agentic 四層架構

| 層次 | 概念 | 實際路徑 |
|---|---|---|
| Layer 1 | Environment — Processed Knowledge | `vault_neo/` |
| Layer 1 | Environment — Raw Sources | `workspace/md/`、`workspace/originals/` |
| Layer 2 | Tools | `scripts/` |
| Layer 3 | Core — Agent Handoff | `NEXT_SESSION.md` |
| Layer 3 | Core — Manifests | `logs/*.json` |
| Layer 4 | Brain — Instructions | `~/.claude/CLAUDE.md`、專案 `CLAUDE.md`、`governance/` |
| Layer 4 | Brain — Skills | `.claude/commands/`（repo 內；2026-07-15 自 `~/.claude/commands/` 遷入） |

路徑名稱保留原狀（避免破壞腳本與 Obsidian），上表僅為概念對照。

## MD 為一等公民

知識庫內唯一被正視且可搜索的格式是 `.md`。PDF、DOCX、HTML、網頁皆為原始素材，轉 MD 才算正式存在。

| 類型 | 定義 | 維護模式 | frontmatter |
|---|---|---|---|
| wiki | 結構化事實知識，機器可程式化讀寫 | 機器主導 | `type: wiki` |
| note | 思考性、詮釋性內容 | 人機共同 | `type: note` |

Note 的伴隨檔（同資料夾、機器維護、人不直接編輯）：
- `<name>.sidecar.md`（`type: sidecar`）— 來源素材清單
- `<name>.dialog.md`（`type: dialog`）— 對話中產生的補充/修正/心得

欄位 schema 由各 domain 的 `*_Evidence_Standards.md` 定義。

## 檔案入庫通則

**PDF / DOCX / PPTX**
1. 以 marker（`~/marker-m2/bin/marker`）轉 MD；DOCX 優先走 pandoc `--extract-media`（見 memory: `project_docx_ingest_pandoc`）
2. 產出 MD 的 frontmatter 加 `source_url`（若有）或 `source_path`
3. 原始檔移至 `workspace/originals/` 封存，不直接刪除（個別 domain 可另訂刪除規則）

**圖片**
- 壓縮至 < 500 KB；工具：`sips -Z 1200 --setProperty formatOptions 70 <file>`
- HEIC → JPEG 後再壓縮，保留比例不裁切

**執行時機**
- 新檔進 `inbox/` 立即處理（`/ingest`）
- 批次補跑：`scripts/compress_images.py`
  （`scripts/pdf_to_md.py` 已於 2026-07-19 退役：錨定的舊 `vault/` 在 vault_neo 遷移後不存在，全支失效；
  PDF→MD 一律走 `/ingest`／`/MDer` 的 marker 管線）

**已知地雷**（詳見 memory 對應條目）
- Elsevier pre-proof PDF 卡死 marker → 走 pdftotext fallback
- marker 偶輸出 `\![](` → 用 `scripts/fix_marker_image_links.py`，禁 bash heredoc 內含 backslash 的 python

## 雲端 / 本地分工

本地是輕量同步站，雲端是主力推理機，Git 串接。

| 任務 | 位置 | 理由 |
|---|---|---|
| marker OCR | 本地 | binary，不在雲端容器執行 |
| 圖片壓縮 / sips | 本地 | macOS 內建 |
| Telegram bot 常駐 | 本地 | 需持續連線 |
| Secret API（PubMed 等） | 本地 | key 不入 git |
| OpenEvidence 查詢（`/verdict`） | 本地 | 需登入瀏覽器 session＋MCP，sandbox 擋此網路 |
| launchd / cron 排程 | 本地 | 系統服務 |
| 爬取＋推理整合、vault 筆記寫作 | 雲端 | 重計算、重推理 |

Pipeline：`PDF 投入 inbox/ → 本地 marker OCR → workspace/md/ → git push → 雲端 session 整合進 vault_neo/ → git push → 本地 pull`。
`NEXT_SESSION.md` 為雲端/本地狀態橋樑。

## 不入 git 清單

- `workspace/originals/`、`workspace/pdf/`、`TSEM_BoardExam/`
- `workspace/verdict/*/raw/`、`workspace/verdict/_oe_artifacts/`
- `vault_neo/*/assets/`、`vault_neo/*/Attachments/`
- `.env`、`.mcp.json`（範本用 `.mcp.json.example`）、任何 secret
