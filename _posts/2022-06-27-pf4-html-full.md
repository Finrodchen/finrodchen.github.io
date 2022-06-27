---
title: 使用Flask框架製作網頁(4)─公告欄網頁完全體
tags: python
---

## 前情提要

上一篇文章我們把公告欄網頁背後的資料庫和程式都完成了，在程式中建立了寫入、刪除和更新資料庫中資料的功能。在這篇文章中，我們要修改之前完成的`index.html`和`update.html`網頁，並實際測試網頁建立公告的功能。

## index.html

我們先回顧一下首頁原本的長相：

```html
{% extends 'base.html' %}  <!-- 指定參照的基底網頁 -->

{% block head %}  <!-- head區塊開始 -->

    <title>簡易公佈欄 by Finrodchen</title>

{% endblock %}  <!-- head區塊結束 -->

{% block body %}  <!-- body區塊開始 -->

    <h1 style="text-align: center;">
        重要事項公告欄  <!-- h1標題 -->
    </h1>

    <table border='1' align='center'>
        <tr>  <!-- 表格表頭 -->
            <th>公告事項</th>
            <th>說明</th>
            <th>公告人</th>
            <th>建立時間</th>
            <th>操作</th>
        </tr>
        <tr>  
            <!-- 
            這裡我先填入內容，下一篇文章導入資料庫
            的時候會改為讀取資料庫中的內容。
            -->
            <td>公告001</td>
            <td>這是公告001的說明</td>
            <td>小明</td>
            <td>2020/06/08</td>
            <td>
                刪除
                <br>
                更新
            </td>
        </tr>
    </table>

    <br>
    <hr>
    <br>

    <form align='center'>
        <!--
        分別建立3個要輸入的項目
        在input標籤中定義輸入屬性為文字
        給予欄位名稱及ID標記(用於對應資料庫內容)
        最後在placeholder屬性中提示使用者應填寫的資訊
        -->
        <label for="title">公告事項</label><br>
        <input type="text" name="title" id="title" placeholder="請輸入公告事項"><br><br>
        
        <label for="content">公告說明</label><br>
        <input type="text" name="content" id="content" placeholder="請輸入公告說明"><br><br>
        
        <label for="people">公告人</label><br>
        <input type="text" name="people" id="people" placeholder="請輸入公告發佈人"><br><br>
        
        <input type="submit" value="新增公告事項">

        <!-- 因為目前尚未連結資料庫，所以這個表單還無法收集資料 -->

    </form>
      
{% endblock %}  <!-- body區塊結束 -->
```

在首頁的程式碼中，我們要修改的部分是實際公告顯示的位置，以及增加更新及刪除的按鈕。

### 公告區塊

我們先單獨把表格部分擷取出來：

```html
<table border='1' align='center'>
        <tr>  <!-- 表格表頭 -->
            <th>公告事項</th>
            <th>說明</th>
            <th>公告人</th>
            <th>建立時間</th>
            <th>操作</th>
        </tr>
        <tr>  
            <!-- 
            這裡我先填入內容，下一篇文章導入資料庫
            的時候會改為讀取資料庫中的內容。
            -->
            <td>公告001</td>
            <td>這是公告001的說明</td>
            <td>小明</td>
            <td>2020/06/08</td>
            <td>
                刪除
                <br>
                更新
            </td>
        </tr>
    </table>
```

為了讓flask認出會與資料庫互動的區域，我們要把這一段表格用`{%  %}`標籤包起來，並寫入一個for迴圈，讓表格會隨著不斷寫入新的公告而自動新增。

```html
        {% for task in tasks %} <!-- task for迴圈 -->
        <tr>  
            <td>公告001</td>
            <td>這是公告001的說明</td>
            <td>小明</td>
            <td>2020/06/08</td>
            <td>
                刪除
                <br>
                更新
            </td>
        </tr>
         {% endfor %} <!-- 迴圈結束 -->
```

接著把在表格讀取資料庫中的公告資料，並在「刪除/更新」文字上新增超連結連到對應的功能：

```html
{% for task in tasks %} <!-- task for迴圈 -->
        <tr>  
            <td>{{ task.title }}</td>
            <td>{{ task.content }}</td>
            <td>{{ task.people }}</td>
            <td>{{ task.date_created.date() }}</td>
            <!-- 從資料庫讀取對應欄位的資料 -->
            <td>
                <a href="/delete/{{ task.id }}">刪除</a>
                <br>
                <a href="/update/{{ task.id }}">更新</a>
                <!-- 超連結刪除與更新的功能 -->
            </td>
        </tr>
         {% endfor %} <!-- 迴圈結束 -->
```

另外在供使用的輸入的資料表單中，要加入`action`和`method`兩個屬性，告知flask框架要從這個表單中收集資料：

```html
    <form action="/" method="POST">
    <!-- 表單用於首頁，此表單會發布資料到資料庫中 -->
```

以上就完成了首頁的修改。

## update.html

一樣我們先回顧update.html的長相：

