# Python爬虫

## 1、安装环境并设置清华源

**软件安装：**

​	python3.7:   <https://www.python.org/>

​	pycharm破解版:  <http://www.zdfans.com/html/25815.html>

**pip升级并设置清华镜像：**

**临时使用**

```shell
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
```

注意，`simple` 不能少, 是 `https` 而不是 `http`

**设为默认**

升级 pip 到最新的版本 (>=10.0.0) 后进行配置：

```shell
pip install pip -U
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

如果您到 pip 默认源的网络连接较差，临时使用本镜像站来升级 pip：

```shell
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U
```

本人比较喜欢用juputer,在这里推荐一波：

```shell
pip install jupyter
```

启动命令：

```shell
jupyter notebook
```

安装一些常用库：

 matplot、scipy,scrapy,bs4,urllib2,numpy,pandas、pymysql等等

## 2、正则表达式

参考菜鸟教程：<https://www.runoob.com/regexp/regexp-syntax.html>

**特殊字符**

所谓特殊字符，就是一些有特殊含义的字符，如上面说的 runoo*b 中的 *，简单的说就是表示任何字符串的意思。如果要查找字符串中的 * 符号，则需要对 * 进行转义，即在其前加一个 \: runo\*ob 匹配 runo*ob。

许多元字符要求在试图匹配它们时特别对待。若要匹配这些特殊字符，必须首先使字符"转义"，即，将反斜杠字符\ 放在它们前面。下表列出了正则表达式中的特殊字符：

| 特别字符 | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| $        | 匹配输入字符串的结尾位置。如果设置了 RegExp 对象的 Multiline 属性，则 $ 也匹配 '\n' 或 '\r'。要匹配 $ 字符本身，请使用 \$。 |
| ( )      | 标记一个子表达式的开始和结束位置。子表达式可以获取供以后使用。要匹配这些字符，请使用 \( 和 \)。 |
| *        | 匹配前面的子表达式零次或多次。要匹配 * 字符，请使用 \*。     |
| +        | 匹配前面的子表达式一次或多次。要匹配 + 字符，请使用 \+。     |
| .        | 匹配除换行符 \n 之外的任何单字符。要匹配 . ，请使用 \. 。    |
| [        | 标记一个中括号表达式的开始。要匹配 [，请使用 \[。            |
| ?        | 匹配前面的子表达式零次或一次，或指明一个非贪婪限定符。要匹配 ? 字符，请使用 \?。 |
| \        | 将下一个字符标记为或特殊字符、或原义字符、或向后引用、或八进制转义符。例如， 'n' 匹配字符 'n'。'\n' 匹配换行符。序列 '\\' 匹配 "\"，而 '\(' 则匹配 "("。 |
| ^        | 匹配输入字符串的开始位置，除非在方括号表达式中使用，此时它表示不接受该字符集合。要匹配 ^ 字符本身，请使用 \^。 |
| {        | 标记限定符表达式的开始。要匹配 {，请使用 \{。                |
| \|       | 指明两项之间的一个选择。要匹配 \|，请使用 \|。               |

主要有以下限定符

限定符用来指定正则表达式的一个给定组件必须要出现多少次才能满足匹配。有 * 或 + 或 ? 或 {n} 或 {n,} 或 {n,m} 共6种。

正则表达式的限定符有：

| 字符  | 描述                                                         |
| ----- | ------------------------------------------------------------ |
| *     | 匹配前面的子表达式零次或多次。例如，zo* 能匹配 "z" 以及 "zoo"。* 等价于{0,}。 |
| +     | 匹配前面的子表达式一次或多次。例如，'zo+' 能匹配 "zo" 以及 "zoo"，但不能匹配 "z"。+ 等价于 {1,}。 |
| ?     | 匹配前面的子表达式零次或一次。例如，"do(es)?" 可以匹配 "do" 、 "does" 中的 "does" 、 "doxy" 中的 "do" 。? 等价于 {0,1}。 |
| {n}   | n 是一个非负整数。匹配确定的 n 次。例如，'o{2}' 不能匹配 "Bob" 中的 'o'，但是能匹配 "food" 中的两个 o。 |
| {n,}  | n 是一个非负整数。至少匹配n 次。例如，'o{2,}' 不能匹配 "Bob" 中的 'o'，但能匹配 "foooood" 中的所有 o。'o{1,}' 等价于 'o+'。'o{0,}' 则等价于 'o*'。 |
| {n,m} | m 和 n 均为非负整数，其中n <= m。最少匹配 n 次且最多匹配 m 次。例如，"o{1,3}" 将匹配 "fooooood" 中的前三个 o。'o{0,1}' 等价于 'o?'。请注意在逗号和两个数之间不能有空格。 |

由于章节编号在大的输入文档中会很可能超过九，所以您需要一种方式来处理两位或三位章节编号。限定符给您这种能力。下面的正则表达式匹配编号为任何位数的章节标题：

```shell
/Chapter [1-9][0-9]*/
```



在python里，需要引入re库

```python
import re

a = 'xy123'
#测试 .
b = re.findall('x..',a)
print(b)  #['xy12']
#  *
d= 'xyxyxy123'
c = re.findall('x*',d)
print(c)	#输出：['x', '', 'x', '', 'x', '', '', '', '', '']
# ？的使用
e = re.findall('x?',a)
print(e)  #['x', '', '', '', '', '']
#只需要掌握（.*?)
cd = 'dsfgndojfgdixxixxmgreenoxxlovexxfjurigixxyouxxgjfng'
f = re.findall('xx.*xx',cd) #前面是xx结尾，中间可以有无数个字符，只需要后面也是xx结尾就行
print(f)  #输出：['xxixxmgreenoxxlovexxfjurigixxyouxx']  是符合贪心算法的获得最大匹配字符串长度

g = re.findall('xx.*?xx',cd)
print(g)  #['xxixx', 'xxlovexx', 'xxyouxx'] 非贪心算法
#获取xx**xx中间的值需要使用（）
h = re.findall('xx(.*?)xx',cd)
print(h)  # ['i', 'love', 'you']


```

.*与 .星号?差别

表达式 .* 就是单个字符匹配任意次，即贪婪匹配。 表达式 .*? 是满足条件的情况只匹配一次，即最小匹配.



## 3、Requests小爬虫程序

requests安装： 

```python
pip install requests
```

使用requests获取源代码：

1. 直接获取源代码
2. 修改http头获取源代码

```python
import re
import requests
uri = 'http://tieba.baidu.com/f?ie=utf-8&kw=python'
html = requests.get(uri)
print(html.text)
```

获取源代码如下：

```json
<!DOCTYPE html>
<!--STATUS OK-->
<html>
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <link rel="search" type="application/opensearchdescription+xml" href="/tb/cms/content-search.xml" title="百度贴吧" />
    	<meta itemprop="dateUpdate" content="2019-08-29 23:07:05" />

        <meta name="keywords" content="python,程序设计,科学技术,Python,编程">
    <meta name="description" content="本吧热帖: 1-【本吧广告举报专贴】 2-python学习交流qun，从基础到项目实战，全方面系统分享 3-问大家一个问题，方框的语句出现什么问题了，跟着教程一起写，我 4-编程任务完成不了？都 可以来这： 5-别人成功转行Python，你却失败放弃，python开发老人给你的建议 6-从事10年python开发了，现在把整理出来的资源和大家分享">
    <title>python吧-百度贴吧--python学习交流基地。--这里有一群python爱好者，python学习交流基地。Python 是著名的胶水语言,意味着几乎没有 Python 做不了的事情。 </title>
    
```

其中：

```python

```

## 4、xpath

- 安装lxml
- from lxml import etree
- Selector = etree.HTML(网页源代码)
- Selector.xpath(一段神奇的符号)

## 5、scrapy

安装scrapy













