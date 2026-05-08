# 第 4 堂：策略骨架——均線與動能策略

**目標：** 完成一套完整的「想法 → 寫策略 → 回測 → 看績效」流程

---

## 課程內容

### MA Cross 策略（均線交叉）

```python
# 基本邏輯
# 快線（MA5）上穿慢線（MA20）→ 買進
# 快線（MA5）下穿慢線（MA20）→ 賣出

df['MA_fast'] = df['close'].rolling(5).mean()
df['MA_slow'] = df['close'].rolling(20).mean()

df['signal'] = 0
df.loc[df['MA_fast'] > df['MA_slow'], 'signal'] = 1   # 買
df.loc[df['MA_fast'] < df['MA_slow'], 'signal'] = -1  # 賣
```

### RSI 動能策略

RSI（相對強弱指標）= 100 - (100 / (1 + RS))
RS = 平均漲幅 / 平均跌幅

```python
delta = df['close'].diff()
gain = delta.where(delta > 0, 0).rolling(14).mean()
loss = (-delta.where(delta < 0, 0)).rolling(14).mean()

rs = gain / loss
df['RSI'] = 100 - (100 / (1 + rs))

# 經典用法：
# RSI > 70 → 超買（可能反轉）
# RSI < 30 → 超賣（可能反彈）
```

### 進場邏輯 + 停損/停利規則

**沒有停損的策略，不是策略，是赌博。**

| 規則 | 說明 |
|------|------|
| 固定停損 | 進場價 -5% 止損 |
| 移動停損（Trailing Stop） | 從高點回落 8% 止損 |
| 固定停利 | 報酬 >20% 鎖利 |
| ATR 停損 | 以真實波幅的倍數設定停損 |

```python
# ATR（Average True Range）停損
atr = df['ATR'].iloc[-1]
stop_loss = entry_price - 2 * atr   # 2倍ATR作為停損
take_profit = entry_price + 3 * atr  # 3倍ATR作為停利
```

### 為什麼大部分書本策略賺不到錢

1. **資訊落後：** 書出版時策略已經公開，有效性打折
2. **滑價（Slippage）：** 實際成交價比訊號價格差 0.1-0.5%
3. **交易成本：** 頻繁交易的手續費 + 交易税侵蝕報酬
4. **過擬合：** 參數優化到完美贴合歷史，未來失效

> 實盤前，先用**扣除交易成本 + 0.2% 滑價**的假設做回測。

---

## 實驗：完整回測流程

目標：走完以下 5 步

1. **想法：** 「股價站上 MA20 且 RSI < 70 時買入」
2. **寫策略：** 把規則翻譯成 Python 程式碼
3. **回測：** 計算信號、報酬、交易次數
4. **看績效：** 總報酬、夏普比率、最大回撤
5. **問問題：** 這個策略在什麼情境下有效？什麼情境會虧錢？

```python
import yfinance as yf
import pandas as pd
import numpy as np

# Step 1: 抓資料
df = yf.Ticker("2330.TW").history(period="2y")[['Close']].copy()

# Step 2: 計算信號
df['MA20'] = df['Close'].rolling(20).mean()
delta = df['Close'].diff()
gain = delta.where(delta > 0, 0).rolling(14).mean()
loss = (-delta.where(delta < 0, 0)).rolling(14).mean()
rs = gain / loss
df['RSI'] = 100 - (100 / (1 + rs))

# 進場信號
df['long_signal'] = (df['Close'] > df['MA20']) & (df['RSI'] < 70)

# Step 3: 計算報酬
df['daily_return'] = df['Close'].pct_change()
df['strategy_return'] = df['daily_return'] * df['long_signal'].shift(1)

# Step 4: 績效
total_return = (1 + df['strategy_return'].dropna()).prod() - 1
sharpe = df['strategy_return'].mean() / df['strategy_return'].std() * np.sqrt(252)
print(f"總報酬: {total_return:.2%}")
print(f"夏普比率: {sharpe:.2f}")
```

---

## 課後作業

修改上面的策略參數（MA 天數、RSI 區間），觀察：
1. 總報酬和夏普比率的變化
2. 交易頻率的變化
3. **學會問：「為什麼這個參數效果更好？」而不是「哪組參數賺最多？」**

---

## 重點複習

- **停損是策略的保險**，沒有停損的策略在黑天鵝事件會歸零
- 回測出來的「最佳參數」通常是**過擬合**，真實市場不會復現
- 書本策略賺不到錢不是因為策略爛，是因為**市場在變**
