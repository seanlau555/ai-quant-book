# 背景知識：數據源與 API 對比

> 數據是 quantitative trading 的基礎。選對數據源，直接影響策略開發效率及實盤表現。

---

## 1. 主要數據源一覽

| 數據源 | 類型 | 價格 | Latency | 適用場景 |
|--------|------|------|---------|----------|
| **Binance API** | Crypto | 免費 | 即時 | Crypto 策略 |
| **Yahoo Finance** | 股票/ETF | 免費 | 15-20 分鐘 | 學習/backtest |
| **Alpha Vantage** | 多資產 | 免費/收費 | 15 分鐘 | Prototyping |
| **Polygon.io** | 美股 | $29-199/月 | 即時 | 美股實盤交易 |
| **Alpaca Markets** | 美股 | 免費 | 即時 | 美股交易 |
| **Bloomberg** | 全資產 | $2400+/月 | 即時 | 機構 |
| **Refinitiv** | 全資產 | $1800+/月 | 即時 | 機構 |
| **Nasdaq Data Link** | 另類數據 | 按需報價 | 每日 | Factor 研究 |

---

## 2. 免費數據源詳解

### 2.1 Binance API

**優點**：
- 完全免費，無需付款
- 即時數據，latency 極低
- 支援 WebSocket 即時串流
- 歷史數據完整（多年歷史）
- REST + WebSocket 雙接口

**缺點**：
- 只有 Crypto
- Rate Limit：1200 次請求/分鐘（REST），WebSocket 每個連線有訂閱數量限制
- 高 volatility 期間 API 可能不穩定

**Rate Limit 處理**：
```python
import time
from binance.exceptions import BinanceAPIException

def fetch_with_retry(func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return func()
        except BinanceAPIException as e:
            if e.code == -1003:  # Rate limit
                time.sleep(60)  # Wait 1 minute
            else:
                raise
```

**適用場景**：Crypto 策略開發、24/7 交易系統

---

### 2.2 Yahoo Finance (yfinance)

**優點**：
- 完全免費
- 涵蓋全球股票、ETF、指數
- 歷史數據豐富（數十年歷史）
- Python 庫易於使用

**缺點**：
- 數據延遲 15-20 分鐘
- 沒有官方 API，依賴網頁抓取（可能被封鎖）
- 數據質量無保證（偶有錯誤）
- 沒有 Level-2 數據

**使用示例**：
```python
import yfinance as yf

# Get stock data
aapl = yf.Ticker("AAPL")
df = aapl.history(period="1y", interval="1d")

# Batch download
data = yf.download(["AAPL", "GOOGL", "MSFT"], period="1mo")
```

**適用場景**：學習、backtest、日線頻率策略研究

---

### 2.3 Alpha Vantage

**優點**：
- 有免費方案（每日 25 次請求）
- 涵蓋股票、forex、crypto
- 提供技術指標 API
- 有官方支援

**缺點**：
- 免費方案限制嚴重（2024 年起每日只有 25 次調用）
- 收費版由 $49.99/月起
- 數據延遲 15 分鐘
- 歷史深度有限

**Rate Limit 比較**：
| 方案 | 價格 | 請求上限 |
|------|------|----------|
| Free | $0 | 25 次/日（限制嚴重） |
| Basic | $49.99/月 | 30 次/分鐘 |
| Premium | $249.99/月 | 120 次/分鐘 |

**適用場景**：Prototyping、多資產研究

---

### 2.4 Alpaca Markets

**優點**：
- 免費提供美股即時數據
- 數據 + 交易 API 二合一（無需另外訂閱數據）
- 支援 WebSocket 即時串流
- 內置 paper trading 環境
- 股票交易免佣金
- 支援碎股（fractional shares）

**缺點**：
- 只有美股
- 需要註冊賬戶
- 部分功能需要已入金的賬戶

**使用示例**：
```python
from alpaca.data import StockHistoricalDataClient
from alpaca.data.requests import StockBarsRequest
from alpaca.data.timeframe import TimeFrame
from datetime import datetime, timedelta

# Initialize client (no keys needed for free tier)
client = StockHistoricalDataClient()

# Get historical bars
request = StockBarsRequest(
    symbol_or_symbols=["AAPL", "MSFT"],
    timeframe=TimeFrame.Day,
    start=datetime.now() - timedelta(days=30)
)
bars = client.get_stock_bars(request)
```

**適用場景**：美股策略開發、paper trading、使用同一套 API 實盤交易

---

## 3. 收費數據源詳解

### 3.1 Polygon.io

**優點**：
- 美股即時數據
- 歷史 tick 數據
- 支援 WebSocket 即時串流
- 價格合理（$29/月起）

**缺點**：
- 只有美股
- Basic 方案沒有 Level-2
- 需要美國付款方式

**定價**：
| 方案 | 價格 | 功能 |
|------|------|------|
| Basic | $29/月 | 延遲數據 + 歷史數據 |
| Developer | $79/月 | 即時數據 |
| Premium | $199/月 | Level-2 + 完整功能 |

---

### 3.2 Bloomberg Terminal

**優點**：
- 全球最全面的金融數據
- 即時 + 歷史 + 新聞 + 研究
- Level-2、order book 數據
- 專業分析工具

**缺點**：
- 價格高昂（$2400+/月/終端機）
- 需要專用硬件
- API 使用複雜

**適用場景**：機構投資者、專業 quant 團隊

---

### 3.3 Refinitiv（前身為 Thomson Reuters）

**優點**：
- 覆蓋全球市場
- 高質量 tick 數據
- 歷史數據深厚
- Eikon API 易於使用

**缺點**：
- 定價昂貴
- 合約複雜

**適用場景**：機構策略、HFT

---

## 4. 另類數據源

### 4.1 Nasdaq Data Link（前身為 Quandl）

提供非傳統金融數據：
- 衞星影像數據
- 情緒分析數據
- 宏觀經濟數據
- 供應鏈數據

### 4.2 新聞與情緒數據

| 來源 | 類型 | 用途 |
|------|------|------|
| NewsAPI | 新聞聚合 | 情緒分析 |
| Twitter/X API | 社交媒體 | 市場情緒 |
| Reddit API | 討論區 | 散戶情緒 |
| SEC EDGAR | 監管文件 | 基本面分析 |

---

## 5. 數據源選擇決策樹

```
What asset class are you trading?
│
├─ Cryptocurrency → Binance API (Free, Real-time)
│
├─ US Stocks
│   ├─ Learning/Backtesting → Yahoo Finance (Free)
│   ├─ Prototyping → Alpaca Markets (Free, Real-time)
│   ├─ Data + Trading Combined → Alpaca Markets (Free)
│   └─ Premium Data Only → Polygon.io ($29+/mo)
│
├─ A-Shares (China)
│   ├─ Learning/Backtesting → Tushare / AKShare (Free)
│   └─ Live Trading → Broker API / Wind
│
└─ Institutional Needs → Bloomberg / Refinitiv
```

---

## 6. 常見陷阱

### 6.1 數據質量問題

- **缺失值**：假期、停牌造成的空白
- **異常值**：未調整的拆股、派息
- **時區問題**：不同交易所的時區各異

### 6.2 Survivorship Bias

免費數據源通常只包含現時仍存在的股票，缺少已除牌股票的數據。

### 6.3 Lookahead Bias

部分數據源的「歷史數據」可能包含事後修正（例如盈利重述）。

---

## 7. 實用建議

1. **起步階段**：Yahoo Finance + Binance API（免費）
2. **Prototyping**：Alpaca Markets（免費即時數據 + paper trading）
3. **美股實盤**：Alpaca Markets（免費）或 Polygon.io（$29+/月，享進階功能）
4. **機構**：Bloomberg / Refinitiv（全面但昂貴）
5. **務必驗證**：比對多個數據源，檢查數據質量

> **警告**：IEX Cloud 已於 2024 年 8 月停止公開 API 服務。如有使用 IEX Cloud 的舊代碼，請遷移至 Alpaca Markets 或 Polygon.io。

---

> **核心原則**：數據質量 > 數據數量。寧願用少量高質量數據，也不要用大量有問題的數據去訓練 model。
