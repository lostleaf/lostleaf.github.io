---
title: 【课程笔记】全实战进阶系列-CTA策略-回测原理
date: 2019-09-20 23:40:02
tags:
- Quant
- 全实战进阶系列-CTA策略
---

[全实战进阶系列-CTA策略](https://appszu5scwd6134.h5.xiaoeknow.com/), 平台小鹅通，by 用 Python 的交易员

<!-- More -->

## 回测 Demo
基于 `examples/cta_backtesting`

- 两个 notebook
  - `backtesting.ipynb`: 常规的单标 CTA
  - `portfolio.ipynb`: 组合回测

- `backtesting.ipynb`
  - 创建回测
    ``` python
    engine = BacktestingEngine()
    engine.set_parameters(...) # 回测参数
    engine.add_strategy(AtrRsiStrategy, {}) # 添加策略
    ```
  - 运行回测
    ``` python
    engine.load_data() # 载入数据
    engine.run_backtesting() # 跑回测, 得到成交记录
    df = engine.calculate_result() # 从成交记录到盯市盈亏
    engine.calculate_statistics() # 计算统计指标
    engine.show_chart() # 显示结果
    ```

## 引擎(BacktestingEngine)代码
- `set_parameters`: 仅仅保存参数
- `add_strategy`: 添加策略类(继承 `CtaTemplate`)和实例化策略对象
- `load_data`: 载入历史数据, 保存在 `self.history_data`, 会调用 `load_bar_data` 或者 `load_tick_data` 从数据库读取数据
- `run_backtesting`: 跑回测，本身支持 Tick / K 线，但是图形界面只支持 K 线
  - 调用 strategy (`CtaTemplate`子类) 的 `on_init` 函数进行初始化
  - 如果 strategy 调用了 `self.load_bar`，engine 则会把对应的历史数据推送给策略进行初始化
  - 完成后 engine 标记 `self.strategy.inited = True`
  - 下一步，engine 调用 `on_start` 函数启动策略
  - 启动完成后，engine 标记 `self.strategy.trading = True`
  - 对于剩下的数据，调用 `new_bar` / `new_tick` 将剩下的数据用于回测
- `calculate_result`: 把交易聚合成逐日盈亏
- `calculate_statistics`: 基于逐日盈亏计算统计指标，比较多利用了 pandas 的向量化
- `show_chart`: 用 `matplotlib` 可视化指标
- `new_bar` / `new_tick`
  - 缓存最新的 K 线 / Tick 数据，模拟撮合限价单(`cross_limit_order`)和停止单(`cross_stop_order`)
  - 将最新的 K 线 / Tick 推给 `strategy.on_bar` (防止未来函数，因为实盘走完 K 线之后才会下单)
  - 更新收盘价，方便计算逐日盯市盈亏
- `cross_limit_order` (K 线)
  - `long_cross_price = self.bar.low_price`, 对于 long (下在 bid), 如果最低价大于**买单**，则有成交机会
  - `short_cross_price = self.bar.high_price`, 对于 short (下在 ask), 如果最高价大于**卖单**，则有成交机会
  - `long_best_price = self.bar.open_price`, `short_best_price = self.bar.open_price`, 按 taker 模拟撮合, 可能按开盘价成交
  - 模拟撮合，遍历所有 limit order:
    - 对于状态等于 `Status.SUBMITTING` 的，更新为 `Status.NOTTRADED`，调用 `on_order` 回调
    - 若 `order.price >= long_cross_price`/`order.price <= short_cross_price`, 计算是否成交, 不成交则直接 `continue`
    - 若成交, 则认为全部成交, 更新状态为 `Status.ALLTRADED`，调用 `on_order` 回调
    - 按照限价单的 price, 以及 best_price, 计算成交价格
    - 生成成交记录 `TradeData`, 维护 `strategy.pos`, 调用 `on_trade` 回调
- `cross_stop_order` (K 线)
  - `long_cross_price = self.bar.high_price`, `short_cross_price = self.bar.low_price`, `short_best_price = long_best_price = self.bar.open_price`
  - 模拟撮合，遍历所有 stop order:
    - 若 `order.price <= long_cross_price`/`order.price >= short_cross_price`, 停止单触发，新增一个限价单`OrderData`
    - 按照停止单的 price, 以及 best_price, 计算成交价格
    - 生成成交记录 `TradeData`
    - 更新原停止单状态为, `StopOrderStatus.TRIGGERED`, 调用 `on_stop_order` 回调
    - 对生成的限价单, 调用 `on_order` 回调
    - 维护 `strategy.pos`, 调用 `on_trade` 回调