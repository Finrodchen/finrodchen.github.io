---
title: 台股資料爬蟲─Python筆記(2)
tags: python
---

![logo](https://i.imgur.com/y9uS8fm.png)

## 台股個股線圖

這篇的內容很單純，就是要畫出這個圖啦！！！

在[前一篇文章](https://finrodchen.net/2020/05/26/%e5%8f%b0%e8%82%a1%e8%b3%87%e6%96%99%e7%88%ac%e8%9f%b2-python%e5%ad%b8%e7%bf%92%e7%ad%86%e8%a8%98/)中，我們已經能使用`twstock`模組抓取特定個股的交易歷史，並且使用`pandas`模組把清單(list)加上表頭並轉換成Data Frame資料表，接著再將資料表的內容儲存為`.csv`檔案。接下來只要我們定期更新個股股價，便能開始研究價格走勢了。

在這張圖中我們可以看到幾個主要項目：

### K線

![K](https://i.imgur.com/zA6NLux.png)

K線又稱作蠟燭線，一根蠟燭表示一天自開盤到收盤的價格表現，其中紅色代表上漲，綠色代表下跌，而中間的蠟燭上下限標記開盤及收盤價格，K線的上下橫線則為當日最高與最低價格。

### 均線

均線紀錄了一段特定時間中股票持有成本的平均值，英文原名是Moving Average(MA)，舉例來說，下表是某檔股票5日的收盤價格，將5日的價格加總除以5即為5日均價，以每一天為中心計算前後兩日加總的均價，就能連成一條5日價格均線了。

時間|D-2|D-1|D0|D+1|D+2|5日均價
---|---|---|---|---|---|---
價格|52.5|53.0|59.5|55.2|49.5|53.94

而隨著加總平均計算的天數不同，均線代表的意涵也不同，以下是常見的均線技術分析：

均線種類|說明|分析意義
---|---|---
5日均線|周線，極短線操作指標|飆股跌破周線可能是出場時機
10日均線|雙周線，短線操作指標|強勢股跌破雙周線可能會進入短期整理
20日均線|月線，多頭操作指標|跌破月線可能會進入短期空頭修正格局
60日均線|季線，中期操作指標|跌破季線可能會進入中期空頭修正格局
240日均線|年線，長期操作指標|跌破季線可能會進入長期空頭修正格局

除了這5種算法以外，還有許多不同日數加總的判斷方式(例如6日線、22日線等)，但對趨勢判斷的影響不會很大。程式中我設定顯示5、10及20日均線用於分析。

### 交易量

圖片下方的副圖表為每日的交易量，長條圖顏色對應當日的漲跌顯示。

## Python程式中使用的模組

在這個程式中會用到以下幾個模組：

- pandas：讀取`.csv`檔案，將股價資料轉為Data Frame資料表格式
- matplotlib：Python科學繪圖的主要模組，功能超級強大
- mplfinance：matplotlib旗下專門用於金融分析的繪圖模組，詳細的使用方式可參照[開發者GitHub](https://github.com/matplotlib/mplfinance)

## 資料前處理

首先引入需要的模組

```python
import pandas as pd
import matplotlib
import mplfinance as mpf
```

設定我們要讀取的股票代號，再由pandas的`pd.read_csv()`功能讀取`.csv`檔案

```python
target_stock = '0050'
df = pd.read_csv(f'./data/{target_stock}.csv', parse_dates=True, index_col=1) 
```

接著要對資料表中的Turnover表頭做一點修改，由於mplfinance模組中對於交易量的辨認是Volume這個字，所以我們使用pandas的`df.rename()`功能調整表頭。

```python
df.rename(columns={'Turnover':'Volume'}, inplace = True) 
```

這樣一來資料就準備好了，可以開始畫圖了！

## 開始畫圖吧

由於mplfinance內建的漲/跌標記顏色是美國的版本(綠漲紅跌)，所以我們要先使用mplfinance中自訂圖表外觀功能`mpf.make_marketcolors()`將漲/跌顏色改為台灣版本(紅漲綠跌)，接著再將這個設定以`mpf.make_mpf_style()`功能保存為自訂的外觀。

```python
mc = mpf.make_marketcolors(up='r', down='g', inherit=True)
s  = mpf.make_mpf_style(base_mpf_style='yahoo', marketcolors=mc)
```

其他圖表的細節設定(類型、均線、顯示交易量、圖表標題、套用外觀等...)我們則建立一個可變參數將它們都放在裡面，這個做法當你有多張圖表要共用設定參數時會非常方便喔。

```python
kwargs = dict(type='candle', mav=(5,20,60), volume=True, figratio=(10,8), figscale=0.75, title=target_stock, style=s)
```

最後只要很簡單的使用`mpf.plot()`就完成啦！

```python
mpf.plot(df, **kwargs)
```

## 台股走勢圖繪圖成果

![plot](https://i.imgur.com/7ZSnlks.png)

啟動程式後，會開啟一個視窗，除了有我們畫出的股價走勢圖外，matplotlib模組提供了圖表放大縮小與儲存圖檔的功能，也可以再調整圖表顯示的細節，功能真的超級強大。

## 完整程式碼

```python
import pandas as pd
import matplotlib
import mplfinance as mpf
# 導入pandas、matplotlib、mplfinance模組，將mplfinance模組縮寫為mpf
# 這邊要導入matplotlib的原因是因為mplfinance繪圖時需要調用mptplotlib模組

target_stock = '0050' #設定要繪製走勢圖的股票

df = pd.read_csv(f'./data/{target_stock}.csv', parse_dates=True, index_col=1) #讀取目標股票csv檔的位置

df.rename(columns={'Turnover':'Volume'}, inplace = True) 
#這裡針對資料表做一下修正，因為交易量(Turnover)在mplfinance中須被改為Volume才能被認出來

mc = mpf.make_marketcolors(up='r',down='g',inherit=True)
s  = mpf.make_mpf_style(base_mpf_style='yahoo',marketcolors=mc)
#針對線圖的外觀微調，將上漲設定為紅色，下跌設定為綠色，符合台股表示習慣
#接著把自訂的marketcolors放到自訂的style中，而這個改動是基於預設的yahoo外觀

kwargs = dict(type='candle', mav=(5,20,60), volume=True, figratio=(10,8), figscale=0.75, title=target_stock, style=s) 
#設定可變參數kwargs，並在變數中填上繪圖時會用到的設定值

mpf.plot(df, **kwargs)
#選擇df資料表為資料來源，帶入kwargs參數，畫出目標股票的走勢圖
```
