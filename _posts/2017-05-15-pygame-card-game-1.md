---
layout:	post
title:	PyGame遊戲實作 - Part 1
date:	2017-05-14
categories: python
--- 

過去兩個多月, 上了八次的Python課程, 從基本的print(), basic data type(str, list, tuple, set, dict), 
list comprehension, Object Oriented(class, inheritance)到appJar, PyGame, numpy等函式庫. 
大致上完成了基本的入門介紹. 我給學生們出了一個利用PyGame APIs來寫一個小遊戲的練習. 

自己也有些手癢, 於是就依照我給學生的建議, 也來自己寫一個撲克牌遊戲. 這裡就是一些實作時的紀錄.

第一部份先嘗試將52張牌呈現在畫面上, 然後做一些翻牌的動作.

## Card類別

先在網路上找到了一張撲克牌的[影像檔](http://felineclipart.com/clipart/BcarbGGRi.htm), 包含了52張牌, 兩張joker和cover牌.
以下是我的`Card` class(card.py檔案)

```python
class Card:

    IMAGE_SIZE=(2179,1216) #1
    CARD_WIDTH=IMAGE_SIZE[0]/13 #2
    CARD_HEIGHT=IMAGE_SIZE[1]/5 #3
        
    def __init__(self, id, suit, number): #4
	""" self.id: card id. 1-52 for Normal cards, 53 for black Joker, 54 for red Joker, 0 for cover card
		self.suit: card suit. 1 for CLUB, 2 for DIAMOND, 3 for HEART, 4 for SPACE, 5 for JOKER, 0 for COVER CARD
		self.number: card number. 1 TO 13 for Normal cards,  14 for JOKER, O for COVER CARD """
        self.id = id 
        self.suit= suit
        self.number=number
        
    def get_image_rect(self, ratio): #5
        if self.id>=1 and self.id<=52:
            x=(self.number-1)*Card.CARD_WIDTH
            y=(self.suit-1)*Card.CARD_HEIGHT
        elif self.id==0:
            x=2*Card.CARD_WIDTH
            y=4*Card.CARD_HEIGHT
        elif self.id==53:
            x=0
            y=4*Card.CARD_HEIGHT
        elif self.id==54:
            x=Card.CARD_WIDTH
            y=4*Card.CARD_HEIGHT
        return (x*ratio,y*ratio,Card.CARD_WIDTH*ratio,Card.CARD_HEIGHT*ratio)

    def __str__(self):
        return "[%d, %d, %d]" % (self.id, self.suit, self.number)
    def __repr__(self):
        return "[%d, %d, %d]" % (self.id, self.suit, self.number)
    
    @classmethod
    def create_cards(cls): #6
        cards=[]
        card=Card(0, 0, 0)
        cards.append(card)
        id=1
        for i in range(1,5):
            for j in range(1,14):
                card=Card(id, i, j)
                cards.append(card)
                id+=1
        card=Card(53, 5, 14)
        cards.append(card)
        card=Card(54, 5, 14)
        cards.append(card)
        return cards

```

註解:
1. `IMAGE_SIZE` : static variable用來儲存所下載的影像檔的尺寸
1. `CARD_WIDTH`=IMAGE_SIZE[0]/13 : static variable用來儲存每一張card的原始大小(寬)
1. `CARD_HEIGHT`=IMAGE_SIZE[1]/5 : static variable用來儲存每一張card的原始大小(高)
1. 類別的建構子宣告了三個成員變數`id`(代碼), `suit`(花色), `number`(牌面號碼), 正常情況下只會使用到52張牌, 但是完整起見, 仍然定義兩張joker cards(id=53或54, suit=5, number=14). 還有cover card(id=0, suit=0, number=0)
1. `get_image_rect()`成員方法: 用來取得某張card它在整個影像檔上所佔的區域範圍, 參數可以傳入一個`縮放比例`(`ratio`變數), 回傳值為一個tuple, 包含位置及大小. 這個方法會在card要呈現在根視窗的時候(blit)被呼叫. 
1. `create_cards()` class methods: 用來產生所有的card物件, 然後存到一個list物件(`cards`變數)中. 

## PyGame 程式

```python
import pygame
import os
import random
from card import Card

pygame.display.init()
#
WIDTH=800
HEIGHT=600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
RATIO = (WIDTH/13)/Card.CARD_WIDTH #1
CARD_WIDTH = WIDTH//13 #2
CARD_HEIGHT = int((WIDTH/13)*(Card.CARD_HEIGHT/Card.CARD_WIDTH)) #3
#
white = (255,255,255)
screen.fill(white)
#
done = False 
dir_name = os.path.dirname(os.path.realpath(__file__))
image_path = os.path.join(dir_name,"cards.png")
img = pygame.image.load(image_path).convert()  
#
img = pygame.transform.scale(img, 
	(int(Card.IMAGE_SIZE[0]*RATIO), 
	int(Card.IMAGE_SIZE[1]*RATIO))) #4
#
cards = Card.create_cards() #5
#
cover_card = cards[0]
cover_card_rect = cover_card.get_image_rect(RATIO) #6
# 
sli = cards[1:53] #7
random.shuffle(sli) #8
#
dsp_origin_x=1 #9
dsp_origin_y=1 #9
for i in range(0,52): #10
    m=i//13
    n=i%13
    pos_x=dsp_origin_x+n*CARD_WIDTH
    pos_y=dsp_origin_y+m*CARD_HEIGHT
    card = sli[i]
    rect = card.get_image_rect(RATIO)
    screen.blit(img, (pos_x, pos_y), rect) #11
#
clock = pygame.time.Clock()
counter=0 #12
while not done: #13
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            done = True
    s = counter//52 
    p = counter%52
    q = p//13
    r = p%13
    if s%2==0: 
        rect = cover_card_rect
    else:
        card = sli[p]
        rect = card.get_image_rect(RATIO)
        
    pos_x=dsp_origin_x+r*CARD_WIDTH
    pos_y=dsp_origin_y+q*CARD_HEIGHT
    screen.blit(img, (pos_x, pos_y), rect)
    #
    counter+=1
	#
    pygame.display.flip()
    clock.tick(20)
    
```

註解:
1. `RATIO = (WIDTH/13)/Card.CARD_WIDTH` : 計算影像的縮放比例, Card.CARD_WIDTH是每一張card的原始寬度, WIDTH是根視窗的寬度.
1. `CARD_WIDTH = WIDTH//13` : 計算要呈現的每一張card的尺寸(寬).
1. `CARD_HEIGHT = int((WIDTH/13)*(Card.CARD_HEIGHT/Card.CARD_WIDTH))`  : 計算要呈現的每一張card的尺寸(高)
1. `img = pygame.transform.scale(img, (int(Card.IMAGE_SIZE[0]*RATIO), int(Card.IMAGE_SIZE[1]*RATIO)))` : 將載入的影像檔做縮放
1. `cards = Card.create_cards()` : 產生所有的card物件
1. `cover_card_rect = cover_card.get_image_rect(RATIO)` : 儲存cover card在影像檔中的區域範圍.
1. `sli = cards[1:53]` : list slice的動作, 以取得52張Normal card物件, 存入到一個list物件(`sli`變數)
1. `random.shuffle(sli)` : 洗牌的動作. `sli` list物件中的52張牌的順序已經改變. 
1. `dsp_origin_x=1, dsp_origin_y=1` : 在根視窗上顯示52張牌的原點位置(左上角)設在(1,1)
1. 這個迴圈用來呈現`sli` list物件中儲存的52張牌. `rect = card.get_image_rect(RATIO)` 取得某張card在影像檔中的區域範圍. 然後再做blit. **因為可以只針對部份區域的影像做blit, 因此rect變數正是某張card在影像檔中的區域範圍.**
1. `screen.blit(img, (pos_x, pos_y), rect)` : 在根視窗(`screen`)上與影像檔(`img`)中的`rect`區域, 在`pos_x,pos_y`的位置進行blit運算. 
1. counter變數用來紀錄while主要迴圈跑的次數
1. while主要迴圈中, 在每一個frame內, 翻一張card, 先闔起所有52張卡, 然後再掀開來, 一直循環.

 
:sweat_smile:
