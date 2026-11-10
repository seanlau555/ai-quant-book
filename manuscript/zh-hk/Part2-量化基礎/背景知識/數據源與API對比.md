# 背景知識：數據源與API對比

> 數據是量化交易的基礎。選擇正確的數據源直接影響策略開發效率和實盤表現。

---

## 一、主流數據源概覽

| 數據源 | 類型 | 價格 | 延遲 | 適用場景 |
|-------|------|------|------|---------|
| **Binance API** | 加密貨幣 | 免費 | 實時 | 加密貨幣策略 |
| **Yahoo Finance** | 股票/ETF | 免費 | 15-20分鐘 | 學習/回測 |
| **Alpha Vantage** | 多資產 | 免費/付費 | 15分鐘 | 原型開發 |
| **Polygon.io** | 美股 | $29-199/月 | 實時 | 美股實盤 |
| **Alpaca Markets** | 美股 | 免費/付費 | 實時 | 美股開發/交易 |
| **Bloomberg** | 全資產 | $2400+/月 | 實時 | 機構級 |
| **Refinitiv** | 全資產 | $1800+/月 | 實時 | 機構級 |
| **Nasdaq Data Link** | 另類數據 | 按需定價 | 日頻 | 因子研究 |

> ⚠️ **重要提醒**：IEX Cloud 已於 2024 年 8 月停止服務。如果你的代碼依賴 IEX Cloud，需要遷移到其他數據源（推薦 Polygon.io 或 Alpaca）。

---

## 二、免費數據源詳解

### 2.1 Binance API

**優點**：
- 完全免費，無需付費
- 實時數據，延遲極低
- 支持 WebSocket 實時推送
- 歷史數據完整（可追溯多年）
- REST + WebSocket 雙接口

**缺點**：
- 僅限加密貨幣
- Rate Limit：1200次/分鐘（REST），每個連接限制訂閲數量（WebSocket）
- 高波動時 API 可能不穩定

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
                time.sleep(60)  # 等待1分鐘
            else:
                raise
```

**適用場景**：加密貨幣策略開發、7x24小時交易系統

---

### 2.2 Yahoo Finance (yfinance)

**優點**：
- 完全免費
- 覆蓋全球股票、ETF、指數
- 歷史數據豐富（可追溯數十年）
- Python 庫簡單易用

**缺點**：
- 數據延遲 15-20 分鐘
- 無官方 API，依賴網頁抓取（可能被封）
- 數據質量不保證（偶有錯誤）
- 無 Level-2 數據

**使用示例**：
```python
import yfinance as yf

# 獲取股票數據
aapl = yf.Ticker("AAPL")
df = aapl.history(period="1y", interval="1d")

# 批量獲取
data = yf.download(["AAPL", "GOOGL", "MSFT"], period="1mo")
```

**適用場景**：學習、回測、日頻策略研究

---

### 2.3 Alpha Vantage

**優點**：
- 免費層可用（25次/天，2024年後大幅收緊）
- 覆蓋股票、外匯、加密貨幣
- 提供技術指標 API
- 有官方支持

**缺點**：
- 免費層限制嚴格
- 付費版 $49.99/月起
- 數據延遲 15 分鐘
- 歷史數據深度有限

**Rate Limit 對比**（2024-2025 定價）：
| 層級 | 價格 | 請求限制 |
|-----|------|---------|
| 免費 | $0 | 25次/天 |
| 基礎 | $49.99/月 | 75次/分鐘 |
| 專業 | $149.99/月 | 300次/分鐘 |
| 企業 | $249.99/月 | 600次/分鐘 |

> **注意**：Alpha Vantage 免費層請求限制已大幅收緊，僅適合輕量學習使用。

**適用場景**：原型開發、多資產研究

---

### 2.4 Alpaca Markets

**優點**：
- 免費實時美股數據（需開户）
- 同時提供交易 API
- WebSocket 實時推送
- 支持 Paper Trading（模擬交易）
- Python SDK 完善

**缺點**：
- 需要開設 Alpaca 賬户
- 僅限美股和加密貨幣
- 歷史數據需付費（更深度）

**使用示例**：
```python
from alpaca.data import StockHistoricalDataClient
from alpaca.data.requests import StockBarsRequest
from alpaca.data.timeframe import TimeFrame

client = StockHistoricalDataClient(api_key, secret_key)
request = StockBarsRequest(
    symbol_or_symbols=["AAPL"],
    timeframe=TimeFrame.Day,
    start="2024-01-01"
)
bars = client.get_stock_bars(request)
```

**適用場景**：美股策略開發與實盤交易一體化

---

## 三、付費數據源詳解

### 3.1 Polygon.io

**優點**：
- 美股實時數據
- 歷史 Tick 數據
- WebSocket 實時推送
- 價格合理（$29/月起）

**缺點**：
- 僅限美股
- 基礎版無 Level-2
- 需要美國支付方式

**定價**：
| 層級 | 價格 | 功能 |
|-----|------|-----|
| 基礎 | $29/月 | 延遲數據 + 歷史 |
| 開發者 | $79/月 | 實時數據 |
| 高級 | $199/月 | Level-2 + 全功能 |

---

### 3.2 Bloomberg Terminal

**優點**：
- 全球最全面的金融數據
- 實時 + 歷史 + 新聞 + 研究
- Level-2、訂單簿數據
- 專業分析工具

**缺點**：
- 價格昂貴（$2000+/月/終端）
- 需要專用硬件
- API 使用複雜

**適用場景**：機構投資者、專業量化團隊

---

### 3.3 Refinitiv (原 Thomson Reuters)

**優點**：
- 全球市場覆蓋
- Tick 數據質量高
- 歷史數據深度好
- Eikon API 易用

**缺點**：
- 價格昂貴
- 合同複雜

**適用場景**：機構級策略、高頻交易

---

## 四、另類數據源

### 4.1 Nasdaq Data Link（原 Quandl）

提供非傳統金融數據：
- 衞星圖像數據
- 情感分析數據
- 宏觀經濟數據
- 供應鏈數據

### 4.2 新聞與情感數據

| 來源 | 類型 | 用途 |
|-----|------|-----|
| NewsAPI | 新聞聚合 | 情感分析 |
| Twitter/X API | 社交媒體 | 市場情緒 |
| Reddit API | 論壇 | 散户情緒 |
| SEC EDGAR | 監管文件 | 基本面分析 |

---

## 五、數據源選擇決策樹

```
你的交易品種是什麼？
│
├─ 加密貨幣 → Binance API (免費、實時)
│
├─ 美股
│   ├─ 學習/回測 → Yahoo Finance (免費)
│   ├─ 原型開發 → Alpha Vantage (付費) 或 Alpaca (免費)
│   ├─ 實盤+數據 → Alpaca Markets (免費開户)
│   └─ 高級數據 → Polygon.io ($29+/月)
│
├─ A股
│   ├─ 學習/回測 → Tushare / AKShare (免費)
│   └─ 實盤交易 → 券商 API / Wind
│
└─ 機構級需求 → Bloomberg / Refinitiv
```

> **2024-2025 推薦**：對於美股開發者，Alpaca Markets 已成為最受歡迎的免費選擇，因其同時提供數據和交易 API。

---

## 六、常見陷阱

### 6.1 數據質量問題

- **缺失值**：節假日、停牌導致的數據空洞
- **異常值**：拆股、分紅未調整
- **時區問題**：不同交易所時區不同

### 6.2 Survivorship Bias

免費數據源通常只包含當前存在的股票，已退市股票數據缺失。

### 6.3 Look-Ahead Bias

某些數據源的"歷史數據"可能包含後來修正的值（如財報重述）。

---

## 七、實用建議

1. **起步階段**：Yahoo Finance + Binance API（免費）
2. **原型開發**：Alpaca Markets（免費且同時支持交易）
3. **美股實盤**：Alpaca（數據+交易一體）或 Polygon.io（純數據）
4. **機構級**：Bloomberg / Refinitiv（全面但昂貴）
5. **始終驗證**：對比多個數據源，檢查數據質量

> ⚠️ **避坑提醒**：Alpha Vantage 免費層現在只有 25 次/天，已不適合正式開發使用。

---

> **核心原則**：數據質量 > 數據數量。寧願用少量高質量數據，也不要用大量有問題的數據訓練模型。
