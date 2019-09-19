---
title: 【课程笔记】全实战进阶系列-CTA策略-K线逻辑
date: 2019-09-15 17:28:12
tags:
- Quant
- 全实战进阶系列-CTA策略
---

[全实战进阶系列-CTA策略](https://appszu5scwd6134.h5.xiaoeknow.com/), 平台小鹅通，by 用 Python 的交易员

<!-- More -->

## K 线合成 ([BarGenerator](https://github.com/vnpy/vnpy/blob/master/vnpy/trader/utility.py#L120))

- 两个作用：
  1. 用 Tick 数据合成 1 分钟 K 线
  2. 用 1 分钟 K 线合成其他周期的 K 线；n 分钟，n 要是 60 的约数; n 小时，任意 n

- `__init__` 参数
  1. `on_bar`: 合成 K 线回调函数
  2. `windows`: 对应 n 分钟 / n 小时的 n
  3. `on_window_bar`: 合成 n 分钟 / n 小时 K 线的回调函数
  4. `interval`: 周期 (分钟，小时等); `vnpy.trader.constant.Interval`

- `update_tick`: 
  1. 输入 tick data，合成一分钟 K 线
  2. 每走完 1 分钟 K 线，自动调用 `on_bar` 回调函数
  3. 主要是实盘的时候用，回测可能就是用的 K 线数据

- `update_bar`:
  1. 输入一分钟 bar data, 合成 n 分钟或 n 小时 K 线
  2. 每走完 n 分钟或 n 小时 K 线，自动调用 `on_window_bar` 回调函数
  3. 分钟线合成逻辑: 整除切分——只考虑了分钟，没考虑小时，整分钟推送一次，所以 n 要是 60 的约数
    ``` python
    if self.interval == Interval.MINUTE: # x-minute bar
      if not (bar.datetime.minute + 1) % self.window:
        finished = True
    ```

- n 分钟 K 线策略
  1. `on_bar` 回调加上 `bg.update_bar`
  2. 注册 `on_Nmin_bar`

- 自定义分钟合成: 参考小时线合成方式，改写 `BarGenerator` 代码成计数切分

- 自定义分钟线起始结束分钟, 防止整分钟下单拥堵: 修改 `elif self.bar.datetime.minute != tick.datetime.minute:`

## 时间序列 ([ArrayManager](https://github.com/vnpy/vnpy/blob/master/vnpy/trader/utility.py#L269))

- 两个作用
  1. K 线的时间序列容器: OHLCV 数组容器
  2. 计算技术指标: 基于 TA-lib

- `__init__`: `size`: 至少需要的 K 线数量

- 使用方法
  1. `am.update_bar(bar)`
  2. 检查 `am.inited`
  3. 计算技术指标，`sma` 等

- 扩展指标(Aroon)
``` python
class NewArrayManager(ArrayManager):
    def __init__(self, size=100):
        super().__init__(size)

    def aroon(self, n, array=False):
        aroon_up, aroon_down = talib.AROON(
            self.high, self.low, n
        )
        if array:
            return aroon_up, aroon_down
        return aroon_up[-1], aroon_down[-1]
```

## 条件逻辑

``` python
fast_rsi = am.rsi(self.fast_window, array=True)
slow_rsi = am.rsi(self.slow_window, array=True)

fast_rsi_0 = fast_rsi[-1]
slow_rsi_0 = slow_rsi[-1]

fast_rsi_1 = fast_rsi[-2]
slow_rsi_1 = slow_rsi[-2]

self.rsi_count = 0
```

- 大于小于
``` python
if fast_rsi_0 > 70:
  print('Over bought')
elif fast_rsi_0 < 30:
  print('Over sold')
```

- 快速指标**上穿/下穿**慢速指标
  
``` python
rsi_cross_over = (fast_rsi_1 < slow_rsi_1 and fast_rsi_0 >= flow_rsi_0)
rsi_cross_below = (fast_rsi_1 > slow_rsi_1 and fast_rsi_0 <= flow_rsi_0)
```

- 条件计数
``` python
# 联系多少个周期满足条件
if fast_rsi_0 > fast_rsi_0:
    self.rsi_count += 1
else:
    self.rsi_count = 0

if self.rsi_count >= 3:
    print('Very strong trend')
```