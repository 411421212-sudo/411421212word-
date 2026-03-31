# Pandoc 使用教學

> **Pandoc** 是一款免費、開源的「萬用文件格式轉換器」，可以將 Markdown、HTML、LaTeX、Word (.docx)、PowerPoint (.pptx) 等數十種格式互相轉換。

---

## 目錄

1. [安裝 Pandoc](#1-安裝-pandoc)
2. [基本語法](#2-基本語法)
3. [常用轉換範例](#3-常用轉換範例)
4. [將本專案投影片轉為 .pptx](#4-將本專案投影片轉為-pptx)
5. [進階技巧](#5-進階技巧)
6. [常見問題排除](#6-常見問題排除)

---

## 1. 安裝 Pandoc

### Windows

1. 前往官網下載安裝程式：<https://pandoc.org/installing.html>
2. 執行下載的 `.msi` 安裝檔，依指示完成安裝
3. 開啟「命令提示字元 (cmd)」或「PowerShell」，輸入以下指令確認安裝成功：

```
pandoc --version
```

> 若看到版本號（例如 `pandoc 3.x.x`），代表安裝成功。

### macOS

使用 **Homebrew** 安裝（推薦）：

```bash
brew install pandoc
```

### Linux (Ubuntu / Debian)

```bash
sudo apt-get update
sudo apt-get install pandoc
```

---

## 2. 基本語法

Pandoc 的指令格式如下：

```
pandoc [輸入檔案] -o [輸出檔案]
```

| 部分 | 說明 |
|------|------|
| `pandoc` | 執行 Pandoc 程式 |
| `[輸入檔案]` | 要轉換的來源檔案（例如 `README.md`） |
| `-o` | 指定輸出檔案（**output** 的縮寫） |
| `[輸出檔案]` | 轉換後的目標檔案，副檔名決定輸出格式 |

Pandoc 會根據輸出檔的**副檔名**自動判斷目標格式，非常方便。

---

## 3. 常用轉換範例

### Markdown → PowerPoint (.pptx)

```bash
pandoc 投影片.md -o 投影片.pptx
```

### Markdown → Word (.docx)

```bash
pandoc 投影片.md -o 投影片.docx
```

### Markdown → PDF

> 需額外安裝 LaTeX（例如 [MiKTeX](https://miktex.org/) 或 [TeX Live](https://tug.org/texlive/)）。

```bash
pandoc 投影片.md -o 投影片.pdf
```

### Markdown → HTML 網頁

```bash
pandoc 投影片.md -o 投影片.html
```

### Word (.docx) → Markdown

```bash
pandoc 文件.docx -o 文件.md
```

---

## 4. 將本專案投影片轉為 .pptx

本專案已有一份 Markdown 投影片：`OLED中文顯示投影片.md`。

### 步驟說明

**Step 1**：開啟終端機（命令提示字元 / PowerShell / Terminal），切換到本專案資料夾：

```bash
cd 你的專案資料夾路徑
# 例如 Windows：cd C:\Users\YourName\Downloads\411421212word-
# 例如 macOS/Linux：cd ~/Downloads/411421212word-
```

**Step 2**：執行轉換指令：

```bash
pandoc OLED中文顯示投影片.md -o OLED中文顯示投影片.pptx
```

**Step 3**：轉換完成後，資料夾內會出現 `OLED中文顯示投影片.pptx`，用 Microsoft PowerPoint 或 LibreOffice Impress 開啟即可。

### 使用自訂投影片範本（選用）

若想讓輸出的 .pptx 套用特定的視覺樣式（字型、顏色、背景），可加上 `--reference-doc` 參數：

```bash
pandoc OLED中文顯示投影片.md --reference-doc=我的範本.pptx -o OLED中文顯示投影片.pptx
```

> **提示**：可以用現有的 `Word論文寫作指南.pptx` 作為範本基礎，以保持風格一致。

---

## 5. 進階技巧

### 指定輸入 / 輸出格式（不依賴副檔名）

使用 `-f`（from）和 `-t`（to）明確指定格式：

```bash
pandoc -f markdown -t pptx OLED中文顯示投影片.md -o output.pptx
```

### 輸出帶有目錄的 Word 文件

```bash
pandoc 投影片.md --toc -o 投影片.docx
```

### 轉換時套用 CSS 樣式（HTML 輸出）

```bash
pandoc 投影片.md -o 投影片.html --css=style.css --self-contained
```

### 批次轉換多個 Markdown 檔案

```bash
pandoc *.md -o 合併文件.docx
```

---

## 6. 常見問題排除

### ❌ 問題：執行 `pandoc` 後顯示「找不到指令」

**原因**：Pandoc 未正確加入系統的 PATH 環境變數。

**解決方式**：
- **Windows**：重新開啟命令提示字元後再試；若仍失敗，手動將 Pandoc 安裝路徑（通常為 `C:\Users\YourName\AppData\Local\Pandoc`）加入系統 PATH。
- **macOS/Linux**：執行 `which pandoc` 確認安裝位置，再確認 PATH 設定。

---

### ❌ 問題：轉換 PDF 時出現「pdflatex not found」錯誤

**原因**：缺少 LaTeX 環境。

**解決方式**：
- 安裝 [MiKTeX](https://miktex.org/)（Windows）或執行 `sudo apt-get install texlive-full`（Linux）
- 或改用 `--pdf-engine=wkhtmltopdf` 參數（需另外安裝 wkhtmltopdf）：

```bash
pandoc 投影片.md -o 投影片.pdf --pdf-engine=wkhtmltopdf
```

---

### ❌ 問題：中文字在輸出的 PDF 中顯示為方框或亂碼

**原因**：預設的 LaTeX 字型不支援中文。

**解決方式**：在 Markdown 檔案最上方加入以下 YAML 前置資料，指定支援中文的字型：

```yaml
---
CJKmainfont: "Microsoft YaHei"
---
```

完整指令：

```bash
pandoc 投影片.md -o 投影片.pdf --pdf-engine=xelatex
```

---

## 快速參考卡

```
常用指令速查
─────────────────────────────────────────────────────
轉為 PowerPoint：  pandoc 檔案.md -o 檔案.pptx
轉為 Word：        pandoc 檔案.md -o 檔案.docx
轉為 PDF：         pandoc 檔案.md -o 檔案.pdf
轉為 HTML：        pandoc 檔案.md -o 檔案.html
查看版本：         pandoc --version
查看說明：         pandoc --help
官方文件：         https://pandoc.org/MANUAL.html
─────────────────────────────────────────────────────
```
