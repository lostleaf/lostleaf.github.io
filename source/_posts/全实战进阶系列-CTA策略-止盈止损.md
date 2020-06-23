---
title: 【课程笔记】全实战进阶系列-CTA策略-止盈止损
date: 2020-06-21 21:11:02
tags:
- 量化交易
- CTA
categories:
- Vnpy-CTA策略 课程笔记
---
[全实战进阶系列-CTA策略](https://appszu5scwd6134.h5.xiaoeknow.com/), 平台小鹅通，by 用 Python 的交易员

<!-- More -->

## BoolDemoStrategy
`BollDemoStrategy`，一个跑在 15 分钟 K 线上的策略

[代码Github](https://gist.github.com/lostleaf/2af58045a98ae00f0b8db733cf204f1e#file-boll_demo_strategy_20-py)

策略参数变量定义
```python
class BollDemoStrategy(CtaTemplate):
    boll_window = 18 # MA 窗口
    boll_dev = 3.4 # 标准差倍数
    fixed_size = 1 # 下单固定手数

    boll_up = 0 # 上轨
    boll_down = 0 # 下轨
    boll_mid = 0 # 中轨

    parameters = [
        "boll_window", "boll_dev", 
        "fixed_size"
    ]
    variables = [
        "boll_up", "boll_down", "boll_mid"
    ]
```

开平仓逻辑
```python
def on_15min_bar(self, bar: BarData):
    self.cancel_all() # 撤销未成交订单

    am = self.am # 将K线推送给 array manager
    am.update_bar(bar)
    if not am.inited:
        return

    # 计算布林带上中下轨
    self.boll_up, self.boll_down = am.boll(self.boll_window, self.boll_dev)
    self.boll_mid = am.sma(self.boll_window)
    
    if self.pos == 0: 
        # 空仓，在上(下)轨分别下条件单做多(空)
        self.buy(self.boll_up, self.fixed_size, True)
        self.short(self.boll_down, self.fixed_size, True)

    elif self.pos > 0:
        # 持多仓，下穿中轨，平仓 
        if bar.close_price <= self.boll_mid:
            self.sell(bar.close_price - 5, abs(self.pos))
        
    elif self.pos < 0:
        # 持空仓，上穿中轨，平仓
        if bar.close_price >= self.boll_mid:
            self.cover(bar.close_price + 5, abs(self.pos))
        
    self.put_event()
```

回测结果，不太好

![Backtest](bolldemo_backtest.png)

## 固定价格止损
添加参数

```python
class BollDemoStrategy(CtaTemplate):
    boll_window = 18
    ...

    fixed_sl = 20 # fixed stop loss

    long_entry = 0
    long_sl = 0
    short_entry = 0
    short_sl = 0

    parameters = [
        "boll_window", "boll_dev", 
        "fixed_size", 

        "fixed_sl"
    ]
    variables = [
        "boll_up", "boll_down", "boll_mid",

        "long_entry", "long_sl",
        "short_entry", "short_sl"
    ]
```

在下单逻辑中加入止损
```python
def on_15min_bar(self, bar: BarData):
    #初始化 array manager，及计算指标逻辑不变
    ...     

    if self.pos == 0:
        ...

        # 记录开多仓和开空仓的点位
        self.long_entry = self.boll_up
        self.short_entry = self.boll_down

    elif self.pos > 0:
        ...

        # 持多仓，在 long_entry - fixed_sl 处下止损单
        self.long_sl = self.long_entry - self.fixed_sl
        self.sell(self.long_sl, abs(self.pos), True)

    elif self.pos < 0:
        ...

        # 持空仓，在 short_entry + fixed_sl 处下止损单
        self.short_sl = self.short_entry + self.fixed_sl
        self.cover(self.short_sl, abs(self.pos), True)
```

加入止损后，回测结果有所提升

![Fixed stop loss](boll_fixedstop_backtest.png)

## 可变止损

### 按百分比止损

替换
```python
fixed_sl = 20
```
为
```
percent_sl = 0.01
```

开平仓相应的替换
```python
self.long_sl = self.long_entry * (1 - self.percent_sl)

self.short_sl = self.short_entry * (1 + self.percent_sl)
```

![Percent stop loss](boll_percentstop_backtest.png)

### 按技术指标来止损

使用 atr 的一个倍数止损

[代码Github](https://gist.github.com/lostleaf/2af58045a98ae00f0b8db733cf204f1e#file-boll_demo_strategy_21-py)

在最原始的 boll 策略的基础上，添加参数
```python
class BollDemoStrategy(CtaTemplate):
    ...
    atr_window = 20
    atr_multiplier = 2

    atr_value = 0

    parameters = [
        "boll_window", "boll_dev", 
        "fixed_size", 

        "atr_window", "atr_multiplier"
    ]
    variables = [
        "boll_up", "boll_down", "boll_mid",
        "long_entry", "long_sl",
        "short_entry", "short_sl",

        "atr_value"
    ]
```

对于下单逻辑, 计算 atr 
``` python
self.atr_value = am.atr(self.atr_window)
```

止损逻辑替换为

```python
self.long_sl = self.long_entry - self.atr_value * self.atr_multiplier

self.short_sl = self.short_entry + self.atr_value * self.atr_multiplier
```

![Atr stop loss](boll_atrstop_backtest.png)

## 移动止损

根据当前K线的最高价，不断调整止损点位

[代码Github](https://gist.github.com/lostleaf/2af58045a98ae00f0b8db733cf204f1e#file-boll_demo_strategy_22-py)

定义参数的变量

```python
class BollDemoStrategy(CtaTemplate):
    ...
    # 前面的跟 ATR 止损一样

    intra_trade_high = 0 # 交易中的最高价
    intra_trade_low = 0  # 交易中的最低价
    
    parameters = [
        "boll_window", "boll_dev", 
        "fixed_size", 
        "atr_window", "atr_multiplier"
    ]
    variables = [
        "boll_up", "boll_down", "boll_mid",
        "atr_value",
        "intra_trade_high", "long_sl",
        "intra_trade_low", "short_sl"
    ]
```

下单逻辑

```python
def on_15min_bar(self, bar: BarData):
    ...
    # 相同的方法初始化 array manager，以及计算布林带和 ATR
    
    if self.pos == 0:
        ...

        # 初始化最高价和最低价
        self.intra_trade_high = bar.high_price
        self.intra_trade_low = bar.low_price

    elif self.pos > 0:
        ...
        
        # 更新最高价
        self.intra_trade_high = max(self.intra_trade_high, bar.high_price)
        # 更新最低价（不重要）
        self.intra_trade_low = bar.low_price

        # 用最高价下止损停止单
        self.long_sl = self.intra_trade_high - self.atr_value * self.atr_multiplier
        self.sell(self.long_sl, abs(self.pos), True)

    elif self.pos < 0:
        ...
        
        # 更新最低价
        self.intra_trade_low = min(self.intra_trade_low, bar.low_price)
        # 更新最高价（不重要）
        self.intra_trade_high = bar.high_price

        # 用最低价下止损停止单
        self.short_sl = self.intra_trade_low + self.atr_value * self.atr_multiplier
        self.cover(self.short_sl, abs(self.pos), True)
```

## 止损进阶

[代码Github](https://gist.github.com/lostleaf/2af58045a98ae00f0b8db733cf204f1e#file-boll_demo_strategy_23-py)

整合多个出场价格，锚定真实的开仓价格


定义变量和参数

```python
class BollDemoStrategy(CtaTemplate):

    boll_window = 18
    boll_dev = 3.4
    fixed_size = 1

    boll_up = 0
    boll_down = 0
    boll_mid = 0

    atr_window = 20
    atr_multiplier = 2    
    atr_value = 0

    intra_trade_high = 0
    long_sl = 0
    intra_trade_low = 0
    short_sl = 0

    fixed_tp = 100 # 止盈区间
    long_entry = 0
    long_tp = 0 # 多仓止盈点位
    short_entry = 0
    short_tp = 0 # 空仓止盈点位
    
    parameters = [
        "boll_window", "boll_dev", 
        "fixed_size", 
        "atr_window", "atr_multiplier",
        "fixed_tp"
    ]
    variables = [
        "boll_up", "boll_down", "boll_mid",
        "atr_value",
        "intra_trade_high", "long_sl",
        "intra_trade_low", "short_sl",
        "long_entry", "long_tp",
        "short_entry", "short_tp"
    ]
```

收到成交回报，计算止盈点位

```python
def on_trade(self, trade: TradeData):
    if self.pos != 0:
        if trade.direction == Direction.LONG:
        # 多仓止盈
            self.long_entry = trade.price
            self.long_tp = self.long_entry + self.fixed_tp
        else:
        # 空仓止盈
            self.short_entry = trade.price
            self.short_tp = self.short_entry - self.fixed_tp
```

下单逻辑
```python
def on_15min_bar(self, bar: BarData):
    ...
    # 初始化 array manager, 计算布林指标和atr
    
    if self.pos == 0:
        self.buy(self.boll_up, self.fixed_size, True)
        self.short(self.boll_down, self.fixed_size, True)

        self.intra_trade_high = bar.high_price
        self.intra_trade_low = bar.low_price

        # 空仓时，初始化入场点位和止盈为0
        self.long_entry = 0
        self.long_tp = 0
        self.short_entry = 0
        self.short_tp = 0

    elif self.pos > 0:
        # self.sell(self.boll_mid, abs(self.pos), True) 为避免反向开仓，这里不能下停止单
        
        self.intra_trade_high = max(self.intra_trade_high, bar.high_price)
        self.intra_trade_low = bar.low_price

        # 止损点位为布林带中轨和移动止损的较大值
        self.long_sl = self.intra_trade_high - self.atr_value * self.atr_multiplier
        self.long_sl = max(self.boll_mid, self.long_sl)
        self.sell(self.long_sl, abs(self.pos), True)

        # 用限价单止盈，如果用停止单，则会立即触发
        if self.long_tp:
            self.sell(self.long_tp, abs(self.pos))

    elif self.pos < 0:
        # self.cover(self.boll_mid, abs(self.pos), True) 为避免反向开仓，这里不能下停止单

        self.intra_trade_low = min(self.intra_trade_low, bar.low_price)
        self.intra_trade_high = bar.high_price

        # 止损点位为布林带中轨和移动止损的较小值
        self.short_sl = self.intra_trade_low + self.atr_value * self.atr_multiplier
        self.short_sl = min(self.boll_mid, self.short_sl)
        self.cover(self.short_sl, abs(self.pos), True)

        # 用限价单止盈，如果用停止单，则会立即触发
        if self.short_tp:
            self.cover(self.short_tp, abs(self.pos))

```