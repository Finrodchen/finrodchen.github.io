---
title: 「Marp」用VSCode做簡報?
tags: markdown
---

![logo](https://i.imgur.com/okZsFwr.png)

Marp是利用Markdown語法編寫簡報的功能，可以簡單的作出俐落的簡報(但沒有什麼花俏的轉場，動畫之類的)，不過可以僅用編輯器就生出來，同時可以使用Markdown語法給予簡報內容整齊又美觀的排版。

Mapr本身是一個開源專案，目前最方便的用法是安裝VSCode的[擴充功能](https://marketplace.visualstudio.com/items?itemName=marp-team.marp-vscode)，便能以雙欄式操作即時呈現簡報結果。

## 起手式

要將Markdown的`.md`檔案變身成簡報，只要在文件前端插入Marp的屬性就可以了：

```markdown
---
marp: true
theme: default
paginate: true
---
```

`marp: true`表示指定這個檔案是Marp簡報檔案

`theme: default`簡報主題預設有三種：default, gaia, uncover，可以另外以CSS自訂

`paginate: true`表示顯示簡報頁碼，如果第一頁(或任何一頁不要頁碼)，可以在那一頁開頭插入`<!-- _paginate: false-->`就可以取消當頁面的頁碼顯示

`---`只要放3條dash line就是換下一頁了超簡單

## 常用指令

### 放在文件開頭屬性的指令(整個簡報的通用設定)

`marp: true` 指定文件為Marp簡報
`theme: default` 選定佈景主題
`paginate: true` 開啟頁碼(預設右下角)
`backgroundcolor: white` 背景為白色(若要使用佈景主題的設定，此項可忽略)
`color: black` 文字為黑色(若要使用佈景主題的設定，此項可忽略)
`header: ___` 頁首(預設左上角，可放圖片)
`footer: ___` 頁尾(預設左下角，可放圖片)

### 放在頁面開頭的的指令(用於單頁面調整)

`<!-- _paginate: false-->` 前綴`_`表示只作用在當頁面，取消頁碼
`<!-- _backgroundColor: aqua -->` 變更當頁面背景
`<!-- _color: white -->` 變更當頁面文字顏色
`<!--fit-->`把這一串code放在標題的`#`和標題文字之間，標題就會自動被放大到填滿頁面

#### 頁面中的指令(內容呈現)

`![](image.jpg)` 插入圖片(可調尺寸，濾鏡)
`![bg](image.jpg)` 變更當頁面背景(可調尺寸，濾鏡，圖片縮放方式，也可調整靠左/靠右)

還有其他較複雜/延伸的使用方式可參考[官方連結](https://marpit.marp.app/markdown)
不過基本上與Markdown差不多啦！

## 自訂主題

### 基本環境設定

若要自訂簡報主題就需要一點CSS技巧，首先要生成一個CSS檔案(路徑與簡報檔案相同)，接著在VSCode工作區的`Marp:Theme`設定中加入CSS檔案的檔名就完成了。

### 主題CSS檔案內容

```css
/* @theme theme-name */

section {
  width: 1280px;
  height: 960px;
  font-size: 40px;
  padding: 40px;
}

h1 {
  font-size: 60px;
  color: #09c;
}

h2 {
  font-size: 50px;
}
```

`/* @theme theme-name */`在Metadata中設定主題的名字

`section`標籤後面是簡報全域設定

接著可以依照一般CSS操作分別對`h1~h6, ol, ul, ul, p`設定細節

設定完成後回到簡報修改主題名稱就能套用新主題了！更多設定一樣可參考[官方連結](https://marpit.marp.app/markdown)

## 輸出簡報檔案

Marp擴充功能可以輸出多種檔案：

副檔名|說明
---|---
pdf|通用檔案，方便傳遞
html|會同時生成播放功能界面，可直接放上網站使用
pptx|PowerPoint檔案
jpg|只會輸出第一張投影片圖檔
png|只會輸出第一張投影片圖檔
