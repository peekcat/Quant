# JoinQuant学习记录

## 1 基础入门

JoinQuant平台非常适合快速入门量化交易，不仅在该平台学习量化知识，还可以在平台上完成策略的编写及回测。

### 1.1 定投

使用量化定投平安银行（000001.SZ）

- 写法一

  ```python
  def initialize(context):
      这里是用来写初始化代码的地方,例子中就是选定要交易的股票为平安银行
  
  def handle_data(context,data):
      这里是用来写周期循环代码的地方,例子中就是买100股的平安银行
  ```

- 写法二

  ```python
  def initialize(context):
      run_daily(period,time='every_bar')
      这里是用来写初始化代码的地方,例子中就是选定要交易的股票为平安银行
  
  def period(context):
      这里是用来写周期循环代码的地方,例子中就是买100股的平安银行
  ```

> 写法一将逐步弃用，写法二是聚宽系统改进后的新写法，**推荐使用写法二**

- 选定要交易的股票为平安银行：

  ```
  g.security = '000001.XSHE'
  ```

- 买100股的平安银行（市价单写法）:

  ```
  order(g.security, 100)
  ```

- 以写法二为例把剩下的代码补上后，完整代码为：

  ```python
  def initialize(context):
      # 参考标的
      TARGET_CODE = '000300.XSHG'
      # 设定沪深300作为基准
      set_benchmark(TARGET_CODE)
      run_daily(before_market_open, time='before_open', reference_security=TARGET_CODE)
      run_daily(period, time='every_bar', reference_security=TARGET_CODE)
  
  def before_market_open(context):
      log.info('before_market_open runtime：'+str(context.current_dt.time()))
      g.security = '000001.XSHE'
  
  def period(context):
      log.info('market_open runtime:'+str(context.current_dt.time()))
      security = g.security
      order(security, 100)
  ```

### 1.2 单均线策略

案例为基础入门的单均线策略，context为JoinQuant框架维护的变量，用于保存初始化数据。

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
    set_order_cost(OrderCost(open_tax=0, close_tax=0.001, \
                             open_commission=0.0001, \
                             close_commission=0.0001, \
                             min_commission=0), type='stock')

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

> run_daily 用于调用策略
>
> ​	time 调用时间
>
> ​	reference_security 参考标的

### 1.3 下单API

order(security,amount)，含义是买卖一定数量的（单位：股）股票。security是股票代码，amount是数量，amount为负数时就是代表卖出了，需要知道的是，国内股票买入最小单位是1手即100股。

```python
# 买入100股平安银行
order("000001.XSHE",100)
# 卖出100股平安银行
order("000001.XSHE",-100)
```

order_target(security,amount)，含义是通过买卖，将股票仓位调整至一定数量（单位：股）。security是股票代码，amount是数量。

```python
# 调整平安银行的持股数量至1000股
# 即，如果目前平安银行的持股数量低于1000股就买入，高于就是卖出，不高不低就不动。
order_target("000001.XSHE",1000)
```

order_value(security,value)，含义是买卖一定价值量（单位：元）股票。security是股票代码，value是价值量。value为负数时就是代表卖出。

```python
# 买入10000元的平安银行
# 如果当前股票市价是10元，则代表买入1000股
# 如果除不开系统会自动调整成相近的合理数量。卖出时也会。
order_value("000001.XSHE",10000)
# 卖出10000元的平安银行
# 如果当前股票市价是100元，则代表卖出100股
order_value("000001.XSHE",-10000)
```

order_target_value(security,value)，通过买卖，将股票仓位调整至一定价值量（单位：元）。security是股票代码，value是价值量。

```python
# 调整平安银行的持股价值量至10000元
# 即，如果目前平安银行的持股价值量（按股票市价算）低于10000元就买入，高于就是卖出，不高不低就不动。
order_target_value("000001.XSHE",10000)
```

### 1.4 Context

- context是一个回测系统建立的Context类型的对象，其中存储了如当前策略运行的时间点、所持有的股票、数量、持仓成本等数据。
- 对象可以理解为特殊类型的变量，对象的结构往往比我们之前见过的list与dict更复杂，被定义好的对象是有名字的，比如context是一个变量，它的变量类型是一个Context类型的对象，就像dict包括key与value，Context类型的对象也包括很多属性，而且可以嵌套另一个种类型的对象，结构见下图。

![Img](./assets/3f21926604474d702db5efe2c9154cb1.png)

常用的Context数据写法

- 当前时间 context.current_dt
- 当前时间的“年-月-日”的字符串格式 context.current_dt.strftime("%Y-%m-%d")
- 前一个交易日 context.previous_date
- 当前可用资金 context.portfolio.available_cash
- 持仓价值 context.portfolio.positions_value
- 累计收益 context.portfolio.returns
- 运行频率 context.run_params.frequency
- 当前持有股票 context.portfolio.positions.keys()
- 当前持有的某股票的开仓均价 context.portfolio.positions['xxxxxx.xxxx'].avg_cost
- 当前持有的某股票的可卖持仓量 context.portfolio.positions['xxxxxx.xxxx'].closeable_amount

### 1.5 止损

狭义的止损是指当亏损达到一定幅度后下单卖出该股票的操作，目的是减少进一步的亏损。广义则指在狭义的思路上衍生的复杂的减少亏损的方法。更多的情况下指狭义的止损。通过context的数据可以得到持有股票的成本和现价，从而可以算出该股票的盈亏情况，运用条件判断语句根据盈亏情况从而决定是否卖出股票，从而实现止损操作。

```python
def initialize(context):
    run_daily(period,time='every_bar')
    g.security = '000001.XSHE'

def period(context):
    # 买入股票
    order(g.security, 100)
    # 获得股票持仓成本
    cost=context.portfolio.positions['000001.XSHE'].avg_cost
    # 获得股票现价
    price=context.portfolio.positions['000001.XSHE'].price
    # 计算收益率
    ret=price/cost-1
    # 打印日志
    print('成本价：%s，现价：%s，收益率：%s' % cost, price, ret)
    # 如果亏损达到10%则卖出股票
    if ret<-0.1:
        order_target('000001.XSHE',0)
        print('触发止损')
```

### 1.6 多股票策略

多股票策略即是用list保存股票，用for循环处理list中的数据

```python
def initialize(context):
    # 参考标的
    TARGET_CODE = '000300.XSHG'
    # 设定沪深300作为基准
    set_benchmark(TARGET_CODE)
    # 开启动态复权模式(真实价格)
    set_option('use_real_price', True)
    # 股票手续费：买入万一，卖出万一加印花税千一
    set_order_cost(OrderCost(open_tax=0, close_tax=0.001, \
                             open_commission=0.0001, \
                             close_commission=0.0001, \
                             min_commission=0), type='stock')

    run_daily(before_market_open, time='before_open', reference_security=TARGET_CODE)
    run_daily(period, time='every_bar', reference_security=TARGET_CODE)

def before_market_open(context):
    log.info('before_market_open runtime：'+str(context.current_dt.time()))
    # stock code list
    g.security = ['000001.XSHE', '000002.XSHE']
    
def period(context):
    for stock in g.security:
        # 买入股票
        order(stock, 100)
        # 获得股票持仓成本
        cost=context.portfolio.positions[stock].avg_cost
        # 获得股票现价
        price=context.portfolio.positions[stock].price
        # 计算收益率
        ret=price/cost-1
        print('成本价：%s，现价：%s，收益率：%s' % cost, price, ret)
        # 如果亏损达到10%则卖出股票
        if ret<-0.1:
            order_target(stock,0)
            print('触发止损')
```

### 1.7 实现一个自己的策略

根据想法设计买入和卖出股票的流程

```python
  设定好要交易的股票数量stocksnum 
  每天找出市值排名最小的前stocksnum只股票作为要买入的股票
  若已持有的股票的市值已经不够小而不在要买入的股票中，则卖出这些股票
  买入要买入的股票，买入金额为当前可用资金的stocksnum分之一
```

伪代码实现

```python
def initialize(context):
    run_daily(period,time='every_bar')
    # 代码：设定好要交易的股票数量stocksnum

def period(context):
    # 代码：找出市值排名最小的前stocksnum只股票作为要买入的股票
    # 代码：若已持有的股票的市值已经不够小而不在要买入的股票中，则卖出这些股票。
    # 代码：买入要买入的股票，买入金额为可用资金的stocksnum分之一
```

代码实现

```python
def initialize(context):
    run_daily(period,time='every_bar')
    # 设定好要交易的股票数量stocksnum
    g.stocksnum = 10

def period(context):
    # 代码：找出市值排名最小的前stocksnum只股票作为要买入的股票
    # 获取当天的股票列表
    scu = get_all_securities(date= context.current_dt).index.tolist()
    # 选出在scu内的市值排名最小的前stocksnum只股票
    q=query(valuation.code)
    		.filter(valuation.code.in_(scu))
      	.order_by(valuation.market_cap.asc())
        .limit(g.stocksnum)
    df = get_fundamentals(q)
    # 选取股票代码并转为list
    buylist=list(df['code'])

    # 代码：若已持有的股票的市值已经不够小而不在要买入的股票中，则卖出这些股票。
    # 对于每个当下持有的股票进行判断：现在是否已经不在buylist里，如果是则卖出
    for stock in context.portfolio.positions:
        if stock not in buylist: #如果stock不在buylist
            order_target(stock, 0) #调整stock的持仓为0，即卖出

    # 代码：买入要买入的股票，买入金额为可用资金的stocksnum分之一
    # 将资金分成g.stocksnum份
    position_per_stk = context.portfolio.cash/g.stocksnum
    # 用position_per_stk大小的g.stocksnum份资金去买buylist中的股票
    for stock in buylist:
        order_value(stock, position_per_stk)
```

优化代码

策略是每天进行一次选股并交易，我们觉得这太频繁了，希望能实现通过一个变量period控制操作的周期，即每period天进行一次选股并交易。

```python
def initialize(context):
    run_daily(period,time='every_bar')
    # 设定好要交易的股票数量
    g.stocksnum = 7
    # 设定交易周期
    g.period = 13
    # 记录策略进行天数
    g.days = 0

def period(context):
    # 判断策略进行天数是否能被轮动频率整除余1
    if g.days % g.period == 0:
        # 代码：找出市值排名最小的前stocksnum只股票作为要买入的股票
    # 获取当天的股票列表
    scu = get_all_securities(date= context.current_dt).index.tolist()
        # 选出在scu内的市值排名最小的前stocksnum只股票
        q=query(valuation.code)
        		.filter(valuation.code.in_(scu))
          	.order_by(valuation.market_cap.asc())
          	.limit(g.stocksnum)
        df = get_fundamentals(q)
        # 选取股票代码并转为list
        buylist=list(df['code'])

        # 代码：若已持有的股票的市值已经不够小而不在要买入的股票中，则卖出这些股票。
        # 对于每个当下持有的股票进行判断：现在是否已经不在buylist里，如果是则卖出
        for stock in context.portfolio.positions:
            if stock not in buylist: #如果stock不在buylist
                order_target(stock, 0) #调整stock的持仓为0，即卖出

        # 代码：买入要买入的股票，买入金额为可用资金的stocksnum分之一
        # 将资金分成g.stocksnum份
        position_per_stk = context.portfolio.cash/g.stocksnum
        # 用position_per_stk大小的g.stocksnum份资金去买buylist中的股票
        for stock in buylist:
            order_value(stock, position_per_stk)
    # 策略进行天数增加1        
    g.days = g.days + 1
```

## 2 进阶学习

编写一个优秀的策略分为以下四步

- 灵感细化
- 逐步实现策略
- 调整与改进策略
- 自测与自学
