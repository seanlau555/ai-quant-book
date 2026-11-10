# Part 4：Multi-Agent 系統

> 本階段目標：**建立協作架構。** 理解點解需要多個 expert agent、點樣設計它們的分工與協作，以及 Risk Agent 如何擁有 veto power。

---

## 課程

| 課程 | 主題 | 成果 |
|--------|-------|--------------|
| [第11課](第11課：為什麼需要多智能體.md) | 為什麼需要 Multi-Agent | Multi-Agent 架構設計 |
| [第12課](第12課：市場狀態識別.md) | Regime Detection | Regime Agent |
| [第13課](第13課：Regime誤判與系統性崩潰模式.md) | Regime 誤判與系統性崩潰 | 診斷清單、degradation 策略 |
| [第14課](第14課：LLM在量化中的應用.md) | LLM 在量化中的應用 | LLM 增強層設計 |
| [第15課](第15課：風險控制與資金管理.md) | 風險控制與資金管理 | Risk Agent |
| [第16課](第16課：組合構建與風險暴露管理.md) | Portfolio 構建與 exposure 管理 | Portfolio Agent、factor monitoring |
| [第17課](第17課：在線學習與策略進化.md) | Online Learning 與策略進化 | Evolution Agent |

---

## 背景知識

| 文件 | 說明 | 建議閱讀時間 |
|----------|-------------|----------------------|
| [Multi-Agent 框架對比](背景知識/多智能體框架對比.md) | Shannon vs AutoGen vs CrewAI | 15 分鐘 |
| [量化開源框架對比](背景知識/量化開源框架對比.md) | VectorBT vs Backtrader vs FinRL | 10 分鐘 |
| [均值方差 Portfolio 優化](背景知識/均值方差組合優化.md) | Markowitz Model、efficient frontier、risk parity | 20 分鐘 |

---

## 完成本階段後

你將能夠：
- 設計 multi-agent 架構：Meta Agent、Expert Agents、Risk Agent
- 實現 Regime Detection：識別 trending、ranging 與 crisis 市場
- 理解 Regime 誤判的五種模式及應對策略
- 理解 LLM 的正確用法（增強，而非取代）
- 建立由 Risk Agent 擁有 veto power 的多層 risk control 系統
- 設計 portfolio 層：position sizing、factor exposure monitoring
- 設計 online learning 與策略淘汰機制

---

## 下一階段

→ [Part 5：生產與實戰](../Part5-生產與實戰/README.md) - 部署到真實環境
