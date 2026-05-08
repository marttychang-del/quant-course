# 第 6 堂：統計套利與事件驅動（初階）
[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/marttychang-del/quant-course/blob/main/session-06/06_pair_trading.ipynb)

**目標：** 理解兩種不需要預測市場方向的Alpha來源

---

## 課程內容

### Pair Trading 配對交易原理

**核心邏輯：** 兩檔高度相關的股票，長期會維持均衡關係。當關係偏離時，賭它會回歸。

```python
import yfinance as yf
import numpy as np

# 抓兩檔高度相關的股票（這裡用 AAPL vs MSFT 示範）
aapl = yf.Ticker("AAPL").history(period="1y")['Close']
msft = yf.Ticker("MSFT").history(period="1y")['Close']

# 價格比率（spread 的代理）
ratio = aapl / msft
mean_ratio = ratio.mean()
std_ratio = ratio.std()

print(f"比率均值: {mean_ratio:.4f}")
print(f"比率標準差: {std_ratio:.4f}")

# Z-score：偏離多少個標準差
z_score = (ratio.iloc[-1] - mean_ratio) / std_ratio
print(f"目前 Z-score: {z_score:.2f}")
```

**交易邏輯：**
- Z-score > 2：比率偏高，Short AAPL / Long MSFT（賭比率收斂）
- Z-score < -2：比率偏低，Long AAPL / Short MSFT
- Z-score 回到 0 附近：平倉

> **風險提示：** 比率偏離不回歸是很常見的——基本面改變了，歷史均衡關係不再成立。

### 台股特殊事件

| 事件 | 說明 | 量化切入點 |
|------|------|-----------|
| **除權息** | 每年 7-8 月密集除權息，指數自然蒸發 | 避開高股息ETF成分股或利用期貨反向套利 |
| **季度營收** | 每月營收公告（3, 5, 8, 11 月） | 營收年增率 > 20% 的動能策略 |
| **庫存消化** | 半導體庫存週期（約 3-4 季） | 觀察通路庫存數據領先股價 |

### Earnings Surprise 事件策略

財報超預期（EPS > 預期）後，股價通常會有正向反應：

```python
# 概念框架（實際需要財報資料庫如 Alpha Vantage）
# 1. 取得 EPS 公告日期
# 2. 比較實際 EPS vs 分析師共識
# 3. 超預期幅度越大，公告後正報酬機率越高

# 粗略統計（以美股為例）：
# 超預期 > 5%：平均 +1.5%（次日）
# 超預期 > 10%：平均 +2.5%（次日）
# 低於預期：平均 -2%（次日）
```

### 量化策略的生命週期

一個量化策略通常會經歷：

1. **開發期（1-3 月）：** 想法 → 回測 → 參數優化
2. **模擬期（1-3 月）：** Paper trade，確認實盤可行性
3. **實盤期（6-24 月）：** 小資金上線，逐步增加部位
4. **衰減期：** 市場結構改變 or 策略被太多人用，Alpha 消失
5. **終結 or 重新校準**

> **實務提醒：** 策略開始賺錢後反而要提高警覺——可能是市場還沒注意到你，也可能是你正在承受隱藏風險。

---

## 實驗：用相關係數找出台股兩檔有套利潛力的股票對

```python
import yfinance as yf
import pandas as pd

# 嘗試不同股票對
pairs = [
    ("2330.TW", "2454.TW"),  # 台積電 vs 聯發科（半導體雙雄）
    ("2891.TW", "5880.TW"),  # 中信金 vs 兆豐金（金融）
]

for stock_a, stock_b in pairs:
    a = yf.Ticker(stock_a).history(period="1y")['Close']
    b = yf.Ticker(stock_b).history(period="1y")['Close']
    
    # 取共同日期
    aligned = pd.concat([a, b], axis=1).dropna()
    corr = aligned.iloc[:, 0].corr(aligned.iloc[:, 1])
    
    # Pair Trading Z-score
    ratio = aligned.iloc[:, 0] / aligned.iloc[:, 1]
    z = (ratio.iloc[-1] - ratio.mean()) / ratio.std()
    
    print(f"{stock_a} vs {stock_b}")
    print(f"  相關係數: {corr:.4f}")
    print(f"  目前 Z-score: {z:.2f}")
    print(f"  {'有機會' if abs(z) > 1.5 else '無明顯偏離'}")
    print()
```

---

## 課後作業

1. 用上面的程式碼，測試 3 組不同的台股股票對
2. 記錄每組的相關係數和 Z-score
3. 說明哪一組**目前**看起來最有套利潛力，為什麼

---

## 重點複習

- **配對交易的核心風險**是比率不回歸——基本面改變會讓歷史均衡失效
- 台股除權息、季度營收是**可預測的事件**，可以提前佈局
- 量化策略一定會衰減，**能活多久取決於策略護城河有多深**
- 策略開發完成後，**資金管理**比策略本身更重要
