---
layout: post
title:  "Pygame介紹 - Part 2"
date:   2017-03-23
categories: pygame
---
 
這是PyGame介紹文的Part 2. [Part 1]({{site.baseurl}}{% post_url 2017-03-17-pygame-intro %})介紹了基本程式架構及draw module. 繼續看如何利用keyboard事件來改變Rect的顏色的範例

## 改變Rect的顏色
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
1. 當鍵盤某個鍵被按下時, 會觸發`pygame.KEYDOWN`型態的事件, 搭配event.key, 會告訴我們到底是那一個key被按下, 這裡使用`pygame.K_SPACE`(空白鍵)
1. 將`draw.rect()`放在while迴圈之中, 則會被重複執行, 每一次執行會根據目前color的變數值, 來畫新的Rect

## 移動Rect
接下來, 利用上下左右四個key, 來移動Rect. 這四個key分別為`pygame.K_UP`, `pygame.K_DOWN`, `pygame.K_LEFT`, `pygame.K_RIGHT`

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
1. x, y 變數代表Rect物件的位置, 會隨著被按下的方向key而改變
1. 取得一個Clock物件, 可以用來追蹤時間, 主要是為了控制while迴圈執行的頻率(frame rate)
1. 檢核event.type是否為`pygame.KEYDOWN`, event.key是否為`pygame.K_UP`
1. `screen.fill((0,0,0))`用來在畫Rect物件前, 先將Surface物件的背景填滿黑色, 因此只會看到一個Rect物件
1. `clock.tick(60)` 用來控制while迴圈執行的頻率(frame rate), 60代表while迴圈執行的頻率約60 frames/sec, 也就是每執行一個循環不會快過1/60秒(約16 msec).
1. `clock.get_time()` 取得最後兩次呼叫clock.tick()的時間間隔(單位為msec) 

:sweat_smile:
