# 資料前處理報告 (Data Preparation Report)

> 期中評量 · NLP 自然語言處理 115-S1
> 填寫方式：請把所有「（ ）」與 `___` 替換成你自己的內容，並刪除 `>` 開頭的說明文字。

- **學號 / 姓名**：U12627007 / 江俞廷
- **專案名稱**：NLP 自然語言處理 (115 S1) - 課程專案
- **完成的任務**：- [x] 1-A 頁首頁尾清除　- [x] 1-B 雙語標題合併　- [x] 1-C 雙語條文合併

---

## 1. 資料集概覽 (Dataset Overview)

> 數字來源：`01_Data_Ingestion.ipynb` 第 2 節 (掃描 PDF) 與第 5 節 (批次處理) 的輸出。

| 類別 (Cluster) | PDF 數量 | 清洗前 chunk 數 | 清洗後 chunk 數 |
|---|---|---|---|
| Academic_Affairs | 3 | 96 | 65 |
| Library_Information | 3 | 248 | 225 |
| Student_Affairs | 3 | 66 | 55 |
| **合計** | **9** | **410** | **345** |

**問題 (2–3 句)：** 清洗後 chunk 數量變少了多少？是哪個任務造成的？有沒有哪個任務**不影響** chunk 數量？為什麼？

清洗後 chunk 從 410 個減少到 345 個，少了 65 個（約 15.9%），其中以《大學部學則》最明顯（43 → 21）。因為切塊規則 `\n(?=##\s)` 只在 `## ` 標題處切開，減少主要來自任務 1-B 與 1-C：原本中文標題（如「## 第一條」）和英文標題（如「## Article 1」）各自是一個 `##` 標題、各自成為一個 chunk，合併後標題數變少，chunk 數也跟著變少。任務 1-A 不影響 chunk 數量，因為它只刪除段落中的頁首頁尾文字，沒有新增或刪除 `## ` 標題，所以切塊的位置不會改變。

## 2. 清洗步驟 (Cleaning Steps)

> **截圖來源：** `data/interim/<類別>/` 中的 `raw_*.md` (清洗前) 與 `cleaned_*.md` (清洗後)，也就是批次處理存下的**真實檔案**。
>
> **[重要] 不可使用 notebook 中的範例。** notebook 任務 1-A、1-B、1-C 的測試文字 (第二十一條、明志科技大學 / Ming Chi University of Technology、第十五條 / Article 15) **不能**作為你的截圖。請在真實檔案中**另找一處**，可以是其他 PDF 的檔案。使用範例的截圖**不給分**。
>
> **截圖方法 (VS Code 比較檢視)：**
> 1. 在檔案總管對 `raw_<檔名>.md` 按右鍵 → **Select for Compare**。
> 2. 對同一份文件的 `cleaned_<檔名>.md` 按右鍵 → **Compare with Selected**。
> 3. 左右並排顯示，紅色 = 刪除，綠色 = 新增或修改。按 `Ctrl + F` 搜尋，找到該任務造成的變更。
> 4. 每個任務截**一張**圖，存到 `reports/figures/`，**檔名必須完全相同** (否則圖片不會顯示)：
>    * 任務 1-A → `1a_compare.png`
>    * 任務 1-B → `1b_compare.png`
>    * 任務 1-C → `1c_compare.png`

### 2.1 任務 1-A：頁首頁尾溢出清除

- 檔名：`data/interim/Academic_Affairs/cleaned_A232020018_大學部學則.md`　搜尋的文字：`第五章 轉學`
- 截圖 (存為 `reports/figures/1a_compare.png`)：![1-A](reports/figures/1a_compare.png)
- 改變了什麼：PDF 每一頁上方都印有文件名稱「大學部學則」當頁首，Docling 轉換時把它當成 `## 大學部學則` 標題，夾在內文中間。清洗前（左，raw 第 664～668 行）頁首剛好卡在中文章名 `## 第五章 轉學、轉系（組）` 和英文章名 `## Chapter 5 Transfers and Changes of Major` 之間；清洗後（右，cleaned 第 575 行）頁首被刪除，中英文章名也因此能被 1-B 合併成 `## 第五章 轉學、轉系（組） / Chapter 5 Transfers and Changes of Major`。
- 這對之後的問答檢索有什麼幫助：頁首如果留著，會因為切塊規則 `\n(?=##\s)` 變成一個只有「大學部學則」四個字的空 chunk，檢索時是雜訊；它也會把章名和內文隔開，讓中英文標題無法相鄰而合併失敗。刪除後，章名、英文翻譯和底下的條文能保持在同一個 chunk，檢索到的內容更完整。
- 這個任務有沒有**改錯**的地方 (把不該改的也改了)？若有，寫出檔名與搜尋的文字；若沒有，寫「沒有發現」：有。同一個檔案 `cleaned_A232020018_大學部學則.md` 開頭（搜尋 `Academic Regulations for Undergraduate Studies`），raw 第 5 行的「大學部學則」是整份文件真正的中文標題，不是頁首，但規則只看文字內容是否等於「大學部學則」，所以連真正的標題也一起刪掉，清洗後的檔案開頭只剩英文名稱，找不到中文文件名稱。

### 2.2 任務 1-B：雙語標題合併

- 檔名：`data/interim/Academic_Affairs/cleaned_A232020018_大學部學則.md`　搜尋的文字：`第一篇 總則`
- 截圖 (存為 `reports/figures/1b_compare.png`)：![1-B](reports/figures/1b_compare.png)
- 改變了什麼：清洗前（左，raw 第 224～226 行）中文標題 `## 第一篇 總則` 和英文標題 `## Part 1 General Provisions` 是兩個獨立的 `##` 標題，中間隔著空行；清洗後（右，cleaned 第 212 行）合併成一行 `## 第一篇 總則 / Part 1 General Provisions`。
- 這對之後的問答檢索有什麼幫助：合併前，切塊規則 `\n(?=##\s)` 會把中文標題切成一個只有標題、沒有內文的 chunk，英文標題則帶著內文成為另一個 chunk。合併後中英文標題和內文在同一個 chunk，不論用中文「總則」或英文「General Provisions」搜尋，都能找到完整內容，也減少了沒有內文的空 chunk。
- 這個任務有沒有**改錯**的地方 (把不該改的也改了)？若有，寫出檔名與搜尋的文字；若沒有，寫「沒有發現」：有。同一個檔案 `cleaned_A232020018_大學部學則.md` 開頭（搜尋 `Academic Regulations for Undergraduate Studies`），raw 原本是校名的中英文標題、文件名稱「大學部學則」、文件英文名稱 `## Academic Regulations for Undergraduate Studies` 四行，清洗後三個不同層次的標題被串成同一行 `## 明志科技大學 / Ming Chi University of Technology / Academic Regulations for Undergraduate Studies`，中文文件名稱「大學部學則」被刪掉，行尾還多出一個 `/`。合併規則只看「相鄰的中英文 `##` 標題」，沒有確認它們是不是同一個標題的翻譯，所以把校名和文件名稱誤合併。

### 2.3 任務 1-C：雙語條文合併

- 檔名：`data/interim/Academic_Affairs/cleaned_A232020018_大學部學則.md`　搜尋的文字：`Article 19`
- 截圖 (存為 `reports/figures/1c_compare.png`)：![1-C](reports/figures/1c_compare.png)
- 改變了什麼：清洗前（左，raw 第 360～363 行）中文條文「第十九條 學生經核准得跨部選修課程……」與英文條文「Article 19 Students may take courses offered by other departments……」是兩個獨立的清單項目，「第二十條」與「Article 20」也一樣；清洗後（右，cleaned 第 322～323 行）每一條的中英文被合併成同一個項目，中間用 ` / ` 連接，例如「第十九條 ……另訂之。 / Article 19 Students may take courses……」。
- 這對之後的問答檢索有什麼幫助：合併後同一條規定的中文和英文緊鄰在一起，檢索到其中一種語言時，另一種語言的對應內容也會一起被帶出來，不會發生中文條文和英文條文被分開、各自回答不完整的情況；用英文問也能直接對到正確的中文條號，方便 LLM 引用「第十九條」作答。
- 這個任務有沒有**改錯**的地方 (把不該改的也改了)？若有，寫出檔名與搜尋的文字；若沒有，寫「沒有發現」：有。同一個檔案 `cleaned_A232020018_大學部學則.md`（搜尋 `formula-not-decoded`），raw 第 228～230 行原本是 Docling 無法辨識而留下的佔位符 `<!-- formula-not-decoded -->`，下一個項目才是「Article 1 The Academic Regulations for Undergraduate Studies」；清洗後（cleaned 第 214 行）合併規則把這個佔位符當成中文條文，和 Article 1 接成 `<!-- formula-not-decoded --> / Article 1 ……`。真正的中文「第一條」在轉換時就遺失了，合併後反而讓這個錯誤更不容易被發現。

