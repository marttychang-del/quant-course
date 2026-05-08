# 第 5 堂：風險管理與資金管理
[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/marttychang-del/quant-course/blob/main/session-05/05_risk_management.ipynb)

**目標：** 建立帳號活下去的核心知識——停損紀律與部位計算

---

## 課程內容

### Kelly Criterion：每次下注多少%

Kelly 公式告訴你：**在有優勢的情況下，每次該押多少帳戶資產**。

```
f* = (bp - q) / b

f* = 每次該押的資金比例
b = 贏時賺到的倍數（扣除本金後的純贏額）
p = 獲勝機率
q = 失敗機率 = 1 - p
```

實務簡化版：
```python
# 假設：
win_rate = 0.55       # 55% 勝率
avg_win = 0.10        # 平均贏 10%
avg_loss = 0.05       # 平均輸 5%

# Kelly %
kelly_pct = (win_rate * avg_win - (1 - win_rate) * avg_loss) / avg_win
print(f"Kelly %: {kelly_pct:.2%}")  # 建議不要用 Kelly 全文，要減半或更低
```

> **實務警告：** Kelly 是理論上限，實際使用建議 ** Kelly（Full Kelly = 破產風險高）。

### 停損紀律

| 停損類型 | 公式 | 適用情境 |
|---------|------|---------|
| 固定停損 | 進場價 × (1 - r%) | 簡單明確 |
| ATR 停損 | 進場價 - 2×ATR | 動態調整，適用所有標的 |
| 移動停損 | 高點回落 X% | 保護帳面獲利 |
| 時間停損 | 持有 N 天未達目標 | 避免無效等待 |

```python
# ATR 停損（推薦）
atr = df['ATR'].iloc[-1]
stop_loss_pct = 2 * atr / entry_price  # 2倍ATR，轉換成百分比
print(f"建議停損: {stop_loss_pct:.2%}")
```

### 部位大小：每筆 Risk 2% 原則

**核心規則：** 單筆交易最多損失帳戶的 2%。

```python
account_size = 1_000_000   # 帳戶 100 萬
risk_per_trade = 0.02       # 每筆風險 2%
max_loss = account_size * risk_per_trade  # 單筆最大損失：2萬

entry_price = 100
stop_loss_price = 95          # 停損價
risk_per_share = entry_price - stop_loss_price  # 每股風險：5元

shares_to_buy = max_loss / risk_per_share
position_value = shares_to_buy * entry_price
print(f"買入股數: {shares_to_buy:.0f} 股")
print(f"總部位: {position_value:.0f} 元")
```

### 多策略分散效果

不相關策略組合，可以**降低整體波動**而不犧牲太多報酬：

```python
# 假設兩策略相關係數 = 0
portfolio_vol = np.sqrt(sig1**2 + sig2**2)  # 遠低於兩者相加
```

> **不相關**是關鍵：兩個都做均線的策略組合起來沒有分散效果。

### 最大回撤（Max Drawdown）

```python
# 最大回撤計算
cumulative = (1 + returns).cumprod()
running_max = cumulative.cummax()
drawdown = (cumulative - running_max) / running_max
max_dd = drawdown.min()
print(f"最大回撤: {max_dd:.2%}")
```

> Max Drawdown 是衡量策略風險的**最誠實指標**——夏普比率 2.0 但 Max DD 50% 的策略，遠比夏普 1.0 但 Max DD 10% 的策略危險。

---

## 實驗：計算 Kelly % 和 2% 風險下的部位

使用自己第 4 堂設計的策略參數，計算：
1. Kelly %（理論值）
2. 建議實戰使用比例（Kelly ÷ 2）
3. 2% 風險下的部位規模

```python
# 請根據自己策略的歷史勝率/盈虧比填入
win_rate = 0.___        # 例如 0.55
avg_win   = 0.___       # 例如 0.10（平均贏10%）
avg_loss  = 0.___       # 例如 0.05（平均輸5%）

kelly = (win_rate * avg_win - (1 - win_rate) * avg_loss) / avg_win
print(f"Kelly %: {kelly:.2%}")
print(f"建議實戰: {kelly/2:.2%}")
```

---

## 課後作業

1. 用自己感興趣的股票，回測 100 筆以上交易，計算：
   - 勝率
   - 平均贏 / 平均輸
   - Kelly %
   - Max Drawdown
2. 根據 2% 原則計算：在 100 萬帳戶下，每次最多可以損失多少？

---

## 重點複習

- **資金管理比進場訊號更重要**——再好的策略，滿倉一次失敗就歸零
- Kelly 公式是理論上限，實戰用 ** Kelly 已經夠了
- **Max Drawdown** 才是真正衡量策略風險的指標
- 2% 原則是懶人散戶最簡單的資金管理規則
