---
layout: post
title:  "Pygame介紹 - Part 4"
date:   2017-03-28
categories: pygame
---

這是PyGame介紹文的Part 4. 

[Part 1]({{site.baseurl}}{% post_url 2017-03-17-pygame-intro %})介紹了基本程式架
構及draw module. 

[Part 2]({{site.baseurl}}{% post_url 2017-03-23-pygame-intro-2 %})介紹如何在pyga
me主迴圈裡重複畫新的矩形以及控制主迴圈執行的頻率 

[Part 3]({{site.baseurl}}{% post_url 2017-03-23-pygame-intro-3 %})介紹使用影像檔, 影像blit的運算, 然後顯示在根視窗上

接著看看如何在Surface上顯示文字.

Pygame顯示文字的方式, 也是利用Surface物件, 類似影像blit的運作方式. 因此文字並不是直接在某個已經存在的Surface物件上顯示, 而是利用Font物件的render()來產生一個文字影像Surface物件, 然後與要呈現的底層Surface物件做blit運算.

## Font物件

產生文字影像Surface物件的第一步驟, 就是要產生一個Font物件. 代表所要使用的字型與大小. 有以下幾種方式:
- `font.SysFont("dejavuserif",36)` : 使用系統提供的font, 例如dejavuserif. 可以利用`font.get_fonts()`來取得目前系統所具備的fonts
- `font.Font(None,36)` : 使用系統預設的font.
- `font.Font("resources/fonts/papyrus.ttf")` : 指定所要使用的font的檔案路徑.

範例:

```python
import pygame

pygame.init()
width=640
height=480
screen = pygame.display.set_mode((width, height))
clock = pygame.time.Clock()
done = False

font = pygame.font.SysFont("dejavuserif", 36) #1

text_surface = font.render("Hello World!", True, (255, 0, 0)) #2

while not done:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            done = True
    
    screen.fill((255, 255, 255))
    position =  width//2 - text_surface.get_width() // 2, height//2 - text_surface.get_height() // 2
    screen.blit(text_surface, position) #3
    
    pygame.display.flip()
    clock.tick(60)

```

說明: 
1. 使用`font.SysFont()`指定要使用的系統字型dejavuserif, 大小36
2. 呼叫`font.render()`產生Surface物件, 指定要顯示的文字, 顏色, 還有一個antialias的屬性, 後面再詳述.
3. `screen.blit()` 將文字影像blit到根視窗上. 位置至中.

## Font.render()

貼上它的說明文件:
```
render()
	draw text on a new Surface
	render(text, antialias, color, background=None) -> Surface
	This creates a new Surface with the specified text rendered on it. pygame provides no way to directly draw text on an existing Surface: instead you must use Font.render() to create an image (Surface) of the text, then blit this image onto another Surface.

	The text can only be a single line: newline characters are not rendered. Null characters (‘x00’) raise a TypeError. Both Unicode and char (byte) strings are accepted. For Unicode strings only UCS-2 characters (‘u0001’ to ‘uFFFF’) are recognized. Anything greater raises a UnicodeError. For char strings a LATIN1 encoding is assumed. The antialias argument is a boolean: if true the characters will have smooth edges. The color argument is the color of the text [e.g.: (0,0,255) for blue]. The optional background argument is a color to use for the text background. If no background is passed the area outside the text will be transparent.

	The Surface returned will be of the dimensions required to hold the text. (the same as those returned by Font.size()). If an empty string is passed for the text, a blank surface will be returned that is one pixel wide and the height of the font.

	Depending on the type of background and antialiasing used, this returns different types of Surfaces. For performance reasons, it is good to know what type of image will be used. If antialiasing is not used, the return image will always be an 8-bit image with a two-color palette. If the background is transparent a colorkey will be set. Antialiased images are rendered to 24-bit RGB images. If the background is transparent a pixel alpha will be included.

	Optimization: if you know that the final destination for the text (on the screen) will always have a solid background, and the text is antialiased, you can improve performance by specifying the background color. This will cause the resulting image to maintain transparency information by colorkey rather than (much less efficient) alpha values.

	If you render ‘n’ a unknown char will be rendered. Usually a rectangle. Instead you need to handle new lines yourself.

	Font rendering is not thread safe: only a single thread can render text at any time.
```

這裡做一些重點提示:
- 要顯示的文字僅限於單一行文字
- antialias參數代表是否要將字型做邊緣平滑化的處理
- background: 代表所要使用的背景色, 若不提供則代表使用透明色
- 回傳的Surface物件, 其大小會使用剛好可以包含所要呈現的文字的最小尺寸. 

 

