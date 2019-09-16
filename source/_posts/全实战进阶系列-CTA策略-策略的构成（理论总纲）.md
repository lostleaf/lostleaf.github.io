---
title: 【课程笔记】全实战进阶系列-CTA策略-策略的构成（理论总纲）
date: 2019-09-16 14:42:30
tags:
- Quant
---

[全实战进阶系列-CTA策略](https://appszu5scwd6134.h5.xiaoeknow.com/), 平台小鹅通，by 用 Python 的交易员

<!-- More -->

## 策略构成

- 信号：赌涨还是跌
  - 做多
  - 做空
  - 不玩：不玩也是一种信号

- 过滤：是否应该下注

- 出场：手气不顺要跑

## 信号：趋势跟踪类

- 趋势确立之后，跟随交易
- 胜率略高，盈亏比略低
- 信号
  - 均线（不推荐）：SMA, EMA...；原因，均线缠绕，快速买卖亏钱
  - 震荡指标：RSI 超买超卖；MACD 柱放大

## 信号：趋势突破类

- 趋势突破是，追入交易
- 胜率略低，盈亏比略高
- 信号
  - 通道：Donchian，Keltner, Boll...
  - 形态：日内高低点，三角形态

## 过滤

- 趋势强度
  - ATR 波动范围
  - STD 标准差
- 趋势方向
  - DMI 过滤
  - CCI 位置
  - RSI 阈值

## 出场逻辑

- 反向信号：金叉入场，出现死叉
- 固定止盈：入场价 + x%
- 固定止损：入场价 - x%
- 移动止损：从最高的位置回落超过 x%

## [AtrRsi 策略](https://github.com/vnpy/vnpy/blob/master/vnpy/app/cta_strategy/strategies/atr_rsi_strategy.py)

- 股指期货一分钟线策略
- 信号：RSI 进入超买/超卖区
- 过滤：当前 ATR 大于 ATR 的移动平均
- 出场：移动止损