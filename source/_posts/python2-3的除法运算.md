title: python2/3的除法运算
categories: Python
tags: Python
date: 2016-09-19 23:17:52
---

python 2和python 3的除法有一些区别，这个对于没有系统学过python的同学可能有点难理解。
python 3中对两个整数用`/`进行除法运算时，返回的是一个浮点数。
```
>>> 8/5 # Fractions aren't lost when dividing integers
1.6
```
那如果想得到整数结果呢？使用`//`。但还是要注意当两个整数中有一个负数，返回的结果可能跟你的第一直觉不一样。
```
>>> # Integer division returns the floor:
... 7//3
2
>>> 7//-3
-3
```
而在python 2中 `/` 和`//`表示的都是整数除法。

例如
```
>>> 11/2 
5

>>> 3//2 
1
```
通过使用语句`from __future__ import division`
可以改变这种状况 让`/`除变成浮点数除法。

## 参考
- https://docs.python.org/3.1/tutorial/introduction.html#using-python-as-a-calculator

