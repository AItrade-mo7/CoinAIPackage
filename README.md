# CoinAI.net

https://trade.mo7.cc/

BTC 出炉
EMA_272_CAP_5_CAPMax_2.5_level_5

ETH 需要的 EMA 参数

276 220 396 194

EMA_396_CAP_6_CAPMax_1_level_5

## 2023-04-14 19:25 第二次筛选

参数名称: EMA_266_CAP_5_CAPMax_3_level_5
开始时间: 2022-02-07T23:00:00
结束时间: 2022-10-01T00:00:00
InstID: ETH-USDT

参数名称: EMA_342_CAP_7_CAPMax_2.5_level_5
开始时间: 2022-10-26T06:00:00
结束时间: 2023-04-01T00:00:00
InstID: BTC-USDT

## 新一轮参数

Run-2023/05/04 17:46:14 GetWinArr.go:85: 排序方式:Money
参数名称: EMA*294_CAP_6_CAPMax_3_CAPMin*-0.5_level_1
开始时间: 2021-01-26T04:00:00
结束时间: 2023-01-01T00:00:00
InstID: BTC-USDT
开仓频率: 2.6164874551971326
胜率: 0.2877697841726619
盈亏比: 4.7193012560651858
最终金钱: 2456.214
平仓后历史最低余额: {"Value":"863.82","TimeStr":"2021-02-01T12:00:00","Time":0}
持仓过程中最低盈利比率: {"Value":"-6.46","TimeStr":"2021-01-30T07:00:00","Time":0}
杠杆倍率: 1
总手续费: 230.277024091945

Run-2023/05/04 17:48:32 GetWinArr.go:85: 排序方式:Money
参数名称: EMA*80_CAP_2_CAPMax_0.5_CAPMin*-0.5_level_1
开始时间: 2021-01-25T23:00:00
结束时间: 2023-01-01T00:00:00
InstID: ETH-USDT
开仓频率: 0.6196943972835314
胜率: 0.2386223862238622
盈亏比: 5.2028012381953871
最终金钱: 12341.808
平仓后历史最低余额: {"Value":"739.828","TimeStr":"2021-02-21T16:00:00","Time":0}
持仓过程中最低盈利比率: {"Value":"-7.467","TimeStr":"2021-01-26T06:00:00","Time":0}
杠杆倍率: 1
总手续费: 3796.555197364715

## 回测与算法优化计划

EMA = EMA 步长 \
CAP_P = CAP 步长 \
CAP_MN = CAP 边界值范围 \
Level = 杠杆倍数

> CAP 值结构:\
> CAP 值受到 CAP_P 和 EMA 步长综合影响,\
> EAM 越长,则 CAP 变化越平缓,每条 K 线之间 CAP 值间隔越小,反之变化幅度越大(市场方向反应速度关键值) \
> CAP_P 越长则 CAP 变化越迟钝,反之越敏感(盈亏比关键值) .\

1. 优先跑 主调 EMA \
   范围 50-600 间隔 2 \
   CAP_P = [3] \
   CAP_MN = ±[0.5] \
   Level = [1] 1 倍杠杆 \
   共 `(600-50)/2 = 275` 个计算任务 \
   1 分钟一个任务，16 核心并行计算则耗时 `275/16≈17` 共 17 分钟分钟 \
   这边一定会得出一些盈亏比非常惊人的数据.

2. 按照结余金钱和盈亏比综合排序 筛选 出最优秀的 N 个参数, \
   一般来说 N = 5 \
   此时得出的结果为: 『盈亏比最大』 \
   胜率可能不高,但是盈亏比一定是很高的存在, 此时则每次大的单边都能吃到.

3. 胜率调整 \
   CAP_P = (2-7) 共计 6 个参数 \
   CAP_MN = ±(1-6) 间隔 0.5 共计 `((6-1)/0.5)*2 = 20` 20 个参数 \
   需要计算 `20*6*5=600`条, 共计耗时 `600/16≈38` 分钟 \
    CAP 步长越长则钝化越严重, CAP_MN 则会排除掉很多震荡仓位. 进一步提高胜率. \
    此步骤得出的结果进行胜率筛选,找出最佳的 胜率和 盈亏比参数.

4. 算法修改 \
   目前的算法为: \
   固定的 EMA 值,固定的 CAP_P 值 , 固定的 CAP_MN.\
   不断地计算 CAP 值, CAP 小于 CAP_N 则开空 CAP 大于 CAP_M 则开多,否则保持空仓 \
   当前若为持仓状态时 `CAP_MN * 0.8` (相当于 收窄 CAP 边界范围,想从持仓回到空仓则需要更多力量) \

   算法需要两轮计算
   `275/16≈17` 共 17 分钟 \
   `600/16≈38` 共计 38 分钟\
   总计 `17+38=55` 分钟. 产出一个策略 则需要 55 分钟\

   算法名为 Hunter ,意思是: 猎人,没事了就空仓,当时机成熟则全仓进去.

## 新算法构想

根据持仓情况作对比,持仓时间越久,则不断增加平仓的概率.

空仓时间越久,则根据市场情况不断增加持仓概率.
