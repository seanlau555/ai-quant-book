# 背景知識：Alpha 與 Beta

> 在量化投資中，Alpha (α) 代表“超額收益”，Beta (β) 代表“市場基準收益”。理解兩者的關係是構建對沖策略和風險管理的核心。

---

## 一、 核心概念定義

### 1.1 Beta (β): 系統性風險與收益
**Beta** 衡量投資組合相對於大市（基準）的波動敏感度。它反映了你從“市場上升”中分到的那部分利潤，也反映了你承擔的結構性風險。

- **計算公式**: $\beta_i = \frac{Cov(r_i, r_m)}{Var(r_m)}$
  - $r_i$: 資產收益率
  - $r_m$: 市場基準收益率
- **基準值**:
  - **β = 1**: 與市場同步（如滬深300指數基金）
  - **β > 1**: 激進型（多為成長股、科技股）
  - **β < 1**: 防禦型（多為公用事業、大消費）
  - **β < 0**: 負相關（極少數，如反向槓桿ETF）
  - **β ≈ 0**: **市場中性 (Market Neutral)**，收益不隨大市波動。

### 1.2 Alpha (α): 超額收益
**Alpha** 是經過風險調整（排除 Beta 影響）後的純粹盈利。它被視為投資經理的“技能”或模型的“獨門秘籍”。

- **現代定義**: 無法被已知因子（如市值、價值、動量）解釋的收益。
- **現狀**: 隨着市場效率提高，傳統的 Alpha 正在逐漸衰減（Alpha Decay）並演變為基準收益。

---

## 二、 CAPM 模型與公式

經典資本資產定價模型 (CAPM) 揭示了回報的構成：

$$E(R_i) = R_f + \beta_i (E(R_m) - R_f) + \alpha_i$$

- $R_f$: 無風險利率 (Risk-free rate)
- $E(R_m) - R_f$: 市場風險溢價
- **Alpha 的本質**: 實際回報率與 CAPM 預測回報率之間的差額。

---

## 三、 演進：從 Alpha 到 Smart Beta

現代量化將收益進一步拆解：

1.  **Traditional Beta**: 純大市波動。
2.  **Smart Beta (因子收益)**: 介於 α 與 β 之間。通過系統性暴露於某些因子（如：市值 Size, 價值 Value, 動量 Momentum）獲得的收益。
3.  **Pure Alpha**: 真正的超額技能（如：高頻套利、另類數據挖掘、AI 深度模式識別）。

| 維度 | Beta (β) | Smart Beta | Alpha (α) |
| :--- | :--- | :--- | :--- |
| **收益來源** | 市場整體表現 | 因子暴露 | 獨特信息/算法 |
| **成本** | 極低 (指數基金) | 較低 | 高 (對沖基金) |
| **可規模化** | 極高 | 高 | 有限 |
| **透明度** | 完全透明 | 規則透明 | 黑盒/非公開 |

---

## 四、 中美市場差異

| 特點 | A股市場 | 美股市場 |
| :--- | :--- | :--- |
| **Alpha 環境** | **Alpha 豐富**。散戶多，定價不效率。 | **Beta 驅動**。機構化程度高，Alpha 難求。 |
| **獲取難度** | 較高但相對穩健（波動大）。 | 極高（高度有效市場）。 |
| **中性策略** | 雖有效但受限於對沖工具成本。 | 工具極其豐富，流動性極佳。 |

---

## 五、 Python 實戰：計算 Alpha 和 Beta

使用 `statsmodels` 進行線性迴歸計算：

```python
import numpy as np
import pandas as pd
import statsmodels.api as sm

def calculate_alpha_beta(portfolio_returns, market_returns, rf_rate=0.02/252):
    """
    計算日頻 Alpha 和 Beta
    """
    # 1. 計算超額收益
    y = portfolio_returns - rf_rate
    x = market_returns - rf_rate
    
    # 2. 線性迴歸: y = alpha + beta * x
    x = sm.add_constant(x)
    model = sm.OLS(y, x).fit()
    
    alpha = model.params[0]
    beta = model.params[1]
    
    return alpha, beta

# 示例數據
# alpha_daily, beta = calculate_alpha_beta(returns_df['strategy'], returns_df['benchmark'])
# print(f"Daily Alpha: {alpha_daily:.4f}, Beta: {beta:.2f}")
```

---

## 六、 頂級量化基金的追求

- **Renaissance (文藝復興)**: Medallion 基金以 **低 Beta, 極高 Alpha** 聞名。其邏輯是尋找不隨大市波動的“統計學套利”機會。
- **量化中性策略**: 通過做多 Alpha 組合，同時做空對應規模的期指（Beta 歸零），從而實現在不論大市升跌的情況下都能穩定獲利。

---

> **總結**: 對於散戶，關注 Beta 帶來的長期增長；對於量化研究者，核心任務是利用原始數據挖掘具有預測能力的 **Alpha 信號**。