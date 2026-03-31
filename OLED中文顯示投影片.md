# 在 MicroPython 環境下控制 OLED 顯示中文字

---

## Slide 1：標題頁

# 在 MicroPython 環境下控制 OLED 顯示中文字

### 原理分析與自訂點陣圖 (Bitmap) 實作方案

**報告人：倍蓉**

---

## Slide 2：MicroPython 預設字型不支援中文的原因

### 底層架構限制

- 內建的 **ssd1306** 驅動模組基於 **framebuf（影格緩衝區）** 處理影像
- 預設的 `framebuf.text()` 僅內建標準的 **8×8 像素 ASCII 字庫**
- 僅支援英數字與基本符號

### 硬體資源考量

| 考量因素 | 說明 |
|----------|------|
| **字模尺寸** | 中文字體複雜，至少需 **16×16 像素**矩陣才能具備基本辨識度 |
| **記憶體佔用** | 常用中文字數千個，若將完整字庫編譯進韌體，會大幅消耗 ESP32 有限的 **Flash 記憶體**與 **RAM** |
| **官方決策** | 因資源限制，官方預設不納入中文字庫 |

---

## Slide 3：顯示中文的常見解決方案

### 方案一：自訂點陣圖（Custom Bitmap / 陣列取模）

- **原理**：「以圖代字」——透過字模軟體（如 **PCtoLCD2002**）將特定中文字轉換為十六進位（Hex）陣列，直接寫入程式碼
- **優點**：極度節省記憶體，實作簡單
- **缺點**：僅適合選單、靜態資訊等**字數有限**的場景

### 方案二：引入外部字型庫（External Font Library）

- **原理**：將中文字型檔（`.fon` 或 `.bin`）燒錄至 ESP32 檔案系統，並在程式中掛載第三方字型渲染函式庫
- **優點**：支援**動態且任意**中文字顯示
- **缺點**：消耗較多硬體資源，前置環境設定繁瑣

---

## Slide 4：本次實作採用方案 — 自訂點陣圖 (Bitmap)

### 選擇原因

> 專案僅需顯示**固定的個人資訊**（姓名、系別、年級），字數有限，自訂點陣圖是最高效的選擇。

### 實作流程

```
步驟 1  →  步驟 2  →  步驟 3
   ↓           ↓           ↓
字模轉換    宣告陣列    區塊傳輸顯示
```

1. **字模轉換**：利用外部軟體將中文字串轉換為對應大小的矩陣圖片資料
2. **宣告陣列**：將資料宣告為位元組陣列（Bytearray）
3. **區塊傳輸**：透過記憶體區塊複製（Block Transfer）將資料繪製到 OLED 螢幕主緩衝區

---

## Slide 5：核心程式碼解析 (Step 1)

### Step 1 — 宣告點陣圖陣列 (Bytearray)

```python
bitmap_grade1 = bytearray([
    0x01, 0x00, 0x00, 0x00, 
    # ... (省略) ...
    0xc0, 0x06, 0x00, 0x00
])
```

### 原理解析

- 此陣列儲存「**大一**」兩個字的像素資料
- 轉換邏輯：

$$\text{黑白像素} \rightarrow \text{二進位（0 為暗，1 為亮）} \rightarrow \text{十六進位（Hex）}$$

- 陣列規格：
  - **寬度**：32 像素（= 4 bytes，每 byte 含 8 bits）
  - **高度**：16 像素

---

## Slide 6：核心程式碼解析 (Step 2)

### Step 2 — 建立獨立的影格緩衝區 (FrameBuffer)

```python
fb = framebuf.FrameBuffer(bitmap_grade1, 32, 16, framebuf.MONO_HLSB)
```

### 原理解析

| 參數 | 說明 |
|------|------|
| `bitmap_grade1` | 來源資料（一維 bytearray） |
| `32` | 影像矩陣**寬度**（像素） |
| `16` | 影像矩陣**高度**（像素） |
| `framebuf.MONO_HLSB` | 資料排列格式（見下表） |

#### `MONO_HLSB` 格式說明

| 縮寫 | 全稱 | 意義 |
|------|------|------|
| **MONO** | Monochrome | 單色影像 |
| **H** | Horizontal | 資料以水平方向排列 |
| **LSB→MSB** | Most Significant Bit first | 最高有效位元（MSB）在前 |

> ⚠️ 此參數**必須**與字模軟體輸出的設定**完全一致**，否則顯示結果將出現錯位或亂碼。

---

## Slide 7：核心程式碼解析 (Step 3)

### Step 3 — 執行區塊傳輸 (Block Transfer) 與顯示

```python
oled.blit(fb, 48, 24)
oled.show()
```

### 原理解析

- **`blit`（Block Transfer）**：電腦圖學基本操作，將一個記憶體區塊的影像資料直接覆蓋到另一個區塊

- **`oled.blit(fb, 48, 24)`**：
  - 將 32×16 中文點陣圖（`fb`）完整複製到 OLED 主顯示緩衝區（`oled`）
  - 圖片左上角錨定在螢幕座標 $(X=48,\ Y=24)$

- **`oled.show()`**：
  - 觸發 ESP32 透過 **I2C 通訊協定**
  - 將主緩衝區資料**一次性**推送到實體 OLED 螢幕
  - 完成最終顯示

---

## Slide 8：總結

### 核心結論

1. **MicroPython 預設環境**因硬體資源限制（記憶體、Flash 大小）**不支援中文**顯示
2. 針對**靜態或少量**的中文顯示需求，**自訂點陣圖（Bitmap）** 是一種高效且節省記憶體的解決方案
3. 掌握以下記憶體操作流程，是控制微控制器顯示模組的**核心底層技巧**：

$$\text{Bytearray} \rightarrow \text{FrameBuffer} \rightarrow \text{blit()}$$

### 技術流程總覽

```
字模軟體         MicroPython 程式碼               實體 OLED
(PCtoLCD2002)
     |                    |                           |
中文字  →  Hex 陣列  →  bytearray  →  FrameBuffer  →  blit()  →  show()  →  顯示
```

---

*本投影片內容可使用 Pandoc 轉換為 .pptx 格式：`pandoc OLED中文顯示投影片.md -o presentation.pptx`*
