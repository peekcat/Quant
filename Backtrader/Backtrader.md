# Backtrader

Backtrader是一款纯Python的回测+实盘框架。从软件工程的角度，这个项目非常值得学习。这个框架的代码风格非常Pythonic，也值得借鉴和学习。作者是一个很严谨的德国人，从他的代码审查和社区管理可见一斑。

> backtrader允许您专注于编写可重复使用的交易策略，指标和分析器，而不必花时间构建基础架构。

## 1 安装backtrader

### 1.1 安装

```bash
pip install backtrader
```

基本要求是： Python 2.7 Python 3.2 / 3.3 / 3.4 / 3.5 如有绘图需求，则要求 Matplotlib> = 1.4.1

```bash
pip install "backtrader[plotting]"
# 或指定matplotlib版本安装
pip install backtrader tushare matplotlib==3.2.2 -i https://mirrors.cloud.tencent.com/pypi/simple
```

### 1.2 引入backtrader

```python
import backtrader as bt

if __name__ == '__main__':
    cerebro = bt.Cerebro()
    print("starting portfolio value: %.2f" % cerebro.broker.getvalue())
    cerebro.run()
    print("final portfolio value: %.2f" % cerebro.broker.getvalue())

"""
starting portfolio value: 10000.00
final portfolio value: 10000.00
"""
```

### 1.3 历史数据

雅虎财经可以下载历史每日成交数据，以[TSLA](https://hk.finance.yahoo.com/quote/TSLA/)为例。

如若数据无法下载，需要手动构建`Date,Open,High,Low,Close,Adj Close,Volume`的基本数据

## 2 快速开始

### 2.1 基本数据

交易数据（Data Feeds）、技术指标（Indicators）和策略（Strategies）都是折线（Line）。 折线（Line）是由一系列的点组成的。 

通常交易数据（Data Feeds）包含以下几个组成部分： 开盘价（ Open）、最高价（High）、最低价（Low）、收盘价（Close）、成交量（Volume）、持仓量（OpenInterest）等。 比如：所有的开盘价（ Open）按时间组成一条折线（Line），那么一组交易数据（Data Feeds）就应该包含了6条折线（Line）。 

再加上时间（DateTime）一共有7条折线（Line）。 

### 2.2 数据索引

当访问一条折线（Line）的数据时，会默认从下标为0的位置开始，最后一个数据通过下标-1来获取。这样的设计和Python的迭代器是一致的，所以折线（Line）是可以迭代遍历的。

例如：创建一个简单移动平均值的策略（均值策略）： self.sma = SimpleMovingAverage(.....)
访问此移动平均线的当前值的最简单方法：av = self.sma[0]
所以在回测过程中，无需知道已经处理了多少条/分钟/天/月，“0”一直指向当前值。 

按照Python遍历数组的方式，用下标-1来访问最后一个值： previous_value = self.sma[-1] 同理：-2、-3下标也是可以照常使用。

### 2.3 设定现金

```python
# 导入backtrader框架  
import backtrader as bt 
# 创建Cerebro引擎  
cerebro = bt.Cerebro() 
# Cerebro引擎在后台创建broker(经纪人)，系统默认资金量为10000 
# 设置投资金额100000.0 
cerebro.broker.setcash(100000.0) 
# 引擎运行前打印期出资金  
print('组合期初资金: %.2f' % cerebro.broker.getvalue()) 
cerebro.run() 
# 引擎运行后打期末资金  
print('组合期末资金: %.2f' % cerebro.broker.getvalue())
```

组合期初资金: 100000.00
组合期末资金: 100000.00
