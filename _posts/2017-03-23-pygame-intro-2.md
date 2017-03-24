---
layout: post
title:  "Pygame介紹 - Part 2"
date:   2017-03-23
categories: pygame
---
 
這是PyGame介紹文的Part 2. 
[Part 1]({{site.baseurl}}{% post_url 2017-03-17-pygame-intro %})介紹了基本程式架構及draw module. 

接著繼續看如何利用keyboard事件來改變矩型的顏色.

## 改變矩型顏色
```python
import pygame

pygame.display.init()

screen = pygame.display.set_mode((400, 300))
done = False
is_red = True
while not done:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            done = True
        if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE: #1
            is_red = not is_red
    if is_red:
        color = (255,0,0)
    else:
        color = (255,255,255)
    pygame.draw.rect(screen,color,pygame.Rect(10,10,60,60)) #2
    pygame.display.flip()
```
說明:
1. 當鍵盤某個鍵被按下時, 會觸發`pygame.KEYDOWN`型態的事件, 搭配event.key, 會告訴我們到底是那一個key被按下, 這裡使用`pygame.K_SPACE`(空白鍵).
1. 將`draw.rect()`放在while迴圈之中, 則會被重複執行, 每一次執行會根據目前color, 位置大小, 畫上新的矩形.

注意: 這個範例**並不是去改變已經畫在畫布上的矩形,  而是每次重新畫一個矩形.** 但是因為是畫在相同的位置, 所以感覺像是變化了顏色.

## 移動矩型位置
接下來, 利用上下左右四個key, 來移動矩型. 這四個key分別為`pygame.K_UP`, `pygame.K_DOWN`, `pygame.K_LEFT`, `pygame.K_RIGHT`

```python
import pygame

pygame.display.init()

screen = pygame.display.set_mode((400, 300))
x=10 #1
y=10
done = False
is_red = True
clock = pygame.time.Clock() #2
while not done:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            done = True
        if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
            is_red = not is_red
        if event.type == pygame.KEYDOWN and event.key == pygame.K_UP: #3
            y-=30
        if event.type == pygame.KEYDOWN and event.key == pygame.K_DOWN:
            y+=30
        if event.type == pygame.KEYDOWN and event.key == pygame.K_LEFT:
            x-=30
        if event.type == pygame.KEYDOWN and event.key == pygame.K_RIGHT:
            x+=30 
    
    if is_red:
        color = (255, 0, 0)
    else:
        color = (255, 255, 255)
    screen.fill((0,0,0)) #4
    pygame.draw.rect(screen,color,pygame.Rect(x,y,60,60))
    pygame.display.flip()
    clock.tick(60) #5
    print("time lapse:",clock.get_time()) #6
```
說明:
1. x, y 變數代表要畫的矩形的位置, 會隨著被按下的方向key而改變
1. 取得一個Clock物件, 可以用來追蹤時間, 主要是為了控制while迴圈執行的`頻率(frame rate)`
1. 檢核event.type是否為`pygame.KEYDOWN`, event.key是否為`pygame.K_UP`
1. `screen.fill((0,0,0))`用來在畫矩形前, 先將Surface物件的背景填滿黑色, 也就是清除畫布的意思, 因此只會看到新畫上去的矩形
1. `clock.tick(60)` 用來控制while迴圈執行的`頻率(frame rate)`, 60代表while迴圈執行的頻率約60 frames/sec, 也就是每執行一個循環不會快過1/60秒(約16 msec).
1. `clock.get_time()` 取得最後兩次呼叫clock.tick()的時間間隔(單位為msec), 所以應該會看到時間間隔約為16 msec. 

注意: 跟改變顏色的範例一樣, 這個範例**並不是去改變已經畫在畫布上的矩形,  而是每次重新畫一個矩形.**

## 移動矩型位置 - 2nd try

上述範例可以改寫, 不要每次畫矩形時, 都產生一個新的Rect物件, 實際上可以共用一個Rect物件, 讓其位置做變動即可.

```python
import pygame

pygame.display.init()

screen = pygame.display.set_mode((400, 300))
red=(255,0,0)
white=(255,255,255)
my_rect = pygame.Rect(10,10,60,60) #1
done = False
is_red=True
clock = pygame.time.Clock()
while not done:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            done = True
        if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
            is_red = not is_red
        if event.type == pygame.KEYDOWN and event.key == pygame.K_UP:
            my_rect.move_ip(0,-30) #2
        if event.type == pygame.KEYDOWN and event.key == pygame.K_DOWN:
            my_rect.move_ip(0,30)
        if event.type == pygame.KEYDOWN and event.key == pygame.K_LEFT:
            my_rect.move_ip(-30,0)
        if event.type == pygame.KEYDOWN and event.key == pygame.K_RIGHT:
            my_rect.move_ip(30,0)
    if is_red:
        color=red
    else:
        color=white
    screen.fill((0,0,0))
    pygame.draw.rect(screen,color,my_rect) #3
    pygame.display.flip()
    clock.tick(60)
```
說明:
1. 建立Rect物件my_rect
1. 每一次當方向key被按下時, 呼叫`my_rect.move_ip()`, 將Rect的位置移動. 見後面詳細說明.
1. draw.rect()使用my_rect, 並沒有每一次呼叫都產生新的Rect物件

檢視一下Rect的說明文件:
```
move()
	moves the rectangle
	move(x, y) -> Rect
	Returns a new rectangle that is moved by the given offset. The x and y arguments can be any integer value, positive or negative.
 
move_ip()
	moves the rectangle, in place
	move_ip(x, y) -> None
	Same as the Rect.move() method, but operates in place.
```

移動函數有兩個版本:
- move(x,y): 不會改變原來的Rect物件, 而會產生一個新的Rect物件, 做位移x,y後, 回傳該新的Rect物件
- move_ip(x,y): 不會產生新的Rect物件, 而是改變原本的Rect物件的位置, 沒有回傳值

Rect物件有以下一些屬性, 可以用來改變位置或大小

```
The Rect object has several virtual attributes which can be used to move and align the Rect:

x,y
top, left, bottom, right
topleft, bottomleft, topright, bottomright
midtop, midleft, midbottom, midright
center, centerx, centery
size, width, height
w,h

All of these attributes can be assigned to:

rect1.right = 10
rect2.center = (20,30)

Assigning to size, width or height changes the dimensions of the rectangle; 
all other assignments move the rectangle without resizing it. 
Notice that some attributes are integers and others are pairs of integers.

```
因此上面移動函數move_ip(x,y)也可以換成直接改變x,y屬性:
```
my_rect.move_ip(0,-30)
相當於
my_rect.y -= 30

```
```
my_rect.move_ip(30,0)
相當於
my_rect.x += 30
```

:sweat_smile:
