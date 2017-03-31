---
layout: post
title:  "Pygame介紹 - Part 1"
date:   2017-03-17
categories: pygame
---
# [Pygame](http://www.pygame.org/)介紹
Pygame是基於Python語言所開發的, 用來設計多媒體遊戲的一組函式庫模組. 簡單來說它是:
- 開放原始碼
- 用來開發獨立應用程式(非web形式)
- 2D遊戲(非3D)

接下來看看其基本程式架構

## 基本程式架構

```python
import pygame

pygame.display.init() #1

screen = pygame.display.set_mode((400, 300)) #2

done = False
while not done: #3
    for event in pygame.event.get(): #4
        if event.type == pygame.QUIT:
            done = True
        
    pygame.display.flip() #5
pygame.quit() #6
```

幾個重點說明:

1. 第一步驟先呼叫`pygame.init()`來初始化pygame的所有模組, 如果程式中不會同時使用到各個模組, 也可以選定要初始化的個別模組, 這裡就直接初始化display模組`pygame.display.init()`.
1. 建立根視窗(display Surface): `display.set_mode()`傳入所要建立的根視窗解析度大小, 回傳`Surface`物件. 注意**解析度是一個tuple參數(包括在(,)中)**
1. while迴圈是pygame的主要迴圈(main loop), 它會一直執行, 直到收到一個QUIT事件, 才會將變數done更新為True, 然後離開主要迴圈. 這個main loop主要重複做三件事情:
   - 處理事件, 例如鍵盤按鍵, 滑鼠按鍵等
   - 更新遊戲的屬性, 狀態等資料(例如遊戲人物的位置, 體力值等)
   - 更新畫面
1. 主要迴圈中包含一個for-in迴圈, 這是處理事件的迴圈, 每次從event queue(佇列)中讀取所有的event物件(`pygame.event.get()`), 並且清除queue. `pygame.QUIT`事件是一個停止的事件, 用來關閉程式.若有其他的事件須處理, 其邏輯也會寫在此迴圈中.
1. pygame使用雙重buffer機制來更新畫面, `display.flip()`就是切換buffer的動作. 這個動作會更新整個畫面. 還有另一個相關的函式`display.update()`也是用來更新畫面, 但提供更新部份畫面的功能, 可以提高程式執行的效率.
1. 最後程式要結束執行之前, 呼叫`pygame.quit()`來結束所啟動的pygame modules. 這個函式在python程式結束時, 會自動呼叫, 因此實際上可以不用做呼叫. 後面的範例都省略.

## draw模組
接下來畫畫一些基本的圖形, 例如rectangle, circle等

### Rectangle
首先看看它的說明文件:
```
pygame.draw.rect()
    draw a rectangle shape
    rect(Surface, color, Rect, width=0) -> Rect

    Draws a rectangular shape on the Surface. The given Rect is the area of the rectangle. The width argument is the thickness to draw the outer edge. If width is zero then the rectangle will be filled.

    Keep in mind the Surface.fill() method works just as well for drawing filled rectangles. In fact the Surface.fill() can be hardware accelerated on some platforms with both software and hardware display modes.
```

例如:
```
pygame.draw.rect(screen, (0,0,255), pygame.Rect(10,10,60,60), 5)
```

參數:
1. 底層Surface物件, 也就是這個rect要在那一個Surface物件上呈現
1. 邊框顏色(如果width參數為0, 則為填入的顏色), 注意**顏色參數為一個包含三個整數值的tuple, 分別代表三原色(red, green, blue)的成份, 範圍從0到255**. 這裡的(0,0,255)為藍色.
1. Rect物件, 代表所要畫的矩形的位置及大小
1. width代表邊框大小, 預設值為0, 代表所畫的的矩形為填滿的

備註: Rect物件會在後面的文章再詳細說明, 這邊舉一個產生該物件的例子:
*`pygame.Rect(10,10,60,60)`, 參數分別代表矩形的左上角座標x,y及寬width,高height, 這裡使用的座標系統: 原點在左上角, 向右是+x, 向下是+y的方向*

要注意的是**Rect物件, 僅僅是代表一個具有x,y位置, 長, 寬的一種資料物件, 它並不是那個顯示在畫面上的矩形形狀**

這一行要放在哪裡呢? 因為它只是一個靜態的Rect, 我們可以把它放在while迴圈之外. 
如果放在while迴圈內, 則這行敘述會一直被重複執行, 每一次畫一個新的Rect, 浪費cpu.
 
```python
import pygame

pygame.display.init()

screen = pygame.display.set_mode((400, 300))
pygame.draw.rect(screen, (0,0,255),pygame.Rect(10,10,60,60), 5) ###
done = False

while not done:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
                done = True
    
    pygame.display.flip()
```
### Circle
```
pygame.draw.circle()
    draw a circle around a point
    circle(Surface, color, pos, radius, width=0) -> Rect
    Draws a circular shape on the Surface. The pos argument is the center of the circle, and radius is the size. The width argument is the thickness to draw the outer edge. If width is zero then the circle will be filled.
```

例如:

```
pygame.draw.circle(screen, (255, 255, 255), (100, 100), 30, 5)
```
注意: 無論是前面的`draw.rect()`或是這裡的`draw.circle()`, 都是在底層的Surface物件上畫上相關的形狀. 這兩個函數都有回傳值, 也就是一個`Rect物件`, 代表所畫的形狀的外界範圍. 

### 其他圖案元件
pygame.draw模組中包含以下這些圖案元件
```
pygame module for drawing shapes
pygame.draw.rect	—	draw a rectangle shape (矩形)
pygame.draw.polygon	—	draw a shape with any number of sides (多邊形)
pygame.draw.circle	—	draw a circle around a point (圓)
pygame.draw.ellipse	—	draw a round shape inside a rectangle (橢圓)
pygame.draw.arc	—	draw a partial section of an ellipse (弧形)
pygame.draw.line	—	draw a straight line segment (直線)
pygame.draw.lines	—	draw multiple contiguous line segments (多條線)
pygame.draw.aaline	—	draw fine antialiased lines (antialiased直線)
pygame.draw.aalines	—	draw a connected sequence of antialiased lines (多條antialiased直線)

```

:sweat_smile:
