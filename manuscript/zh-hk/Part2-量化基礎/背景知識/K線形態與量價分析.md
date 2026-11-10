# K 線形態與量價分析

> **核心要點**：K 線形態與量價關係是傳統技術分析的基石，但單獨使用時預測力有限。正確的做法是將它們用作 feature engineering 的輸入，而非直接的交易 signal。

---

## 為甚麼要理解這些？

如果你用過任何交易軟件（TradingView、Thinkorswim、Interactive Brokers），一定見過各式各樣的 K 線形態標註和成交量柱狀圖。不少交易員依靠這些視覺形態來作決策。

**作為量化從業者，你需要理解這些概念，原因如下**：

1. **溝通語言**：與交易員和分析員交流時的通用詞彙
2. **Feature engineering**：這些形態可以量化，成為 ML 模型的輸入 feature
3. **策略評估**：了解傳統方法的局限，有助你設計更好的系統
4. **市場心理**：K 線形態反映買家與賣家之間的角力

---

## 1. K 線基礎

### 1.1 K 線的組成

單根 K 線包含四個價格點：

![Candlestick Anatomy](assets/candlestick-anatomy.svg)

### 1.2 K 線的資訊內容

| 組成部分 | 反映的資訊 |
|----------|------------|
| 實體長度 | 買方/賣方主導的強度 |
| 上影線 | 來自上方的賣壓 |
| 下影線 | 來自下方的買盤支持 |
| 實體位置 | 日內買賣角力的結果 |

---

## 2. 常見的單根 K 線形態

### 2.1 Doji

```
    │
   ─┼─  ← Open ≈ Close
    │
```

**特徵**：實體極小或沒有實體，上下影線長度可以不同

| 類型 | 形狀 | 含義 |
|------|------|------|
| 標準 Doji | 上下影線相近 | 買賣雙方均衡，猶豫不決 |
| Dragonfly Doji | 長下影線，無上影線 | 下方有強力支持 |
| Gravestone Doji | 長上影線，無下影線 | 上方有強力阻力 |

**傳統解讀**：可能出現趨勢反轉的 signal
**實際效果**：單獨使用時，預測準確率約 50-55%

### 2.2 Hammer 與 Hanging Man

```
   │
   █  ← Small body at top
   │
   │ ← Long lower shadow (at least 2x body length)
```

**分別在於所處的背景**：
- **Hammer**：出現在下跌趨勢的底部 → 可能向上反轉
- **Hanging Man**：出現在上升趨勢的頂部 → 可能向下反轉

### 2.3 Shooting Star

```
   │ ← Long upper shadow
   │
   █  ← Small body at bottom
```

**含義**：出現在上升趨勢的頂部，顯示上方阻力強勁，可能見頂

### 2.4 大 bullish candle與大 bearish candle

| 形態 | 特徵 | 含義 |
|------|------|------|
| 大 bullish candle | 實體長，影線短 | 多方強勢主導 |
| 大 bearish candle | 實體長，影線短 | 空方強勢主導 |

**量化定義**：實體長度 > 近期 ATR 的 1.5 倍

---

## 3. 常見的多根 K 線形態

### 3.1 Engulfing Pattern

```
Bullish Engulfing (at downtrend):    Bearish Engulfing (at uptrend):

    █                                      │
    █  First small bearish               ███ First small bullish
   ┌─┐                                   ┌─┐
   │ │ Second large bullish              █ █ Second large bearish
   │ │ Completely covers first           █ █ Completely covers first
   └─┘                                   └─┘
```

**條件**：
1. 第二根 K 線的實體完全覆蓋第一根
2. 兩根 K 線顏色相反
3. 出現在明確的趨勢中

### 3.2 Morning Star 與 Evening Star

```
Morning Star (bottom reversal):

█     ← First: Large bearish
█
█
  ┼   ← Second: Small body (doji or small candle), gaps down
 ┌┐
 ││   ← Third: Large bullish, closes well into first candle's body
 ││
```

**Evening Star**：Morning Star 的鏡像，出現在頂部

### 3.3 Three Black Crows 與 Three White Soldiers

| 形態 | 結構 | 含義 |
|------|------|------|
| Three Black Crows | 連續三根大 bearish candle，每根開市更低、收市更低 | 強烈看淡 |
| Three White Soldiers | 連續三根大 bullish candle，每根開市更高、收市更高 | 強烈看好 |

---

## 4. 成交量分析基礎

### 4.1 成交量的含義

成交量 = 買賣雙方成交的股數

**核心原則**：價格是方向，成交量是動能

| 量價關係 | 含義 | 可靠性 |
|----------|------|--------|
| 價升 + 量升 | 升勢有資金支持 | 趨勢很可能延續 |
| 價升 + 量跌 | 升勢缺乏動能 | 趨勢可能衰竭 |
| 價跌 + 量升 | 恐慌拋售 | 可能加速下跌，或接近底部 |
| 價跌 + 量跌 | 跌勢動能減弱 | 可能企穩 |

### 4.2 關鍵的量價形態

#### 放量突破

```
Price ─────────┐
              └──────── Breakout
Volume       ████████ ← Volume significantly increases (>1.5× average)
```

**含義**：突破獲資金確認，可信度高

#### 縮量回調

```
Price    /\
       /  \  ← Pullback
      /
Volume ██  █ ← Volume shrinks during pullback
```

**含義**：回調屬正常的獲利回吐，並非趨勢反轉

#### 量價背離

```
Price   /\  /\  /\ ← Price makes new highs
      /  \/  \/
Volume ██ █  ▪ ← Volume decreases
```

**含義**：升勢動能衰竭，可能見頂（與 MACD 背離相似）

### 4.3 常見的成交量 Indicator

| Indicator | 公式 | 用途 |
|-----------|------|------|
| **Volume Ratio** | 當前成交量 / N 日平均成交量 | 衡量每日交易活躍程度 |
| **Turnover Rate** | 成交量 / 流通股數 | 衡量股票換手率 |
| **OBV** | 累計成交量（上升日加、下跌日減、持平日不變） | 資金流向 |
| **VWAP** | Σ(Price × Volume) / ΣVolume | 機構交易基準 |

---

## 5. 形態識別的實證研究

### 5.1 學術研究結論

多項學術研究測試過 K 線形態的預測力：

| 研究 | 樣本 | 結論 |
|------|------|------|
| Lo, Mamaysky & Wang (2000) | 美國股票 1962-1996 | 部分形態在統計上顯著，但經濟意義有限 |
| Marshall, Young & Rose (2006) | 35 個市場 | 大部分形態沒有預測力 |
| Caginalp & Laurent (1998) | S&P 500 成分股 | 某些形態在特定條件下有效 |

**總結**：
- 僅憑形態的勝率通常為 50-55%
- 扣除交易成本後，大部分形態策略都不賺錢
- 形態的有效性可能隨時間衰減（被市場學會）

### 5.2 為甚麼形態「看起來有效」

1. **Confirmation bias**：人們記得成功的例子，忘記失敗的
2. **Hindsight bias**：回看過去的圖表，總能找到「完美」的形態
3. **Survivorship bias**：只有成功的交易員才會分享他們的故事
4. **定義模糊**：怎樣才算「標準」的 hammer？沒有精確的定義

### 5.3 形態在何時可能有效？

研究顯示，形態在以下條件下表現較佳：

- **配合成交量**：有成交量確認的形態更可靠
- **處於關鍵價位**：出現在 support/resistance 附近
- **市場環境**：處於特定的 volatility regime
- **時間框架**：較長的時間框架（日線、週線）比分鐘圖更可靠

---

## 6. 量化實作：將形態作為 Feature

### 6.1 量化形態的挑戰

傳統的形態分析屬於「圖案配對」，但定義模糊：
- 「長下影線」——多長才算長？
- 「小實體」——多小才算小？
- 「處於趨勢中」——如何定義趨勢？

**解決方法**：將形態轉換為連續的 feature，而非離散的 signal

### 6.2 Feature Engineering 範例

