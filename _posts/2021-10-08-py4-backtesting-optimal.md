---
title: 最佳化股票交易策略─Python筆記(4)
tags: python
---

上一篇文章我們完成了使用`Backtesting.py`模組做的個股基本交易策略回測程式，其中用的是常見的均線交錯判定買/賣點，而在本篇當中，我們要嘗試使用模組中`Optimize`的功能，找出特定個股過往最適合的均線判定策略，做為未來交易策略的參考。程式的構成基本上是相同的，但在執行回測時會加入最佳化設定，由程式計算合適的交易方式，達到極大的利潤。

## 引入模組

```python
from backtesting import Backtest, Strategy #引入回測和交易策略功能

from backtesting.lib import crossover #從lib子模組引入判斷均線交會功能
from backtesting.test import SMA #從test子模組引入繪製均線功能

import pandas as pd #引入pandas讀取股價歷史資料CSV檔
```

## 策略定義、導入資料及建立回測機器

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
            
stock = "TSLA" #設定要測試的股票標的名稱

df = pd.read_csv(f"./data/{stock}.csv", index_col=0) #pandas讀取資料，並將第1欄作為索引欄
df = df.interpolate() #CSV檔案中若有缺漏，會使用內插法自動補值，不一定需要的功能
df.index = pd.to_datetime(df.index) #將索引欄資料轉換成pandas的時間格式，backtesting才有辦法排序

test = Backtest(df, SmaCross, cash=10000, commission=.002)
# 指定回測程式為test，在Backtest函數中依序放入(資料來源、策略、現金、手續費)

```

## 最佳化回測結果

程式寫到這裡，我們已經把回測條件都建立好了，接下來要使用模組中的`Optimize`功能，由程式計算若我們的目標為利潤的最大值，則特斯拉股票在均線交叉的交易策略下要如何執行呢?

```python
result = test.run() #原始結果

opt_result = test.optimize(n1=range(5, 50, 5),  #將回測機器加入optimize屬性，定義短均線的周期為5~50(每次加5)，長均線的周期為10~120(每次加5)
                    n2=range(10, 120, 5),
                    maximize='Equity Final [$]',  #最佳化目標為最終資產最大化
                    constraint=lambda p: p.n1 < p.n2)  #限制n1及n2的範圍，只會計算n1小於n2的情況
      
print("Original strategy")
print(result) #印出result結果
print() #空一行
print("Optimize strategy")
print(opt_result)  #印出opt_result結果
```

為了方便比較差異，這邊選擇將最佳化前後的結果一起呈現，並加入文字說明區分前後結果。執行這個程式需要等候大約30秒~60秒的計算時間，接著便能看到回測結果：

```python
Original strategy
Start                     2010-06-29 00:00:00
End                       2020-06-29 00:00:00
Duration                   3653 days 00:00:00
Exposure [%]                          96.5782
Equity Final [$]                      51692.9
Equity Peak [$]                         52497
Return [%]                            416.929
Buy & Hold Return [%]                 4124.99
Max. Drawdown [%]                    -80.6771
Avg. Drawdown [%]                    -15.2517
Max. Drawdown Duration     1037 days 00:00:00
Avg. Drawdown Duration      120 days 00:00:00
Trades                                    149
Win Rate [%]                           38.255
Best Trade [%]                        353.968
Worst Trade [%]                      -17.9826
Avg. Trade [%]                        2.81325
Max. Trade Duration         197 days 00:00:00
Avg. Trade Duration          24 days 00:00:00
Expectancy [%]                        11.1337
SQN                                  0.708816
Sharpe Ratio                        0.0875347
Sortino Ratio                        0.672486
Calmar Ratio                        0.0348704
_strategy                            SmaCross
dtype: object

Optimize strategy
Start                       2010-06-29 00:00:00
End                         2020-06-29 00:00:00
Duration                     3653 days 00:00:00
Exposure [%]                            96.9614
Equity Final [$]                         155089 #最終資產提升了大約3倍
Equity Peak [$]                          246015
Return [%]                              1450.89
Buy & Hold Return [%]                   4124.99
Max. Drawdown [%]                      -82.7275
Avg. Drawdown [%]                      -8.48113
Max. Drawdown Duration       1102 days 00:00:00
Avg. Drawdown Duration         54 days 00:00:00
Trades                                      138
Win Rate [%]                            44.9275
Best Trade [%]                          147.373
Worst Trade [%]                        -23.6136
Avg. Trade [%]                          2.77656
Max. Trade Duration           113 days 00:00:00
Avg. Trade Duration            26 days 00:00:00
Expectancy [%]                          10.4776
SQN                                    0.655627
Sharpe Ratio                           0.144873
Sortino Ratio                          0.508596
Calmar Ratio                          0.0335628
_strategy                 SmaCross(n1=15,n2=20) #在最終資產最大化的條件下，適合的交易策略是以15及20天的均線交叉操作
dtype: object
```

## 選擇不同的最佳化目標

在`Optimize`功能中參數中，我們可以在`maximize`參數中放入不同的項目來調整回測機器，這邊我們要嘗試的是以`SQN`指數作為最佳化的指標。

「SQN」的全文為System Quality Number，常被翻譯為「系統品質指數」，他的計算方式是，SQN指數越高，通常表示這個交易策略越穩定：

```python
((期望獲利/標準差)*交易次數)開根號
```

影響SQN高低的面向有以下幾點：

1) 期望獲利越大→SQN越高
2) 每次交易結果之間的標準差越小→SQN越高
3) 交易次數越多→SQN越高

而在SQN指數好壞的評估下，常用的表格如下：
SQN值|評價
---|---
<1.00|很爛的系統
1.01~2.00|普通的系統，可以考慮使用
2.01~3.00|好的系統，建議使用
3.01~5.00|很好的系統，應努力維持
5.01~7.00|極佳的系統，難能可貴
>7.00|聖杯系統，百年一遇

我們在程式碼中，將`maxmize`參數計算的條件改為`SQN`，再執行一次程式：

```python
Original strategy
Start                     2010-06-29 00:00:00
End                       2020-06-29 00:00:00
Duration                   3653 days 00:00:00
Exposure [%]                          96.5782
Equity Final [$]                      51692.9
Equity Peak [$]                         52497
Return [%]                            416.929
Buy & Hold Return [%]                 4124.99
Max. Drawdown [%]                    -80.6771
Avg. Drawdown [%]                    -15.2517
Max. Drawdown Duration     1037 days 00:00:00
Avg. Drawdown Duration      120 days 00:00:00
Trades                                    149
Win Rate [%]                           38.255
Best Trade [%]                        353.968
Worst Trade [%]                      -17.9826
Avg. Trade [%]                        2.81325
Max. Trade Duration         197 days 00:00:00
Avg. Trade Duration          24 days 00:00:00
Expectancy [%]                        11.1337
SQN                                  0.708816
Sharpe Ratio                        0.0875347
Sortino Ratio                        0.672486
Calmar Ratio                        0.0348704
_strategy                            SmaCross
dtype: object

Optimize strategy
Start                      2010-06-29 00:00:00
End                        2020-06-29 00:00:00
Duration                    3653 days 00:00:00
Exposure [%]                           96.1402
Equity Final [$]                        130315  #最終資產約增加了2.5倍
Equity Peak [$]                         132342
Return [%]                             1203.15
Buy & Hold Return [%]                  4124.99
Max. Drawdown [%]                     -67.3037
Avg. Drawdown [%]                     -9.49771
Max. Drawdown Duration      2120 days 00:00:00
Avg. Drawdown Duration        89 days 00:00:00
Trades                                     116
Win Rate [%]                           40.5172
Best Trade [%]                         387.895
Worst Trade [%]                       -17.9826  
Avg. Trade [%]                         4.83321
Max. Trade Duration          201 days 00:00:00
Avg. Trade Duration           31 days 00:00:00
Expectancy [%]                         12.6711
SQN                                    1.37751  #SQN值較為增加
Sharpe Ratio                          0.122412
Sortino Ratio                          1.18451
Calmar Ratio                         0.0718119
_strategy                 SmaCross(n1=5,n2=30)  #適用的均線交叉判斷為5日及30日
dtype: object
```

這邊可以看到最佳化SQN的結果，與最佳化最終資產相比，最終的資產減少了約12%，但在交易回落與損失來說，SQN最佳化後的表現都比較好一點，但整體來說也不算是多棒的策略就是了...

## 應用在台股

上一篇文章我們同樣將這個程式用台積電的歷史價格跑了一次，結果是賠錢的，所以我們這次分別用最終資產和SQN去跑，看看結果如何？程式碼如下(因為要算2個`Optimize`，程式執行時間會比較長)：

```python
opt_result_equity = test.optimize(n1=range(5, 50, 5),  #將回測機器加入optimize屬性，定義短均線的周期為5~50(每次加5)，長均線的周期為10~120(每次加5)
                    n2=range(10, 120, 5),
                    maximize='Equity Final [$]',  #最佳化目標為最終資產最大化
                    constraint=lambda p: p.n1 < p.n2)  #限制n1及n2的範圍，只會計算n1小於n2的情況

