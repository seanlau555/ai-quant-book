# 背景：Alpha 與 Beta

> 在量化投資中，Alpha (α) 代表「超額回報」，Beta (β) 代表「市場基準回報」。理解兩者的關係，是構建 hedge 策略和風險管理的核心。

---

## 1. 核心概念定義

### 1.1 Beta (β)：系統性風險與回報
**Beta** 衡量一個 portfolio 相對於市場（benchmark）的波動敏感度。它反映你從「市場上升」中獲得的那部分利潤，同時也反映你所承擔的結構性風險。

- **公式**：$\beta_i = \frac{Cov(r_i, r_m)}{Var(r_m)}$
  - $r_i$：資產回報
  - $r_m$：市場 benchmark 回報
- **基準數值**：
  - **β = 1**：與市場同步（例如 S&P 500 指數基金）
  - **β > 1**：進取型（通常是增長股、科技股）
  - **β < 1**：防守型（通常是公用事業股、必需消費品股）
  - **β < 0**：負相關（罕見，例如反向槓桿 ETF）
  - **β ≈ 0**：**Market Neutral**，回報不隨市場波動。

### 1.2 Alpha (α)：超額回報
**Alpha** 是經風險調整後（剔除 Beta 影響後）的純利潤。它被視為基金經理的「技術」，或模型的「獨門秘方」。

- **現代定義**：無法由已知 factor（例如市值、價值、momentum）解釋的回報。
- **現況**：隨着市場效率提高，傳統 alpha 正逐漸衰減（Alpha Decay），並演變為 benchmark 回報。

---

## 2. CAPM 模型與公式

經典的資本資產定價模型（CAPM）揭示了回報的構成：

$$E(R_i) = R_f + \beta_i (E(R_m) - R_f) + \alpha_i$$

- $R_f$：無風險利率
- $E(R_m) - R_f$：市場風險溢價
- **Alpha 的本質**：實際回報與 CAPM 預測回報之間的差額。

---

## 3. 演進：從 Alpha 到 Smart Beta

現代量化投資進一步拆解回報：

1. **傳統 Beta**：純粹的市場波動。
2. **Smart Beta（Factor 回報）**：介乎 α 與 β 之間。透過系統性地持有某些 factor（例如 Size、Value、Momentum）的 exposure 而取得的回報。
3. **Pure Alpha**：真正的超額技術（例如高頻套利、另類數據挖掘、AI 深層模式識別）。

| 維度 | Beta (β) | Smart Beta | Alpha (α) |
| :--- | :--- | :--- | :--- |
| **回報來源** | 整體市場表現 | Factor exposure | 獨有資訊／算法 |
| **成本** | 極低（指數基金） | 相對較低 | 高（hedge fund） |
| **Scalability** | 極高 | 高 | 有限 |
| **透明度** | 完全透明 | 規則透明 | 黑箱／不公開 |

---

## 4. 中美市場差異

| 特徵 | A 股（中國）市場 | 美國股票市場 |
| :--- | :--- | :--- |
| **Alpha 環境** | **Alpha 豐富**。散戶眾多，定價欠缺效率。 | **Beta 主導**。機構化程度高，alpha 難尋。 |
| **難度** | 較高，但相對穩定（波動大）。 | 極高（高度有效的市場）。 |
| **Neutral 策略** | 有效，但受限於 hedge 工具的成本。 | 工具極為豐富，liquidity 極佳。 |

---

## 5. Python 實現：計算 Alpha 與 Beta

使用 `statsmodels` 進行線性回歸：

```python
import numpy as np
import pandas as pd
import statsmodels.api as sm

def calculate_alpha_beta(portfolio_returns, market_returns, rf_rate=0.02/252):
    """
    Calculate daily Alpha and Beta
    """
    # 1. Calculate excess returns
    y = portfolio_returns - rf_rate
    x = market_returns - rf_rate

    # 2. Linear regression: y = alpha + beta * x
    x = sm.add_constant(x)
    model = sm.OLS(y, x).fit()

    alpha = model.params[0]
    beta = model.params[1]

    return alpha, beta

# Example usage
# alpha_daily, beta = calculate_alpha_beta(returns_df['strategy'], returns_df['benchmark'])
# print(f"Daily Alpha: {alpha_daily:.4f}, Beta: {beta:.2f}")
```

---

## 6. 頂級量化基金追求什麼

- **Renaissance（Medallion Fund）**：以**低 Beta、極高 Alpha** 聞名。其邏輯是尋找不隨市場波動的「statistical arbitrage」機會。
- **Market Neutral 策略**：透過 long 一個 alpha portfolio，同時 short 相應數量的指數期貨（Beta → 零），無論市場上升還是下跌，都可獲得穩定利潤。

---

> **總結**：對散戶而言，應專注於從 Beta 獲取長期增長；對量化研究員而言，核心任務是從原始數據中挖掘具有預測力的 **alpha 信號**。
