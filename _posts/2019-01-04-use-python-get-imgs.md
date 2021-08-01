---
layout: post
title:  "如何用python实现爬虫自动爬取百度图片原图"
date:     2019-01-04 16:31:25
tags:
    - python
    - 爬虫
author: "YL"
subtitle: ""
catalog: false
header-style: text
---
# 如何用python实现爬虫自动爬取百度图片原图



# 说点什么
其实一直以来，对于`python`这个语言还是很感兴趣的，但是以前一直在做图像处理相关的东西，所以对这种无法触及底层内存处理的语言一直没怎么关注过，不过最近实在是被C++的字符串处理和复杂芜杂的网络框架给整崩溃了，而且看到大家都说`python`很好玩，就趁着最近没事来学一下`python`。

昨天跟着廖雪峰老师的`python`教程（比较推荐它的基础教程），看了看基本的数据结构和逻辑之后，决定还是直接从实际的一个小项目来练手（对我来说这是我熟悉一门新语言的最好的方式），所以直接就选择比较简单的`python`爬虫项目来练手了。

写爬虫最重要的就是要了解我们平时访问的网页是什么东西，其实我们平时访问的网页其源码就是一个字符串文件（当然也可以说二进制文件）

每次当我们在浏览器上输入一个网址的时候，比如 `www.google.com `这个网址，浏览器就会根据其绑定的ip地址，帮助我们访问远端服务器并发送请求，远端服务器就会将这个网页的源码直接打包发给我们的浏览器（`html`文件格式），然后由我们的浏览器将这一字符串文件进行解析，比如通过`div、h1、h2`这些布局对网页进行格式化的渲染和展示，对`img`这些资源继续向远端服务器发送请求获取图片等，然后填装到网页上。
# 爬虫是什么
怎么说呢，爬虫，往简单的地方来说，就是模拟浏览器去获取一个网页的源码，至于你拿这个源码干嘛，那就不重要了。

复杂的说，爬虫其实是一切网页数据处理的综合，比如：自动爬取网页源码、分析网页数据、获取关键数据并进行保存，或者自动下载需要的网页图片、音频、视频的工具。

**所以当我们在写一个pythonm爬虫的时候，我们在写什么？**
当我们需要一个爬虫的时候，一般是我们遇到了比较复杂的数据需求，这个需求对于手工来说很复杂，单这个需求正是爬虫可以很简单地解决的。

所以首先确定我们的需求，我们需要这个爬虫去做什么？

比如我们觉得一个教学网站的数据很有意思，但是其格式太过于复杂，或者需要频繁的翻页不能凭复制粘贴很简单地拿到的时候，爬虫就有了作用。

总之，还是拿一个爬虫实例来进行说明吧。

# 爬取百度图片

## ps：python 版本  —— `Python 3.7.0`

第一个爬虫，当然不能选取太复杂的需要涉及到很多复杂的网络规则的东西，所以我们就爬一些很简单地允许爬虫爬的网站吧。

比如爬取并下载某个关键词下的**百度图片**的图片数据。

## 1、确立需求

- 可以修改关键词爬取我们想要的不同的图片集
- 可以选择爬取的图片数量
- 爬取的图片大小是原图片大小
- 爬取鲁棒性


暂时就确定上面的几个需求吧，然后开始真正的爬虫代码的编写

## 2、观察网页源码
首先打开百度图片，随意搜索一个关键词，比如这次用我比较喜欢的动漫角色  **栗山未来**  来进行测试

右键查看网页源代码之后可以看到，每张图片的原地址开头都是很简单的`"objURL"`，结尾都是一个小逗号，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190104162331379.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NvZGVkb2N0b3I=,size_16,color_FFFFFF,t_70)

根据这点我们可以确定这个爬虫该如何进行操作了，很简单——从网页源码中提取出这些url并进行下载就可以了 `\(^o^)/~`
## 3、获取网页源码

其实这一步我有点不想写的，不过担心有看我的博客刚学`python`的朋友可能会出现错误，还是说一下吧。

首先，我们需要引入`python`中的网络库`urllib`,由于我们要用到的`urlopen`和`read`方法在我这个版本的`python`中，直接引用`urllib`会出现错误，所以一般采用此`import`：`import urllib.request`

整个的提取网页源码的方法如下：

```python
import urllib.request
def get_html(httpUrl):
	page = urllib.request.urlopen( httpUrl )#打开网页
	htmlCode = page.read( )#读取网页
	return htmlCode
```

我们可以把这个方法打包成一个py文件，以后直接就可以在其他py文件中`import`一下这个方法，然后直接使用我们写好的`get_html`方法就可以了。

然后将上方获取到的二进制文件，解码成一个正常地字符串：

```python
html_code=get_html(search_url)
html_str=html_code.decode(encoding = "utf-8")#将二进制码解码为utf-8编码，即str
```


## 4、提取图片地址

说到提取网页源码中的关键数据，其实就是字符串匹配，这要是在`C++`里面估计得把我累死，当然我也的确写过类似的，甚至更复杂的，实在是惨痛的回忆。

这里向大家推荐一下比较简单的字符串检索和匹配方案——**正则表达式**，又称规则表达式。（英语：`Regular Expression`，在代码中常简写`为regex`、`regexp`或`RE`）

当然具体的如何使用正则表达式我就不多说了，大家自行搜索各个视频网站，看个大概就够了，或者直接看这里——[正则表达式基础](http://www.runoob.com/regexp/regexp-syntax.html)

了解一下正则表达式的基本逻辑就ok了。

进入正题，上面的一步已经分析了每张图片的原地址开头都是很简单的`"objURL"`，所以我们正则的关键就是把`"objURL"`后方的地址拿出来就够了，这很简单，我就直接把写好的正则表达式拿出来了：

```python
reg=r'"objURL":"(.*?)",'
```
ps:字符串前的r主要是防止少写了转义字符引起的字符缺失

这个正则的作用如下：
**匹配以`"objURL":"`开头，并且以`",`结尾的任意字符串**

然后就是简单地编译一下这个正则（记得`import`一下`re`包)：
```python
import re
reg=r'"objURL":"(.*?)",'
reg_str = r'"objURL":"(.*?)",'      #正则表达式
reg_compile = re.compile(reg_str)
```


然后用上方编译之后的正则表达式对第三步获取到的字符串进行解析，如下：

```python
pic_list = reg_compile.findall(html_str)
```
上方的`pic_list`就是一个简单的列表
将列表中的数据输出：
```python
    for pic in pic_list:
      print(pic)
```
输出如下：

```
http://b-ssl.duitang.com/uploads/item/201609/02/20160902174427_4H2V8.jpeg
http://b-ssl.duitang.com/uploads/item/201607/18/20160718133442_cnmKP.jpeg
http://cdnq.duitang.com/uploads/item/201504/05/20150405H2814_VvZfS.jpeg
http://wxpic.7399.com/nqvaoZtoY6GlxJuvYKfWmZlkxaJhpc-bna-SmqKfYm/lmqN-oo5Gop3nXfWqYhdpxmoPGZHOozqx7fpyToXCcmYaMvIe0g6GuocOqdnmP1WGrsLCZoY-ub36elaeBr42tb6nSh5qApHqCrnyPq6K0emd6152Ti7Crh3ZiYHGvq5acpNpu
http://i1.hdslb.com/bfs/archive/801e00579f4b5bd2b83dcdd665dcc7819fce4470.jpg
http://wxpic.7399.com/nqvaoZtoY6GlxJuvYKfWmZlkxaJhpc-bna-SmqKfYm/lmqN-oo5Gop3nXfWqYhdpxmoPGZHOozqx7fpyToXCcmYaHv3vFrntnotGGeaRpsKp8nbqmZnCXh5-tqZOIrqOgmZDWmsNxoJ2bs4Jsqoy-fm2BznWlf82clpxiYHGvq5acpNpu
```
可以明显地看到，其中有几个地址并不是图片地址，这可以说是我的正则出了一点问题，不过也并无伤大雅（实际是我懒得改了），稍微一点判断即可完美解决（由于所有图片地址结尾都为g，如`png、jpg、jpeg`  `^_^`）



```
for pic in pic_list:
	if pic[len(pic)-1]=='g':
		print(pic)
```

完美解决，w(ﾟДﾟ)w

ps：哈哈哈，当然是开玩笑的啦这里，大家记得自己想个好方法进行改正，算是一个小测试吧，毕竟真的无伤大雅。


## 5、下载图片

下载图片也很简单，我最开始已经说过，图片其实就是存在远端服务器的一个图片文件，只要服务器允许，我们一个GET请求就可以轻松地拿到这个图片了，毕竟浏览器也是这么做的，至于你拿到干什么，是用来真的填充网页还是自己用就没人在乎了。

下载图片`python`有很多的方法，如下存在三种

(以下代码非本人所写，版本是否正确也不确定，存在错误请自行百度，第一种在我的版本未出现问题）：

```python
def urllib_download(url):
    from urllib.request import urlretrieve
    urlretrieve(url, '1.png')     
 
def request_download(url):
    import requests
    r = requests.get(url)
    with open('1.png', 'wb') as f:
        f.write(r.content)                      
 
def chunk_download(url):
    import requests
    r = requests.get(url, stream=True)    
    with open('1.png', 'wb') as f:
        for chunk in r.iter_content(chunk_size=32):
            f.write(chunk)
 
```
所以我们只需要把上方的列表中的每个图片网址进行下载就可以了，我选择的是上方的第一种，这是存在于`urllib.request`中的一个方法，其定义如下：

```python
urllib.request.urlretrieve(url, filename, reporthook, data)
参数说明：
url：外部或者本地url
filename：指定了保存到本地的路径（如果未指定该参数，urllib会生成一个临时文件来保存数据）；
reporthook：是一个回调函数，当连接上服务器、以及相应的数据块传输完毕的时候会触发该回调。我们可以利用这个回调函数来显示当前的下载进度。
data：指post到服务器的数据。该方法返回一个包含两个元素的元组(filename, headers)，filename表示保存到本地的路径，header表示服务器的响应头。
```

然后就是简单的一个循环下载就ok了：


```python
def download_pic(pic_adr,x):
	urllib.request.urlretrieve(pic_adr, '%s.jpg' %x)
	# './images/%s.jpg'，这里也可以自己选择创建一个文件吧所有图片放进去，而不是直接放在当前目录下
	
x=0
for pic in pic_list:
	if pic[len(pic)-1]=='g':
		print(pic)
		download_pic(pic,x)
		x += 1
```
给每一个图片取一个土逼的名字当然是必须的，就递增1234这样取名就可以了。

这样我们就已经结束了整个爬虫的过程，执行之后，效果如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190104162525413.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NvZGVkb2N0b3I=,size_16,color_FFFFFF,t_70)

可以看到，效果还是不错的，毕竟放在那里让它自己下载就可以了，大家也可以想一下如何翻页之类的效果，其实也很简单，我就随便说一下实现一个更改关键词的效果来提示吧（其实是因为有点懒不想写）

## 6、更改搜索关键词

看百度搜索页面的网址：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190104162535863.png)

观察这一网址，我们可以很简单地发现，原来最后的keyword后面的就是关键词啊，复制下来改一下就可以了嘛，但是复制下来发现事情并不简单：

```
http://image.baidu.com/search/flip?tn=baiduimage&ipn=r&ct=201326592&cl=2&lm=-1&st=-1&fm=result&fr=&sf=1&fmq=1546589116401_R&pv=&ic=0&nc=1&z=&hd=&latest=&copyright=&se=1&showtab=0&fb=0&width=0&height=0&face=0&istype=2&ie=utf-8&ctd=1546589116402%5E00_1519X723&word=%E6%A0%97%E5%B1%B1%E6%9C%AA%E6%9D%A5
```
为什么`word`后面变成了`%E6%A0%97%E5%B1%B1%E6%9C%AA%E6%9D%A5`
这么一堆东西？

很简单，因为`url`是`ASCII`码，而这里我们用到了中文，用一点小操作转一下就可以了：

```python
keyword=urllib.parse.quote(keyword)
search_url="http://image.baidu.com/search/flip?tn=baiduimage&ipn=r&ct=201326592&cl=2&lm=-1&st=-1&fm=result&fr=&sf=1&fmq=1546580974349_R&pv=&ic=0&nc=1&z=&hd=&latest=&copyright=&se=1&showtab=0&fb=0&width=&height=&face=0&istype=2&ie=utf-8&ctd=1546580974351%5E00_1519X723&word="
search_url=search_url+keyword       #加上关键字
```
`urllib.parse.quote`就是解码的代码，在后面加上关键字就oK了。

简单吧，这个简单的python爬虫就写完了。。

虽然还是想自己写字符串识别，但是正则实在是太好用了。还是推荐大家多学一点正则的骚技巧吧。

就这样。

需要修改的自己复制下来完整代码进行更改就好了，下载多页这些也是很简单的操作，就懒得再说了，或者自己加上多线程，加上代理，加上动态ip都是可以的，反正我的第一个爬虫就写到这里了。


# 完整源代码（百度图片爬虫）

```python
import urllib
import time
from urllib.request import urlretrieve
import re
import urllib.request
def get_html(httpUrl):#获取网页源码
	page = urllib.request.urlopen( httpUrl )#打开网页
	htmlCode = page.read( )#读取网页
	return htmlCode

def get_keyword_urllist(keyword):#爬取当前关键词下的图片地址
	keyword=urllib.parse.quote(keyword)
	search_url="http://image.baidu.com/search/flip?tn=baiduimage&ipn=r&ct=201326592&cl=2&lm=-1&st=-1&fm=result&fr=&sf=1&fmq=1546580974349_R&pv=&ic=0&nc=1&z=&hd=&latest=&copyright=&se=1&showtab=0&fb=0&width=&height=&face=0&istype=2&ie=utf-8&ctd=1546580974351%5E00_1519X723&word="
	search_url=search_url+keyword       #加上关键字
	html_code=get_html(search_url)
	html_str=html_code.decode(encoding = "utf-8")#将二进制码解码为utf-8编码，即str
	reg_str = r'"objURL":"(.*?)",'      #正则表达式
	reg_compile = re.compile(reg_str)
	pic_list = reg_compile.findall(html_str)
	return pic_list


keyword="栗山未来"#自己修改，或者自己写个input或者一个txt自己读就完事了，甚至你写一个配置表，把爬取数量、爬取关键词、爬取图片大小都写好都可以
pic_list=get_keyword_urllist(keyword)

x=0
for pic in pic_list:
	if pic[len(pic)-1]=='g':
		print(pic)
		name = keyword+str(x)
		time.sleep(0.01)
		urllib.request.urlretrieve(pic, './images/%s.jpg' %name)
		x += 1
		
```


如果不能运行，请查看一下包含的模板是不是对的，或者版本是否正确。


# 重点
最后，大家如果运行了上方的源码应该会发现，虽然能够下载但是下载速度实在是太慢了，好一点几秒钟就解决了，但是慢一点的话可能需要十多秒钟。
针对于此，我重写了一下上方的代码，加上了多线程的处理，把爬取的时间缩短到了1s以内，直接上传到了我的github上，大家可以自行下载

# [快速爬取百度图片——爬虫1.0版本](https://github.com/xinyoulinxi/BaiDuImagesAutoDownload)