## 3. 評估指標與 AI 解讀查證 (Metrics & Verifying the AI's Interpretation)

### 3.1 指標數據表

> 數字來源：專案根目錄的 `metrics.json`。[1]、[2] 必填；[3]、[4] 選填。
> 「參考值」由老師在課堂上公布 (完成全部任務的結果)。

| 指標 | Raw | Cleaned (你的) | Cleaned (參考值) |
|---|---|---|---|
| [1] 中位數長度 Median Length | 205.0 | 273.0 | 270.5 |
| [2] 拓撲錨定率 TAR | 1.0 | 1.0 | 1.0 |
| [3] 深度變異量 SDV (選填) | 0.0 | 0.0 | 0.0 |
| [4] 中位數資訊熵 Entropy (選填) | 4.72 | 5.05 | 5.06 |

> **儀表板截圖：** 你自己的 `01_Data_Ingestion.ipynb` **第 6 節**輸出的 4 拼圖儀表板 (第 8 週執行並保留的輸出)。截圖後存為 `reports/figures/dashboard.png`。第 3.2 節也會用到這張截圖。

![dashboard](reports/figures/dashboard.png)

你的 Cleaned 結果與參考值一致嗎？若一致，寫「一致」；若不一致，找出原因。（1 句）

不一致：TAR 與 SDV 相同，但我的中位數長度 273.0 高於參考值 270.5、中位數熵 5.054 略低於 5.062，而且連清洗前的 Raw 熵也不同（我 4.720、參考 4.723），表示差異有一部分在清洗前就存在，可能是 Docling／RapidOCR 轉換 PDF 的結果不完全相同，另一部分則來自我的 1-A～1-C 正則與參考版本不同（例如第 2 節發現的誤刪「大學部學則」標題、誤合併 `formula-not-decoded`），讓部分 chunk 的合併結果和長度不一樣。

### 3.2 AI 的解讀

> 把第 3.1 節的儀表板截圖 (`dashboard.png`) 交給 AI，請它解讀結果。將 AI 的回答**完整貼在這裡**，並註明使用的 AI 工具。

- 使用的 AI 工具：Claude (Anthropic)

```text
這是一份比較清洗前 (Raw) 與清洗後 (Cleaned) chunk 品質的 4 拼圖儀表板：

[1] Chunk 長度分布（對數刻度箱型圖）
清洗後的中位數長度約 273 字元，高於清洗前的約 205 字元，整個箱子也往上移。這表示清洗把原本被切得很碎的小 chunk 合併成較完整的段落，最可能的原因是雙語標題合併 (1-B) 讓中文標題與英文標題不再各自成為一個 chunk。不過兩邊在 10^3 以上都有大量離群值，最長接近 10^4 字元，代表仍有少數極長的 chunk 沒有被切開。

[2] 拓撲錨定率 TAR
Raw 和 Cleaned 都是 1.0，表示 100% 的 chunk 都以 Markdown 標題開頭，切塊都錨定在文件結構上。清洗沒有破壞結構完整性。

[3] 結構深度
所有 chunk 的標題深度都是第 2 層 (##)，沒有第 1 層或第 3 層，所以深度變異量 SDV 為 0。清洗後第 2 層的 chunk 數從約 410 個減少為約 345 個，減少的部分來自標題合併。

[4] 夏農資訊熵
清洗後的中位數熵約 5.05，高於清洗前的約 4.72，表示每個 chunk 的字元多樣性提高，重複的雜訊（如頁首頁尾）變少，資訊密度變高。兩邊在 1～2.5 之間都有低熵離群值，這些很可能是只有標題、或由表格符號組成的 chunk，對檢索幫助不大。

總結：清洗讓 chunk 變得更長、更完整、資訊密度更高，同時維持 100% 的結構錨定。主要剩下的問題是少數超長 chunk 和低熵 chunk，需要進一步處理。
```

### 3.3 你的查證 (至少兩點)

