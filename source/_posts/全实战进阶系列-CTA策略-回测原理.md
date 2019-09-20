---
title: 【课程笔记】全实战进阶系列-CTA策略-回测原理
date: 2019-09-20 23:40:02
tags:
- Quant
- 全实战进阶系列-CTA策略
---

[全实战进阶系列-CTA策略](https://appszu5scwd6134.h5.xiaoeknow.com/), 平台小鹅通，by 用 Python 的交易员

<!-- More -->

## 回测引擎
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

- `app.cta_strategy.backtesting.BacktestingEngine`
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
    - 缓存最新的 K 线 / Tick 数据，模拟撮合限价单和停止单
    - 将最新的 K 线 / Tick 推给 `strategy.on_bar` (防止未来函数，因为实盘走完 K 线之后才会下单)
    - 更新收盘价，方便计算逐日盯市盈亏
