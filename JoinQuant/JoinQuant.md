# 1  JoinQuant学习记录

JoinQuant平台非常适合快速入门量化交易，不仅在该平台学习量化知识，还可以在平台上完成策略的编写及回测。

最简单的单均线策略

```python
from jqdata import *

## 初始化 仅执行一次
def initialize(context):
    # 参考标的
    TARGET_CODE = '000300.XSHG'
    # 设定沪深300作为基准
    set_benchmark(TARGET_CODE)
    # 开启动态复权模式(真实价格)
    set_option('use_real_price', True)
    # 股票手续费：买入万一，卖出万一加印花税千一
    set_order_cost(OrderCost(close_tax=0.001, open_commission=0.0001, close_commission=0.0001, min_commission=0), type='stock')

    run_daily(before_market_open, time='before_open', reference_security=TARGET_CODE)
    run_daily(market_open, time='open', reference_security=TARGET_CODE)
    run_daily(after_market_close, time='after_close', reference_security=TARGET_CODE)

def before_market_open(context):
    log.info('before_market_open runtime：'+str(context.current_dt.time()))
    # stock code
    g.security = '000001.XSHE'

def market_open(context):
    log.info('market_open runtime:'+str(context.current_dt.time()))
    security = g.security
    # 获取收盘价
    close_data = get_bars(security, count=5, unit='1d', fields=['close'])
    # 获取5日均价
    MA5 = close_data['close'].mean()
    # 获取上一时间点价格
    current_price = close_data['close'][-1]
    # 获取可用资金
    cash = context.portfolio.available_cash

    # 如果上一时间点价格高出五天平均价1%, 则全仓买入
    if (current_price > 1.01*MA5) and (cash > 0):
        log.info("价格高于均价 1%%, 买入 %s" % (security))
        order_value(security, cash)
    # 如果上一时间点价格低于五天平均价, 则全仓卖出
    elif current_price < MA5 and context.portfolio.positions[security].closeable_amount > 0:
        log.info("价格低于均价, 卖出 %s" % (security))
        order_target(security, 0)

def after_market_close(context):
    log.info(str('after_market_close runtime:'+str(context.current_dt.time())))
    trades = get_trades()
    for _trade in trades.values():
        log.info('成交记录：'+str(_trade))
    log.info('#################### It\'s over! ####################')
```

