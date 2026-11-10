# Part 5：Production 與實戰

> 本階段目標：**Production deployment。** 由 backtest 到 live trading，由開發到運維，建立一個真正運行得到的 multi-agent 交易系統。

---

## 課程

| 課程 | 主題 | 成果 |
|--------|-------|--------------|
| [第 18 課](第18課：交易成本建模與可交易性.md) | 交易成本建模與可交易性 | 成本估算模型、tradability 評估 |
| [第 19 課](第19課：執行系統 - 從信號到真實成交.md) | Execution 系統 | Execution Agent |
| [第 20 課](第20課：生產運維.md) | Production 運維 | Monitoring 與 alerting 系統 |
| [第 21 課](第21課：項目實戰.md) | 項目實戰 | 完整系統整合 |
| [第 22 課](第22課：總結與進階方向.md) | 總結與進階方向 | 學習路線圖 |

---

## 背景知識

| 文件 | 說明 | 建議閱讀時間 |
|----------|-------------|------------------------|
| [Execution Simulator 實現](背景知識/執行模擬器實現.md) | 四級 execution simulator、slippage 與 market impact 建模 | 25 分鐘 |
| [策略同質化與容量瓶頸](背景知識/策略同質化與容量瓶頸.md) | 策略擁擠、流動性危機、capacity 估算方法 | 20 分鐘 |
| [程序化交易監管（2024-2025）](背景知識/程序化交易監管（2024-2025）.md) | 美國／中國／歐盟監管框架、高頻交易認定標準、合規要求 | 20 分鐘 |

---

## 附錄

| 文件 | 說明 |
|----------|-------------|
| [附錄 A：實盤交易記錄標準指南](附錄/附錄A：實盤交易記錄標準指南.md) | 實盤交易最低限度 log 清單（為 execution 與 RL 提供反饋） |
| [附錄 B：量化系統的 12 種典型死亡方式](附錄/附錄B：量化系統的12種典型死亡方式.md) | 機構級失敗案例總結（必讀） |
| [附錄 C：人類決策與自動化邊界](附錄/附錄C：人類決策與自動化邊界.md) | 人機協作的邊界劃分 |
| [附錄 D：量化交易常見問題 FAQ](附錄/附錄D：量化交易常見問題FAQ.md) | 常見問題 |

---

## 完成本階段後

你將能夠：
- 建立交易成本模型：slippage、market impact、opportunity cost
- 評估策略的 tradability：由 Gross Alpha 到 Net Alpha
- 實現 Execution Agent：TWAP、VWAP 等 execution algorithm
- 建立 monitoring 系統：PnL tracking、drawdown alert、model drift detection
- 使用 time-travel debugging：精確重播「這筆交易為何蝕錢」
- 完成端到端系統整合：由數據到 execution 的完整閉環
- 識別量化系統的 12 種典型死亡方式
- 了解進階方向：高頻交易、跨市場套利、DeFi quant

---

## 最終成果

讀完全書後，你將擁有：

```
+-------------------------------------------------------------+
|              Multi-Agent Trading System Prototype            |
+-------------------------------------------------------------+
|                                                             |
|  Data Pipeline -> Beta/Hedge -> Regime Detection -> Portfolio|
|      |                                                      |
|  Multi-Agent Signals -> Risk Review -> Cost Assessment ->   |
|                         Execution -> Monitoring             |
|                                                             |
+-------------------------------------------------------------+
```

這不是玩具，而是一個可持續迭代、可真正部署的系統架構。
