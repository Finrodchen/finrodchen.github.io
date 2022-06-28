---
title: 利用GitHub Pages建立自己的部落格
tags: markdown
---

![GitHub Pages](https://i.imgur.com/MqTFW1u.png)
[GitHub Pages](https://pages.github.com/)是由GitHub提供的服務，讓用戶可以非常方便的將程式存放庫以Jekyll引擎轉換成介面簡潔又快速的網站，同時具備免費的網域和SSL，只要註冊[GitHub](https://github.com/)帳號就能免費使用，本文要介紹的是以最簡單的方式建立自己的blog，一起來看看吧！

## 建立專用Repository
要在GitHub建立自己的blog，首先要有一個GitHub帳號，接著透過[這個連結](https://github.com/Finrodchen/simple-blog-bootstrap/generate)建立一個新的存放庫，庫的名稱要命名為「你的GitHub使用者名稱.github.io」，並將存放庫的隱私屬性設定為「公開」，這樣GitHub就會自動將存放庫裡面的文件編譯成網頁並發佈到網路上了。

```markdown=
存放庫命名原則：
使用者名稱：finrodchen
存放庫名稱：finrodchen.github.io
```
建立好資料庫後等候約1-3分鐘，就可以透過剛剛建立好的存放庫的名稱「你的GitHub使用者名稱.github.io」作為網址，連線到你的部落格了。

像是這個樣子：

![Newblog](https://i.imgur.com/ACGpStK.png)



## Blog基本設定
接下來我們要做一點個人化，在你的存放庫中打開`_config.yml`檔案，可以直接在網頁端操作或是透過任何可以連線存取GitHub存放庫的軟體(VsCode, Subline text等等)來編輯。

在`_config.yml`檔案中，我們要先修改title，author和description三個位置，分別代表Blog的標題，作者名稱以及Blog的簡短敘述。

![](https://i.imgur.com/m6aWuuf.png)

另外也可以在Email和許多預設的社群網站選項中填入自己的帳號(確定要顯示的社群帳號，記得要將程式碼句首的#號移除)，接著Commit`_config.yml`檔案的修正，GitHub便會依照修正的內容重新編譯一次網站。

接下來打開`index.md`檔案，這邊要使用Markdown語法編輯，可以自由寫入任何想呈現於首頁的內容，轉譯後這些內容會在頁面的上半部出現。

## 發表第一篇文章
blog完成基本的個人化後，接下來就要發表你的第一篇文章了，打開`_post`資料夾後你會看到名稱為`2021-03-08-blog-post-title-from-file-name.md
`的檔案，剛好我們可以理解一下Jekyll引擎轉譯發文的規則：

1. 放在`_post`資料夾裡面的文章會自動被編譯成發文
2. 發文的檔名規則為`yyyy-mm-dd-文章名稱.md`，檔名的日期即發文的日期
3. 文章須以Markdown或Html語法編寫

了解規則後我們可以嘗試新增一個檔案，檔名為`2022-06-28-first-blog.md`，檔案內容如下：

```markdown=
---
title: My First Blog
tags: blog
---

This is my first blog, hello world.
```

其中被三條Dash包圍起來的部分是文章的屬性，這裡我們可以設定文章的標題和標籤，標籤未來會用做文章的分類，可以細心的安排。

文章完成後將這個檔案Commit送出，等GitHub重新編譯好網站就能看到你的發文了。

![](https://i.imgur.com/Nj5AEl8.png)

到這裡其實你的blog已經準備好了，只要熟悉Markdown語法就可以輕鬆寫出排版精美整潔的文章，當然如果你想要對網站本身有更多的修改或個人化，就要接著學習Jekyll引擎了。

## 鑽研Jekyll

![](https://i.imgur.com/J4qpcSF.png)

Jekyll引擎是由GitHub的創辦人編寫的，主要撰寫的程式語言為Ruby，轉譯靜態網頁快速有效率，網站版型或結構的調整也非常簡單，主要可以參考Jekyll官方的[說明文件](https://jekyllrb.com/docs/)，針對各項參數調整有完整的說明。另外本文建立的blog是使用minima這個佈景主題，官方也有針對一些[設定項目說明](https://github.com/jekyll/minima/blob/v2.5.0/README.md)可以參考。

## 參考資料

1. [本片文章測試blog存放庫](https://github.com/Finrodchen/testblog)
2. [本篇文章技術引用原作者存放庫](https://github.com/chadbaldwin/simple-blog-bootstrap)
