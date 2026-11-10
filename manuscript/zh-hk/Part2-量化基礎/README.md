# Part 2：量化基礎

> **本階段目標：** 打好扎實的基礎。理解市場如何運作、數據如何處理、策略如何驗證，以及風險從何而來。沒有這些基礎，我們之後建立的 Agent 只是空中樓閣。

---

## 課程

| 課程 | 主題 | 成果 |
|--------|-------|-------------|
| [第02課](第02課：金融市場與交易基礎.md) | 金融市場與交易基礎 | 理解市場結構與交易成本 |
| [第03課](第03課：數學與統計基礎.md) | 數學與統計基礎 | 掌握描述 return 與風險的數學語言 |
| [第04課](第04課：技術指標的真實角色.md) | 技術指標的真實角色 | 理解 indicator 是 feature engineering，而不是聖杯 |
| [第05課](第05課：經典策略範式.md) | 經典策略範式 | 掌握 trend following、mean reversion 等基礎策略 |
| [第06課](第06課：數據工程的殘酷現實.md) | 數據工程的殘酷現實 | 數據 pipeline 腳本 |
| [第07課](第07課：回測系統的陷阱.md) | Backtest 系統的陷阱 | Backtesting 框架模板 |
| [第08課](第08課：Beta、對沖與市場中性.md) | Beta、Hedging 與 Market Neutrality | Beta 分解與 hedge ratio 計算 |

---

## 背景知識

| 文件 | 說明 | 建議閱讀時間 |
|----------|-------------|----------------------|
| [交易所與 Order Book 機制](背景知識/交易所與訂單簿機制.md) | L1/L2/L3 數據的分別 | 15 分鐘 |
| [High-Frequency Trading 與 Market Microstructure](背景知識/高頻市場微結構.md) | 理解 HFT 策略、Kyle's Lambda | 20 分鐘 |
| [HF Market Microstructure](背景知識/高頻市場微結構.md) | 深入探討 order book、spread、market impact | 20 分鐘 |
| [Tick-Level Backtesting 框架](背景知識/Tick級回測框架.md) | Event-driven backtesting、queue position 模擬 | 25 分鐘 |
| [Sharpe Ratio 的統計陷阱](背景知識/夏普比率的統計陷阱.md) | Estimation error、multiple testing、Deflated Sharpe | 20 分鐘 |
| [K 線形態與量價分析](背景知識/K線形態與量價分析.md) | K 線形態、成交量分析及量化應用 | 20 分鐘 |
| [加密貨幣交易特點](背景知識/加密貨幣交易特點.md) | 24/7 市場的獨特挑戰 | 10 分鐘 |
| [數據源與 API 對比](背景知識/數據源與API對比.md) | 揀選合適的數據源 | 10 分鐘 |

---

## 完成本階段後

你將能夠：
- 理解不同市場的特性：股票、期貨、加密貨幣
- 用數學語言描述 return 與風險
- 認清技術指標的真正價值與限制
- 建立數據 pipeline，處理 API rate limit、缺失數據等問題
- 避開常見的 backtesting 陷阱（overfitting、lookahead bias 等）
- 理解 Beta 與 Alpha 的分別，知道 return 從何而來
- 計算 hedge ratio，並明白為何 market neutrality 很難達到

---

## 下一階段

→ [Part 3：Machine Learning](../Part3-機器學習/README.md) - 從傳統模型到 Agent
