# Part 3：機器學習

> 本階段目標：**由 Model 到 Agent。** 理解 machine learning 在量化交易中的正確用法，以及如何由「預測模型」演進為「做決策的 Agent」。

---

## 課程

| 課程 | 主題 | 成果 |
|--------|-------|--------------|
| [第09課](第09課：監督學習在量化中的應用.md) | Supervised Learning 在量化中的應用 | ML 策略框架、IC/IR 評估 |
| [第10課](第10課：從模型到Agent.md) | 從 Model 到 Agent | 單一 Agent 策略模組 |

---

## 背景知識

| 文件 | 說明 | 建議閱讀時間 |
|----------|-------------|------------------------|
| [LLM 量化研究進展](背景知識/LLM量化研究進展.md) | Large language model 在量化交易中的最新應用 | 15 分鐘 |
| [Triple Barrier 標籤方法](背景知識/Triple Barrier標籤方法.md) | 用 take-profit 與 stop-loss 定義 ML label | 15 分鐘 |
| [時間序列 Cross-Validation（Purged CV）](背景知識/時間序列交叉驗證（Purged CV）.md) | 防止時間序列中的 information leakage | 15 分鐘 |
| [強化學習（Reinforcement Learning）在交易中的應用](背景知識/強化學習在交易中的應用.md) | 將 RL 演算法應用於交易 | 20 分鐘 |
| [另類數據（NLP 與衛星）](背景知識/另類數據（NLP與衛星）.md) | 非傳統數據來源：文字 sentiment、衛星圖像等 | 15 分鐘 |
| [Meta-Labeling 方法](背景知識/Meta-Labeling方法.md) | 用第二層 model 預測訊號的可靠程度 | 15 分鐘 |
| [Feature Engineering 常見陷阱](背景知識/特徵工程常見陷阱.md) | Future leakage、overfitting 等常見錯誤 | 10 分鐘 |
| [ML 在金融中的限制](背景知識/機器學習在金融中的限制.md) | 核心挑戰：低 signal-to-noise ratio、distribution drift 等 | 15 分鐘 |
| [Model 架構選擇指南](背景知識/模型架構選擇指南.md) | LSTM/GRU/Transformer/CNN 比較、RL 演算法選擇 | 20 分鐘 |
| [Model Drift 與 Retraining](背景知識/模型漂移與再訓練.md) | K-S/CUSUM drift detection、retraining 觸發條件與策略 | 20 分鐘 |
| [MLOps 基礎設施](背景知識/MLOps基礎設施.md) | Feature Store、Model Registry、Drift Monitor | 30 分鐘 |
| [前沿 ML 與 RL 方法（2025）](背景知識/前沿ML與RL方法（2025）.md) | SOTA 技術：Decision Transformer、AlphaAgent、GNN、Diffusion Models | 30 分鐘 |

---

## 完成本階段後

你將能夠：
- 理解為何「預測價格」是錯誤的目標
- 設計合理的 label（Triple Barrier Method）
- 理解 Agent 的核心組成：State、Action、Reward、Environment
- 建立能產生交易訊號的單一 Agent 策略

---

## 下一階段

→ [Part 4：Multi-Agent 系統](../Part4-多智能體/README.md) - 建立協作式 Agent 系統