```python
def calculate_candlestick_features(df):
    """
    Convert candlestick patterns to continuous features
    df needs: open, high, low, close, volume columns
    """
    # Basic calculations
    df['body'] = df['close'] - df['open']
    df['body_abs'] = df['body'].abs()
    df['upper_shadow'] = df['high'] - df[['open', 'close']].max(axis=1)
    df['lower_shadow'] = df[['open', 'close']].min(axis=1) - df['low']
    df['range'] = df['high'] - df['low']

    # Feature 1: Body ratio (0=doji, 1=no shadows)
    df['body_ratio'] = df['body_abs'] / df['range'].replace(0, np.nan)

    # Feature 2: Upper shadow ratio
    df['upper_shadow_ratio'] = df['upper_shadow'] / df['range'].replace(0, np.nan)

    # Feature 3: Lower shadow ratio
    df['lower_shadow_ratio'] = df['lower_shadow'] / df['range'].replace(0, np.nan)

    # Feature 4: Close position (where close sits in day's range)
    df['close_position'] = (df['close'] - df['low']) / df['range'].replace(0, np.nan)

    # Feature 5: Relative body size (compared to recent ATR)
    atr = calculate_atr(df, period=14)
    df['relative_body'] = df['body_abs'] / atr

    # Feature 6: Engulfing ratio (current body covers previous body)
    df['engulfing_ratio'] = df['body_abs'] / df['body_abs'].shift(1)

    return df

def calculate_volume_features(df, periods=[5, 20]):
    """
    Calculate volume-related features
    """
    for p in periods:
        # Volume ratio
        df[f'volume_ratio_{p}'] = df['volume'] / df['volume'].rolling(p).mean()

    # OBV (On-Balance Volume)
    df['obv'] = (np.sign(df['close'].diff()).fillna(0) * df['volume']).cumsum()

    # OBV trend (slope of OBV moving average)
    df['obv_slope'] = df['obv'].rolling(10).apply(
        lambda x: np.polyfit(range(len(x)), x, 1)[0]
    )

    # Price-volume correlation (recent price change vs volume correlation)
    df['price_volume_corr'] = df['close'].pct_change().rolling(20).corr(
        df['volume'].pct_change()
    )

    return df
```

### 6.3 Feature 使用建議

| Feature 類型 | 適合的模型 | 備註 |
|--------------|------------|------|
| 連續形態 feature | Tree models、Neural networks | 需要 normalization |
| 二元形態 label | Logistic regression、規則系統 | 定義必須一致 |
| 序列形態 | LSTM、Transformer | 需要時間窗口 |

---

## 7. Multi-Agent 視角

在 multi-agent 系統中，K 線形態與成交量分析可以這樣定位：

| Agent | 用法 |
|-------|------|
| **Trend Agent** | 用大 K 線判斷趨勢強度 |
| **Mean Reversion Agent** | 用 doji、hammer 識別潛在反轉點 |
| **Regime Agent** | 量價背離作為市場 regime 轉變的其中一個 signal |
| **Risk Agent** | 異常的成交量激增作為風險預警 signal |

**關鍵原則**：
- 形態 feature 只是其中一項輸入，而非唯一依據
- 結合其他 feature（MACD、RSI、基本面）作綜合判斷
- Agent 輸出的是概率，而非確定性的 signal

---

## 8. 常見誤區

### 誤區 1：形態是「秘密暗號」

> 「學會看 K 線形態就能保證持續獲利」

**真相**：如果形態真的那麼有效，用的人越多，效果就越差（reflexivity）

### 誤區 2：形態在任何地方都有效

> 「Hammer 一定代表見底」

**真相**：同一個形態在不同的市場環境和不同的標的上，表現可能截然不同

### 誤區 3：成交量永遠有意義

> 「成交量大代表機構在買入」

**真相**：成交量大可能是散戶恐慌、algorithmic trading、index rebalancing 等多種原因

### 誤區 4：形態可以精確預測

> 「Evening star 代表明天一定下跌」

**真相**：形態最多只提供概率上的 edge（如果有的話），而非確定性的預測

---

## 重點回顧

- [x] K 線形態是買賣雙方角力的視覺呈現
- [x] 僅憑形態的預測力有限（勝率 50-55%）
- [x] 成交量確認可以提高形態的可靠性
- [x] 正確的用法是將形態量化為 ML feature，而非直接作為 signal
- [x] 在 multi-agent 系統中，形態只是眾多輸入之一

---

## 延伸閱讀

- [第04課：技術指標的真實角色](../第04課：技術指標的真實角色.md) - MACD、RSI 等指標詳解
- [第09課：監督學習在量化中的應用](../../Part3-機器學習/第09課：監督學習在量化中的應用.md) - 如何將 feature 輸入 ML 模型
- [背景知識：特徵工程常見陷阱](../../Part3-機器學習/背景知識/特徵工程常見陷阱.md) - 避免 feature engineering 中的錯誤

---

## 參考文獻

- Nison, S. (1991). *Japanese Candlestick Charting Techniques*
- Lo, A., Mamaysky, H., & Wang, J. (2000). Foundations of Technical Analysis. *Journal of Finance*
- Murphy, J. (1999). *Technical Analysis of the Financial Markets*
