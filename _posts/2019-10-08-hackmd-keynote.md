---
title: 用HackMD做簡報
tags: markdown
---


![hackmd](https://i.imgur.com/nHKflbD.png)

> 「為了推廣Markdown。」

HackMD是由[Markdown 台灣社團](https://www.facebook.com/groups/markdown.tw)發起人吳承翰開發的線上多人協作共筆平台，使用Markdown語法作為編輯文件的工具，可以輕易的編輯出美觀易讀的文件，適合用來傳播知識或是做為筆記平台，更多HackMD開發的故事可以閱讀Star Rocket這篇[專訪](https://medium.com/starrocket/hackmd-product-story-1e332f83d343)，這個工具的發展過程相當有趣。

## HackMD功能簡介

HackMD的使用很簡單，但功能其實很強大，其中最基本也最重要的功能就是`寫作`

### 寫作

寫作當然是HackMD的核心，使用Markdown語法撰寫文件，同時能即時預覽套用語法後的結果，再加上工具列的設計讓不熟悉Markdown語法的人也能輕鬆上手。

### 書本&簡報

書本和簡報模式其實一樣是奠基於基本的文章編寫，接著使用特定的語法將文章轉換成分章節的書本或是分頁面的簡報檔案，只要掌握基本的操作方式，可以輕易的完成並在瀏覽器上顯示。

其中特別要提的是簡報模式，與我之前寫過使用[Marp插件製作簡報](https://finrodchen.rocks/2020/05/05/%e3%80%8cmarp%e3%80%8d%e7%94%a8vscode%e5%81%9a%e7%b0%a1%e5%a0%b1/)所產出的結果相比，HackMD能讓簡報有轉場、物件動畫效果，還能任意調整簡報內文字的顏色，可以製作出簡潔美觀但又不會過於死板的簡報，只要有瀏覽器就能使用，不用攜帶檔案，更不用安裝軟體，非常方便。

### GitHub連結

HackMD支援語GitHub連結，可以直接從特定Repo裡面抓取`.md`檔案，在HackMD的編輯器中寫作，完成後也能再推回Repo中，也能相容Git的版本控制，不需要再額外使用命令列或是VSCode等等的編輯器。

### 多人協作

HackMD中的文件可以設定為多人編輯，並能針對每個人調整權限，甚至可以藉由「主持」的方式指派特定人員為主持人，其他的協作者可以看到主持人視窗捲動狀態，即時溝通或修正特定的文字段落。

更進一步的HackMD使用方式和介紹可以參考[官方的說明書](hhttps://hackmd.io/c/tutorials-tw/%2Fs%2Ftutorials-tw)。

## 簡報功能設定

### 啟動簡報功能

啟動HackMD的簡報功能，僅需要在文件的開頭(標題之前)插入YAML的描述：

```yaml
---
title: #投影片標題(可以用第一行H1標題取代)
tags: #標籤(用逗點隔開)
disqus: #討論區
slideOptions: #投影片選項
  theme: #投影片主題
  transition: #投影片轉場
---
```

### 分頁方式

每一頁簡報的區隔則是用3個dashline來區分，這邊HackMD還提供了另一個「章節」的功能，藉由放入4個dashline可以將下一頁降階為子章節(鍵盤換頁時要按`↓`才會顯示)。

```markdown
# 我是主標題

---
# 我是第2章

____
## 我是第2.1章

____
## 我是第2.2章

___
# 我是第3章

```

### 單張簡報調整

放在文件開頭的YAML描述會套用到整份簡報，若想要針對單張簡報修改設定，只要在特定頁面標題前加入：

```yaml
<!-- .slide: -->

例如加入 <!-- .slide: data-transition="fade" -->
#表示只有加入控制碼的該頁面會有"fade"的轉場效果
```

再放入需要的` slideoption: `參數就可以只調整特定頁面的內容。

### 主題與轉場

HackMD提供多種主題及轉場：
項目|參數
---|---
主題|black, white, league, sky, beige, simple, serif, blood, night, moon, solarized
使用方法|於`slideoption:theme:`後面放入想使用的主題
轉場|none, fade, slide, convex, concave, zoom
使用方法|於`slideoption: transition:`後面放入想使用的轉場

### 簡報呈現

製作完成的簡報可藉由HackMD的分享功能以簡報模式分享或公開發表，使用者便可以這個網址啟動這份簡報，看到文件呈現為簡報的成果。

![test](https://i.imgur.com/ImGwi2h.png)

### 簡報輸出

HackMD上的簡報可以在簡報模式瀏覽時選擇下方的列印功能，再列印為PDF就可以輸出了。

## 可用參數

HackMD官方列出了全部` slideoption: `可用的參數與說明，以下引用自官方文件：

```yaml
# Display controls in the bottom right corner
controls: true

# Display a presentation progress bar
progress: true

# Set default timing of 2 minutes per slide
defaultTiming: 120

# Display the page number of the current slide
slideNumber: false

# Push each slide change to the browser history
history: false

# Enable keyboard shortcuts for navigation
keyboard: true

# Enable the slide overview mode
overview: true

# Vertical centering of slides
center: true

# Enables touch navigation on devices with touch input
touch: true

# Loop the presentation
loop: false

# Change the presentation direction to be RTL
rtl: false

# Randomizes the order of slides each time the presentation loads
shuffle: false

# Turns fragments on and off globally
fragments: true

# Flags if the presentation is running in an embedded mode,
# i.e. contained within a limited portion of the screen
embedded: false

# Flags if we should show a help overlay when the questionmark
# key is pressed
help: true

# Flags if speaker notes should be visible to all viewers
showNotes: false

# Global override for autolaying embedded media (video/audio/iframe)
# - null: Media will only autoplay if data-autoplay is present
# - true: All media will autoplay, regardless of individual setting
# - false: No media will autoplay, regardless of individual setting
autoPlayMedia: null

# Number of milliseconds between automatically proceeding to the
# next slide, disabled when set to 0, this value can be overwritten
# by using a data-autoslide attribute on your slides
autoSlide: 0

# Stop auto-sliding after user input
autoSlideStoppable: true

# Use this method for navigation when auto-sliding
autoSlideMethod: Reveal.navigateNext

# Enable slide navigation via mouse wheel
mouseWheel: false

# Hides the address bar on mobile devices
hideAddressBar: true

# Opens links in an iframe preview overlay
previewLinks: false

# Transition style
transition: 'slide' 
# none/fade/slide/convex/concave/zoom

# Transition speed
transitionSpeed: 'default'
# default/fast/slow

# Transition style for full page slide backgrounds
backgroundTransition: 'fade'
# none/fade/slide/convex/concave/zoom

# Number of slides away from the current that are visible
viewDistance: 3

# Parallax background image
parallaxBackgroundImage: ''
# e.g. "'https://s3.amazonaws.com/hakim-static/reveal-js/reveal-parallax-1.jpg'"

# Parallax background size
parallaxBackgroundSize: ''
# CSS syntax, e.g. "2100px 900px"

# Number of pixels to move the parallax background per slide
# - Calculated automatically unless specified
# - Set to 0 to disable movement along an axis
parallaxBackgroundHorizontal: null
parallaxBackgroundVertical: null

# The display mode that will be used to show slides
display: 'block'

# Enable spotlight mode
spotlight:
  enabled: true
  
# Enable timer (in minutes)
allottedMinutes: 5
```

## 結果呈現

[連結為這一篇教學轉為簡報模式的結果](https://hackmd.io/@s02260441/Byk8kze9I)
