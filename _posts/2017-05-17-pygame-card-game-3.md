---
layout:	post
title:	PyGame遊戲實作 - Part 3
date:	2017-05-17
categories: python
--- 

撲克牌遊戲[第一部份]({{site.baseurl}}{% post_url 2017-05-15-pygame-card-game-1 %})先將52張牌呈現在畫面上, 然後做一些翻牌的動作.

[第二部分]({{site.baseurl}}{% post_url 2017-05-16-pygame-card-game-2 %}) 引入一套PyGame的簡單framework, 提供模組化的設計, 可以更方便的進行複雜遊戲的設計. 

第三部份將建立第二個場景, 也就是遊戲的主要場景. 這個撲克牌遊戲就是來考驗記憶力, 每次挑選兩張牌, 若是點數一樣, 兩張牌就會自畫面中移除, 否則將再蓋上.

### Card類別

與第一部份時建立的Card類別差異如下:
- 增加static variable `cards` , 這個list物件用來儲存所有的card物件. create_cards(cls)類別方法只會建立這個物件一次. 
- 新稱成員變數self.valid, self.rect兩個. self.valid用來判斷一張牌是否已經被移除. self.rect用來cache它在影像檔中的位置與大小.
- 新增成員方法`invalide(self)` 用來改變self.valid屬性為False
- 成員方法`get_image_rect(self, ratio)`也只會計算一次self.rect

```python
class Card:
    IMAGE_SIZE=(2179,1216)
    CARD_WIDTH=IMAGE_SIZE[0]/13
    CARD_HEIGHT=IMAGE_SIZE[1]/5
    cards = None
    @classmethod
    def create_cards(cls):
        if Card.cards == None:
            Card.cards=[]
            card=Card(0, 0, 0)
            Card.cards.append(card)
            id=1
            for i in range(1,5):
                for j in range(1,14):
                    card=Card(id, i, j)
                    Card.cards.append(card)
                    id+=1
            card=Card(53, 5, 14)
            Card.cards.append(card)
            card=Card(54, 5, 14)
            Card.cards.append(card)
        return Card.cards
        
    def __init__(self, id, suit, number):
        # 1-52 FOR NORMAL CARDS, 53 FOR BLACK JOKER, 54 FOR RED JOKER, 0 FOR COVER CARD
        self.id = id 
        #1: CLUB, 2: DIAMOND, 3: HEART, 4:SPACE, 5. JOKER, 0 FOR COVER CARD
        self.suit= suit
        # 1 TO 13,  14 FOR JOKER, O FOR COVER CARD
        self.number=number
        # valid is True for the cards that are still valid
        self.valid=True
        self.rect = None
        
    def invalid(self):
        self.valid=False
        
    def get_image_rect(self, ratio):
        if self.rect==None:
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
            else:
                print("Error when calling get_image_rect()")
            self.rect =  (x,y,Card.CARD_WIDTH,Card.CARD_HEIGHT)
        return (self.rect[0]*ratio, self.rect[1]*ratio, self.rect[2]*ratio, self.rect[3]*ratio)
    def __str__(self):
        return "[%d, %d, %d]" % (self.id, self.suit, self.number)
    def __repr__(self):
        return "[%d, %d, %d]" % (self.id, self.suit, self.number)
    
```
### SceneHome類別

這是遊戲啟動後的第一個場景. 一樣會呈現52張牌及掀牌的動畫. 但是沒有先做洗牌的步驟. 加入了一個START 按鈕,來做啟動遊戲的觸發.
與第二部份時建立的SceneHome差異如下:

- `__init__(self)`中建立了一個按鈕來偵測滑鼠點擊的動作, 然後啟動第二個場景 
- `on_event(self, event)`方法中處理pygame.MOUSEBUTTONDOWN事件.


```python
class SceneHome(Scene):
    RATIO = (Director.WIDTH/13)/Card.CARD_WIDTH
    CARD_WIDTH = Director.WIDTH//13
    CARD_HEIGHT = int((Director.WIDTH/13)*(Card.CARD_HEIGHT/Card.CARD_WIDTH))
    CARDS_IMAGE= None
    #
    @classmethod
    def get_cards_img(cls):
        if SceneHome.CARDS_IMAGE==None:
            dir_name = os.path.dirname(os.path.realpath(__file__))
            image_path = os.path.join(dir_name,"cards.png")
            img = pygame.image.load(image_path).convert() 
            #scale(Surface, (width, height), DestSurface = None) -> Surface
            SceneHome.CARDS_IMAGE = pygame.transform.scale(img, 
                (int(Card.IMAGE_SIZE[0]*SceneHome.RATIO), 
                int(Card.IMAGE_SIZE[1]*SceneHome.RATIO)))
        return SceneHome.CARDS_IMAGE

    def __init__(self, director):
        super().__init__(director)
        self.img = SceneHome.get_cards_img()
        #
        self.cards = Card.create_cards()
        #
        self.cover_card = self.cards[0]
        self.cover_card_rect = self.cover_card.get_image_rect(SceneHome.RATIO)
        # 
        self.sli = self.cards[1:53]
        #沒有先洗牌
        #random.shuffle(self.sli)
        #
        white = (255,255,255)
        director.screen.fill(white)
        #
        self.dsp_origin_x=1
        self.dsp_origin_y=1
        for i in range(0,52):
            m=i//13
            n=i%13
            pos_x=self.dsp_origin_x+n*SceneHome.CARD_WIDTH
            pos_y=self.dsp_origin_y+m*SceneHome.CARD_HEIGHT
            card = self.sli[i]
            rect = card.get_image_rect(SceneHome.RATIO)
            director.screen.blit(self.img, (pos_x, pos_y), rect)
        # 建立按鈕 
        font = pygame.font.SysFont("dejavuserif", 36)  
        black = (0,0,0)
        text_sur = font.render("START", True, black)  
        #
        yellow = (255,255,0)
        button_sur = pygame.Surface(text_sur.get_size()) 
        button_sur.fill(yellow) 
        #
        button_sur.blit(text_sur,(0,0))
        self.button_rect = director.screen.blit(button_sur,(350,450))
        #
        self.counter=0
        
    def on_update(self):
        pass
 
    def on_event(self, event):
        if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
            pos = pygame.mouse.get_pos()
            if self.button_rect.collidepoint(pos):
                scene = SceneGame(self.director)
                self.director.change_scene(scene)
 
    def on_draw(self, screen):
        s = self.counter//52
        p = self.counter%52
        q = p//13
        r = p%13
        if s%2==0:
            rect = self.cover_card_rect
        else:
            card = self.sli[p]
            rect = card.get_image_rect(SceneHome.RATIO)
            
        pos_x=self.dsp_origin_x+r*SceneHome.CARD_WIDTH
        pos_y=self.dsp_origin_y+q*SceneHome.CARD_HEIGHT
        screen.blit(self.img, (pos_x, pos_y), rect)
        
        self.counter+=1

```

#### 建立按鈕及event處理邏輯

按鈕的建立比較麻煩, 不像一般GUI函式庫這麼方便, 在pygame中要做到Button的效果, 如果不使用影像檔, 則:
1. 利用`pygame.font` 模組先建立一塊文字`surface物件`
1. 再建立一塊矩形的`surface物件`
1. 將文字`surface物件`blit到矩形`surface物件`, 做成一個類似像button的`surface物件`
1. 再將上面的button `surface物件`blit到根視窗上. 
1. 最後紀錄這個button的位置與大小(`self.button_rect`成員變數), 會在後面的on_event()中使用.

```
# create a button 
font = pygame.font.SysFont("dejavuserif", 36)  
black = (0,0,0)
text_sur = font.render("START", True, black)  
#
yellow = (255,255,0)
button_sur = pygame.Surface(text_sur.get_size()) 
button_sur.fill(yellow) 
#
button_sur.blit(text_sur,(0,0))
self.button_rect = director.screen.blit(button_sur,(350,450))
```

`on_event(self,event)`需要處理`pygame.MOUSEBUTTONDOWN`事件, 並且確定是滑鼠左鍵被按下. 
pygame.MOUSEBUTTONDOWN事件物件具有`pos`, `button`兩個屬性. 
- `pos`代表滑鼠點擊的位置. 
- `button`代表滑鼠點擊的button.
 - 1代表左鍵, 
 - 2代表中間鍵或滾輪, 
 - 3代表右鍵, 
 - 4代表滾輪往上滾動, 
 - 5代表滾輪往下滾動. 

`pygame.mouse.get_pos()`取得mouse當時的位置. 利用rect物件的`collidepoint()`可以判斷一個點位置是否在rect物件範圍內.
在建立button時我們已經紀錄了button的位置與大小(`self.button_rect`), 所以我們判斷滑鼠點擊位置是否位於button範圍內, 
就可以知道使用者是否點擊了這個button. 如果點擊了這個按鈕, 就切換到下一個scene物件.

```
def on_event(self, event):
        if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
            pos = pygame.mouse.get_pos()
            if self.button_rect.collidepoint(pos):
                scene = SceneGame(self.director)
                self.director.change_scene(scene)
 ```

### SceneGame類別

我的第二個場景就是遊戲的主要場景`SceneGame`. 

#### __init__(self)建構子

建構子中一些初始化的步驟, 例如 建立self.img, self.cards, self.cover_card_rect, self.sli等成員變數, 
都與SceneHome相同. 

`random.shuffle(self.sli)`是作洗牌的動作. 接下來寫一個迴圈, 將52張牌蓋住顯示在各自的位置上. 

另外建立兩組的成員變數分別代表兩張所點選的cards: `self.card1`, `self.card2`, 兩張card的擺置的位置(索引值組合)
`self.card1_indices`, `self.card2_indices`. 

最後產生一塊白色的surface物件`self.empty_rect_sur`, 要用來填滿cards因配對後, 被移除掉而空出來的區域.

#### on_event(self,event)方法

根據偵測滑鼠點選的位置, 然後計算到底是那一張牌被點選. 
這裡使用的是方式是利用滑鼠點擊位置(`pos[0]`,`pos[1]`), 分別除以一張卡片的寬度(`SceneHome.CARD_WIDTH`)與
高度(`SceneHome.CARD_HEIGHT`), 便可以知道到底是那一張牌被點選. 

**要特別注意的是滑鼠位置的取得, 這裡必須使用event.pos屬性, 而不使用`pygame.mouse.get_pos()`**. 
理由是萬一MOUSEBUTTONDOWN事件, 因為密集的點擊而被累積起來時, 連續兩次的MOUSEBUTTONDOWN事件觸發, 
可能導致`pygame.mouse.get_pos()`回傳相同的位置. 因為`pygame.mouse.get_pos()`是回傳呼叫這個函數當下的滑鼠位置, 
而不是MOUSEBUTTONDOWN事件發生當下的點擊位置. 而使用event.pos則可以確定就是event當下發生時的點擊位置.

`if card.valid:`來確定必須是`valid==True`狀態下的card物件才可以被點選. 

第一張點選的card物件, 儲存在`self.card1`,它的位置索引值儲存在`self.card1_indices`. 
同樣動作也要用在第二張點選的card物件, 但是還要檢查是否同一張card被重複選了兩次(`if card.id!=self.card1.id:`) 
當兩張卡片被選完之後, 呼叫`pygame.event.set_blocked(pygame.MOUSEBUTTONDOWN)`, 設定暫時不接受任何MOUSEBUTTONDOWN事件.
這個動作可以避免MOUSEBUTTONDOWN事件被連續點擊而累積. 

#### on_draw(self, screen)方法

`if self.card1 and self.card2 :` 用來處理當兩張卡片都點選了之後的畫面處理邏輯. 
這裡先呼叫一次`pygame.display.flip()`來更新畫面, 所以才能看到第二張牌被掀開的結果.
然後呼叫`pygame.time.wait(2000)`, 使程式暫停2秒, 以便玩家可以看到兩張卡片的內容.

`if self.card1.number == self.card2.number:` 若兩張卡片number相同, 則呼叫`invalid()`方法, 
然後將兩張卡片區塊填上白色surface物件`self.empty_rect_sur`. 

`else:` 則將卡片闔上. 最後將`self.card1`, `self.card1_indices`, `self.card2`, `self.card2_indices`
清除, 並且恢復接收MOUSEBUTTONDOWN的事件`pygame.event.set_allowed(pygame.MOUSEBUTTONDOWN)`

```python
class SceneGame(Scene):
    def __init__(self, director):
        super().__init__(director)
        self.img = SceneHome.get_cards_img()
        #
        self.cards = Card.create_cards()
        #
        self.cover_card = self.cards[0]
        self.cover_card_rect = self.cover_card.get_image_rect(SceneHome.RATIO)
        # 
        self.sli = self.cards[1:53]
		#洗牌
        random.shuffle(self.sli)
        #
        white = (255,255,255)
        director.screen.fill(white)
        #
        self.dsp_origin_x=1
        self.dsp_origin_y=1
        for i in range(0,52):
            m=i//13
            n=i%13
            pos_x=self.dsp_origin_x+n*SceneHome.CARD_WIDTH
            pos_y=self.dsp_origin_y+m*SceneHome.CARD_HEIGHT
			#以下兩行comment起來, 因為是要顯示闔起來的牌
            #card = self.sli[i]
            #rect = card.get_image_rect(SceneHome.RATIO)
            director.screen.blit(self.img, (pos_x, pos_y), self.cover_card_rect)
        
        self.card1=None
        self.card2=None
        self.card1_indices=None
        self.card2_indices=None
        self.empty_rect_sur = pygame.Surface((SceneHome.CARD_WIDTH,SceneHome.CARD_HEIGHT)) 
        self.empty_rect_sur.fill(white) 

    def on_update(self):
        pass
 
    def on_event(self, event):
        if event.type == pygame.MOUSEBUTTONDOWN and event.button == 1:
            #pos = pygame.mouse.get_pos()
			pos = event.pos
            i = pos[1]//SceneHome.CARD_HEIGHT
            j = pos[0]//SceneHome.CARD_WIDTH
            if i>=0 and i<4 and j>=0 and j<13:
                card = self.sli[i*13+j]
                #
                if card.valid:
                    to_show = False
                    if self.card1==None:
                        self.card1 = card
                        self.card1_indices=(i,j)
                        to_show=True
                    elif self.card2==None:
                        # 檢查一張card是否被點了兩次
                        if card.id!=self.card1.id:
							# 暫時不接受MOUSEBUTTONDOWN的事件
                            pygame.event.set_blocked(pygame.MOUSEBUTTONDOWN)
                            self.card2 = card
                            self.card2_indices=(i,j)
                            to_show=True
                    if to_show:
                        pos_x=self.dsp_origin_x+j*SceneHome.CARD_WIDTH
                        pos_y=self.dsp_origin_y+i*SceneHome.CARD_HEIGHT
                        rect = card.get_image_rect(SceneHome.RATIO)
                        self.director.screen.blit(self.img, (pos_x, pos_y), rect)

    def on_draw(self, screen):
        if self.card1 and self.card2 :
            pygame.display.flip()
            pygame.time.wait(2000)
            if self.card1.number == self.card2.number:
                self.card1.invalid()
                self.card2.invalid()
                #blit 2 white rects
                pos_x=self.dsp_origin_x+self.card1_indices[1]*SceneHome.CARD_WIDTH
                pos_y=self.dsp_origin_y+self.card1_indices[0]*SceneHome.CARD_HEIGHT
                self.director.screen.blit(self.empty_rect_sur,(pos_x,pos_y))
                #
                pos_x=self.dsp_origin_x+self.card2_indices[1]*SceneHome.CARD_WIDTH
                pos_y=self.dsp_origin_y+self.card2_indices[0]*SceneHome.CARD_HEIGHT
                self.director.screen.blit(self.empty_rect_sur,(pos_x,pos_y))

            else:
                #blit 2 cover card
                pos_x=self.dsp_origin_x+self.card1_indices[1]*SceneHome.CARD_WIDTH
                pos_y=self.dsp_origin_y+self.card1_indices[0]*SceneHome.CARD_HEIGHT
                self.director.screen.blit(self.img, (pos_x, pos_y), self.cover_card_rect )
                #
                pos_x=self.dsp_origin_x+self.card2_indices[1]*SceneHome.CARD_WIDTH
                pos_y=self.dsp_origin_y+self.card2_indices[0]*SceneHome.CARD_HEIGHT
                self.director.screen.blit(self.img, (pos_x, pos_y), self.cover_card_rect )
                #
            self.card1 = None 
            self.card2 = None 
            self.card1_indices=None
            self.card2_indices=None
			#恢復接收MOUSEBUTTONDOWN的事件
			pygame.event.set_allowed(pygame.MOUSEBUTTONDOWN)
```

:sweat_smile:
