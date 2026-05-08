# 第 2 堂：金融數據基礎
[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/marttychang-del/quant-course/blob/main/session-02/02_stock_data.ipynb)

**目標：** 理解股價數據維度，熟悉常見資料來源，建立資料清理基本功

---

## 課程內容

### 股價維度：價格、成交量、OHLC

- **Open（開盤價）** — 該週期第一筆成交價
- **High（最高價）** — 該週期最高成交價
- **Low（最低價）** — 該週期最低成交價
- **Close（收盤價）** — 該週期最後一筆成交價
- **Volume（成交量）** — 該週期總成交股數

進階：
- **Adj Close（前復權收盤價）** — 扣除除權息影響的調整後價格
- **VWAP（成交量加權平均價）** — 機構建倉成本參考

### 常見數據來源

| 來源 | 優點 | 限制 |
|------|------|------|
| Yahoo Finance | 免費、容易取得 | 前復權資料有時gap |
| 券商 API（shioaji/FinMind） | 台股完整資料 | 需要券商帳戶 |
| Alpha Vantage | 有技術指標 | 免費層 25 calls/day |
| Tiingo | 品質高 | 需要 API key |

### 什麼是「因子」（Factor）

策略的核心邏輯，決定股價行為的驅動因素：

- **價格因子：** 移動平均、動能、均值回歸
- **成交量因子：** 放量/縮量、OBV
- **基本面因子：** EPS、營收成長、毛利率、ROE

### 資料清理基本功

#### 缺失值（Missing Data）

```python
# 檢查缺失值
df.isnull().sum()

# 處理方式：
# 1. 刪除（df.dropna()）
# 2. 前向填補（df.fillna(method='ffill')）
# 3. 插值（df.interpolate()）
```

#### 極端值（Outliers）

```python
# 簡單方式：用 3 倍標準差定義極端值
mean = df['close'].mean()
std = df['close'].std()
outliers = df[(df['close'] > mean + 3*std) | (df['close'] < mean - 3*std)]
```

#### 前後復權

- **前復權（Adj Close）：** 把歷史價格往今天靠攏
- **後復權：** 把今天價格往歷史延伸
- **不復權：** 原始交易價格

> 計算**報酬率**時，強烈建議使用前復權價格，否則除權息前後價格會斷鏈。

---

## 實驗：Python 安裝 + 抓一檔股票收盤價

老師帶操作，一步一步來：

```bash
# 1. 安裝 conda（如果還沒有）
# 下載 Miniconda: https://docs.conda.io/en/latest/miniconda.html

# 2. 建立環境
conda create -n quant python=3.11 -y
conda activate quant

# 3. 安裝套件
pip install yfinance pandas jupyter

# 4. 啟動 notebook
jupyter notebook
```

抓取股票：
```python
import yfinance as yf

# 台股：2330.TW（台積電）
# 美股：AAPL
stock = yf.Ticker("2330.TW")
df = stock.history(period="1y")
print(df.head())
print(df.tail())
```

---

## 課後作業

用 yfinance 抓取自己感興趣的股票（台股或美股皆可）：
1. 取得至少 6 個月的歷史 OHLCV 資料
2. 輸出 `df.isnull().sum()` 確認無缺失
3. 畫出收盤價折線圖（程式碼貼上即可）
4. 計算該股票的**平均日報酬率**和**日波動率**

---

## 重點複習

- **Adj Close（前復權）** 是計算報酬的基礎，不要用不復權價格
- 資料清理不做好，回測結果是垃圾進、垃圾出
- Yahoo Finance + yfinance 是最省事的起點
