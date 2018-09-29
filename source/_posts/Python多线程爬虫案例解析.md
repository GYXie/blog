title: Python多线程爬虫案例解析
date: 2016-09-24 01:02:39
categories: 爬虫
tags:
  - Python
  - 爬虫
  - 多线程
---
*本文为作者原创文章，转载请注明出处。*

## 工具
爬网页的一个很重要的工具就是对网页进行分析，所以熟悉Chrome的开发者工具使用很重要（其实很简单，不要被吓到。）
- 使用Chrome可以查看网页的html源码
![](/img/clawer-01.png)
- 使用Chrome开发者工具可以查看网页指定部分的源码
![](/img/clawer-02.png)

## Python包
Python有很多有用的爬虫工具包，这里只使用两个最基本的包（入门简单最好了）。
- urllib2
用来获取网页的源码。简单地可以理解为：你填一个url，它给你返回网页代码。
- pyquery
主要用来定位找到我们感兴趣的html代码。

## 案例背景
一亩三分地论坛上有很多留学申请的录取信息，包含申请者的三维条件和录取情况，可以用来分析学校的录取条件等信息。

## 步骤
1. 获得我们要爬的所有网页的URL
    `http://www.1point3acres.com/bbs/forum-82-1.html`这个网页是该论坛录取汇报的入口。
    ![](/img/clawer-03.png)
    首先我们确定一下有多少录取汇报帖子。
    查看网页源码定位到下面这段html代码。
    ``` html
    <a class="bm_h" href="javascript:;" rel="forum.php?mod=forumdisplay&fid=82&sortid=164&%1=&sortid=164&page=2" curpage="1" id="autopbn" totalpage="1000" picstyle="0" forumdefstyle="">下一页 &raquo;</a>
    ```
    下面这行代码获取帖子数。
    ```
    num =  int(doc('#autopbn').attr('totalpage'))# 获取页数
    ```
    观察帖子的URL可以看到不用的帖子的区别在于后面的数据不同，而且这个数子的范围就在上面获取的帖子数内，所以我们就很容易可以构造出我们的真正要爬的网页的URL。
    ```
    page_url = 'http://www.1point3acres.com/bbs/forum-82-%s.html' % index
    ```
2. 设置多线程
根据系统的核数设置线程的数目。开启多线程可以加快我们的爬取速度。
```
pool = Pool(8)
```
3. 爬取网页
上面已经拿到了所有的URL。根据这个URL用urllib2去获取网页内容就可以了。
```
page = urllib2.urlopen(url)
content =  page.read()
```
4. 解析网页
其实上面已经用到了解析的方法，就是采用pyquery去定位到我们感兴趣的内容。
5. 保存数据
解析到网页后可以设计好一定的格式去保存数据。建议采用CSV格式，即以逗号分隔。这个格式Excel也可以很方面的打开处理。
![](/img/clawer-04.png)

6. 数据清理及分析
爬下来的数据可能包含一些脏数据，可以用正则表达式来做数据清洗。（下次有时间我会写一篇关于正则表达式方面的经验。）

## 完整源码
``` python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
from sys import argv
from pyquery import PyQuery as pyq
from multiprocessing import Pool
import traceback
import codecs
import urllib2
import sys
reload(sys)
sys.setdefaultencoding('UTF-8')

script, output_file, except_file = argv

output = codecs.open(output_file, 'w', 'UTF-8') # 保存爬取的数据的文件
except_urls = codecs.open(except_file, 'w', 'UTF-8') # 保存失败的URL的文件

url = 'http://www.1point3acres.com/bbs/forum-82-1.html'
page = urllib2.urlopen(url)
content =  unicode(page.read(), "gbk")
doc = pyq(content)
num =  int(doc('#autopbn').attr('totalpage'))# 获取页数

print '%s pages' % num

title_url = []
def get_report(url):
	if url == None:
		return
	try:
		page = urllib2.urlopen(url)
		content =  page.read()
		doc = pyq(content)
		title = doc('h1').text()
		postlist = doc('#postlist')
		main = postlist('div:first')
		main = main('.pcb')
		head = main('u').text()
		info_list = main('li').items()
		line = title + '\t' + head
		for item in info_list:
			line = '%s\t%s' % (line, item.text().strip('\r\n'))
		# line = line.replace(' ','')
		user = doc('.avatar')
		user_url = doc('.avtm').attr('href')
		line = '%s\t%s\t%s\t%s\r\n' % (line, main.text().replace('\t', ' '), url, user_url)
		output.write(line)
	except (KeyboardInterrupt, SystemExit):
		raise
	except Exception, e:
		print 'except: url = %s' % url
		except_urls.write('%s\r\n' % url)
		print e
		traceback.print_exc()
	return

def get_post_urls(index):
	try:
		page_url = 'http://www.1point3acres.com/bbs/forum-82-%s.html' % index
		page = urllib2.urlopen(page_url)
		content =  unicode(page.read(), "gbk")
		doc = pyq(content)
		table = doc('#threadlisttableid')
		for tr in table('tr').items():
			post_url = tr('.xst').attr('href')
			get_report(post_url)
			# post_title = tr('.xst').text()
			# cites = tr('cite').items()
			# print post_title
			# target.write('%s\t%s\r\n' % (post_title, post_url))
			# title_url.append('%s\t%s\r\n' % (post_title, post_url))
	except (KeyboardInterrupt, SystemExit):
		raise
	except Exception, e:
		print 'except: %s' % page_url
		print e
		traceback.print_exc()
	return

pool = Pool(8)
pool.map(get_post_urls, range(1, num + 1))
pool.close()
pool.join()

# for item in set(title_url):
# 	target.write(item)
except_urls.close()
output.close()
```


--- 
*这篇文章是针对非计算机专业的同学写的，如果有不清楚的地方欢迎在下面留言。我会不断完善这篇文章，希望能帮到大家。*