> 挑 AI 說得最肯定的兩句話，回到你的檔案或程式中找證據：它說的是真的嗎？

**查證 1**
- AI 的說法：「清洗後第 2 層的 chunk 數從約 410 個減少為約 345 個，減少的部分來自標題合併。」
- 我查了什麼 (哪個檔案、哪個 chunk、哪段程式)：在 `01_Data_Ingestion.ipynb` 新增一格程式，逐一讀取 `data/processed/` 中 9 份 `raw_*_processed.json` 與 `cleaned_*_processed.json`，計算每份的 chunk 數（見第 1 節表格）；另外 `09_midterm_chunks.ipynb` 也讀到 cleaned 共 345 個 chunk。再對照 `01_Data_Ingestion.ipynb` 的切塊規則 `\n(?=##\s)`。
- 結論：部分正確，因為數字正確（Raw 410、Cleaned 345，減少 65 個），而且切塊規則只在 `## ` 處切開，標題合併確實會讓 chunk 變少。但 AI 只看圖就把原因全歸給「標題合併 (1-B)」，儀表板本身並沒有顯示是哪個任務造成的；1-B 與 1-C 都可能減少 `## ` 標題，要分別關掉某個任務重跑，才能確定各自的貢獻。

**查證 2**
- AI 的說法：「清洗後的中位數熵約 5.05，高於清洗前的約 4.72，表示每個 chunk 的字元多樣性提高，重複的雜訊（如頁首頁尾）變少，資訊密度變高。兩邊在 1～2.5 之間都有低熵離群值，這些很可能是只有標題、或由表格符號組成的 chunk。」
- 我查了什麼：`metrics.json` 的 `Information_Median_Entropy`（Cleaned 5.054、Raw 4.720）；`09_midterm_chunks.ipynb` 的 `chunk_entropy()` 函式；以及第 3.4 節 (b) 被指定的低熵 chunk（`cleaned_A242060013_學生請假作業辦法.md` 第 79 行）。
- 結論：部分正確。數字正確；低熵離群值的推測也正確，我拿到的低熵 chunk（熵 2.83）就是一張目錄表格，內容大多是空白、`-`、`|`。但「熵提高＝雜訊變少、資訊密度變高」說得太肯定：`chunk_entropy()` 是用「字元頻率」計算，chunk 越長、出現的不同字元越多，熵本來就容易變高；而清洗後中位數長度也從 205 增加到 273，所以熵上升有一部分只是因為 chunk 變長，不一定代表內容品質真的變好。

### 3.4 實際證據：你被指定的 chunk

> 執行 `09_midterm_chunks.ipynb`：填入你的學號與老師在課堂上公布的 SEED，按 Run All。
> 每位同學拿到的 chunk 不同。請**連同驗證碼**一起填入。
> 程式只顯示前 400 字元的預覽。請在 VS Code 打開程式列出的**閱讀用檔案** (`data/interim/` 中的 `cleaned_*.md`，與 JSON 內容相同但有正常換行)，按 `Ctrl + G` 輸入程式給的**行號** (或按 `Ctrl + F` 搜尋「搜尋用」文字)，從該行讀到下一個 `## ` 標題為止，就是**完整**的 chunk。

- **SEED**：1106

**(a) 很長的 chunk**
- 資料檔 (JSON)：`data/processed/Student_Affairs/cleaned_A242030012_學生獎懲規定_processed.json`　位置：第 1 個 (從 0 開始數)　長度：4828 字元　驗證碼：80241ff2
- 閱讀用檔案 (`.md`)：`data/interim/Student_Affairs/cleaned_A242030012_學生獎懲規定.md`　從第 51 行開始
- 前 3 行內容：
  ```text
  ## 108.12.12 臺教學(二)字第 1080177185 號函備查

  Approved for recordation by Letter Tai-Jao-Xue-(II) No. 1080177185 dated 2019.12.12
  ```
- 這是什麼樣的文字？為什麼會這麼長？這是《學生獎懲規定》開頭的「修訂沿革」（歷次校務會議修訂日期、教育部備查函號，中英對照），後面接著一個列出各條文標題的目錄表格（如「第二條 適用對象 / Article 2 Applicability」）。切塊規則只在 `## ` 標題處切開，而 Docling 只把第一行備查函號辨識成 `##` 標題，後面的修訂紀錄和整張目錄表格都沒有 `## ` 標題，所以全部被併進同一個 chunk，長達 4828 字元。
- 之後做問答檢索時，這麼長的 chunk 可能造成什麼問題？這個 chunk 混合了修訂日期、函號和目錄表格，主題很雜，轉成向量後語意被稀釋，問「某條文規定什麼」時可能誤把這個只有目錄、沒有內文的 chunk 檢索出來；而且太長可能超過 embedding 模型的長度上限被截斷，放進 LLM 時也會佔用大量 context。

**(b) 資訊熵很低的 chunk**
- 資料檔 (JSON)：`data/processed/Student_Affairs/cleaned_A242060013_學生請假作業辦法_processed.json`　位置：第 3 個 (從 0 開始數)　資訊熵：2.83　驗證碼：9a61c2b9
- 閱讀用檔案 (`.md`)：`data/interim/Student_Affairs/cleaned_A242060013_學生請假作業辦法.md`　從第 79 行開始
- 內容：
  ```text
  ## Table of Contents

  |           |                                                 |   頁次 |
  |-----------|-------------------------------------------------|--------|
  | 第一條    | 目的                                            |      1 |
  | Article 1 | Objective/Purpose                               |        |
  | 第二條    | 適用對象                                        |      1 |
  | Article 2 | Applicability                                   |        |
  | 第三條    | 請假方式                                        |      1 |
  | Article 3 | Method of Application                           |        |
  | 第四條    | 假別                                            |      2 |
  | Article 4 | Types of Leave                                  |        |
  | 第五條    | 請假規定                                        |      6 |
  | Article 5 | Regulations for Leave Application               |        |
  | 第六條    | 請假核決權責                                    |      7 |
  | Article 6 | Responsibility and Authority for Leave Approval |        |
  | 第七條    | 實施與修訂                                      |      8 |
  | Article 7 | Implementation and Amendment                    |        |
  ```
- 為什麼它的資訊熵這麼低？它對問答檢索有用嗎？這個 chunk 是《學生請假作業辦法》的目錄（Table of Contents），被 Docling 轉成 Markdown 表格。資訊熵是用「字元」出現頻率計算的，而表格為了對齊欄位，塞了大量重複的空白、`-` 和 `|`，這幾個字元佔了大部分內容，分布非常集中，所以熵只有 2.83。它對問答檢索幾乎沒有用：只有條號、條文標題和頁次，沒有任何實際規定內容，問「請假要怎麼申請」時可能被檢索出來卻答不出東西，反而擠掉真正有用的條文 chunk。可以在清洗時刪除目錄區塊，或在入庫前過濾掉熵太低的 chunk。

## 4. 一個尚未解決的問題 (One Remaining Problem)

> 打開 `data/interim/` 中任一個 `cleaned_*.md` (與 `data/processed/` 的 JSON 內容相同，但有正常換行，比較好讀)，找出一個清洗後**仍然有問題**的 chunk。
> 每個 chunk 從一個 `## ` 標題開始，到下一個 `## ` 標題之前結束。
> 例如：只有標題沒有內文、表格亂掉、中英文沒有對齊、還有殘留的頁首頁尾、斷行造成的多餘空白，或是**清洗造成的錯誤** (誤合併、誤刪除)。
> **不可**使用第 3.4 節的兩個 chunk。

- **檔名** (`cleaned_*.md`)：`data/interim/Academic_Affairs/cleaned_A232020018_大學部學則.md`　**從第 499 行開始** (VS Code 左側的行號)　**標題**：`## 第四章 請假、休學、復學、退學 / Chapter 4 Excused Absences, Leaves of Absence (Suspension), Resumption of ...`
- **我是怎麼找到的** (例如：搜尋了什麼、看了哪個檔案)：在做第 2 節 1-A 截圖時，用 VS Code 比較檢視對照 `raw_A232020018_大學部學則.md` 與 `cleaned_A232020018_大學部學則.md`，搜尋 `大學部學則` 找頁首，在 raw 第 660～661 行附近發現第五十二條的中文被切成兩行；再打開 cleaned 檔用 `Ctrl + G` 跳到第 572 行確認，清洗後仍然斷開。
- **問題 chunk (貼上部分內容)**：

```text
第五十一條　學生有下列情形之一者，應予開除學籍：

- 一、有本學則第十條情形之一者。
- 二、本學則第五十條第一項第八款勒令退學者。 / Article 51 Students shall be expelled from the University under any of the following circumstances:
1. Students who are involved in one of the circumstances listed in Article 10 of the Academic Regulations.
2. Those who are ordered to withdraw from Article 50, Paragraph 1, Item 8 of the Academic Regulations.
- 第五十二條　學生退學由學校發給轉學或修業證明書，但入學資格未經
- 核准或開除學籍者，不發給任何修業證明文件。 / Article 52 When a student withdraws from school, the school will issue a transfer or study certificate. However, if the admission qualification is not approved or the student is expelled, no certificate of study will be issued.
```

- **問題是什麼**：有兩個問題。(1) 第五十二條的中文在「未經」和「核准」之間被切成兩個清單項目（第 572、573 行），一句話被拆開，前半句意思不完整；1-C 合併時只把後半句「核准或開除學籍者……」和 `Article 52` 接在一起，前半句「第五十二條　學生退學由學校發給轉學或修業證明書」反而沒有對應的英文。(2) 第五十一條的英文 `Article 51 ...` 沒有接在中文條文「第五十一條……應予開除學籍：」後面，而是被接到最後一個子項「二、……勒令退學者。」後面（第 569 行），英文的子項 `1.`、`2.` 也和中文子項分開，中英文對不齊。
- **可能的原因** (PDF 本身？Docling 轉換？清洗的正則表示式？切塊規則？)：(1) PDF 換頁：第五十二條剛好跨頁，Docling 轉換時把下一頁開頭的「核准或開除學籍者……」當成新的清單項目，前面加上 `- `；1-A 刪掉頁首後也沒有把斷開的兩行接回來。(2) 1-C 的正則表示式只看「中文清單項目的下一個項目是不是 `Article N` 開頭」，就把它接到前一個項目後面；它沒有檢查前一個項目是不是以「第 N 條」開頭、條號是否和 `Article N` 相同，所以第五十一條的英文被接到子項「二、」，第五十二條的英文只接到後半句。
- **具體的修正方法** (例如：一條規則、一個正則表示式的構想、長度上限)：(1) 新增「斷句接回」規則：如果一個清單項目不是以句號「。」、冒號「：」結尾，而下一個項目不是以「第…條」、「一、」或 `Article` 開頭，就把兩行合併成一行，例如用 `re.sub(r"([^。：；\n])\n- (?!第[一二三四五六七八九十百]+條|[一二三四五六七八九十]+、|Article)", r"\1", text)`。(2) 1-C 合併時先抓出中文的條號（「第五十一條」→ 51）和英文的 `Article (\d+)`，只有兩邊號碼相同才合併，並把英文接在**整條中文（含所有子項）**的最後，而不是接在前一行。
- **對問答檢索的影響**：如果使用者問「被開除學籍的學生拿得到修業證明嗎？」，檢索可能只抓到前半句「學生退學由學校發給轉學或修業證明書」，LLM 會誤答「可以拿到」，漏掉後面「但入學資格未經核准或開除學籍者，不發給」的例外規定，答案剛好相反。另外，這個 chunk 從第 499 行一路包含第四章所有條文，直到第 574 行才結束，內容很長、主題很多，向量語意被稀釋，也會讓第五十一、五十二條更不容易被精準檢索到。

## 5. AI 協作 (AI Collaboration)

- 本次作業的 AI 協作紀錄請見 [`AI_USAGE.md`](AI_USAGE.md) 的「Week 04 / 08」與「Week 09」段落。
- 這次使用 AI 最大的收穫或教訓是：AI 解讀儀表板時說得很肯定，但不一定正確。例如它說「熵提高表示雜訊變少、資訊密度變高」，可是我回去看 `chunk_entropy()` 才發現熵是用字元頻率計算，chunk 變長熵本來就會變高，所以熵上升不一定代表品質變好；它也把 chunk 減少全部歸給 1-B，但儀表板本身看不出是哪個任務造成的。教訓是 AI 的解讀只能當作假設，要回到程式碼、`metrics.json` 和實際的 chunk 去驗證，尤其要看我自己被指定的 chunk 和 raw／cleaned 的比較檔，才知道清洗真正做了什麼、還有哪裡改錯。
