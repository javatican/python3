---
layout: post
title:  "Pygame介紹 - Part 3"
date:   2017-03-23
categories: pygame
---
 
這是PyGame介紹文的Part 3. 
[Part 1]({{site.baseurl}}{% post_url 2017-03-17-pygame-intro %})介紹了基本程式架構及draw module. 
[Part 2]({{site.baseurl}}{% post_url 2017-03-23-pygame-intro-2 %})介紹如何在pygame主迴圈裡重複畫新的矩形以及控制主迴圈執行的頻率 

繼續看如何使用Surface物件, 以及使用影像檔.

## Surface物件

在第一個範例時, 我們就用到了一個Surface物件, 也就是`pygame.display.setMode()`所產生的根視窗, 稱之為視窗, 其實不對, 應該是一個類似畫布的影像. 
Surface物件具有固定的解析度以及每個pixel(像素)的`bit depth`. 所謂的`bit depth`代表每一個pixel可以使用多少個bit來儲存其顏色. 
例如24-bit的depth代表可以有2^24個顏色階層, 約16.7百萬種. 

### 產生Surface物件

如何產生一個Surface物件呢? 直接呼叫建構子即可. 有兩個建構子可以使用:
```
Surface((width, height), flags=0, depth=0, masks=None) -> Surface
Surface((width, height), flags=0, Surface) -> Surface
```
備註: 第一個Surface建構子, 接受的depth參數的預設值為0, 代表會使用系統最佳的bit depth. 

### 如何呈現Surface物件

產生出來的Surface物件, 若要顯示出來, 就必須在根視窗(記得也是一個影像, 位於最底層的Surface)上呈現. 這個動作, 在電腦圖像領域有一個專有名詞來代表它, 稱為[Bit blit](https://en.wikipedia.org/wiki/Bit_blit), 是`bit block trasfer`的意思, 也就是**多個圖檔利用布林函數整合成一個圖檔**.

Surface物件有blit()函數可以完成這個工作. 說明文件如下:
```
blit()
	draw one image onto another
	blit(source, dest, area=None, special_flags = 0) -> Rect
	Draws a source Surface onto this Surface. The draw can be positioned with the dest argument. Dest can either be pair of coordinates representing the upper left corner of the source. A Rect can also be passed as the destination and the topleft corner of the rectangle will be used as the position for the blit. The size of the destination rectangle does not effect the blit.

	An optional area rectangle can be passed as well. This represents a smaller portion of the source Surface to draw.

```
看一個完整範例:
```python
import pygame

pygame.display.init()

screen = pygame.display.set_mode((400, 300)) 
screen.fill((100,0,0))
done = False
sur1 = pygame.Surface((100,100)) #1 
sur2 = pygame.Surface((100,100), pygame.SRCALPHA) #2
screen.blit(sur1,(100,200)) #3
screen.blit(sur2,(200,200))
while not done:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            done = True
        
    pygame.display.flip()
```
說明:
1. 呼叫Surface建構子, 產生Surface物件, 解析度為100*100, 預設會是一個全黑的影像
1. 同樣是呼叫Surface建構子, 但是多傳入了一個參數`pygame.SRCALPHA`, 表示這個影像的pixel顏色包含了透明度(alpha)值, 預設是全透明的影像.
1. `screen.blit(sur1,(100,200))`將sur1呈現在底層的screen上, 放置的位置在(100,200).

### 載入影像檔
在PyGame中, 每一個載入的影像檔, 也是一個Surface物件. 可以利用`pygame.image.load()`來載入. 看一看說明文件:
```
pygame.image.load()
	load new image from a file
	load(filename) -> Surface
	load(fileobj, namehint=””) -> Surface
	Load an image from a file source. You can pass either a filename or a Python file-like object.

	Pygame will automatically determine the image type (e.g., GIF or bitmap) and create a new Surface object from the data. In some cases it will need to know the file extension (e.g., GIF images should end in ”.gif”). If you pass a raw file-like object, you may also want to pass the original filename as the namehint argument.

	The returned Surface will contain the same color format, colorkey and alpha transparency as the file it came from. You will often want to call Surface.convert() with no arguments, to create a copy that will draw more quickly on the screen.

	For alpha transparency, like in .png images, use the convert_alpha() method after loading so that the image has per pixel transparency.

	Pygame may not always be built to support all image formats. At minimum it will support uncompressed BMP. If pygame.image.get_extended() returns ‘True’, you should be able to load most images (including PNG, JPG and GIF).

	You should use os.path.join() for compatibility.

	eg. asurf = pygame.image.load(os.path.join('data', 'bla.png'))
```

 
:sweat_smile:
