---
title: 回測股票交易策略─Python筆記(3)
tags: python
permalink: /Python/
---

之前我們建立了自動抓取股票歷史交易數據的程式，藉由這個程式我們能自由擷取想研究的股票的歷史資料。而這些資料除了可以用來繪製線圖以外，最常被使用的就是「交易策略回測」了。利用過往的成交資料，套入自訂的交易策略並模擬交易，由程式中設定的條件交由電腦幫我們下單，藉這樣的回測可以驗證你的交易策略是否有效。

首先我們先複習一下抓取歷史股價資料的程式，原先我使用的是`twstock`模組，但考慮到抓取美股資料的需求，因此將資料來源轉換成`yfinance`也就是從Yahoo財經抓取資料，並存成CSV，供接下來的程式使用。

```python
import yfinance as yf
import pandas as pd
from pandas_datareader import data
from datetime import datetime

yf.pdr_override() #以pandasreader常用的格式覆寫

target_stock = 'TSLA'  #股票代號變數

start_date = datetime(2010, 1, 1)
end_date = datetime(2020, 6, 30) #設定資料起訖日期

df = data.get_data_yahoo([target_stock], start_date, end_date) #將資料放到Dataframe裡面

filename = f'./data/{target_stock}.csv' #以股票名稱命名檔案，放在data資料夾下面

df.to_csv(filename) #將df轉成CSV保存
```

執行這個程式就可以得到特斯拉自2010/01/01~2020/06/30的歷史股價資料了(這邊因為特斯拉是2010/06/29才開始交易，所以最舊一筆資料會是2010/06/29)：

```python
                   Open         High         Low        Close    Adj Close    Volume
Date
2010-06-29   19.000000    25.000000   17.540001    23.889999    23.889999  18766300
2010-06-30   25.790001    30.420000   23.299999    23.830000    23.830000  17187100
2010-07-01   25.000000    25.920000   20.270000    21.959999    21.959999   8218800
2010-07-02   23.000000    23.100000   18.709999    19.200001    19.200001   5139800
2010-07-06   20.000000    20.000000   15.830000    16.110001    16.110001   6866900
...                ...          ...         ...          ...          ...       ...
2020-06-23  998.880005  1012.000000  994.010010  1001.780029  1001.780029   6365300
2020-06-24  994.109985  1000.880005  953.140015   960.849976   960.849976  10959600
2020-06-25  954.270020   985.979980  937.150024   985.979980   985.979980   9254500
2020-06-26  994.780029   995.000000  954.869995   959.739990   959.739990   8854900
2020-06-29  969.010010  1010.000000  948.520020  1009.349976  1009.349976   9026400

[2518 rows x 6 columns]
```

## 使用模組

![logo](https://i.imgur.com/10jgVt3.png)

回測的程式最主要的模組就是`backtesting.py`，這個模組對於新手來說非常簡單易用，而且除了輸出回測交易結果之外，backtesting還會生成一個可互動的線圖網頁，讓用戶能回顧整個交易歷程，相當方便。廢話不多說，開始寫程式吧！

## 引入模組

```python
from backtesting import Backtest, Strategy #引入回測和交易策略功能

from backtesting.lib import crossover #從lib子模組引入判斷均線交會功能
from backtesting.test import SMA #從test子模組引入繪製均線功能

import pandas as pd #引入pandas讀取股價歷史資料CSV檔
```

## 定義交易策略

交易策略有很多個面向，這邊我先示範最基本的均線(Moving Average)判斷，設定5日均線(周線)和20日均線(月線)，當周線往上走超過月線時，表示股票進入上漲的趨勢，這時就選擇買進。而當周線又再度與月線交叉時，表示股票開始下跌，則選擇賣出。這樣一來一往，雖然不會買在起漲點，但也不會被殺在谷底，藉此維持一定水準的報酬。
情況|動作
---|---
周線向上突破月線，開始上漲|買入股票
周現向下跌破月線，開始下跌|賣出股票

```python
class SmaCross(Strategy): #交易策略命名為SmaClass，使用backtesting.py的Strategy功能
    n1 = 5 #設定第一條均線日數為5日(周線)
    n2 = 20 #設定第二條均線日數為20日(月線)，這邊的日數可自由調整

    def init(self):
        self.sma1 = self.I(SMA, self.data.Close, self.n1) #定義第一條均線為sma1，使用backtesting.py的SMA功能算繪
        self.sma2 = self.I(SMA, self.data.Close, self.n2) #定義第二條均線為sma2，使用backtesting.py的SMA功能算繪

    def next(self):
        if crossover(self.sma1, self.sma2): #如果周線衝上月線，表示近期是上漲的，則買入
            self.buy()
        elif crossover(self.sma2, self.sma1): #如果周線再與月線交叉，表示開始下跌了，則賣出
            self.sell()
```

## 導入要回測的股票標的

完成策略的定義之後，就可以導入我們目標的股票，並用pandas將其格式化。

```python
stock = "TSLA" #設定要測試的股票標的名稱

df = pd.read_csv(f"./data/{stock}.csv", index_col=0) #pandas讀取資料，並將第1欄作為索引欄
df = df.interpolate() #CSV檔案中若有缺漏，會使用內插法自動補值，不一定需要的功能
df.index = pd.to_datetime(df.index) #將索引欄資料轉換成pandas的時間格式，backtesting才有辦法排序
```

## 回測功能

回測功能的程式非常簡單，可以想像backtesting模組內建一個回測大腦，我們只要將設定好的策略及股價資訊提供給它，再設定好我們回測程式起始手上的現金與交易的手續費比例，backtesting就能幫我們完成其它複雜的運算了。

```python
test = Backtest(df, SmaCross, cash=10000, commission=.002)
# 指定回測程式為test，在Backtest函數中依序放入(資料來源、策略、現金、手續費)

result = test.run()
#執行回測程式並存到result中
```

## 輸出結果

回測程式已經完成，接下來要輸出結果，在`backtesting.py`模組中提供兩種結果呈現方式，第一種是單純輸出純文字資料，內容包含交易次數、時間、報酬率、最終資產等等；另一種則是以網頁的方式呈現，是可互動的線圖，使用者可以看到每次買進和賣出的時間點以及資產的累積狀況。

```python
print(result) # 直接print文字結果

test.plot(filename=f"./backtest_result/{stock}.html") #將線圖網頁依照指定檔名保存
```

## 程式執行結果

如果我們執行以上程式碼，便可以看到我們以這個均線交叉策略購買特斯拉的股票，自2010/06/29到2020/06/29的回測結果了。

```python
Start                     2010-06-29 00:00:00  起始時間
End                       2020-06-29 00:00:00  結束時間
Duration                   3653 days 00:00:00  經過天數
Exposure [%]                          96.5782  投資比率
Equity Final [$]                      51692.9  最終資產
Equity Peak [$]                         52497  最高資產
Return [%]                            416.929  報酬率
Buy & Hold Return [%]                 4124.99  買入持有報酬率
Max. Drawdown [%]                    -80.6771  最大交易回落
Avg. Drawdown [%]                    -15.2517  平均交易回落
Max. Drawdown Duration     1037 days 00:00:00  最長交易回落期間
Avg. Drawdown Duration      120 days 00:00:00  平均交易回落期間
# Trades                                  149  交易次數
Win Rate [%]                           38.255  勝率
Best Trade [%]                        353.968  最好交易報酬率
Worst Trade [%]                      -17.9826  最差交易報酬率
Avg. Trade [%]                        2.81325  平均交易報酬率
Max. Trade Duration         197 days 00:00:00  最長交易間隔
Avg. Trade Duration          24 days 00:00:00  平均交易間隔
Expectancy [%]                        11.1337  期望值
SQN                                  0.708816  系統品質指標
Sharpe Ratio                        0.0875347  夏普比率
Sortino Ratio                        0.672486  索丁諾比率
Calmar Ratio                        0.0348704  卡瑪比率
_strategy                            SmaCross  使用策略名稱
```

而[線圖網頁部分則是這樣](https://i-stock.neocities.org/TSLA.html)，可互動的介面非常方便。

## 運用在台股

這邊以台積電做範例，要修改的地方有下面幾項：

1. 擷取歷史資料的程式中`target_stock`變數要指定為`'2330.TW'` (Line 8)
2. 回測程式中的`stock`變數也要指定為`'2330.TW'` (Line 22)
3. 回測程式中的手續費要改為`.004`，以平均0.04%交易費用計算 (Line 29)

接著執行程式我們就能得到均線交叉策略購買台積電的回測結果以及[線圖](https://i-stock.neocities.org/2330.TW.html)了(是賠錢的嗚嗚嗚)。

```python
Start                     2010-01-04 00:00:00
End                       2020-06-29 00:00:00
Duration                   3829 days 00:00:00
Exposure [%]                          97.8062
Equity Final [$]                      3128.54
Equity Peak [$]                       10518.2
Return [%]                           -68.7146
Buy & Hold Return [%]                  380.74
Max. Drawdown [%]                    -79.3499
Avg. Drawdown [%]                    -32.6934
Max. Drawdown Duration     3449 days 00:00:00
Avg. Drawdown Duration     1255 days 00:00:00
# Trades                                  162
Win Rate [%]                          33.3333
Best Trade [%]                        22.9392
Worst Trade [%]                      -10.8921
Avg. Trade [%]                      -0.629596
Max. Trade Duration          98 days 00:00:00
Avg. Trade Duration          24 days 00:00:00
Expectancy [%]                        3.53282
SQN                                  -2.42403
Sharpe Ratio                         -0.13655
Sortino Ratio                       -0.302971
Calmar Ratio                      -0.00793443
_strategy                            SmaCross
```

下一篇文章我們就要進階運用`backtesting.py`模組裡面的功能，優化我們的交易策略，看看能不能把台積電轉回賺錢的狀態XDDD。

## 完整程式碼

```python
from backtesting import Backtest, Strategy #引入回測和交易策略功能

from backtesting.lib import crossover #引入判斷均線交會功能
from backtesting.test import SMA #引入繪製均線功能

import pandas as pd #引入pandas讀取股價歷史資料CSV檔

class SmaCross(Strategy): #交易策略命名為SmaClass，使用backtesting.py的Strategy功能
    n1 = 5 #設定第一條均線日數為5日(周線)
    n2 = 20 #設定第二條均線日數為20日(月線)，這邊的日數可自由調整

    def init(self):
        self.sma1 = self.I(SMA, self.data.Close, self.n1) #定義第一條均線為sma1，使用backtesting.py的SMA功能算繪
        self.sma2 = self.I(SMA, self.data.Close, self.n2) #定義第二條均線為sma2，使用backtesting.py的SMA功能算繪

    def next(self):
        if crossover(self.sma1, self.sma2): #如果周線衝上月線，表示近期是上漲的，則買入
            self.buy()
        elif crossover(self.sma2, self.sma1): #如果周線再與月線交叉，表示開始下跌了，則賣出
            self.sell()

stock = "TSLA" #設定要測試的股票標的名稱

df = pd.read_csv(f"data/{stock}.csv", index_col=0) #pandas讀取資料，並將第1欄作為索引欄
df = df.interpolate() #CSV檔案中若有缺漏，會使用內插法自動補值，不一定需要的功能
df.index = pd.to_datetime(df.index) #將索引欄資料轉換成pandas的時間格式，backtesting才有辦法排序


test = Backtest(df, SmaCross, cash=10000, commission=.002)
# 指定回測程式為test，在Backtest函數中依序放入(資料來源、策略、現金、手續費)

result = test.run()
#執行回測程式並存到result中

print(result) # 直接print文字結果

test.plot(filename=f"./backtest_result/{stock}.html") #將線圖網頁依照指定檔名保存
```
