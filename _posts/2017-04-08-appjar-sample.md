---
layout: post
title:  python GUI APIs - appJar 範例
date:   2017-04-08
categories: python_gui
---

這是利用python GUI APIs - appJar所完成的一個點餐的應用. 用在火鍋點餐, 有選擇的鍋底及加點的菜色. 
鍋底是單選的項目, 加點的則可以多選. 當選項有改變時, 可以立即在右側的選單中看到當下選擇的項目.

程式如下:

```
#coding=utf-8
from appJar import gui

main_selected=None #1
dict_selected={} #2
app=None #3

def main():
    togging1={"黃金魚蛋":False, "蝦餃":False, "魚餃":False, "貢丸":False, "魚板":False,  "蟹肉棒":False,  "玉米":False, "米血":False, "鴨血":False,  "芋頭":False}
    togging2={"高麗菜":False,"青江菜":False,"花椰菜":False,"金針菇":False,"豆芽菜":False,"茼蒿":False}
    togging3={"雞肉":False,"牛肉片":False,"鯛魚":False,"羊肉片":False,"豬肉片":False}
    
    global app #4
    app = gui()
    app.addLabelOptionBox("鍋底", ["- 請選擇 -","- 大腸臭臭鍋 -", "大腸臭臭鍋-豬肉片", "大腸臭臭鍋-雞肉",
                            "大腸臭臭鍋-羊肉片", "大腸臭臭鍋-鯛魚", "- 海鮮豆腐鍋 -", "海鮮豆腐鍋-豬肉片", "海鮮豆腐鍋-牛肉片",
                            "海鮮豆腐鍋-鯛魚", "- 泡菜鍋 -","泡菜鍋-豬肉片","泡菜鍋-鯛魚","泡菜鍋-雞肉","泡菜鍋-牛肉片"],0,0,3,1)
    app.setOptionBoxFunction("鍋底", obf)
    
    app.addLabel("l1","加點",1,0,3,1)
    
    app.startToggleFrame("青菜類",2,0,1,1)
    app.addProperties("properties2",togging2)
    app.setPropertiesFunction("properties2",changed)
    app.stopToggleFrame()
    
    app.startToggleFrame("肉類",2,1,1,1)
    app.addProperties("properties3",togging3)
    app.setPropertiesFunction("properties3",changed)
    app.stopToggleFrame()
    
    app.startToggleFrame("火鍋料",2,2,1,1)
    app.addProperties("properties1",togging1)
    app.setPropertiesFunction("properties1",changed)
    app.stopToggleFrame()
    
    app.startLabelFrame("您的點餐內容",0,3,1,3)
    app.addListBox("list",[])
    app.setListBoxBg("list","lightgray")
    app.stopLabelFrame()
    
    app.addButton("完成",press,3,0,3,1)
    
    app.hideLabel("l1") #5
    app.hideButton("完成") #5
    app.hideLabelFrame("您的點餐內容") #5
    app.hideToggleFrame("肉類") #5
    app.hideToggleFrame("火鍋料") #5
    app.hideToggleFrame("青菜類") #5
    
    app.go()
    
def extract_items(data): #6
    """ pass in a dict with key like 'name' and value of True or False, 
    so the function will extract the items with True value and put them in a list
    """
    output=[main_selected] #7
    for key in data:
        if data[key]:
            output.append(key)
    return output  
    
def obf(widget): #8
    """OptionBox event handler, 當湯底的OptionBox有改變時會觸發"""
    global main_selected
    main_selected=app.getOptionBox(widget)
    selected = extract_items(dict_selected) #9
    app.updateListItems("list", selected) #9

    app.showLabel("l1") #10
    app.showButton("完成") #10
    app.showLabelFrame("您的點餐內容") #10
    app.showToggleFrame("肉類") #10
    app.showToggleFrame("火鍋料") #10
    app.showToggleFrame("青菜類") #10
    
def changed(widget):
    global dict_selected
    prop = app.getProperties(widget) #11
    for key,value in prop.items(): #12
        dict_selected[key]=value #12
    selected = extract_items(dict_selected) #13
    app.updateListItems("list", selected) #13
    
def press(button):
    if button=="完成":
        if app.yesNoBox("qu","請確認您的餐點無誤, 是否送出?"):
            app.infoBox("info1","謝謝您的惠顧!")
            app.stop()
        else:
            app.infoBox("info2","請繼續選購!")
            
if __name__ == '__main__':
    main()
```


1. main_selected變數: 所選擇的鍋底
1. dict_selected變數：dict物件, 用來紀錄所有加選的菜色的狀態
1. app變數: gui物件. 必須定義在global scope, 以便讓各個函數內都可以引用
1. 因為app是global變數, 在函數內必須先宣告成global才能使用. 若沒有宣告成global而直接使用, python interpreter會當作一個新的local變數來初始化.
1. 程式啟動後, 會先隱藏下方的加點的選單, 當選擇湯底完成之後才會顯示
1. extract_items()函數: 用來將可加選的菜色(dict物件)中, 有挑選的(value=True)萃取出來, 放到一個list物件中
1. 先將所選的鍋底(main_selected)放到list物件中
1. obf()函數是湯底OptionBox的事件處理函數. 每次當湯底選項有改變時會被呼叫. 
1. obf()函數內容中須呼叫extract_items(), 產生最新的list, 然後更新ListBox
1. 顯示可能被隱藏的下方加選的選單
1. 取得加選properties中的內容(dict物件), 其中有挑選的value為True,否則為False
1. 將properties dict物件內容, 更新到dict_selected中, 因為dict物件的key是唯一, 因此若properties dict中的item資料有更新, 可以覆蓋掉dict_selected中舊的item值.
1. 呼叫extract_items(), 產生最新的list, 然後更新ListBox

:sweat_smile:
