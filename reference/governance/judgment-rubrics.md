> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。

# 判斷力 Rubric

> 讀者：主對話模型（預設 T3 前沿級；措辭保持 T2 可執行，供委派與降級場景沿用）。
> 每條 rubric：判準 → 正例 → 反例 → checklist。
> 正反例全部取自本專案實際發生過的事件（括號內為 memory 條目名，可 grep MEMORY.md 找到全文）。
> 卡在判斷時讀對應一節即可，不必整檔讀。

## R1 何時升級模型

**判準——符合任一才升級**：
- 任務本質是跨文件合成、新設計、或歧義裁決（不是機械執行）。
- 已按 model-dispatch §5 換過 prompt 重試，仍錯且**錯法不同**（表示能力不足，非指示不清）。
- 產出是 load-bearing：要入 vault 的 fact、對外發布的內容、不可逆操作的依據。

**判準——以下三種是假升級題，升級救不了**：
1. **指示不清**：agent 錯得「很合理」（照字面做了但不是你要的）→ 修 prompt，補目標與動機。
2. **缺資料**：agent 在猜（輸出含 hedging 或泛泛之詞）→ 補檔案路徑、原文、範例。
3. **幻覺**：agent 編造細節 → 接地治：給原文、要求逐項附出處；換大模型一樣會編，只是編得更像真的。

**正例**（該升級）：一份應變計畫草案的主框架——跨四份執行細則的合成＋新設計，
主對話／opus 級任務（journal_2026-07-03）。

**反例**（誤升級）：Haiku reviewer 判 FAIL 但引用了 Evidence Standards 裡不存在的規則。
正解不是換 Sonnet 重審，而是對回規範原文、判定幻覺、維持 PASS（`feedback_haiku_reviewer_fabricates_rules`）。
另一反例：Elsevier pre-proof PDF 卡死 marker——不是升級題，是**換工具**題（pdftotext fallback，
`project_elsevier_preproof_marker_kills`）。

**Checklist**（升級前逐條問）：
- [ ] 失敗軌跡在手（原 prompt＋輸出＋錯在哪）？沒有就先整理，升級必附。
- [ ] 排除了指示不清／缺資料／幻覺三種假升級題？
- [ ] 升完打算怎麼降回來（pattern 化後批次套用）？

## R2 何時算真的完成

**判準——四項全過才叫完成**：
1. 驗收條件逐條打勾，每條指得出 artifact 證據（`檔案:行號` 或指令輸出）。
2. 產物本體驗過：檔案 read-back／程式碼實跑（不是「寫完了」）。
3. 下游登記完成：該進 `_PageIndex`／catalog／manifest 的條目已在。
4. 無靜默 TODO：沒做的部分明說，不藏在「大致完成」裡。

**正例**：一篇 deepnote 的完成＝檔案存在＋frontmatter schema 通過＋`_PageIndex` 有條目＋reviewer PASS——
四項各自能指出證據位置。

**反例**：看 wrapper log 末尾沒有錯誤就回報「已完成／待處理」，與實際 artifact 不符——
曾因此誤報管線狀態（`feedback_pipeline_status_verify_artifact`）。log 是過程，artifact 才是結果。

**Checklist**：
- [ ] 每條驗收條件都有證據位置？
- [ ] 產物用 read-back 或實跑驗過（不是產出者自己說）？
- [ ] catalog／index／manifest 更新了？
- [ ] 回報裡沒有「應該」「大致」「理論上」修飾完成宣告？

## R3 何時停下來問使用者

**必問**（先停再動）：
- 刪檔、force push、`git reset`、`rm -rf`；覆寫**非自己建立**且內容與描述不符的檔案。
- 對外發布（push 到公開處、上傳第三方、發訊息給人）、花錢、動 launchd／cron 排程。
- 內容方向的醫療判斷：某條 fact 該不該進 vault 存疑、兩來源衝突無法由證據等級裁決時。
- 任務範圍實質改變（做著 A 發現其實該做 B）——這是方向決策，屬使用者。
- 升級到 opus 仍失敗、兩輪重試用盡（model-dispatch §5 的終點）。

