# 第 3 堂：基礎統計與金融時間序列

**目標：** 建立統計直覺，理解均值回歸 vs 趨勢動能的核心邏輯

---

## 課程內容

### 基礎統計

```python
import numpy as np

# 均值（Mean）
mean = np.mean(returns)

# 標準差（Standard Deviation）— 衡量波動幅度
std = np.std(returns)

# 相關係數（Correlation）— 兩檔股票連動程度，-1 到 +1
corr = np.corrcoef(stock_a_returns, stock_b_returns)[0, 1]

# Covariance（共變異數）
cov = np.cov(stock_a_returns, stock_b_returns)[0, 1]
```

### 常態分佈與金融數據的肥尾

金融市場的報酬分布通常：
- **峰值（Leptokurtic）：** 中央峰值比常態分佈高
- **肥尾（Fat Tail）：** 極端事件的發生機率遠高於常態分佈預期

> 3 個標準差事件，在常態分佈下約 99.7% 不會發生，但金融市場大約每 3-5 年就會遇到一次。

**實務含義：** 以常態分佈算 VaR（風險值）會嚴重低估極端損失。

### 移動平均

```python
# 簡單移動平均（SMA）
df['MA20'] = df['close'].rolling(window=20).mean()
df['MA60'] = df['close'].rolling(window=60).mean()

# 指數移動平均（EMA）— 近期價格權重更高
df['EMA20'] = df['close'].ewm(span=20).mean()
```

### 波動率（Historical Volatility, HV）

```python
# 日波動率（ annualized = 日波動率 * sqrt(252) ）
log_returns = np.log(df['close'] / df['close'].shift(1))
daily_vol = log_returns.std()
annualized_vol = daily_vol * np.sqrt(252)  # 252 交易日
print(f"年度波動率: {annualized_vol:.2%}")
```

### 均值回歸 vs 趨勢動能

| 策略類型 | 核心假設 | 適用市場 |
|---------|---------|---------|
| **均值回歸** | 價格會回到長期均值 | 區間震盪、價值股 |
| **趨勢動能** | 趨勢會延續 | 趨勢明顯的多頭/空頭市場 |

> 兩種邏輯都是對的，問題是**市場什麼時候切換狀態**——這也是量化策略會衰減的原因。

---

## 實驗：計算自己感興趣的股票 N 日 HV

```python
import yfinance as yf
import numpy as np

ticker = "2330.TW"  # 換成你想看的股票
df = yf.Ticker(ticker).history(period="1y")

log_returns = np.log(df['Close'] / df['Close'].shift(1)).dropna()

for n in [20, 60, 120]:
    hv = log_returns.rolling(window=n).std() * np.sqrt(252)
    print(f"{n}日HV（最新）: {hv.iloc[-1]:.2%}")
```

觀察：
- HV 高（>30%）代表近期波動大
- HV 低（<15%）代表價格相對穩定
- HV 的**趨勢**比單一數字更有意義

---

## 課後作業

選擇 2-3 檔自己感興趣的股票：
1. 計算各自 60 日 HV（年度化）
2. 畫出 HV 走勢圖（程式碼截圖）
3. 觀察：不同股票的 HV 是否有連動？還是各走各的？

---

## 重點複習

- **均值和標準差**是理解所有金融統計的起點
- 金融數據**不是**常態分佈，肥尾風險是真实存在的
- **HV 是衡量風險的核心指標**，比 Beta 更直覺
- 均值回歸和趨勢動能都是有效的，關鍵是**判斷市場處於哪種狀態**
