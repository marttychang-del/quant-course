# 第 8 堂：從回測到實盤 + 系統化框架

**目標：** 建立可運作的量化交易日常流程，完成期末專案提案

---

## 課程內容

### 回測陷阱（前三大殺手）

| 陷阱 | 說明 | 解決方案 |
|------|------|---------|
| **前視偏差（Look-ahead bias）** | 使用了當時還没公布的資料 | 確保信號只用「當下已知的資料」 |
| **存活者偏差（Survivorship bias）** | 只用現在還存在的股票回測 | 使用包含已下市股票的完整資料庫 |
| **最佳化偏差（Optimization bias）** | 用同一份數據反覆調參數 | 預留樣本外數據，或用 Walk-Forward |

```python
# Walk-Forward 示意
# 把資料切成三段：訓練 → 驗證 → 測試
train = df[:'2022']
validate = df['2023']
test = df['2024']

# 在 train 上優化參數
best_params = optimize(train)

# 在 validate 上確認有效性
validate_result = run_backtest(validate, best_params)

# 在 test 上最終驗證（這才是真正的考試）
test_result = run_backtest(test, best_params)
```

### 券商 API 串接（基本概念）

| 券商 | API | 語言 | 文件 |
|------|-----|------|------|
| Interactive Brokers（IBKR） | TWS / IB Gateway | Python（ib_insync） | 完整但學習曲線中等 |
| 元大（台） | 不開放 API | — | — |
| 永豐（台） | Shinobi | Python | 文件有限 |
| 嘉信（美） | API | 多語言 | 完整 |

```python
# IBKR 示例（概念碼）
from ib_insync import IB

ib = IB()
ib.connect('127.0.0.1', 7497, clientId=1)

# 取得合約
contract = Stock('AAPL', 'SMART', 'USD')
data = ib.reqMktData(contract)

# 下單
order = MarketOrder('BUY', 100)
trade = ib.placeOrder(contract, order)
```

> **實務提醒：** 串 API 前，先在 Paper Trade（模擬環境）跑三個月，確認一切正常再轉實盤。

### 量化交易的日常

#### 每週（每週六或日）

- [ ] 檢視本週信號執行情況（有無漏單、錯誤）
- [ ] 計算本週策略績效（報酬、Max DD）
- [ ] 檢查市場環境指標（VIX、利率、信用利差）
- [ ] 調整下周倉位（如果有）

#### 每日（收盤前 30 分鐘）

- [ ] 確認明日有沒有事件（財報、央行利率決策）
- [ ] 檢查策略參數是否需要再平衡
- [ ] 確認帳戶風險（整體部位是否超 2% 原則）

#### 盤後（收盤後 30 分鐘）

- [ ] 記錄今日交易（日誌）
- [ ] 更新持倉績效
- [ ] 產出簡單的績效報告

### 資源推薦

**書籍：**
- 《主力的思維》— 理解市場結構
- 《計量交易的101個原則》— 量化實務
- 《 Fama-French 論文原文》— 因子理論基礎

**工具：**
- Python（pandas, yfinance, backtrader）
- Jupyter Notebook
- GitHub（策略版本控制）

**持續學習：**
- QuantConnect / Backtrader（回測平台）
- SSRN（學術論文）
- WSB / Reddit r/quant（社群觀點，但要小心）

---

## 期末專案提案（需於第 8 堂結束前提交）

請在 GitHub repo 的 `projects/` 目錄提交以下內容：

### 提案格式

```
projects/
└── YOUR_NAME_proposal.md
```

### 提案內容

1. **策略名稱**
2. **市場標的**（台股/美股/期貨/ETF）
3. **進場邏輯**（文字描述 + 量化規則）
4. **停損規則**
5. **資金管理原則**（Kelly or 2%）
6. **預期資料來源**
7. **初步回測時間框架**

---

## 重點複習

- **Walk-Forward** 是避免過擬合的最基本方法
- **實盤前的 Paper Trade** 不是可選項，是必選項
- 量化交易的價值不是「戰勝市場」，而是**紀律、一致性、可複製性**
- **持續記錄和檢討**才是長期進步的核心
