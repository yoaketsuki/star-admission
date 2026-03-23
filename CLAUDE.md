# 繁星推薦錄取標準儀表板 — 專案 Memory

## 專案簡介

**月島（鳳新高中教務處）** 使用，每年分析繁星推薦校系變化和鐘擺效應，給學生和家長參考。

- **GitHub Pages**：https://yoaketsuki.github.io/star-admission/
- **Git Repo**：https://github.com/yoaketsuki/star-admission
- **狀態**：已發布，持續迭代中
- **技術棧**：純前端 HTML + CSS + JavaScript（無框架），Chart.js 4.4.1，Botanical Garden 配色

---

## 資料規模

- 110～115 學年度，12 個 PDF，**17,836 筆**錄取記錄
- 比序資料 **13,359 筆**（75% 覆蓋率）

---

## 檔案結構

### Repo 根目錄（star-admission）

| 檔案 | 說明 |
|------|------|
| `index.html` | GitHub Pages 主頁面（同 `繁星推薦儀表板.html`） |
| `data.json` | 所有校系錄取資料，17,836 筆 |
| `dept_order.json` | 各校科系官方顯示排序 |
| `繁星錄取標準_110~115.csv` | 各學年度原始 CSV |
| `繁星錄取標準_全部.csv` | 合併完整 CSV |

### 本機工作目錄（A14_繁星推薦_分析）

| 路徑 | 說明 |
|------|------|
| `extract_fanxing.py` | PDF 提取腳本（pdfplumber page.chars 座標法） |
| `update_dashboard.py` | 儀表板更新腳本 |
| `update_bisxu_inline.py` | 比序嵌入腳本 |
| `references/110-115繁星錄取標準/` | PDF 來源 |
| `outputs/繁星錄取標準_*.csv` | CSV 輸出 |
| `outputs/繁星推薦儀表板.html` | 本機版儀表板 |
| `outputs/index.html` | GitHub Pages 版（同上） |

---

## data.json 資料結構

```json
{
  "y":  110,           // 學年度
  "s":  "國立臺灣大學",  // 學校名稱
  "c":  "00101",        // 校系代碼
  "n":  "中國文學系",    // 科系名稱
  "q":  6,              // 招生名額
  "a":  6,              // 錄取人數
  "r1": 2,              // 第一輪錄取人數
  "p1": 3,              // 第一輪最低在校排名 %
  "r2": 4,              // 第二輪錄取人數
  "p2": 6,              // 第二輪最低在校排名 %
  "ek1": "頂標",        // 國文檢定
  "ek2": "英文檢定",    // 英文檢定
  "ek3": "數A檢定",     // 數A（若有）
  "ek4": "數B檢定",     // 數B（若有）
  "ek5": "社/自檢定",   // 社會或自然
  "ek6": "英聽",        // 英聽（若有）
  "bx": [               // 比序項目 [[名稱, r1標準, r2標準], ...]
    ["在校學業", "3%", "6%"],
    ["學測英文", null, "15"]
  ]
}
```

---

## 技術要點與坑

| 問題 | 解法 |
|------|------|
| `extract_tables()` 編碼亂碼 | 改用 `page.chars` 逐字座標重建 |
| 校系代碼跨年度會變 | 儀表板用 `name + school` 分組，不用代碼 |
| **data.json 與嵌入資料不同步** | **HTTP 環境優先用 data.json；file:// 用嵌入資料。重新提取後必須同時更新兩者！** |
| 嵌入資料 script 位置 | 必須放在主程式 script **之前**（file:// fallback） |
| CSV 映射遺漏 | JSON 嵌入時 CSV 檢定欄位要映射成 ek1-ek7 |
| Bash 裡 Python 字串 | 不能用 `${}`，改用獨立 `.py` 檔 |
| GitHub Pages CDN 快取 | 網址加 `?v=N` 強制刷新（如 `?v=3`） |

### 比序 PDF 座標
- x=585-645：比序名稱
- x=695-730：第一輪值
- x=775-810：第二輪值

---

## 已完成功能

- PDF 提取（pdfplumber）+ 110-115 學年度全資料
- 學測檢定標準（ek1-ek7）+ 比序項目（bx）
- 官方分則表交叉驗證 18/18 OK
- 互動式儀表板 HTML（Botanical Garden 配色）
- 排版：資料表（含比序）→ 學測檢定表 → 趨勢折線圖
- 校系按官方學系代碼數字排序
- 搜尋：學校簡稱（ALIAS 40+）+ 科系簡稱（DEPT_ALIAS 50+）
- 群組快捷：頂大（5校）、中字輩（4校）、醫學系
- 切換學校/搜尋時自動清除舊分析面板
- 回首頁按鈕（橘色 #f47b20、62px 圓形、SVG 小屋圖示、帶橘暈陰影）、Footer 提示文字

---

## 待辦

- [ ] 醫學系按鈕改為學校分類，只顯示醫學系/牙醫學系
- [ ] 學測檢定標準變化標示（Sunset Boulevard 配色）
- [ ] 校內學生繁星排序功能（規劃中，與錄取標準交叉比對做選填建議）
- [ ] 資料品質報告：每次提取後自動檢查缺失、異常、校系增減
- [ ] 趨勢報告自動生成
- [ ] UI 持續迭代
- [ ] 併校合併已改程式碼，尚未 push（需先 git pull --rebase）

---

## 每年新增學年度流程

1. 新 PDF 放進 `references/`
2. 執行 `extract_fanxing.py`
3. 執行 `update_dashboard.py`
4. Push 到 GitHub

## 發布流程（本機改完每次都要做）

```bash
# 本機改 繁星推薦儀表板.html 後：
cd outputs/
cp 繁星推薦儀表板.html index.html
git add -A
git commit -m "說明"
git push
# 等 1-2 分鐘，GitHub Pages 自動更新
# 若還是舊版，網址加 ?v=N 強制刷新
```

### 異地 Claude（線上版）的協作流程
線上版 Claude 無法直接推 main，會建立 `claude/` 開頭的分支。
本機 Claude 或月島需手動合併：
- **方法 A**：在 GitHub 網頁上開 PR → Merge（已驗證可行，PR #1）
- **方法 B**：本機跑 `git checkout main && git merge claude/xxx && git push`

---

## 更新紀錄

| 日期 | 內容 |
|------|------|
| 2026-03-23 | PR #1 合併：首頁按鈕改版（橘色、62px、SVG 圖示更清晰） |
| 2026-03-23 | 併校資料合併：SCHOOL_MERGE 字典（交大+陽明→陽明交通大學），已改程式碼，尚未 push |
| 2026-03-23 | 規劃中：醫學系學校分類、學測檢定變化標示（Sunset Boulevard）、校內繁星排序 |