```html
{% extends 'base.html' %}  <!-- 指定參照的基底網頁 -->

{% block head %}  <!-- head區塊開始 -->

        <title>公告更新</title>  <!-- 網頁標題 -->

{% endblock %}  <!-- head區塊結束 -->

{% block body %}  <!-- body區塊開始 -->

    <h2 style="text-align: center;">
        公告更新  <!-- h2標題`,設定為置中-->
    </h2>

    <form align='center'>
        <!--
        分別建立3個要輸入的項目
        在input標籤中定義輸入屬性為文字
        給予欄位名稱及ID標記(用於對應資料庫內容)
        新增了value屬性，用於顯示預設的內容在表單輸入欄位中
        與資料庫連結後value屬性會直接引用特定公告的資料內容
        -->
        <label for="title">公告事項</label><br>
        <input type="text" name="title" id="title" value="公告001"><br><br>
        
        <label for="content">公告說明</label><br>
        <input type="text" name="content" id="content" value="這是公告001的說明"><br><br>
        
        <label for="people">公告人</label><br>
        <input type="text" name="people" id="people" value="小明"><br><br>
        
        <input type="submit" value="更新公告">
        <!-- 因為目前尚未連結資料庫，所以這個表單還無法收集資料 -->
        
    </form>
      
{% endblock %}  <!-- body區塊結束 -->
```

其中我們要修改的是資料表單對應的網頁動作，以及每一個欄位中`value`屬性的內容，修正如下：

```html
<form action="/update/{{ task.id }}" method="POST" align="center">
<!-- 更新指定ID的公告，並會發布資料到資料庫中 -->

<label for="title">公告事項</label><br>
        <input type="text" name="title" id="title" value="{{ task.title }}"><br><br>
        
        <label for="content">公告說明</label><br>
        <input type="text" name="content" id="content" value="{{ task.content }}"><br><br>
        
        <label for="people">公告人</label><br>
        <input type="text" name="people" id="people" value="{{ task.people }}"><br><br>
        
        <!-- value中會預先讀取目前的公告資訊，方便更新 -->
```

這樣就改好更新用頁面了。

## 本機測試

進入python console啟動`app.py`，點選 <http://127.0.0.1:5000> 連結，就在瀏覽器操作這個公告欄啦！可以測試一下新增/更新/刪除的功能是否正常，都沒問題就大功告成95%啦，再來就是要將這個公告欄部署到網路上，就完成一個真正可用的網頁公告欄了。

![test](https://i.imgur.com/BQXihb4.jpg)

## 完整程式碼

### index.html

```html
{% extends 'base.html' %}  <!-- 指定參照的基底網頁 -->

{% block head %}  <!-- head區塊開始 -->

    <title>簡易公佈欄 by Finrodchen</title>

{% endblock %}  <!-- head區塊結束 -->

{% block body %}  <!-- body區塊開始 -->

    <h1 style="text-align: center;">
        重要事項公告欄  <!-- h1標題 -->
    </h1>

    <table border='1' align='center'>
        <tr>  <!-- 表格表頭 -->
            <th>公告事項</th>
            <th>說明</th>
            <th>公告人</th>
            <th>建立時間</th>
            <th>操作</th>
        </tr>
        {% for task in tasks %} <!-- task for迴圈 -->
        <tr>  
            <td>{{ task.title }}</td>
            <td>{{ task.content }}</td>
            <td>{{ task.people }}</td>
            <td>{{ task.date_created.date() }}</td>
            <!-- 從資料庫讀取對應欄位的資料 -->
            <td>
                <a href="/delete/{{ task.id }}">刪除</a>
                <br>
                <a href="/update/{{ task.id }}">更新</a>
                <!-- 超連結刪除與更新的功能 -->
            </td>
        </tr>
         {% endfor %} <!-- 迴圈結束 -->
    </table>

    <br>
    <hr>
    <br>

    <form action="/" method="POST">
    <!-- 表單用於首頁，此表單會發布資料到資料庫中 -->
    
        <!--
        分別建立3個要輸入的項目
        在input標籤中定義輸入屬性為文字
        給予欄位名稱及ID標記(用於對應資料庫內容)
        最後在placeholder屬性中提示使用者應填寫的資訊
        -->
        
        <label for="title">公告事項</label><br>
        <input type="text" name="title" id="title" placeholder="請輸入公告事項"><br><br>
        
        <label for="content">公告說明</label><br>
        <input type="text" name="content" id="content" placeholder="請輸入公告說明"><br><br>
        
        <label for="people">公告人</label><br>
        <input type="text" name="people" id="people" placeholder="請輸入公告發佈人"><br><br>
        
        <input type="submit" value="新增公告事項">

    </form>
      
{% endblock %}  <!-- body區塊結束 -->
```

### update.html

```html
{% extends 'base.html' %}  <!-- 指定參照的基底網頁 -->

{% block head %}  <!-- head區塊開始 -->

        <title>公告更新</title>  <!-- 網頁標題 -->

{% endblock %}  <!-- head區塊結束 -->

{% block body %}  <!-- body區塊開始 -->

    <h2 style="text-align: center;">
        公告更新  <!-- h2標題`,設定為置中-->
    </h2>

    <form action="/update/{{ task.id }}" method="POST" align="center">
<!-- 更新指定ID的公告，並會發布資料到資料庫中 -->
        <!--
        分別建立3個要輸入的項目
        在input標籤中定義輸入屬性為文字
        給予欄位名稱及ID標記(用於對應資料庫內容)
        新增了value屬性，用於顯示預設的內容在表單輸入欄位中
        與資料庫連結後value屬性會直接引用特定公告的資料內容
        -->
        <label for="title">公告事項</label><br>
        <input type="text" name="title" id="title" value="{{ task.title }}"><br><br>
        
        <label for="content">公告說明</label><br>
        <input type="text" name="content" id="content" value="{{ task.content }}"><br><br>
        
        <label for="people">公告人</label><br>
        <input type="text" name="people" id="people" value="{{ task.people }}"><br><br>
        
        <!-- value中會預先讀取目前的公告資訊，方便更新 -->
        
    </form>
      
{% endblock %}  <!-- body區塊結束 -->
```