opt_result_sqn = test.optimize(n1=range(5, 50, 5),  #將回測機器加入optimize屬性，定義短均線的周期為5~50(每次加5)，長均線的周期為10~120(每次加5)
                    n2=range(10, 120, 5),
                    maximize='SQN',  #最佳化目標為SQN最大化
                    constraint=lambda p: p.n1 < p.n2)  #限制n1及n2的範圍，只會計算n1小於n2的情況
      
print("Original strategy")
print(result) #印出result結果
print() #空一行
print("Optimize strategy 1")
print(opt_result_equity)  #印出opt_result_equity結果
print() #空一行
print("Optimize strategy 2")
print(opt_result_sqn)  #印出opt_result_sqn結果
```

執行後結果如下：

```python
Original strategy
Start                     2010-01-04 00:00:00
End                       2020-06-29 00:00:00
Duration                   3829 days 00:00:00
Exposure [%]                          97.8062
Equity Final [$]                       4330.1
Equity Peak [$]                       10837.6
Return [%]                            -56.699
Buy & Hold Return [%]                  380.74
Max. Drawdown [%]                    -73.3687
Avg. Drawdown [%]                    -22.5779
Max. Drawdown Duration     3449 days 00:00:00
Avg. Drawdown Duration      941 days 00:00:00
Trades                                    162
Win Rate [%]                          34.5679
Best Trade [%]                        23.1396
Worst Trade [%]                      -10.6913
Avg. Trade [%]                      -0.429462
Max. Trade Duration          98 days 00:00:00
Avg. Trade Duration          24 days 00:00:00
Expectancy [%]                        3.46975
SQN                                  -1.80742
Sharpe Ratio                       -0.0931454
Sortino Ratio                       -0.209145
Calmar Ratio                      -0.00585348
_strategy                            SmaCross
dtype: object

Optimize strategy 1
Start                       2010-01-04 00:00:00
End                         2020-06-29 00:00:00
Duration                     3829 days 00:00:00
Exposure [%]                            96.4482
Equity Final [$]                          26042
Equity Peak [$]                         28079.7
Return [%]                               160.42
Buy & Hold Return [%]                    380.74
Max. Drawdown [%]                      -29.0209
Avg. Drawdown [%]                      -4.70563
Max. Drawdown Duration       1016 days 00:00:00
Avg. Drawdown Duration         58 days 00:00:00
Trades                                       45
Win Rate [%]                            55.5556
Best Trade [%]                          28.0365
Worst Trade [%]                        -7.73389
Avg. Trade [%]                           2.3475
Max. Trade Duration           323 days 00:00:00
Avg. Trade Duration            83 days 00:00:00
Expectancy [%]                          6.12501
SQN                                     1.84177
Sharpe Ratio                           0.266962
Sortino Ratio                           1.03557
Calmar Ratio                          0.0808899
_strategy                 SmaCross(n1=45,n2=60)
dtype: object

Optimize strategy 2
Start                       2010-01-04 00:00:00
End                         2020-06-29 00:00:00
Duration                     3829 days 00:00:00
Exposure [%]                            96.4482
Equity Final [$]                          26042
Equity Peak [$]                         28079.7
Return [%]                               160.42
Buy & Hold Return [%]                    380.74
Max. Drawdown [%]                      -29.0209
Avg. Drawdown [%]                      -4.70563
Max. Drawdown Duration       1016 days 00:00:00
Avg. Drawdown Duration         58 days 00:00:00
Trades                                       45
Win Rate [%]                            55.5556
Best Trade [%]                          28.0365
Worst Trade [%]                        -7.73389
Avg. Trade [%]                           2.3475
Max. Trade Duration           323 days 00:00:00
Avg. Trade Duration            83 days 00:00:00
Expectancy [%]                          6.12501
SQN                                     1.84177
Sharpe Ratio                           0.266962
Sortino Ratio                           1.03557
Calmar Ratio                          0.0808899
_strategy                 SmaCross(n1=45,n2=60)
dtype: object
```

結果可以看到，在有限的選擇中，程式算出了相同的結果：在短均線45天、長均線60天交叉操作的狀況下同時會取得最大資產與最佳的SQN，而SQN值來到1.84，距離好的系統已經不遠。

## 其他變化

在`backtesting.py`輸出的結果中，還有其他常用的系統評估指標：
指標|說明
---|---
夏普比率|(報酬率 – 無風險利率)/標準差，表示在承受1%的風險下，能得到多少比率的報酬？夏普比率越高，獲利能力就越強
索丁諾比率|與夏普比率類似，但計算標準差時只採計負標準差，也就是排除賺錢的情況，僅用可能賠錢的情況來判斷系統強度，原則上較夏普比率更有鑑別度
卡瑪比率|年化報酬率/最大回落，卡瑪比率越高表示系統勝率與報酬率越穩定

可以嘗試將這些指標都帶入程式測試，尋找滿意的策略做為參考。
