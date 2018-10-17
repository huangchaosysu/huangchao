---
layout: single
author_profile: true
title: "python爬虫入门-02"
date: 2018-10-07 10:30:53
# toc: true
tags:
  - python
  - 爬虫
categories:
  - python
  - python爬虫
---


爬取网易云音乐播放数大于500万的歌单。

打开歌单的url: http://music.163.com/#/discover/playlist，然后用lxml.html提取播放数<span class="nb">3715</span>。结果表明，我们什么也没提取到。难道我们打开了一个假的网页？

动态网页：所谓的动态网页，是指跟静态网页相对的一种网页编程技术。静态网页，随着html代码的生成，页面的内容和显示效果就基本上不会发生变化了——除非你修改页面代码。而动态网页则不然，页面代码虽然没有变，但是显示的内容却是可以随着时间、环境或者数据库操作的结果而发生改变的。

值得强调的是，不要将动态网页和页面内容是否有动感混为一谈。这里说的动态网页，与网页上的各种动画、滚动字幕等视觉上的动态效果没有直接关系，动态网页也可以是纯文字内容的，也可以是包含各种动画的内容，这些只是网页具体内容的表现形式，无论网页是否具有动态效果，只要是采用了动态网站技术生成的网页都可以称为动态网页。

现在我们明白了，这是一个动态网页，我们得到它的时候，歌单还没请求到呢，当然什么都提取不出来！

我们之前的技术不能执行那些让页面产生各种神奇效果的JavaScript 代码。如果网站的HTML页面没有运行JavaScript，就可能和你在浏览器里看到的样子完全不同，因为浏览器可以正确地执行JavaScript。用Python 解决这个问题只有两种途径：直接从JavaScript 代码里采集内容，或者用Python 的第三方库运行JavaScript，直接采集你在浏览器里看到的页面。我们当然选择后者。今天第一课，不深究原理，先简单粗暴的实现我们的小目标。

Selenium：是一个强大的网络数据采集工具，其最初是为网站自动化测试而开发的。近几年，它还被广泛用于获取精确的网站快照，因为它们可以直接运行在浏览器上。Selenium 库是一个在WebDriver 上调用的API。WebDriver 有点儿像可以加载网站的浏览器，但是它也可以像BeautifulSoup对象一样用来查找页面元素，与页面上的元素进行交互（发送文本、点击等），以及执行其他动作来运行网络爬虫。安装方式与其他Python第三方库一样

安装: pip3 install Selenium

验证一下:

![](/assets/images/spider/spider02.png)

Selenium 自己不带浏览器，它需要与第三方浏览器结合在一起使用。例如，如果你在Firefox 上运行Selenium，可以直接看到一个Firefox 窗口被打开，进入网站，然后执行你在代码中设置的动作。虽然这样可以看得更清楚，但不适用于我们的爬虫程序，爬一页就打开一页效率太低，所以我们用一个叫PhantomJS的工具代替真实的浏览器。

PhantomJS：是一个“无头”（headless）浏览器。它会把网站加载到内存并执行页面上的JavaScript，但是它不会向用户展示网页的图形界面。把Selenium和PhantomJS 结合在一起，就可以运行一个非常强大的网络爬虫了，可以处理cookie、JavaScript、header，以及任何你需要做的事情。

PhantomJS并不是Python的第三方库，不能用pip安装。它是一个完善的浏览器，所以你需要去它的官方网站下载，然后把可执行文件拷贝到Python安装目录的Scripts文件夹，像这样：

开始干活！

打开歌单的第一页：  http://music.163.com/#/discover/playlist/?order=hot&cat=%E5%85%A8%E9%83%A8&limit=35&offset=0

用Chrome的“开发者工具”F12先分析一下，很容易就看穿了一切。

![](/assets/images/spider/spider03.png)

播放数nb (number broadcast)：29915

封面 msk (mask)：有标题和url

同理，可以找到“下一页”的url，最后一页的url是“javascript:void(0)”。

最后，让我们用代码完成工作。

```
from selenium import webdriver

url = "https://music.163.com/#/discover/playlist/?order=hot&cat=%E5%85%A8%E9%83%A8&limit=35&offset=0"

driver = webdriver.PhantomJS()

driver.get(url)  # 加载页面

driver.switch_to.frame("contentFrame")  # 切换到歌单所在的iframe

# 歌单所有项目的包裹元素
data = driver.find_element_by_id('m-pl-container').find_elements_by_tag_name('li')

for d in data:
    nb = d.find_element_by_class_name('nb').text  # 播放数
    msk = d.find_element_by_css_selector("a.msk")
    print(msk.get_attribute('title'), nb, msk.get_attribute('href'))
```

结果

```
我不想当锦鲤，我只想我喜欢的人也喜欢我 10万 https://music.163.com/playlist?id=2465170307
威士忌风味·野性浪漫醇香度43% 43918 https://music.163.com/playlist?id=555244168
柔情/迷离•落日余晖下那抹绚烂的幻影 15547 https://music.163.com/playlist?id=619162172
单身都市丽人生存手册 20347 https://music.163.com/playlist?id=2465652213
avex 动漫歌曲宅急便 Part. 2 7898 https://music.163.com/playlist?id=2470305428
『韩』秋日渐近，唱一曲悲凉 14354 https://music.163.com/playlist?id=977452696
全球史诗级万评电音!百种风格【主打】 172万 https://music.163.com/playlist?id=2303371412
2018上半年最热新歌TOP50 1143万 https://music.163.com/playlist?id=2303649893
「高质量英文歌」让你单曲循环的英文歌 364万 https://music.163.com/playlist?id=2256615030
「纯音」嘘，我的悲伤才刚刚睡着 138万 https://music.163.com/playlist?id=2319189362
电影中的钢琴曲｜最会讲故事³ 481万 https://music.163.com/playlist?id=2321804269
单循辑｜我想和你共享耳机 1778万 https://music.163.com/playlist?id=2337333174
『欧美』书店书城常用流行钢琴曲推荐② 197万 https://music.163.com/playlist?id=2309425946
我最大的遗憾，是你的遗憾与我有关 1085万 https://music.163.com/playlist?id=2201957752
予你情诗百首，余生你是我的所有 1726万 https://music.163.com/playlist?id=2230318386
海盐风味｜浪漫迷幻°新鲜度100% 545万 https://music.163.com/playlist?id=2333731148
最怕rapper说情话 463万 https://music.163.com/playlist?id=2349211741
这世界上情歌那么多，却没有一首属于我 1709万 https://music.163.com/playlist?id=2335662972
这些歌名连起来，是我暗恋你多年的秘密 1751万 https://music.163.com/playlist?id=2301753292
青柠味复古 ✘ 凌步夏傍西海岸的空幻 335万 https://music.163.com/playlist?id=2296601213
「伪治愈系Rap」你价值连城伤口的良药 125万 https://music.163.com/playlist?id=2341864198
失落少年孤独心俱乐部 674万 https://music.163.com/playlist?id=2323596120
蓝莓味气泡水° 137万 https://music.163.com/playlist?id=2342277912
盘点那些欧美流行歌手的不烂大街好听歌曲 474万 https://music.163.com/playlist?id=2359934198
同桌的你，曾是我期待上课的理由 752万 https://music.163.com/playlist?id=2253353419
```


正常情况下是这样的，如果你想要爬所有的数据，还需要引入翻页机制（即把url里的页码--offset--替换）

由于selenium在不久的将来会放弃对phantomjs的支持， 所以这里也提供一个替代方案，使用chrome的headless模式。使用这个模式你需要下载chromedriver, 并把这个文件所在目录加到PATH， 也可以拷贝到python路径下面

```
from selenium import webdriver

url = "https://music.163.com/#/discover/playlist/?order=hot&cat=%E5%85%A8%E9%83%A8&limit=35&offset=0"

chromeOptions = webdriver.ChromeOptions()
chromeOptions.add_argument("headless")
driver = webdriver.Chrome(chrome_options=chromeOptions)

driver.get(url)  # 加载页面

driver.switch_to.frame("contentFrame")  # 切换到歌单所在的iframe

# 歌单所有项目的包裹元素
data = driver.find_element_by_id('m-pl-container').find_elements_by_tag_name('li')

for d in data:
    nb = d.find_element_by_class_name('nb').text  # 播放数
    msk = d.find_element_by_css_selector("a.msk")
    print(msk.get_attribute('title'), nb, msk.get_attribute('href'))
```