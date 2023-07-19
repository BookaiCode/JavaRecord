[TOC]


> 先看后赞，养成习惯。 
> 点赞收藏，人生辉煌。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNdxXrwibuF9V7y4ibsCVBeSh79TWlAPEzT4bniaPZF1TNYUlxz9EMGUCGYTzNnuyibD8hjRPibaS136Kw/640?wx_fmt=png)

**讲解我们的爬虫之前，先概述关于爬虫的简单概念（毕竟是零基础教程）**

## 爬虫
网络爬虫（又被称为网页蜘蛛，网络机器人）就是模拟浏览器发送网络请求，接收请求响应，一种按照一定的规则，自动地抓取互联网信息的程序。
原则上，只要是浏览器(客户端)能做的事情，爬虫都能够做。

## 为什么我们要使用爬虫

互联网大数据时代，给予我们的是生活的便利以及海量数据爆炸式的出现在网络中。

过去，我们通过书籍、报纸、电视、广播或许信息，这些信息数量有限，且是经过一定的筛选，信息相对而言比较有效，但是缺点则是信息面太过于狭窄了。不对称的信息传导，以致于我们视野受限，无法了解到更多的信息和知识。

互联网大数据时代，我们突然间，信息获取自由了，我们得到了海量的信息，但是大多数都是无效的垃圾信息。例如新浪微博，一天产生数亿条的状态更新。

在如此海量的信息碎片中，我们如何获取对自己有用的信息呢？

**答案是筛选！**

通过某项技术将相关的内容收集起来，再分析筛选才能得到我们真正需要的信息。

这个信息收集分析整合的工作，可应用的范畴非常的广泛，无论是生活服务、出行旅行、金融投资、各类制造业的产品市场需求等等……都能够借助这个技术获取更精准有效的信息加以利用。

**网络爬虫技术，虽说有个诡异的名字，本能第一反应是那种软软的蠕动的生物，但它却是一个可以在虚拟世界里，无往不前的利器**。

## 爬虫准备工作
**我们平时都说Python爬虫，其实这里可能有个误解，爬虫并不是Python独有的，可以做爬虫的语言有很多例如：PHP，JAVA，C#，C++，Python，选择Python做爬虫是因为Python相对来说比较简单，而且功能比较齐全**。

首先我们需要下载python，我下载的是官方最新的版本 3.8.3

其次我们需要一个Python的代码编辑器，我用的是Pychram。

下载链接：https://www.jetbrains.com/pycharm/download/#section=windows

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNZB7vgjGbxDk1W3xIibbicCtNqiaZlhNargtEctmqy5I189Am0CdN8JOiaarTx8S1kM1uicjFVic0Ks62g/640?wx_fmt=png)

我们还需要一些库来支持爬虫的运行（有些库Python可能自带了)

```python
# -*- codeing = utf-8 -*-
from bs4 import BeautifulSoup  #网页解析，获取数据
import re  #正则表达式，进行文字匹配
import urllib.request, urllib.error  #制定URL，获取网页数据
import xlwt  #进行excel操作
import sqlite3 #进行SQLite数据库操作
```

差不多就是这几个库了，良心的我已经在后面写好注释了。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNZB7vgjGbxDk1W3xIibbicCtj1Kia728IrzPESw8mLOM13obly8AfWyhdosxzKkr8yV9RKoXOSoLSvQ/640?wx_fmt=png)
**爬虫运行过程中，不一定就只需要上面几个库，看你爬虫的一个具体写法了，反正需要库的话我们可以直接在setting里面安装)**


## 爬虫项目讲解
我们要爬取的就是这个网站：https://movie.douban.com/top250

我们的爬取的内容是：**电影详情链接，图片链接，影片中文名，影片外国名，评分，评价数，概况，相关信息**。

这边我已经爬取好了，将爬取内容存入xls表中，看一下效果图：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNdxXrwibuF9V7y4ibsCVBeShQibAq56JCSppBo80qJficvUaFwD1zFmtZibRaKsnJMt9nibObiczuJaz6pg/640?wx_fmt=png)

## 代码分析
先把代码放上来，然后我根据代码逐步解析：

```python
# -*- codeing = utf-8 -*-
from bs4 import BeautifulSoup  # 网页解析，获取数据
import re  # 正则表达式，进行文字匹配`
import urllib.request, urllib.error  # 制定URL，获取网页数据
import xlwt  # 进行excel操作
#import sqlite3  # 进行SQLite数据库操作

findLink = re.compile(r'<a href="(.*?)">')  # 创建正则表达式对象，标售规则   影片详情链接的规则
findImgSrc = re.compile(r'<img.*src="(.*?)"', re.S)
findTitle = re.compile(r'<span class="title">(.*)</span>')
findRating = re.compile(r'<span class="rating_num" property="v:average">(.*)</span>')
findJudge = re.compile(r'<span>(\d*)人评价</span>')
findInq = re.compile(r'<span class="inq">(.*)</span>')
findBd = re.compile(r'<p class="">(.*?)</p>', re.S)


def main():
    baseurl = "https://movie.douban.com/top250?start="  #要爬取的网页链接
    # 1.爬取网页
    datalist = getData(baseurl)
    savepath = "豆瓣电影Top250.xls"    #当前目录新建XLS，存储进去
    # dbpath = "movie.db"              #当前目录新建数据库，存储进去
    # 3.保存数据
    saveData(datalist,savepath)      #2种存储方式可以只选择一种
    # saveData2DB(datalist,dbpath)


# 爬取网页
def getData(baseurl):
    datalist = []  #用来存储爬取的网页信息
    for i in range(0, 10):  # 调用获取页面信息的函数，10次
        url = baseurl + str(i * 25)
        html = askURL(url)  # 保存获取到的网页源码
        # 2.逐一解析数据
        soup = BeautifulSoup(html, "html.parser")
        for item in soup.find_all('div', class_="item"):  # 查找符合要求的字符串
            data = []  # 保存一部电影所有信息
            item = str(item)
            link = re.findall(findLink, item)[0]  # 通过正则表达式查找
            data.append(link)
            imgSrc = re.findall(findImgSrc, item)[0]
            data.append(imgSrc)
            titles = re.findall(findTitle, item)
            if (len(titles) == 2):
                ctitle = titles[0]
                data.append(ctitle)
                otitle = titles[1].replace("/", "")  #消除转义字符
                data.append(otitle)
            else:
                data.append(titles[0])
                data.append(' ')
            rating = re.findall(findRating, item)[0]
            data.append(rating)
            judgeNum = re.findall(findJudge, item)[0]
            data.append(judgeNum)
            inq = re.findall(findInq, item)
            if len(inq) != 0:
                inq = inq[0].replace("。", "")
                data.append(inq)
            else:
                data.append(" ")
            bd = re.findall(findBd, item)[0]
            bd = re.sub('<br(\s+)?/>(\s+)?', "", bd)
            bd = re.sub('/', "", bd)
            data.append(bd.strip())
            datalist.append(data)

    return datalist


# 得到指定一个URL的网页内容
def askURL(url):
    head = {  # 模拟浏览器头部信息，向豆瓣服务器发送消息
        "User-Agent": "Mozilla / 5.0(Windows NT 10.0; Win64; x64) AppleWebKit / 537.36(KHTML, like Gecko) Chrome / 80.0.3987.122  Safari / 537.36"
    }
    # 用户代理，表示告诉豆瓣服务器，我们是什么类型的机器、浏览器（本质上是告诉浏览器，我们可以接收什么水平的文件内容）

    request = urllib.request.Request(url, headers=head)
    html = ""
    try:
        response = urllib.request.urlopen(request)
        html = response.read().decode("utf-8")
    except urllib.error.URLError as e:
        if hasattr(e, "code"):
            print(e.code)
        if hasattr(e, "reason"):
            print(e.reason)
    return html


# 保存数据到表格
def saveData(datalist,savepath):
    print("save.......")
    book = xlwt.Workbook(encoding="utf-8",style_compression=0) #创建workbook对象
    sheet = book.add_sheet('豆瓣电影Top250', cell_overwrite_ok=True) #创建工作表
    col = ("电影详情链接","图片链接","影片中文名","影片外国名","评分","评价数","概况","相关信息")
    for i in range(0,8):
        sheet.write(0,i,col[i])  #列名
    for i in range(0,250):
        # print("第%d条" %(i+1))       #输出语句，用来测试
        data = datalist[i]
        for j in range(0,8):
            sheet.write(i+1,j,data[j])  #数据
    book.save(savepath) #保存

# def saveData2DB(datalist,dbpath):
#     init_db(dbpath)
#     conn = sqlite3.connect(dbpath)
#     cur = conn.cursor()
#     for data in datalist:
#             for index in range(len(data)):
#                 if index == 4 or index == 5:
#                     continue
#                 data[index] = '"'+data[index]+'"'
#             sql = '''
#                     insert into movie250(
#                     info_link,pic_link,cname,ename,score,rated,instroduction,info)
#                     values (%s)'''%",".join(data)
#             # print(sql)     #输出查询语句，用来测试
#             cur.execute(sql)
#             conn.commit()
#     cur.close
#     conn.close()


# def init_db(dbpath):
#     sql = '''
#         create table movie250(
#         id integer  primary  key autoincrement,
#         info_link text,
#         pic_link text,
#         cname varchar,
#         ename varchar ,
#         score numeric,
#         rated numeric,
#         instroduction text,
#         info text
#         )
#
#
#     '''  #创建数据表
#     conn = sqlite3.connect(dbpath)
#     cursor = conn.cursor()
#     cursor.execute(sql)
#     conn.commit()
#     conn.close()

# 保存数据到数据库



if __name__ == "__main__":  # 当程序执行时
    # 调用函数
     main()
    # init_db("movietest.db")
     print("爬取完毕！")

```

**下面我根据代码，从下到下给大家讲解分析一遍**
![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNZB7vgjGbxDk1W3xIibbicCttk8zeIxLsnblNPnN8rzmTO3K6JxgG3cdwPDQYWBYXCxDgdYKtqV1MA/640?wx_fmt=png)

 -*- codeing = utf-8 -*-，开头的这个是设置编码为utf-8 ，写在开头，防止乱码。

 然后下面 **import**就是导入一些库，做做准备工作，（sqlite3这库我并没有用到所以我注释起来了）。

 下面一些**find**开头的是正则表达式，是用来我们筛选信息的。（正则表达式用到 re 库，也可以不用正则表达式，不是必须的。）

大体流程分三步走：

1. 爬取网页
2. 逐一解析数据
 3. 保存网页

**1.爬取网页**

先分析流程1，爬取网页，baseurl 就是我们要爬虫的网页网址，往下走，调用了 getData（baseurl) ，我们来看 getData方法：

```python
  for i in range(0, 10):  # 调用获取页面信息的函数，10次
        url = baseurl + str(i * 25)
```

这段大家可能看不懂，其实是这样的：

因为电影评分Top250，每个页面只显示25个，所以我们需要访问页面10次，25*10=250。

```python
baseurl = "https://movie.douban.com/top250?start="
```
我们只要在baseurl后面加上数字就会跳到相应页面，比如i=1时

[https://movie.douban.com/top250?start=25](https://movie.douban.com/top250?start=25)

我放上超链接，大家可以点击看看会跳到哪个页面，毕竟实践出真知。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNZB7vgjGbxDk1W3xIibbicCtLZ1ic4DlsdDibAoUicVTTCrbbpXgOREicvqlAkSAIicSuWdxpzIIUNJQ0kw/640?wx_fmt=png)

然后又调用了askURL来请求网页，这个方法是请求网页的主体方法，怕大家翻页麻烦，我再把代码复制一遍，让大家有个直观感受。

```python
def askURL(url):
    head = {  # 模拟浏览器头部信息，向豆瓣服务器发送消息
        "User-Agent": "Mozilla / 5.0(Windows NT 10.0; Win64; x64) AppleWebKit / 537.36(KHTML, like Gecko) Chrome / 80.0.3987.122  Safari / 537.36"
    }
    # 用户代理，表示告诉豆瓣服务器，我们是什么类型的机器、浏览器（本质上是告诉浏览器，我们可以接收什么水平的文件内容）

    request = urllib.request.Request(url, headers=head)
    html = ""
    try:
        response = urllib.request.urlopen(request)
        html = response.read().decode("utf-8")
    except urllib.error.URLError as e:
        if hasattr(e, "code"):
            print(e.code)
        if hasattr(e, "reason"):
            print(e.reason)
    return html
```

这个askURL就是用来向网页发送请求用的，**那么这里就有老铁问了，为什么这里要写个head呢？**

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNZB7vgjGbxDk1W3xIibbicCtRktKScWGCAU2pCdwyhribjwMYibLzlibBqreNP3G8rb2YiaUNicjdvmVvng/640?wx_fmt=png)

这是因为我们要是不写的话，访问某些网站的时候会被认出来爬虫，显示错误，错误代码 。
## 418
这是一个梗大家可以百度下，

> 418 I'm a teapot
>
> The HTTP 418 I'm a teapot client error response code indicates that
> the server refuses to brew coffee because it is a teapot. This error
> is a reference to Hyper Text Coffee Pot Control Protocol which was an
> April Fools' joke in 1998.

**我是一个茶壶**

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNdxXrwibuF9V7y4ibsCVBeShknwt7J285S0GMiaCPgRKiabpZ8RmZTsBIOztpCcTJ3lRTibwp27brzH3g/640?wx_fmt=png)

所以我们需要 “装” ，装成我们就是一个浏览器，这样就不会被认出来，伪装一个身份。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNdxXrwibuF9V7y4ibsCVBeSh48RyEy2Ny7hlHibnWoWgQTbAW9cXKibMiaDbXVOdBbZu3zbupqR6mzmQA/640?wx_fmt=png)

来，我们继续往下走，

```python
html = response.read().decode("utf-8")
```

这段就是我们读取网页的内容，设置编码为utf-8，目的就是为了防止乱码。访问成功后，来到了第二个流程：

**2.逐一解析数据**

解析数据这里我们用到了 BeautifulSoup（靓汤） 这个库，这个库是几乎是做爬虫必备的库，无论你是什么写法。

下面就开始查找符合我们要求的数据，用BeautifulSoup的方法以及 re 库的正则表达式去匹配：

```python
findLink = re.compile(r'<a href="(.*?)">')  # 创建正则表达式对象，标售规则   影片详情链接的规则
findImgSrc = re.compile(r'<img.*src="(.*?)"', re.S)
findTitle = re.compile(r'<span class="title">(.*)</span>')
findRating = re.compile(r'<span class="rating_num" property="v:average">(.*)</span>')
findJudge = re.compile(r'<span>(\d*)人评价</span>')
findInq = re.compile(r'<span class="inq">(.*)</span>')
findBd = re.compile(r'<p class="">(.*?)</p>', re.S)
```

匹配到符合我们要求的数据，然后存进dataList， 所以 dataList 里就存放着我们需要的数据了。

最后一个流程：

**3.保存数据**


```python
 # 3.保存数据
saveData(datalist,savepath)      #2种存储方式可以只选择一种
# saveData2DB(datalist,dbpath)
```

保存数据可以选择保存到 xls 表， 需要（xlwt库支持）

也可以选择保存数据到 sqlite数据库， 需要（sqlite3库支持）

这里我选择保存到 xls  表 ，这也是为什么我注释了一大堆代码，注释的部分就是保存到 sqlite 数据库的代码，二者选一就行。

保存到 xls 的主体方法是 saveData （下面的saveData2DB方法是保存到sqlite数据库）：

```python
def saveData(datalist,savepath):
    print("save.......")
    book = xlwt.Workbook(encoding="utf-8",style_compression=0) #创建workbook对象
    sheet = book.add_sheet('豆瓣电影Top250', cell_overwrite_ok=True) #创建工作表
    col = ("电影详情链接","图片链接","影片中文名","影片外国名","评分","评价数","概况","相关信息")
    for i in range(0,8):
        sheet.write(0,i,col[i])  #列名
    for i in range(0,250):
        # print("第%d条" %(i+1))       #输出语句，用来测试
        data = datalist[i]
        for j in range(0,8):
            sheet.write(i+1,j,data[j])  #数据
    book.save(savepath) #保存
```

创建工作表，创列（会在当前目录下创建）：

```python
 sheet = book.add_sheet('豆瓣电影Top250', cell_overwrite_ok=True) #创建工作表
 col = ("电影详情链接","图片链接","影片中文名","影片外国名","评分","评价数","概况","相关信息")
```

然后把 dataList里的数据一条条存进去就行。最后运作成功后，会在左侧生成这么一个文件：

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNZB7vgjGbxDk1W3xIibbicCth0KtqWoNNZUibtafg0hoKonMibRJv2x6ibShbJBvSgjcDag5iamicSdF2tQ/640?wx_fmt=png)

打开之后看看是不是我们想要的结果：
![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNdxXrwibuF9V7y4ibsCVBeShQibAq56JCSppBo80qJficvUaFwD1zFmtZibRaKsnJMt9nibObiczuJaz6pg/640?wx_fmt=png)

**成了，成了！**

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScNZB7vgjGbxDk1W3xIibbicCtOnGewic5hNI9giaiaDlzZo6uFPyOf4uwSUC56ZFoTPOfIia8ZWicq63DcsA/640?wx_fmt=png)

**如果我们需要以数据库方式存储，可以先生成 xls 文件，再把 xls 文件导入数据库中，就可以啦**

**我也在不断的学习中，学到新东西第一时间会跟大家分享，大家可以动动小手，点波关注不迷路。**
