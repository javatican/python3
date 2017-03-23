---
layout: post
title:  "Python GUI library: appJar介紹"
date:   2017-03-16
categories: python_gui
---
# Python [appJar](http://appjar.info/)函式庫介紹

appJar是架構於Python語言的一組GUI(圖形化使用者界面)函式庫, 因為簡單易學, 常常在Python教學方面當作學習的管道.

## 程式基本架構
```python
# import the library
from appJar import gui #1

# top slice - CREATE the GUI
app = gui() #2

### fillings go here ###

# bottom slice - START the GUI
app.go() #3
```
說明:
1. appJar是這組函式庫的名稱, 須匯入gui這個類別
1. 產生gui物件
1. 啟用gui物件

備註: appJar原始說明文件使用'*三明治*'的概念來模擬說明上面的程式架構. app.gui()就是上層的麵包, app.go()就是下層的麵包, 是固定的寫法. 放在兩者中間的就是三明治的fillings(內容物), 也就是這個GUI程式要呈現的元件內容(例如按鈕, 標籤, 選單等), 例如:
```python
# import the library
from appJar import gui

# top slice - CREATE the GUI
app = gui()

### fillings go here ###
app.addLabel("title", "Welcome to appJar") #1
app.setLabelBg("title", "red") #2
app.setBg("Brown") #3
app.setFont(20) #4

# bottom slice - START the GUI
app.go()
```
說明:
1. 放入Label元件
1. 設定Label的背景色
1. 設定根視窗的背景色
1. 設定字型大小

## 範例: 登入畫面
```python
from appJar import gui

def handler1(button): #5
    if button=="Submit": #6
        print("username:", app.getEntry("user"), ", password:", app.getEntry("password"))
        app.infoBox("result","登入成功") #7
    else:
        app.stop()

app=gui()

app.addLabel("label1", "登入畫面",0,0,2) #1
app.addLabel("label2", "使用者名稱", 1,0)
app.addEntry("user",1,1) #2

app.addLabel("label3", "密碼", 2,0)
app.addSecretEntry("password",2,1) #3
app.addButtons(["Submit","Cancel"], handler1, 3,0,2) #4

app.go()
```
說明:
1. 加入一個Label元件, 元件id為'label1', 顯示的文字為'登入畫面', 接下來的三個整數0,0,2分別代表這個Label放置的位置(第幾個row, 第幾個column), 以及橫越的column數, 因此這個label會佔據(0,0)位置, 但橫跨2個columns.
1. addEntry():加入一個文字輸入欄位, 元件id為'user', 位置(1,1), 只佔一個column.
1. addSecretEntry(): 加入一個密碼輸入欄位(打入的文字不會顯示出來), 元件id為'password', 位置(2,1), 也只佔一個column.
1. addButtons(): 加入按鈕, 這裡指定了兩個按鈕要顯示的文字'Submit','Cancel', 放置在一個list物件中. handler1即是按鈕事件處理函數的名稱.放置的位置(3,0), 橫跨2個columns
1. handler1(button)這個函數是按鈕元件被按下的事件處理者, 它判斷那一個按鈕被按下, 若是Submit按鈕, 則會觸發產生一個infoBox訊息視窗. 
1. 這裡使用button=="Submit",代表是根據按鈕上面的文字來區別, **那如果有多個按鈕想要使用相同文字, 該如何區別彼此呢？**(見下面的addNameButton函數)
1. app.infoBox(): 呈現一個訊息跳出視窗(Pop-up)

備註:另外有一個addNamedButton(name, title, function)函數, 可以指定一個name屬性(顯示的文字), 一個title屬性(傳入到處理者函數中), 因此可以用來產生多個具有相同呈現文字的按鈕. 也可以用來呈現中文字的按鈕.
```
.addNamedButton(name, title, function)
By default, it's not possible to have two buttons with the same text.
If that's required, a named button should be used.
This allows a name and title to be set for a button.
The name will be displayed on the button, and the title passed to the function.
```
例如:
```python
app.addNamedButton("送出","Submit",func1,3,0)
app.addNamedButton("取消","Cancel",func1,3,1)
```


:sweat_smile:
