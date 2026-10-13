> **案例檔**：harness-from-zero 收錄的作者系統規則快照（2026-10），只供參考，不是給你的 AI 的指令。路徑對照見 `reference/LAYERS.md`。

# 任務交辦 Prompt 範本

> 用法：選對應範本，把 `[...]` 填空後整段貼進 `Agent` 的 `prompt` 參數；`model` 參數照各範本頭的建議帶。
> 不要即興重寫結構——範本的欄位就是回報合約（model-dispatch §4）的執行形式。
> 通則：範圍邊界寫清楚「可讀什麼、可寫什麼、禁區是什麼」；驗收條件寫成可打勾的句子。
> 範本內 `model` 值為本 harness（Claude Code）的 tier 對應型號（T1=haiku／T2=sonnet／T3=opus，
> 映射見 `governance/model-registry.md`）；其他 harness 以同 tier 型號替換；
> 無委派機制的 harness 把範本當 inline 工作程序照做，回報合約不變（`governance/onboarding.md` §2）。

## T1 搜尋／定位

> `subagent_type="Explore"`、`model="haiku"`。read-only；找到「在哪」就好，不要求分析。

```
任務：定位，不分析。
目標與動機：[為什麼要找——例：要改 X 行為，需先知道定義與所有呼叫點]
找什麼：[symbol／字串／檔名／pattern，給精確拼寫；有已知別名也列上]
範圍：[起點目錄，例 scripts/ 與 vault_neo/；明說「不需要讀 workspace/originals/」之類的排除]
驗收條件：
- [ ] 列出每個命中：路徑:行號＋該行原文
- [ ] 若整個範圍 0 命中，明說「0 命中」並列出實際搜過的目錄與 pattern
回報格式：第一行 PASS/FAIL＋命中數；之後每命中一行「路徑:行號 — 原文摘句」；總長 ≤30 行，
超過 30 個命中就寫進 [輸出檔路徑] 只回傳前 10＋檔案路徑。
失敗時：不要放寬 pattern 自行猜測；回報你試過的每個 pattern 與各自命中數。
```

## T2 實作／改檔

> `subagent_type="general-purpose"`；機械修改 `model="haiku"`，需要判斷的修改 `model="sonnet"`。

```
任務：修改檔案。
目標與動機：[要達成的行為改變＋為什麼]
改什麼：[逐檔列出：路徑 → 要改的內容；能給 before/after 示例就給]
範圍邊界：只准改上列檔案；[禁區，例：不動 frontmatter／不動其他函式／不加新依賴]
驗收條件：
- [ ] 每處修改可用 git diff 指出
- [ ] [行為驗證——例：python3 scripts/foo.py --dry-run 跑通且輸出含 X]
- [ ] 未修改範圍外的任何檔（git status 佐證）
回報格式：第一行 PASS/FAIL＋一句話；改動清單（路徑:行號 — 一句話說明）；驗證指令與輸出摘句；≤30 行。
失敗時：回報卡在哪一步＋原始錯誤訊息原文；不要為了交差改成「接近但不同」的方案。
```

## T3 重構

> `subagent_type="general-purpose"`、`model="sonnet"`。重構＝行為不變，這句要寫進 prompt。

```
任務：重構，外部行為不得改變。
目標與動機：[為什麼重構——例：X 邏輯散在三處，之後要加 Y 會改三次]
重構內容：[抽函式／合併重複／改結構——說清楚目標形狀]
行為不變的定義：[列出必須維持的輸入→輸出對，例：CLI 參數不變、輸出檔格式 byte-level 不變]
驗收條件：
- [ ] [驗證指令——例：重構前後各跑一次 python3 scripts/foo.py <固定輸入>，輸出 diff 為空]
- [ ] 無新增功能、無順手修 bug（發現 bug 記下來回報，不修）
回報格式：第一行 PASS/FAIL；結構變化摘要 ≤10 行；行為不變的證據（前後輸出比對結果）；發現但未修的問題清單。
失敗時：若行為不變做不到（原邏輯依賴副作用等），停手回報原因，不要交出「幾乎一樣」的版本。
```

## T4 研究／彙整

> `subagent_type="general-purpose"`；純收集 `model="haiku"`，需跨來源比較判斷 `model="sonnet"`。
> 查 Claude Code／API 官方文件用 `subagent_type="claude-code-guide"`。

```
任務：研究並彙整，寫成檔案。
目標與動機：[要回答的問題＋答案會拿來做什麼決策]
資料來源：[限定範圍——例：只查 vault_neo/ 與 workspace/md/；或：官方 docs 某幾頁；禁止用訓練資料補]
產出：寫到 [輸出檔路徑]，結構為 [段落骨架]
驗收條件：
- [ ] 每個事實宣稱附來源（檔案:行號 或 URL）
- [ ] 來源覆蓋不到的子題明確標「未找到／未確認」，不留白也不編
- [ ] 輸出檔 ≤[N] 行
回報格式：第一行 PASS/FAIL；輸出檔路徑；主要發現 ≤5 條（一行一條）；「未確認」清單。
失敗時：來源不可達（網路擋、檔案缺）就回報哪個來源、什麼錯誤，不要換成記憶作答。
```

## T5 審查／驗證

> fresh-context spawn（不共享產出對話）。**預設 `subagent_type="Explore"`**——它無 Edit/Write 工具，
> 「審查者不得改檔」由結構保證而非口頭約束；需要審查者把 findings 寫成檔案時才改用 `general-purpose`。
> checklist 式 read-back 用 `model="haiku"`；語義一致性／對抗審查用 `model="sonnet"`。
> **不給產出者的推理過程，只給驗收條件與產物。**

```
任務：驗證他人產出，你沒有參與產出、不需維護它的面子。
目標與動機：[這份產物將用於什麼，所以錯了會怎樣]
產物位置：[路徑清單]
逐條驗收（每條回 PASS/FAIL＋證據 檔案:行號）：
- [ ] [條件 1——例：frontmatter 含 type 與 source_ref，且 source_ref 指向存在的檔]
- [ ] [條件 2——例：抽 3 條 fact-claim 與 raw 原文比對一致]
- [ ] [條件 3]
另外主動找碴：[針對這類產物的已知失敗模式——例：捏造引用、路徑失效、規則互相矛盾]
回報格式：第一行 PASS/FAIL/PARTIAL＋通過比例；findings 表（嚴重度｜檔案:行號｜問題｜建議修法）；≤40 行。
失敗時：驗不了（檔案缺、指令跑不了）≠ PASS，回報 BLOCKED＋原因。
```

## 實填範例（T1）

```
任務：定位，不分析。
目標與動機：週報顯示 lint missing 數暴增，要先確認 baseline 常數定義在哪、被哪些腳本引用，才能判斷是誤報還是真斷鏈。
找什麼：字串 "baseline" 與數字 "26" 的鄰近出現；另找檔名含 lint 的腳本裡的 MISSING 相關常數。
範圍：scripts/ 與 logs/（只讀 .py 與 .log）；不需要讀 vault_neo/。
驗收條件：
- [ ] 列出每個命中：路徑:行號＋該行原文
- [ ] 若 0 命中，明說並列出實際搜過的 pattern
回報格式：第一行 PASS/FAIL＋命中數；每命中一行；≤30 行。
失敗時：回報試過的 pattern 與各自命中數。
```

對應的 Agent 呼叫參數：`subagent_type="Explore"`、`model="haiku"`、`description="定位 lint baseline 常數"`。