**不問，機器自主**（問了反而是失職）：
- 跨頁 bookkeeping：reconcile 的 per-target 套用與否由 Claude 判斷（`feedback_reconcile_machine_decides`）。
- 完成待辦後更新 NEXT_SESSION.md，直接做（`feedback_next_session_update`）。
- `/transcript` 候選 0 時自動跑 scraper 補貨（`feedback_transcript_auto_topup`）。
- 格式選擇、讀哪個檔、先做哪個子任務——可逆的執行細節。

**判準一句話**：不可逆、對外、花錢、方向——問；可逆、對內、執行細節——做。
問的時候給選項＋你的建議，不要開放式拋回。

**Checklist**：
- [ ] 這個動作可逆嗎？可逆 → 做。
- [ ] 影響出得了這台機器／這個 repo 嗎？出得了 → 問。
- [ ] 是「怎麼做」還是「該不該做／做什麼」？後者 → 問。

## R4 方向錯誤訊號：換路，不是重試

**先分類錯誤**：
- **暫時性**（transient）：網路 5xx、timeout、rate limit、偶發空回應 → 重試正確。
  正例：Captivate player 偶發 502，加 3-attempt retry 是正解（`project_emcrit_captivate_subfetch_flaky`）。
- **結構性**（structural）：格式不符、邏輯錯、每次都錯在同處、輸出穩定地錯 → 重試無效，要換路。

**結構性錯誤的換路訊號**（任兩項成立即停止重試）：
- 同類錯誤在**不同修補後**第二次出現。
- 修法開始「對抗工具」：調參數、加 workaround 疊 workaround。
- 每一步都在加特例補丁，通則越修越碎。
- 驗證 whack-a-mole：每輪 FAIL 的項目不同，總數不降。

**換路程序**（兩次修補失敗後，第三次動手前）：
1. 寫下「我目前假設了什麼」（工具可用？輸入格式對？任務理解對？）。
2. 列出 ≥1 條替代路（換工具／換資料源／縮小範圍／升級／問人）。
3. 選路再動。不允許第三次原樣重試。

**正例**：marker 被 Elsevier 垂直浮水印 hang 死——換 pdftotext＋strip 散字，一次解決；
若當時繼續調 marker 參數就是錯（`project_elsevier_preproof_marker_kills`）。

**反例**（該重試卻換路＝過度反應）：單次網路 timeout 就放棄整個資料源。先判類，再選擇。

## R5 品質底線怎麼驗

**按產物類型的最低驗證標準**：

| 產物 | 最低標準 |
|---|---|
| MD note（入 vault） | frontmatter 欄位齊＋`source_ref`/`source_md` 指向存在的檔＋wikilink 可解析＋**fact-claim 抽 ≥3 條回 raw 原文比對** |
| 程式碼／腳本 | 實跑一個 happy path＋一個 edge case；改既有腳本另跑一次原有用途不壞 |
| 批次作業（N 件） | 抽樣 max(3, 10%) 件做**全深度**檢查——不是數件數。件數對≠內容對 |
| 報告／回答 | 每個數字與引用可指到 `檔案:行號`；指不到的標「未確認」 |
| 制度／設定改動 | read-back＋一次真實走查（照新規則走一遍最小案例） |

**正例**：refs=0 的 EMCrit 集數，fidelity gate 抽查引用細節回 raw＋transcript 比對，
攔到憑空捏造的「Ioannidis」人名（journal_2026-07-04）——抽樣深查有效。

**反例**：夜間 batch 驗收只數「20 集都有 transcript 檔」，未抽內容——EP140/141 因 Captivate iframe
共用 bug 抓成同一集內容，數量檢查全過、內容是錯的（journal_2026-05-10）。

**Checklist**（驗收任何批次前）：
- [ ] 抽了 max(3, 10%) 件開檔看內容？
- [ ] fact-claim 類回 raw 了？
- [ ] 驗證者是 fresh-context（不是產出者自驗）？

## 教訓紀錄（append-only，一行一條）

- 2026-07-04 立檔。
- 2026-07-15 讀者前提更新：主對話預設 T3（使用者裁決）；rubric 本文與正反例維持不變（歷史事件照原型號記載）。
