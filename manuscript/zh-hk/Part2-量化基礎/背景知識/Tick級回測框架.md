# 背景知識：Tick 級回測框架

> "用日線回測分鐘策略，就像用世界地圖導航城市街道——精度完全不夠。"

---

## 一、為什麼需要 Tick 級回測？

### 1.1 不同數據粒度的適用場景

| 數據粒度 | 適用策略 | 精度 | 數據量 |
|---------|---------|------|-------|
| 日線 | 趨勢跟蹤、價值投資 | 低 | 小 |
| 分鐘線 | 日內動量、均值迴歸 | 中 | 中 |
| Tick/L2 | 做市、高頻套利、精確執行 | 高 | 大 |

### 1.2 分鐘線回測的侷限

```
場景：5 分鐘 K 線顯示
  Open: $100.00
  High: $100.50
  Low:  $99.80
  Close: $100.20

問題：這 5 分鐘內，價格走勢是什麼樣的？

可能性 1: 先漲後跌再漲
  $100.00 → $100.50 → $99.80 → $100.20

可能性 2: 先跌後漲
  $100.00 → $99.80 → $100.50 → $100.20

可能性 3: 劇烈震盪
  多次觸及高低點

你的止損在 $99.90 會被觸發嗎？用 K 線無法確定。
```

### 1.3 Tick 數據能回答的問題

| 問題 | K 線回測 | Tick 回測 |
|-----|---------|----------|
| 限價單能否成交？ | 只能猜測 | 精確判斷 |
| 成交的具體時間？ | 不知道 | 毫秒級精度 |
| 排隊位置影響？ | 無法模擬 | 可以估算 |
| 價格路徑依賴的止損？ | 不準確 | 準確 |
| 滑點的真實分佈？ | 固定假設 | 實際計算 |

---

## 二、Tick 數據結構

### 2.1 逐筆成交數據（Trade Tick）

```python
@dataclass
class TradeTick:
    """逐筆成交"""
    timestamp: float      # Unix 時間戳（秒，含小數部分表示毫秒）
    symbol: str           # 標的代碼
    price: float          # 成交價
    size: float           # 成交量
    side: str             # 'buy' 或 'sell'（主動方向）
    trade_id: str         # 成交編號
```

**示例數據**：
```
timestamp,symbol,price,size,side,trade_id
1704067200.123,AAPL,185.50,100,buy,T001
1704067200.156,AAPL,185.51,50,buy,T002
1704067200.189,AAPL,185.50,200,sell,T003
1704067200.201,AAPL,185.49,150,sell,T004
```

### 2.2 訂單簿快照（Order Book Snapshot）

```python
@dataclass
class OrderBookSnapshot:
    """訂單簿快照"""
    timestamp: float
    symbol: str
    bids: List[Tuple[float, float]]  # [(價格, 數量), ...]
    asks: List[Tuple[float, float]]  # [(價格, 數量), ...]

    @property
    def mid_price(self) -> float:
        if self.bids and self.asks:
            return (self.bids[0][0] + self.asks[0][0]) / 2
        return 0.0

    @property
    def spread(self) -> float:
        if self.bids and self.asks:
            return self.asks[0][0] - self.bids[0][0]
        return float('inf')
```

### 2.3 訂單簿增量更新（Order Book Delta）

```python
@dataclass
class OrderBookDelta:
    """訂單簿增量更新"""
    timestamp: float
    symbol: str
    side: str           # 'bid' 或 'ask'
    price: float
    size: float         # 新數量（0 表示刪除該價位）
    action: str         # 'add', 'modify', 'delete'
```

---

## 三、事件驅動 vs 向量化回測

### 3.1 向量化回測（Vectorized）

**特點**：
- 用 NumPy/Pandas 批量計算
- 速度快（秒級完成數年數據）
- 適合簡單策略

**侷限**：
- 難以模擬訂單狀態
- 無法處理複雜執行邏輯
- 路徑依賴的邏輯難以表達

```python
# 向量化回測示例
import pandas as pd
import numpy as np

def vectorized_backtest(df: pd.DataFrame,
                        signal_col: str,
                        price_col: str = 'close') -> pd.Series:
    """
    簡單向量化回測
    signal_col: 1=做多, -1=做空, 0=空倉
    """
    # 信號延遲一期（避免未來信息）
    position = df[signal_col].shift(1).fillna(0)

    # 計算收益
    returns = df[price_col].pct_change()
    strategy_returns = position * returns

    # 累計收益
    cumulative = (1 + strategy_returns).cumprod()

    return cumulative
```

### 3.2 事件驅動回測（Event-Driven）

**特點**：
- 逐事件處理
- 可以精確模擬訂單生命週期
- 適合複雜策略和 Tick 數據

**代價**：
- 速度較慢
- 代碼複雜度高

```
事件流:
  t=0.001: MarketData(AAPL, bid=185.50, ask=185.51)
  t=0.002: Signal(BUY, size=100)
  t=0.002: OrderSubmit(LIMIT, 185.50, 100)
  t=0.005: MarketData(AAPL, bid=185.49, ask=185.50)
  t=0.010: Trade(185.50, 50)  ← 部分成交
  t=0.015: MarketData(AAPL, bid=185.50, ask=185.51)
  t=0.020: Trade(185.50, 50)  ← 剩餘成交
  t=0.020: OrderFilled(全部成交)
```

---

## 四、事件驅動框架實現

### 4.1 核心組件

![事件驅動回測架構](../assets/event-driven-backtest-architecture.svg)

### 4.2 事件類型定義

```python
from enum import Enum
from dataclasses import dataclass
from typing import Optional, List

class EventType(Enum):
    MARKET_DATA = "market_data"
    SIGNAL = "signal"
    ORDER = "order"
    FILL = "fill"
    CANCEL = "cancel"

@dataclass
class Event:
    """基礎事件"""
    timestamp: float
    event_type: EventType

@dataclass
class MarketDataEvent(Event):
    """市場數據事件"""
    symbol: str
    bid: float
    ask: float
    bid_size: float
    ask_size: float
    last_price: Optional[float] = None
    last_size: Optional[float] = None

    def __post_init__(self):
        self.event_type = EventType.MARKET_DATA

@dataclass
class SignalEvent(Event):
    """策略信號事件"""
    symbol: str
    direction: int      # 1=買, -1=賣, 0=平倉
    strength: float     # 信號強度 [0, 1]

    def __post_init__(self):
        self.event_type = EventType.SIGNAL

@dataclass
class OrderEvent(Event):
    """訂單事件"""
    symbol: str
    order_type: str     # 'market', 'limit'
    side: str           # 'buy', 'sell'
    quantity: float
    price: Optional[float] = None  # 限價單價格
    order_id: Optional[str] = None

    def __post_init__(self):
        self.event_type = EventType.ORDER

@dataclass
class FillEvent(Event):
    """成交事件"""
    symbol: str
    order_id: str
    side: str
    quantity: float
    price: float
    commission: float

    def __post_init__(self):
        self.event_type = EventType.FILL
```

### 4.3 事件隊列

```python
import heapq
from typing import List

class EventQueue:
    """優先級事件隊列（按時間排序）"""

    def __init__(self):
        self._queue: List[tuple] = []
        self._counter = 0  # 用於相同時間的事件排序

    def push(self, event: Event):
        """添加事件"""
        heapq.heappush(self._queue,
                       (event.timestamp, self._counter, event))
        self._counter += 1

    def pop(self) -> Optional[Event]:
        """取出最早的事件"""
        if self._queue:
            _, _, event = heapq.heappop(self._queue)
            return event
        return None

    def is_empty(self) -> bool:
        return len(self._queue) == 0

    def peek(self) -> Optional[Event]:
        """查看最早的事件（不移除）"""
        if self._queue:
            return self._queue[0][2]
        return None
```

### 4.4 執行模擬器

```python
class TickExecutionSimulator:
    """Tick 級執行模擬器"""

    def __init__(self,
                 commission_rate: float = 0.0003,
                 latency_ms: float = 1.0):
        self.commission_rate = commission_rate
        self.latency_ms = latency_ms
        self.pending_orders = {}
        self.order_counter = 0

    def submit_order(self, order: OrderEvent,
                     current_book: OrderBookSnapshot) -> List[Event]:
        """
        提交訂單，返回產生的事件
        """
        events = []
        self.order_counter += 1
        order.order_id = f"ORD_{self.order_counter:06d}"

        # 模擬延遲
        exec_time = order.timestamp + self.latency_ms / 1000

        if order.order_type == 'market':
            # 市價單立即嘗試成交
            fill = self._execute_market_order(order, current_book, exec_time)
            if fill:
                events.append(fill)
        else:
            # 限價單進入掛單隊列
            self.pending_orders[order.order_id] = {
                'order': order,
                'remaining': order.quantity,
                'submit_time': order.timestamp
            }

        return events

    def on_market_data(self, md: MarketDataEvent) -> List[Event]:
        """
        市場數據更新時檢查掛單
        """
        events = []

        for order_id, pending in list(self.pending_orders.items()):
            order = pending['order']
            remaining = pending['remaining']

            # 檢查是否可成交
            fill_qty, fill_price = self._check_limit_fill(
                order, remaining, md
            )

            if fill_qty > 0:
                fill = FillEvent(
                    timestamp=md.timestamp,
                    symbol=order.symbol,
                    order_id=order_id,
                    side=order.side,
                    quantity=fill_qty,
                    price=fill_price,
                    commission=fill_qty * fill_price * self.commission_rate
                )
                events.append(fill)

                pending['remaining'] -= fill_qty
                if pending['remaining'] <= 0:
                    del self.pending_orders[order_id]

        return events

    def _execute_market_order(self, order: OrderEvent,
                              book: OrderBookSnapshot,
                              exec_time: float) -> Optional[FillEvent]:
        """執行市價單"""
        if order.side == 'buy':
            if not book.asks:
                return None
            # 簡化：取賣一價
            fill_price = book.asks[0][0]
        else:
            if not book.bids:
                return None
            fill_price = book.bids[0][0]

        return FillEvent(
            timestamp=exec_time,
            symbol=order.symbol,
            order_id=order.order_id,
            side=order.side,
            quantity=order.quantity,
            price=fill_price,
            commission=order.quantity * fill_price * self.commission_rate
        )

    def _check_limit_fill(self, order: OrderEvent,
                          remaining: float,
                          md: MarketDataEvent) -> tuple:
        """檢查限價單是否可成交"""
        if order.side == 'buy':
            # 買單：如果賣一價 <= 限價，可成交
            if md.ask <= order.price:
                fill_qty = min(remaining, md.ask_size)
                return fill_qty, md.ask
        else:
            # 賣單：如果買一價 >= 限價，可成交
            if md.bid >= order.price:
                fill_qty = min(remaining, md.bid_size)
                return fill_qty, md.bid

        return 0, 0
```

### 4.5 回測引擎

```python
class TickBacktestEngine:
    """Tick 級回測引擎"""

    def __init__(self, strategy, execution_sim: TickExecutionSimulator):
        self.strategy = strategy
        self.execution = execution_sim
        self.event_queue = EventQueue()
        self.portfolio = Portfolio()
        self.current_book = None

    def load_data(self, data_source):
        """加載 Tick 數據到事件隊列"""
        for tick in data_source:
            if isinstance(tick, TradeTick):
                event = self._trade_to_event(tick)
            elif isinstance(tick, OrderBookSnapshot):
                event = self._book_to_event(tick)
            self.event_queue.push(event)

    def run(self) -> dict:
        """運行回測"""
        while not self.event_queue.is_empty():
            event = self.event_queue.pop()
            self._process_event(event)

        return self._calculate_results()

    def _process_event(self, event: Event):
        """處理單個事件"""
        if event.event_type == EventType.MARKET_DATA:
            self._on_market_data(event)
        elif event.event_type == EventType.SIGNAL:
            self._on_signal(event)
        elif event.event_type == EventType.ORDER:
            self._on_order(event)
        elif event.event_type == EventType.FILL:
            self._on_fill(event)

    def _on_market_data(self, md: MarketDataEvent):
        """處理市場數據"""
        # 更新當前訂單簿
        self.current_book = md

        # 檢查掛單成交
        fills = self.execution.on_market_data(md)
        for fill in fills:
            self.event_queue.push(fill)

        # 策略處理
        signal = self.strategy.on_data(md, self.portfolio)
        if signal:
            self.event_queue.push(signal)

    def _on_signal(self, signal: SignalEvent):
        """處理策略信號"""
        order = self.strategy.signal_to_order(signal, self.portfolio)
        if order:
            self.event_queue.push(order)

    def _on_order(self, order: OrderEvent):
        """處理訂單"""
        fills = self.execution.submit_order(order, self.current_book)
        for fill in fills:
            self.event_queue.push(fill)

    def _on_fill(self, fill: FillEvent):
        """處理成交"""
        self.portfolio.update(fill)
        self.strategy.on_fill(fill)

    def _calculate_results(self) -> dict:
        """計算回測結果"""
        return {
            'total_return': self.portfolio.total_return,
            'sharpe_ratio': self.portfolio.sharpe_ratio,
            'max_drawdown': self.portfolio.max_drawdown,
            'total_trades': self.portfolio.trade_count,
            'total_commission': self.portfolio.total_commission,
            'equity_curve': self.portfolio.equity_curve
        }
```

---

## 五、訂單排隊模擬

### 5.1 為什麼排隊位置重要？

```
場景：你在 $100.00 掛了限價買單

盤口：
  買一: $100.00 × 10,000 股（你排在第 5,000 位）

成交流：
  賣方市價賣出 3,000 股 → 前 3,000 人成交，你還差 2,000
  賣方市價賣出 1,500 股 → 前 4,500 人成交，你還差 500
  價格跳到 $100.05     → 你的單子永遠不會成交了

結論：即使價格"觸達"你的限價，你也未必能成交
```

### 5.2 排隊位置估算

```python
class QueuePositionEstimator:
    """排隊位置估算器"""

    def __init__(self, queue_position_pct: float = 0.5):
        """
        queue_position_pct: 假設你在隊列中的相對位置
                           0 = 隊首, 1 = 隊尾
        """
        self.queue_pct = queue_position_pct

    def estimate_queue_ahead(self,
                             order: OrderEvent,
                             book: OrderBookSnapshot) -> float:
        """估算排在你前面的訂單量"""
        if order.side == 'buy':
            # 找到你的價位在買盤的位置
            for price, size in book.bids:
                if price == order.price:
                    return size * self.queue_pct
            # 價格不在當前盤口，可能全部在你前面
            return float('inf')
        else:
            for price, size in book.asks:
                if price == order.price:
                    return size * self.queue_pct
            return float('inf')

    def update_queue_on_trade(self,
                              queue_ahead: float,
                              trade: TradeTick,
                              order: OrderEvent) -> float:
        """根據成交更新排隊位置"""
        if order.side == 'buy' and trade.side == 'sell':
            # 賣方主動成交，消耗買盤
            if trade.price == order.price:
                queue_ahead = max(0, queue_ahead - trade.size)
        elif order.side == 'sell' and trade.side == 'buy':
            if trade.price == order.price:
                queue_ahead = max(0, queue_ahead - trade.size)

        return queue_ahead

    def can_fill(self, queue_ahead: float, order_size: float) -> tuple:
        """判斷是否可以成交"""
        if queue_ahead <= 0:
            fill_qty = order_size
            return True, fill_qty
        return False, 0
```

### 5.3 完整的限價單模擬

```python
class RealisticLimitOrderSimulator:
    """考慮排隊的限價單模擬器"""

    def __init__(self,
                 queue_estimator: QueuePositionEstimator,
                 commission_rate: float = 0.0003):
        self.queue_est = queue_estimator
        self.commission = commission_rate
        self.orders = {}  # order_id -> order state

    def submit_limit_order(self,
                           order: OrderEvent,
                           book: OrderBookSnapshot) -> str:
        """提交限價單"""
        order_id = f"LMT_{len(self.orders):06d}"

        queue_ahead = self.queue_est.estimate_queue_ahead(order, book)

        self.orders[order_id] = {
            'order': order,
            'queue_ahead': queue_ahead,
            'remaining': order.quantity,
            'status': 'pending'
        }

        return order_id

    def on_trade(self, trade: TradeTick) -> List[FillEvent]:
        """處理市場成交，更新排隊位置"""
        fills = []

        for order_id, state in list(self.orders.items()):
            if state['status'] != 'pending':
                continue

            order = state['order']

            # 更新排隊位置
            state['queue_ahead'] = self.queue_est.update_queue_on_trade(
                state['queue_ahead'],
                trade,
                order
            )

            # 檢查是否可以成交
            can_fill, fill_qty = self.queue_est.can_fill(
                state['queue_ahead'],
                state['remaining']
            )

            if can_fill and fill_qty > 0:
                # 實際成交數量取決於對手方
                actual_fill = min(fill_qty, trade.size)

                fill = FillEvent(
                    timestamp=trade.timestamp,
                    symbol=order.symbol,
                    order_id=order_id,
                    side=order.side,
                    quantity=actual_fill,
                    price=order.price,
                    commission=actual_fill * order.price * self.commission
                )
                fills.append(fill)

                state['remaining'] -= actual_fill
                if state['remaining'] <= 0:
                    state['status'] = 'filled'

        return fills
```

---

## 六、性能優化

### 6.1 數據存儲格式

| 格式 | 讀取速度 | 壓縮率 | 隨機訪問 | 推薦場景 |
|-----|---------|-------|---------|---------|
| CSV | 慢 | 無 | 差 | 小數據、調試 |
| Parquet | 快 | 高 | 好 | 大規模回測 |
| HDF5 | 快 | 中 | 好 | 時序數據 |
| Arrow/Feather | 極快 | 中 | 好 | 內存映射 |

```python
# Parquet 示例
import pandas as pd

# 寫入
df.to_parquet('ticks.parquet', compression='snappy')

# 讀取（只加載需要的列）
df = pd.read_parquet('ticks.parquet',
                     columns=['timestamp', 'price', 'size'])
```

### 6.2 內存優化

```python
import numpy as np

# 使用更小的數據類型
dtype_mapping = {
    'price': np.float32,      # 4 bytes vs 8 bytes
    'size': np.int32,         # 4 bytes
    'side': np.int8,          # 1 byte (0=sell, 1=buy)
}

# 預分配數組
n_ticks = 1_000_000
prices = np.empty(n_ticks, dtype=np.float32)
sizes = np.empty(n_ticks, dtype=np.int32)
```

### 6.3 並行處理

```python
from concurrent.futures import ProcessPoolExecutor
from typing import List

def backtest_single_day(date: str, strategy_params: dict) -> dict:
    """單日回測"""
    # 加載當日數據
    # 運行回測
    # 返回結果
    pass

def parallel_backtest(dates: List[str],
                      strategy_params: dict,
                      n_workers: int = 4) -> List[dict]:
    """並行回測多日"""
    with ProcessPoolExecutor(max_workers=n_workers) as executor:
        futures = [
            executor.submit(backtest_single_day, date, strategy_params)
            for date in dates
        ]
        results = [f.result() for f in futures]

    return results
```

---

## 七、常見誤區

**誤區一：Tick 回測一定比分鐘回測更準**

不一定。如果：
- 策略本身是分鐘級的決策
- 沒有正確模擬排隊和滑點
- 數據質量有問題

那麼 Tick 回測可能引入更多噪音而非更多精度。

**誤區二：有了 Tick 數據就能做高頻策略**

Tick 數據是必要條件，不是充分條件。還需要：
- 低延遲執行能力
- 正確的手續費/返傭假設
- 考慮你的訂單對市場的影響

**誤區三：忽略數據清洗**

Tick 數據常見問題：
- 重複記錄
- 時間戳錯誤
- 價格異常（如負數、極端跳躍）
- 交易所維護期間的垃圾數據

```python
def clean_ticks(df: pd.DataFrame) -> pd.DataFrame:
    """清洗 Tick 數據"""
    # 去重
    df = df.drop_duplicates(subset=['timestamp', 'trade_id'])

    # 排序
    df = df.sort_values('timestamp')

    # 過濾異常價格
    median_price = df['price'].median()
    df = df[df['price'].between(median_price * 0.9,
                                 median_price * 1.1)]

    # 過濾異常成交量
    df = df[df['size'] > 0]

    return df
```

---

## 八、多智能體視角

Tick 級回測在多智能體系統中的角色：

![Tick 回測與 Agent 訓練](../assets/tick-backtest-agent-training.svg)

---

## 九、實用建議

### 9.1 漸進式採用

```
階段 1: 驗證策略邏輯
  - 使用分鐘/小時數據
  - 固定滑點假設
  - 快速迭代

階段 2: 精細化執行假設
  - 使用 Tick 數據回測關鍵信號
  - 根號滑點模型
  - 驗證策略是否仍然盈利

階段 3: 完整 Tick 回測
  - 訂單簿回放
  - 排隊模擬
  - 與實盤數據對比校準
```

### 9.2 關鍵指標對比

```python
def compare_granularity(minute_result: dict,
                        tick_result: dict) -> dict:
    """對比不同粒度回測結果"""
    return {
        'return_diff': tick_result['return'] - minute_result['return'],
        'sharpe_diff': tick_result['sharpe'] - minute_result['sharpe'],
        'fill_rate': tick_result.get('fill_rate', 1.0),
        'avg_slippage': tick_result.get('avg_slippage', 0),
        'verdict': 'tick_worse' if tick_result['return'] < minute_result['return'] * 0.8 else 'acceptable'
    }
```

---

## 十、總結

| 要點 | 說明 |
|-----|------|
| 適用場景 | 高頻策略、精確執行模擬、限價單策略 |
| 核心優勢 | 模擬排隊、精確滑點、路徑依賴邏輯 |
| 實現方式 | 事件驅動架構 |
| 關鍵挑戰 | 數據量大、排隊模擬複雜、計算成本高 |
| 漸進採用 | 先分鐘驗證邏輯，再 Tick 驗證執行 |

---

## 延伸閱讀

- [背景知識：執行模擬器實現](../../Part5-生產與實戰/背景知識/執行模擬器實現.md) - 執行模擬的詳細實現
- [背景知識：交易所與訂單簿機制](交易所與訂單簿機制.md) - 訂單簿基礎
- [第 7 課：回測系統的陷阱](../第07課：回測系統的陷阱.md) - 回測常見問題
