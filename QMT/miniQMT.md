## 如何使用miniQMT

miniQMT是属于QMT的一个子功能，miniQMT将策略编写和实盘分离，在本地配置好的Python环境中编写策略，然后传递给券商的实盘接口进行交易。与QMT最大的不同在于，miniQMT可以直接使用`xtquant`框架，在软件之外写python程序与QMT客户端连接，进行程序化下单。这样QMT客户端就完全变成一个交易终端，量化程序可以独立于QMT运行。

### 1 下载xquant

部分MQT在安装目录`bin.x64/Lib/site_packages`中可以找到`xtquant`文件夹，部分版本可能无法找到`xtquant`文件文件夹，可以在迅投官网下载，[xtquant下载地址](https://dict.thinktrader.net/nativeApi/download_xtquant.html)，下载后解压。

### 2 安装Python环境

miniMQT依赖于Python，20240617版本已经支持到Python 3.12版本，Python和minicoda选其一即可，更推荐使用miniconda。

#### 2.1 Python

[Python下载地址](https://www.python.org/downloads/windows/)

默认会安装在`C:\Users\<USER>\AppData\Local\Programs\Python\`路径下

> <USER> 为当前Windows用户

#### 2.2 miniconda

[miniconda下载地址](https://docs.anaconda.com/miniconda/)

创建minicoda虚拟环境

```bash
conda create -n stock python=3.11
```

启用名为stock的虚拟环境

```bash
conda activate stock
```

验证Python是否安装成功

```bash
python --version
```

检查Python虚拟环境安装目录

```bash
conda env list
```

### 3 安装xtquant

将下载的`xtquant`解压后复制到Python安装目录的`/Lib/site_packages`目录下

```bash
# 进入Python虚拟环境
python

# 从本地python导入xtquant库，如果出现报错则说明安装失败
from xtquant import xtdata
```

### 4 测试miniQMT

[下载PyCharm](https://www.jetbrains.com/pycharm/download/other.html)，选择社区版即可；安装后创建项目，Python环境选择对应创建的虚拟环境。

打开交易终端，选择独立交易模式。

```python
from xtquant import xtdata

xtdata.download_history_data2(stock_list=['600519.SH'], period='1d')
res = xtdata.get_market_data(stock_list=['600519.SH'], period='1d')
# 打印数据说明一切准备就绪
print(res['open'])
```
