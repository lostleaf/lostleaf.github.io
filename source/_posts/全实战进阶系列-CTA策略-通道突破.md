---
title: 【课程笔记】全实战进阶系列-CTA策略-通道突破
date: 2020-03-06 14:51:30
tags:
- Quant
- 全实战进阶系列-CTA策略
---

[全实战进阶系列-CTA策略](https://appszu5scwd6134.h5.xiaoeknow.com/), 平台小鹅通，by 用 Python 的交易员

<!-- More -->

通道是技术分析中常用的指标，例如唐奇安通道、布林通道、肯特纳通道

## 通道指标代码实现
`ArrayManager`([vnpy/trader/utility.py](https://github.com/vnpy/vnpy/blob/master/vnpy/trader/utility.py)) 通道指标实现

### 布林通道
描述高概率的震荡区间，突破区间可能代表趋势形成
```python
def boll(self, n, dev, array=False):
    mid = self.sma(n, array) # 以 n 根K线收盘价移动平均为中轨
    std = self.std(n, array) # 计算 n 根K线收盘价标准差

    up = mid + std * dev     # 中轨 + dev 个标准差为上轨
    down = mid - std * dev   # 中轨 - dev 个标准差为下轨

    return up, down
```

### 肯特纳通道
类似布林通道，使用 atr 代替标准差
```python
    def keltner(self, n, dev, array=False):
        mid = self.sma(n, array)
        atr = self.atr(n, array)

        up = mid + atr * dev
        down = mid - atr * dev

        return up, down
```

### 唐奇安通道
描述压力位/支撑位
```python
def donchian(self, n, array=False):
    # 没有中轨, 使用之前 n 天的最高价最大值/最低价最小值
    up = talib.MAX(self.high, n)
    down = talib.MIN(self.low, n)

    if array:
        return up, down
    return up[-1], down[-1]
```

## KingKeltner 策略
vnpy([vnpy/app/cta_strategy/strategies/king_keltner_strategy.py](https://github.com/vnpy/vnpy/blob/master/vnpy/app/cta_strategy/strategies/king_keltner_strategy.py)) 自带策略，使用肯特纳通道，跑在五分钟 k 线上

```python
def on_5min_bar(self, bar: BarData):
    # 将之前没成交的挂单全部撤掉
    for orderid in self.vt_orderids:
        self.cancel_order(orderid)
    self.vt_orderids.clear()

    # 推送K线给 array manager
    am = self.am
    am.update_bar(bar)
    if not am.inited:
        return

    # 计算肯特纳通道上下轨
    self.kk_up, self.kk_down = am.keltner(self.kk_length, self.kk_dev)

    if self.pos == 0:
        # 开仓逻辑: 突破上轨，多头入场；突破下轨，空头入场
        # 使用 One Cancel Other(oco) 订单，确保单一方向入场
        self.intra_trade_high = bar.high_price
        self.intra_trade_low = bar.low_price
        self.send_oco_order(self.kk_up, self.kk_down, self.fixed_size)

    elif self.pos > 0:
        # 多头出场逻辑：对最高价移动止损 trailing_percent%
        self.intra_trade_high = max(self.intra_trade_high, bar.high_price)
        self.intra_trade_low = bar.low_price

        vt_orderids = self.sell(self.intra_trade_high * (1 - self.trailing_percent / 100),
                                abs(self.pos), True)
        self.vt_orderids.extend(vt_orderids)

    elif self.pos < 0:
        # 空头出场逻辑：对最低价移动止损 trailing_percent%
        self.intra_trade_high = bar.high_price
        self.intra_trade_low = min(self.intra_trade_low, bar.low_price)

        vt_orderids = self.cover(self.intra_trade_low * (1 + self.trailing_percent / 100),
                                    abs(self.pos), True)
        self.vt_orderids.extend(vt_orderids)

    self.put_event()

def on_trade(self, trade: TradeData):
    if self.pos != 0:
        # 多单成交，撤销所有空单；空单成交，撤销所有多单
        if self.pos > 0:
            for short_orderid in self.short_vt_orderids:
                self.cancel_order(short_orderid)

        elif self.pos < 0:
            for buy_orderid in self.long_vt_orderids:
                self.cancel_order(buy_orderid)

        for orderid in (self.long_vt_orderids + self.short_vt_orderids):
            if orderid in self.vt_orderids:
                self.vt_orderids.remove(orderid)

    self.put_event()

def send_oco_order(self, buy_price, short_price, volume):
    # OCO 订单实现，同时下双向停止单，并记录单号；未成交的之后会在 on_trade 被撤销
    self.long_vt_orderids = self.buy(buy_price, volume, True)
    self.short_vt_orderids = self.short(short_price, volume, True)

    self.vt_orderids.extend(self.long_vt_orderids)
    self.vt_orderids.extend(self.short_vt_orderids)
```