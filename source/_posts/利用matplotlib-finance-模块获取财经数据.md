---
title: 利用matplotlib.finance 模块获取财经数据
date: 2017-04-29 00:00:59
tags:
categories: 数据分析
---

从matplotlib.finance模块引入quotes_historical_yahoo_ochl（获取雅虎财经股票历史数据）

datetime.today()获取当天信息，start设置为一年前的今天。

quotes_historical_yahoo_ochl()方法需要三个参数，上市公司股票代码，历史数据起始日期，终止日期。

来查询中国移动一年来的股票数据，并转化为DataFrame：

```
from matplotlib.finance import quotes_historical_yahoo_ochl
from datetime import *
import pandas as pd
today=datetime.today()
start=(today.year-1,today.month,today.day)
quotes=quotes_historical_yahoo_ochl('0941.HK',start,today)
df=pd.DataFrame(quotes)
print df
```

表格六列分别表示日期、开盘价、收盘价、最高价、最低价、成交价。