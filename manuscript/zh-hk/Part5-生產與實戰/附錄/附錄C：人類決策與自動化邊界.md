# 附錄 C：人類決策與自動化邊界

> **哪些決策絕不應該自動化？**

---

## 為什麼需要這些邊界

量化系統的終極誘惑是：**全部自動化。**

但歷史告訴我們，完全自動化是危險的：
- **2010 年 Flash Crash**：高頻 algorithm 互相反饋，道瓊斯指數 5 分鐘內下跌 1000 點
- **Knight Capital**：45 分鐘內虧損 4.4 億美元，沒有人及時介入
- **LTCM**：模型說「安全」，人類判斷被忽視，結果是爆倉

本附錄幫助你劃清邊界：
- 哪些可以自動化？
- 哪些需要 human-in-the-loop？
- 哪些必須由人類決定？

---

## 決策分類框架

```
+-------------------------------------------------------------+
|                    Automation Level Spectrum                 |
+-------------------------------------------------------------+
|                                                             |
|  Full Auto    High Auto    Human Loop    Human Approve    Manual    |
|      |          |            |              |             |         |
|      v          v            v              v             v         |
|   Execute    Stop Loss    Strategy    Param Change    Capital       |
|   Orders     Take Profit   Switch    System Upgrade    Allocation  |
|   Data Clean  Position    Risk       New Strategy    Compliance    |
|   Signal Gen  Adjust     Downgrade   Launch          Legal         |
|              Rebalance   Exception                                  |
|                          Handling                                   |
|                                                                    |
|  <----------------------------------------------------------->     |
|   High Efficiency                              Low Risk            |
|   High Risk                                    Low Efficiency      |
|                                                                    |
+-------------------------------------------------------------+
```

---

## Level 1：Full Automation

### 定義
系統自主決策並執行，無需人類確認。

### 適用場景

| 場景 | 原因 | 條件 |
|------|------|------|
| Order execution | 需要速度 | 在預設規則內 |
| 數據清洗 | 規則明確 | 設有異常警報 |
| Signal 生成 | 模型輸出 | 不直接交易 |
| Logging | 純記錄 | 不含決策成分 |

### 風險控制
```
Even with full automation, you need:
[ ] Hard limits (per-trade max, daily loss max)
[ ] Anomaly detection (alert on deviation from historical behavior)
[ ] Post-hoc audit logs
[ ] Emergency stop button
```

---

## Level 2：High Automation

### 定義
系統自動執行，但設有預設的安全邊界。

### 適用場景

| 場景 | 自動部分 | 需要人類觸發的部分 |
|------|----------|-------------------|
| Stop loss / take profit | 觸發後自動執行 | 修改 stop loss 水平需要批准 |
| 倉位調整 | 在限額內自動調整 | 超出限額需要人類確認 |
| Rebalance | 按規則自動執行 | 偏離門檻時需要人類介入 |

### 案例：智能 Stop Loss

```
Rules:
- 2% loss -> Auto stop 50% of position
- 5% loss -> Auto stop remaining position
- 10% loss -> Auto stop + notify human

Human Reserved Rights:
- Can manually adjust stop level before loss
- But cannot cancel stop after loss occurs
```

---

## Level 3：Human-in-the-Loop

### 定義
系統提出建議，經人類確認後才執行。

### 適用場景

| 場景 | 系統做甚麼 | 人類做甚麼 |
|------|------------|------------|
| 策略切換 | 偵測 regime，建議切換 | 確認切換 |
| 風險降級 | 偵測異常，建議降級 | 批准降級 |
| 異常交易 | 偵測異常，暫停 execution | 決定繼續或取消 |
| 大額訂單 | 提供拆單建議 | 批准 execution |

### 設計原則

```
1. Time Limit
   - Human must respond within X minutes
   - Timeout triggers safe default action

2. Sufficient Information
   - System must provide enough info for decision
   - Not just "confirm/cancel"

3. Accountability Mechanism
   - Record who decided what and when
   - Enables post-mortem review
```

---

## Level 4：Human Approval

### 定義
必須經人類明確批准才能執行。

### 適用場景

| 場景 | 為何需要人類 |
|------|--------------|
| 參數修改 | 可能影響系統行為 |
| 系統升級 | 需要測試及 rollback 計劃 |
| 新策略上線 | 需要完整的審核流程 |
| 資金分配 | 涉及大額資金 |
| 權限變更 | 與安全有關 |

### 批准流程示例

```
Parameter Modification Process:

1. Researcher submits modification request
   - Modification content
   - Modification reason
   - Expected impact
   - Rollback plan

2. Risk control review
   - Does it increase risk
   - Does it comply with limits

3. CTO/PM approval
   - Is it necessary
   - Is timing appropriate

4. Execution
   - First test in simulation environment
   - Then test with small capital live
   - Finally full deployment

5. Monitoring
   - Close monitoring for 24 hours post-launch
   - Rollback immediately on anomaly
```

---

## Level 5：Manual Only

### 定義
不應自動化，必須完全由人類決定。

### 適用場景

| 場景 | 為何不能自動化 |
|------|----------------|
| 資金分配 | 涉及公司／客戶資金 |
| 合規決策 | 涉及法律責任 |
| 危機應對 | 需要綜合判斷 |
| 對外溝通 | 需要人際互動 |
| 戰略決策 | 涉及公司方向 |

### 危機應對案例

```
March 2020 Market Crash

Automated system can do:
- Detect crisis state
- Trigger preset deleveraging rules
- Send alerts

Must be human decision:
- Whether to fully liquidate
- Whether to pause all strategies
- Whether to communicate with investors
- Whether to redeem other funds
- How to report to regulators
```

---

## 邊界判斷清單

當你不確定應否自動化時，問自己：

```
[ ] Is this decision reversible?
  - Reversible -> Lean toward automation
  - Irreversible -> Lean toward manual

[ ] Is there time pressure on this decision?
  - Millisecond level -> Must automate
  - Minute level -> Can be human-in-loop
  - Hour level or above -> Can be pure manual

[ ] Does this decision have legal/compliance implications?
  - Yes -> Lean toward manual
  - No -> Lean toward automation

[ ] How much capital does this decision affect?
  - Small amount -> Can automate
  - Large amount -> Needs human confirmation

[ ] Does this decision require contextual judgment?
  - Requires integrating multiple information sources -> Lean toward manual
  - Rules are clear -> Lean toward automation

[ ] What's the cost of error?
  - Low cost, recoverable -> Lean toward automation
  - High cost, unrecoverable -> Lean toward manual
```

---

## 人機協作最佳實踐

### 1. 明確分工

```
System's Responsibilities:
- Collect and process information
- Execute rule-based operations
- Provide decision recommendations
- Record all operations

Human's Responsibilities:
- Set goals and constraints
- Handle exception situations
- Make value judgments
- Bear ultimate responsibility
```

### 2. 設計良好的互動

```
Bad Interaction:
  "Execute? [Yes/No]"

Good Interaction:
  "Detected the following situation:
   - Market state: Crisis
   - Current drawdown: 8%
   - Recommended action: Reduce to 50%

   Historical similar situations:
   - 2020.03: Deleveraging avoided 15% loss
   - 2018.12: Deleveraging missed 5% rebound

   Options:
   1. Execute deleveraging (recommended)
   2. Don't execute, continue monitoring
   3. Modify deleverage target
   4. Discuss with team"
```

### 3. 信任，但要驗證

```
Trust the System:
- In normal conditions, execute per system recommendation
- Don't frequently override system based on "feelings"

Verify the System:
- Regularly audit quality of system decisions
- Track results of human override vs. following system
- If human override win rate < 50%, consider reducing intervention
```

### 4. 保留緊急通道

```
No matter how automated, always need:

[ ] One-click stop button
[ ] Emergency contact list
[ ] Disaster recovery process
[ ] Backup operation methods (like manual order entry)
[ ] 24/7 on-call arrangements (at least during important periods)
```

---

## 自動化成熟度評估

評估你的系統的自動化程度：

| 維度 | Level 1 | Level 2 | Level 3 | Level 4 | Level 5 |
|------|---------|---------|---------|---------|---------|
| Execution | 手動落單 | 半自動 | 自動 + 確認 | 全自動 | 智能拆單 |
| Monitoring | 人手巡查 | 定時報告 | 實時 dashboard | 自動警報 | 預測性警報 |
| 風險控制 | 事後檢查 | 門檻警報 | 自動降級 | 自動 circuit breaker | 智能風險控制 |
| 異常處理 | 完全人手 | 基於規則 | 人機協作 | 自動 + 升級處理 | 自我修復 |

**建議的演進路徑**：
1. 先實現 execution 層的自動化
2. 再做 monitoring 和警報
3. 再做基於規則的風險控制
4. 最後處理異常處理
5. 始終保留人類 override 通道

---

## 總結

```
Automation Golden Rules:

1. Automate what can be automated (efficiency)
2. Irreversible decisions need human (safety)
3. Always maintain human intervention channel (emergency)
4. Human decisions need accountability mechanism (responsibility)
5. Regularly audit automated decision quality (improvement)

Ultimate Goal:
  Not to replace humans, but to amplify human capability
  Let systems handle tedious tasks, let humans handle important ones
```

---

> **謹記**：全自動系統在正常時期非常高效，在異常時期非常危險。最好的系統是人機協作：機器負責速度和規模，人類負責判斷和責任。
