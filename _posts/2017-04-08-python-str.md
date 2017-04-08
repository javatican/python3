---
layout:	post
title:	Python str APIs
date:	2017-04-08
categories: python
---

花了一些時間整理了Python語言中的string資料處理的函式庫. Python在大數據處理的應用非常廣泛, 因此也提供極多的函式庫, 像是numpy, scipy等. 所以string處理的函式庫, 在這方面的應用就顯得十分重要. 因為內容很多, 所以這裡也是選擇性的擷取一些自己過去經常使用的一些函數做簡要的說明與使用範例. 

## str物件
Python string資料, 使用的是str型態的物件, 它的特性如下:
- Immutable(唯讀,不可改變) unicode字元的集合
- str常數: 可以使用
  - 單引號, 
  - 雙引號
  - triple quoted(三個單引號或三個雙引號）: 可以代表跨行str
- str() : 建構str物件
- Python沒有character型態, 因此`s[0]`等同於`s[0:1]`(長度為1的str物件)
- r”abc\tdef” : `raw string`, 不處理跳脫字元的語法
- 也是sequence物件, sequence物件包括list, tuple, range, str等

```
s1 = "abc\tdef"
print(s1)
s2 = r"abc\tdef" # raw string
print(s2)
```
## Sequence物件[共通的函數](https://docs.python.org/3.6/library/stdtypes.html#typesseq-common)

<img src="{{site.baseurl}}/assets/sequence_str.png" alt="Sequence str common" style="width: 600px;" />

```
import io
s1 = "abcdefg"
s2 = "hijklmn"
# x in s
print('a' in s1)
print('ab' in s1)
# x not in s
print('x' not in s1)
# use +  
print(s1+s2) #1
# use str.join() 
print("\t".join([s1,s2])) #1
# use in-memory stream 
output = io.StringIO() #1
output.write(s1)
print(s2,file=output)
print(output.getvalue())
output.close()
#s*n or n*s
print(s1*3)
print(3*s2)
# s[i] , 若i為負值, 相當於len(s)+ i
print(s1[1])
print(s1[-1]) #相當於s1[6]
# s[i:j]
print(s1[1:3])
print(s1[:3])
print(s1[1:])
# s[i:j:k]
print(s1[1:6:2])
# len(s)
print(len(s2))
# min(s)
print(min(s1))
# max(s)
print(max(s1))
# s.index(x)
print(s1.index('b')) #2
print(s2.index('a')) # raise ValueError
print(s1.index('b',3,6))
# s.count(x)
print(s2.count('a'))

```
說明:
1. 字串串接有幾種方式:
  - 使用 `+`
  - 使用 `str.join()`
  - 使用 in-memory stream `io.StringIO` 
1. `s.index(x)`: 在s中尋找substring x 出現的位置(從0開始), s中若找不到x, 則會raise `ValueError`, 所以通常會搭配`x in s`來使用.

## 其他[str函數](https://docs.python.org/3.6/library/stdtypes.html#string-methods)

- str.encode(encoding="utf-8", errors="strict") : 將str根據encoding轉換成bytes
- str.find(sub[, start[, end]]), str.rfind(sub[, start[, end]]) : 尋找substring
- str.format(*args, **kwargs) : 格式化字串
- str.strip([chars]), str.lstrip([chars]), str.rstrip([chars]) : 移除前面或後面的空白字元
- str.partition(sep), str.rpartition(sep) : 將字串根據sep來做一次分解, 產生三個elements的tuple(seq前面的部份, sep本身, sep後面的部份)
- str.replace(old, new[, count]) # 取代字串
- str.split(sep=None, maxsplit=-1), str.rsplit(sep=None, maxsplit=-1) # 將字串根據sep來做一次或多次的分解, 產生一個list物件, 包含分解後的各個substring 
- str.splitlines([keepends]) # 將多行字串拆解成單一行字串
- str.endswith(suffix[, start[, end]]), str.startswith(prefix[, start[, end]]) #判斷字串是否以substring作為開頭或結尾

備註: str.find()與str.index()的差別: str.index()當找不到substring時, 會raise ValueError, 但str.find()會回傳-1

```
import io
s1 = "abcdefg中文cde"
s2 = "hijklmn"
# str.encode(encoding="utf-8", errors="strict")
print(s1.encode())
# str.find(sub[, start[, end]]), str.rfind(sub[, start[, end]])
print(s1.find('c'))
print(s1.find('c',3,5))
print(s1.find('x')) # return -1 if not found
print(s1.rfind('c'))
# str.format(*args, **kwargs)
print("1+2={0},2*3={1}".format(1+2,2*3))
print("My firstname={first},lastname={last}".format(last="Nieh",first="Ryan"))
# str.strip([chars]), str.lstrip([chars]),  str.rstrip([chars])
s3 = "    abc   "
print("'{0}'".format(s3.strip()))
print("'{0}'".format(s3.lstrip()))
print("'{0}'".format(s3.rstrip()))
# str.partition(sep), str.rpartition(sep)
s4="ryan@yahoo.com.tw"
print(s4.partition('@')) 
print(s4.partition('.')) 
print(s4.rpartition('.')) 
# str.replace(old, new[, count])
s5="tw000tw111tw222tw333"
print(s5.replace('tw','TW',2))
# str.split(sep=None, maxsplit=-1), str.rsplit(sep=None, maxsplit=-1)
s6='12.34.56.78.90'
s7='.12.34.56..78.90.'
print(s6.split('.'))
print(s7.split('.'))
print(s6.split('.',maxsplit=2))
s8='a b c d e'
s9=' a  b  c  d  e '
print(s8.split())
print(s9.split())
# str.splitlines([keepends])
s10="""this is 1st line
this is 2nd line
this is 3rd line
this is 4th line """
print(s10.splitlines())
print(s10.splitlines(True))
# str.startswith(prefix[, start[, end]])
# str.endswith(prefix[, start[, end]])
s11="migulu001.py"
print(s11.startswith('migulu'))
print(s11.endswith('.py'))
```

## printf-style 的格式化字串

Python提供幾種formatting string的功能, 分別是上面介紹的`str.format()函數`, `printf-style格式化字串`, 及 `Formatted string literals`三種方式.
`Formatted string literals`是Python 3.6才推出的新的方式, 以後有機會在針對這個部份做說明. 這裡補充介紹printf-style格式化字串. 

printf-style名詞上而言, printf是C的一個列印函數, 因此是參考C的`printf()`語法而來. Python str物件有一個built-in`運算子%`可以使用, 稱為modulo或string formatting/interpolation operator. 基本上一個str物件的內容中, 可以包含**以%開頭的格式字串**, 而形成一個類似像模板的字串, 裡面的以%開頭的格式字串是可以動態被取代的 . 例如%d(十進位整數), %f(浮點數), %c(字元), %s(字串)等. 
其格式為`format % values`, format是一個字串, 裡面有%開頭的格式字串. values是一個tuple或mapping物件. 

範例: 
```
#printf-style string formatting
print("Good %s!" % "Morning") #1
print("Welcome to my site, %s. You are no. %d visitor." % ('Ryan',39)) #2
print("My name is %(name)s. I am %(age)d years old. My ID is %(id)03d." % {'name':'Ryan', 'age':40, 'id':7}) #3
```
說明:
1. format中僅有一個%s, 因此values只有一個值, 可以不用放在tuple之中
1. format中包含一個%s, 一個%d, 因此values必須有兩個值, 必須放在tuple中
1. format中包含具有名稱的格式字串, `%(name)s`, `%(age)d`, `%(id)03d`, 因此values必須使用mapping物件(如dict). 注意`%(id)03d`的d前面的03, 其代表會列印三位數字的整數, 若不足三位數則前面補0.

:sweat_smile:
