---
layout:	post
title:	PyGame遊戲實作 - Part 2
date:	2017-05-16
categories: python
--- 

撲克牌遊戲[第一部份]({{site.baseurl}}{% post_url 2017-05-15-pygame-card-game-1 %})先將52張牌呈現在畫面上, 然後做一些翻牌的動作.

第二部分將引入一套PyGame的簡單framework, 提供模組化的設計, 可以更方便的進行複雜遊戲的設計. 

## Scene manager PyGame framework

在網路上搜尋PyGame相關的Scene framework, 討論的結果不多, 或許是因為PyGame多用來設計一些簡單的遊戲.
不過即使是簡單的遊戲, 採用一些framework來設計還是可以省掉許多麻煩. 架構清晰, 自然做起工來就有條有理.

### Nerd Paradise
找到的第一套關於PyGame的Scene framework是[Nerd Paradise PyGame tutorial網站](http://www.nerdparadise.com/programming/pygame/part7). 
它的設計利用類別繼承與抽象方法, 定義`BaseScene`抽象類別. 然後搭配一個主要流程控制的`run_game()`函數.
所以我們的工作就是利用類別繼承, 定義BaseScene子類別. 每一個場景就使用一個BaseScene子類別. 

BaseScene類別裡面定義了三個抽象方法需要去實作. 這三個方法在每一個frame中都會被呼叫一次.
- `ProcessInput()` : 每個frame中針對有興趣的event型態做處理邏輯
- `Update()` : 每個frame中要進行的運算
- `Render()` : 每個frame中要更新的畫面.

BaseScene類別還提供了切換Scene物件的方法`SwitchToScene()`與中斷執行的方法`Terminate()`.

### Gestionando Escenas con Pygame

第二套找到的是一個西班牙文網站介紹的[Scene Manager framework](http://razonartificial.com/2010/08/gestionando-escenas-con-pygame/). 
有一個翻譯自這個網站的英文[網站](https://nicolasivanhoe.wordpress.com/2014/03/10/game-scene-manager-in-python-pygame/)提供了我需要的說明.

它與`Nerd Paradise`的作法差異之處在於, 前者使用了一個`Director類別`來作主要流程的控制物件(而後者Nerd Paradise只是使用了一個run_game()函數). 
所以在架構設計上, 前者更容易使用. 但是兩者的基本原理大同小異.

#### Director類別

這個控制流程的物件包含了以下幾個方法

- `__init__(self)`建構子: 在建立Director物件的時候, 建立一些成員變數, 包括
 1. self.screen : PyGame主要根視窗
 1. self.scene : Scene物件
 1. self.quit_flag : 控制主要迴圈是否繼續執行的變數
 1. self.clock : 控制frame rate的 pygame.time.Clock物件
- `loop(self)`方法 : 該方法包含了pygame的主要遊戲迴圈(game loop), event handling 迴圈.
在每一個frame中, 會依序呼叫Scene物件裡面的`on_event()`, `on_update()`, `on_draw()`三個方法.
- `change_scene(self, scene)`方法: 改變Scene物件(切換場景)
- `quit(self)`方法: 結束pygame的執行

```python
class Director:
    WIDTH = 800 #1
    HEIGHT = 600 #1
    CAPTION = "MY CARD GAME" #1
    FPS = 20 #1
    #
    def __init__(self):
        self.screen = pygame.display.set_mode((Director.WIDTH, Director.HEIGHT))
        self.scene = None
        self.quit_flag = False
        self.clock = pygame.time.Clock()
        pygame.display.set_caption(Director.CAPTION)
 
    def loop(self):
 
        while not self.quit_flag:
            time = self.clock.tick(Director.FPS)
 
            # Exit events
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    self.quit()
                if event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_ESCAPE:
                        self.quit()
 
                # Detect events
                self.scene.on_event(event)
 
            # Update scene
            self.scene.on_update()
 
            # Draw the screen
            self.scene.on_draw(self.screen)
            pygame.display.flip()
 
    def change_scene(self, scene):
        self.scene = scene
 
    def quit(self):
        self.quit_flag = True

```

備註:
1. 我定義了一些static variables, 例如: 
 - 根視窗尺寸 - WIDTH, HEIGHT 
 - frame rate - FPS 
 - 遊戲的主題 - CAPTION


#### Scene抽象類別

這個場景Scene抽象類別, 包含了以下幾個方法:

- `__init__(self)`建構子: 在建立Scene物件時, 也指定Director物件, 方便我們取得其中的成員變數, 例如根視窗物件(`screen`)
- `on_event()`方法 : 每個frame中針對有興趣的event型態做處理邏輯
- `on_update()`方法 : 每個frame中要進行的運算
- `on_draw()`方法 : 每個frame中要更新的畫面.

```python
class Scene:
     def __init__(self, director):
         self.director = director
 
     def on_update(self):
         raise NotImplementedError("on_update abstract method must be defined in subclass.")
 
     def on_event(self, event):
         raise NotImplementedError("on_event abstract method must be defined in subclass.")
 
     def on_draw(self, screen):
         raise NotImplementedError("on_draw abstract method must be defined in subclass.")
```

#### Scene子類別

我們目前只有一個場景, 所以定義一個Scene子類別`SceneHome類別`. 我使用static variables來儲存一些重要的變數.
- `RATIO = (Director.WIDTH/13)/Card.CARD_WIDTH` : 計算影像的縮放比例, Card.CARD_WIDTH是每一張card的原始寬度, Director.WIDTH是根視窗的寬度.
- `CARD_WIDTH = Director.WIDTH//13` : 計算要呈現的每一張card的尺寸(寬).
- `CARD_HEIGHT = int((Director.WIDTH/13)*(Card.CARD_HEIGHT/Card.CARD_WIDTH))` : 計算要呈現的每一張card的尺寸(高)
- `CARDS_IMAGE` : 用來儲存載入的影像檔.

`__init__(self)`建構子中, 主要進行兩個動作: 
1. 這個場景中會使用的資料的準備, 例如self.img, self.cards, self.cover_card_rect, self.sli, self.counter等.
 為了方便在其他成員方法中使用這些變數, 我都將之定義為成員變數.
1. 初始畫面的準備, 例如我要顯示這52張牌在根視窗中.

`on_draw(self, screen)`方法: 這個方法會在每一個frame被呼叫一次, 所以我是做掀牌的動作. 

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
		# 洗牌
        random.shuffle(self.sli)
        #
        white = (255,255,255)
        director.screen.fill(white)
        # 顯示52張牌
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
        self.counter=0
    def on_update(self):
        pass
 
    def on_event(self, event):
        pass
 
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

#### main()

最後是main()函數. 其中進行的步驟就是:
1. 初始化pygame modules
1. 建立Director物件
1. 建立Scene物件並且指定其為下一個場景
1. 啟動主要迴圈

```python
import pygame
from director import Director
from my_scenes import SceneHome
 
def main():
    pygame.display.init()
    dr = Director()
    scene = SceneHome(dr)
    dr.change_scene(scene)
    dr.loop()
 
if __name__ == '__main__':
    main()
```

:sweat_smile:
