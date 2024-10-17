# Backtrader

Backtrader是一款纯Python的回测+实盘框架。从软件工程的角度，这个项目非常值得学习。这个框架的代码风格非常Pythonic，也值得借鉴和学习。作者是一个很严谨的德国人，从他的代码审查和社区管理可见一斑。

> backtrader允许您专注于编写可重复使用的交易策略，指标和分析器，而不必花时间构建基础架构。

## 1 .1 安装backtrader

```bash
pip install backtrader
```

如需绘图功能执行下列命令

```bash
pip install "backtrader[plotting]"
# 或指定matplotlib版本安装
pip install backtrader tushare matplotlib==3.2.2 -i https://mirrors.cloud.tencent.com/pypi/simple
```

## 1.2 引入backtrader

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

## 1.3 下载历史数据

雅虎财经可以下载历史每日成交数据，以[TSLA](https://hk.finance.yahoo.com/quote/TSLA/)为例。
