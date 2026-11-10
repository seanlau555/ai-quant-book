# 背景知識：LLM 在量化交易中的研究進展

> Large Language Model（LLM）正在改變量化交易的研究範式。本文整理 LLM 在量化領域的主要研究方向及代表作品。

---

## 1. LLM 在量化交易中的應用場景

| 場景 | 傳統方法 | LLM 方法 | 優勢 |
|----------|-------------------|------------|-----------|
| Sentiment 分析 | 規則／詞典 | 理解上下文 | 更準確、更靈活 |
| 新聞理解 | NLP 模型 | 深層語義理解 | 理解複雜因果關係 |
| 研究報告分析 | 人手閱讀 | 自動提取 | 效率大幅提升 |
| 策略生成 | 人手編碼 | 代碼生成 | 快速製作原型 |
| 市場推理 | 規則系統 | Chain-of-thought 推理 | 多步邏輯 |

---

## 2. 代表性研究作品

### 2.1 FinGPT（2023）

**來源**：Columbia University + AI4Finance

**定位**：開源的金融 LLM

**核心特點**：
- 基於 LLaMA 進行 fine-tune
- 以金融語料訓練
- 支持 sentiment 分析、問答、摘要
- 開源可用

**架構**：
```
Base Model (LLaMA)
    |
Financial Corpus Pre-training (Financial News, SEC Filings)
    |
Task Fine-tuning (Sentiment, QA, Summarization)
    |
FinGPT
```

**限制**：
- 主要用於 NLP 任務
- 無法直接生成交易信號
- 即時處理能力不足

**連結**：https://github.com/AI4Finance-Foundation/FinGPT

---

### 2.2 BloombergGPT（2023）

**來源**：Bloomberg

**定位**：500 億參數的金融專用 LLM

**核心特點**：
- 專有模型，不開源
- 訓練數據包含 Bloomberg 獨家數據
- 在金融 NLP 任務上優於通用模型

**訓練數據**：
| 數據類型 | 規模 |
|-----------|-------|
| 金融數據 | 3,630 億 token |
| 通用數據 | 3,450 億 token |

**表現**：在金融 sentiment 分析、NER 等任務上領先 GPT-3

**限制**：不開源，無法直接使用

---

### 2.3 FLAG-Trader（2024-2025）

**來源**：學術研究

**定位**：LLM + reinforcement learning 交易系統

**核心創新**：
- LLM 生成交易假設
- RL 驗證及優化
- Feedback loop 改進 LLM

**架構**：
```
Market Data + News
    |
LLM (Generate trading hypotheses)
    |
RL Agent (Execute and evaluate)
    |
Reward Feedback
    |
LLM Improvement (Learn from feedback)
```

**優勢**：
- 結合 LLM 的推理能力與 RL 的優化能力
- 可解釋性好
- 可處理文字 + 數值的 multimodal 數據

---

### 2.4 TradingGPT / QuantGPT 系列

**定位**：以 GPT-4 等模型作出交易決策

**方法**：
- 直接使用 GPT-4 API
- 設計特定 prompt
- 生成交易建議

**典型 Prompt**：
```
You are a quantitative analyst. Based on the following market data:
- BTC 24-hour change: +5.2%
- RSI(14): 72
- Volume: Up 30% from yesterday

Please analyze the current market state and provide trading recommendations.
```

**限制**：
- 依賴 API，latency 高
- 成本高
- 有 hallucination 問題

---

### 2.5 MM-DREX（2024）

**定位**：Multimodal 交易系統

**核心創新**：
- 直接「看」K 線圖
- 結合文字 + 圖像
- 視覺形態識別

**輸入**：
```
Text: News, indicator values
Image: Candlestick charts, technical analysis charts
    |
Multimodal LLM
    |
Trading Decision
```

**優勢**：模仿人類交易員「睇圖」的能力

---

## 3. 主要研究方向

### 3.1 增強 Sentiment 分析

**傳統方法**：
```python
# Simple dictionary approach
positive_words = ['bullish', 'surge', 'rally']
negative_words = ['bearish', 'crash', 'plunge']
```

**LLM 方法**：
```python
# Context understanding
prompt = """
Analyze the market sentiment of the following news (-1 to 1):
"Fed signals potential rate cuts, but warns inflation remains sticky"

Please consider:
1. Direct impact
2. Implied expectations
3. Possible market reaction
"""
```

**優勢**：能理解複雜而互相矛盾的信息

---

### 3.2 因子發掘與假設生成

**傳統方法**：人手設計因子

**LLM 方法**：
```python
prompt = """
Based on the following market patterns, propose potentially effective quantitative factors:
1. Small-cap stocks outperform large-cap over the long term
2. High momentum stocks show continuation
3. Low volatility stocks have better risk-adjusted returns

Please propose 3 new factor hypotheses, including:
- Factor definition
- Theoretical basis
- Potential risks
"""
```

**優勢**：快速生成大量假設以供測試

---

### 3.3 代碼生成

**應用**：
- 策略代碼生成
- 數據處理腳本
- 可視化代碼

**示例**：
```python
prompt = """
Implement a dual moving average strategy in Python:
- Short-term MA: 10 days
- Long-term MA: 30 days
- Golden cross = buy, death cross = sell
- Use pandas for data processing
"""
```

**注意**：生成的代碼需要人手審查

---

### 3.4 市場推理

**應用**：多步邏輯推理

**示例**：
```
Question: If the Fed raises rates, what's the impact on tech stocks?

LLM Reasoning Chain:
1. Rate hike -> Interest rates rise
2. Rising rates -> Higher discount rate
3. Higher discount rate -> Future cash flow present value decreases
4. Tech stocks depend on future growth -> Valuations more affected
5. Conclusion: Tech stocks may fall, especially high-valuation growth stocks
```

---

## 4. 現時的限制

### 4.1 Hallucination 問題

LLM 可能生成聽起來合理但實際錯誤的分析：
- 捏造數據
- 錯誤的因果關係
- 過度自信的預測

**應對策略**：必須以真實數據驗證

### 4.2 即時處理能力不足

- API 調用 latency：100-500ms
- 模型 inference 時間：1-10 秒
- 不適合高頻交易

### 4.3 成本問題

| 模型 | 成本 | 適用場景 |
|-------|------|--------------|
| GPT-4 | $30-60/1M token | 研究 |
| GPT-3.5 | $0.5-2/1M token | Production |
| 開源模型 | 運算成本 | 自行部署 |

### 4.4 無法預測價格

**LLM 做得到**：
- 理解新聞
- 分析 sentiment
- 生成假設

**LLM 做不到**：
- 預測明日價格
- 保證盈利
- 取代傳統量化模型

---

## 5. 實務建議

### 5.1 適合使用 LLM 的任務

| 任務 | 建議程度 | 備註 |
|------|----------------------|-------|
| 新聞 sentiment 分析 | 高 | LLM 的核心強項 |
| 研究報告摘要 | 高 | 效率顯著提升 |
| 策略代碼生成 | 中 | 需要人手審查 |
| 因子假設生成 | 中 | 需要 backtest 驗證 |
| 直接作出交易決策 | 低 | 風險高，不建議 |

### 5.2 建議架構

```
Traditional Quantitative Model (Primary)
    ^
LLM Enhancement Layer (Auxiliary)
- Sentiment signals
- News filtering
- Hypothesis generation
    ^
Raw Data
```

### 5.3 開源選項

| 模型 | 規模 | 適用場景 |
|-------|------|--------------|
| LLaMA 3 | 8B-70B | 通用 |
| FinGPT | 7B | 金融 NLP |
| Mistral | 7B | 輕量部署 |
| Qwen | 7B-72B | 中文支持 |

---

## 6. 未來趨勢

1. **Multimodal 融合**：文字 + 數值 + 圖像
2. **Agent 化**：LLM 作為交易 agent 的「大腦」
3. **即時處理**：更低 latency 的 inference
4. **專門化**：更多金融專用模型
5. **合規**：滿足監管對可解釋性的要求

---

> **核心原則**：LLM 是強大的工具，但並非魔法。它可以增強你的分析能力，卻無法取代扎實的量化基礎與嚴格的風險控制。
