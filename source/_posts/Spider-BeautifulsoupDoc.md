---
title: Python3中BeautifulSoup4库的使用
date: 2018-11-12 09:26:18
categories: 
- 爬虫学习
tags:
- 爬虫
- 网页解析
- python
---
## 摘要
Beautiful Soup提供一些简单的、python式的函数用来处理导航、搜索、修改分析树等功能。它是一个工具箱，通过解析文档为用户提供需要抓取的数据，因为简单，所以不需要多少代码就可以写出一个完整的应用程序。


<!-- more -->

Beautiful Soup自动将输入文档转换为Unicode编码，输出文档转换为utf-8编码。你不需要考虑编码方式，除非文档没有指定一个编码方式，这时，Beautiful Soup就不能自动识别编码方式了。然后，你仅仅需要说明一下原始编码方式就可以了。
Beautiful Soup已成为和lxml、html6lib一样出色的python解释器，为用户灵活地提供不同的解析策略或强劲的速度。
## 标签选择器

 - 选择元素

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
print(soup.title)#选择title标签中的内容
print(type(soup.title))#title标签的类型
print(soup.head)#head标签中的内容
print(soup.p)#第一个p标签中的内容

```
返回结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111201211739.png)
soup.标签名可以选择所需要的标签中的内容，返回结果是Tag类型。它查找的是在所有内容中的第一个符合要求的标签，例如上面的<p>的内容。
如果要查询所有的标签，在后面进行介绍。

 - 获取名称

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
print(soup.title.name)
```
返回结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111201308904.png)
这里获取到了标签的名称。

 - 获取属性

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
print (soup.p.attrs['name'])
print(soup.p['name'])
```
返回结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111201553431.png)
在这里，我们把 p 标签的name属性的值打印出来了，name的值为dromouse，上面两种使用方法是等价的。

 - 获取内容

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
print (soup.p.string)
```
返回结果：
![
](https://img-blog.csdnimg.cn/20181111201921613.png)

 
使用标签的.string方法可以轻松获得当前标签中的内容。

 - 嵌套选择

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
print (soup.head.title.string)
```
返回结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111202256959.png)
可以通过嵌套几个不同的标签，层层迭代来获取他的内容，属性，名称。

 - 获取子节点
 
.contents方法：
```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
print(soup.p.contents)
```
返回结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111202820628.png)
.content 属性可以将tag的子节点以列表的方式输出，输出方式为列表，我们可以用列表索引来获取它的某一个元素，如：

```


print soup.head.contents[0]
#返回结果：<title>The Dormouse's story</title>
```


.children方法：

```
print soup.head.children
#返回结果：<listiterator object at 0x7f71457f5710>
```

他返回一个迭代器对象，可以通过遍历获得其中的子节点，如：

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
print(soup.head.children)
for child in  soup.body.children:
    print (child)
```
这样就得到了她的所有子节点：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111203241618.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

 - 获取子孙节点

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')

for child in soup.body.descendants:
    print (child)
```
以上代码可以获取到head标签的所有子孙节点中的内容。
.descendants返回的是一个迭代器的类型，可以通过遍历输出所有的结果。

 - 获取多个内容
.strings  获取多个内容，不过需要遍历获取，比如下面的例子

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')

for string in soup.strings:
    print(repr(string))
```
返回结果为：

```
"The Dormouse's story"
'\n'
'\n'
"The Dormouse's story"
'\n'
'Once upon a time there were three little sisters; and their names were\n'
',\n'
'Lacie'
' and\n'
'Tillie'
';\nand they lived at the bottom of a well.'
'\n'
'...'
'\n'
```
发现将所有标签的内容返回了出来，但是也有许多空的标签，也会进行返回，如果想过滤空行，可以使用.stripped_strings 函数。

.stripped_strings  输出的字符串中可能包含了很多空格或空行,使用 .stripped_strings 可以去除多余空白内容

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')

for string in soup.stripped_strings:
    print(repr(string))
```
返回结果为：

```
"The Dormouse's story"
"The Dormouse's story"
'Once upon a time there were three little sisters; and their names were'
','
'Lacie'
'and'
'Tillie'
';\nand they lived at the bottom of a well.'
'...'

```
这样就避免了空行和空格的出现。

 - 获取父节点
.parent 属性 获取此标签的上一层标签

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
p = soup.p
print (p.parent.name)
```
输出结果为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111232923922.png)
body是p标签的父标签，所以返回为body

 - 获取祖先节点
.parents 属性

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
content = soup.head.title.string
for parent in  content.parents:
    print (parent.name)
```
输出结果为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111233028833.png)
通过元素的 .parents 属性可以递归得到元素的所有父辈节点

 - 获取兄弟节点
 .next_sibling 获取此标签后面的节点
  .previous_sibling  获取此标签前面的节点
 

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
print (soup.p.next_sibling)
#       实际该处为空白
print (soup.p.prev_sibling)
#None   没有前一个兄弟节点，返回 None
print (soup.p.next_sibling.next_sibling)
```
输出结果为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111233324391.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
兄弟节点可以理解为和本节点处在统一级的节点，.next_sibling 属性获取了该节点的下一个兄弟节点，.previous_sibling 则与之相反，如果节点不存在，则返回 None

注意：实际文档中的tag的 .next_sibling 和 .previous_sibling 属性通常是字符串或空白，因为空白或者换行也可以被视作一个节点，所以得到的结果可能是空白或者换行

 - 获取前后节点
 .next_element 获取此标签后一个节点
  .previous_element 获取此标签前一个节点
  

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
print (soup.head.next_element)

```
输出结果为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111233708248.png)

与 .next_sibling  .previous_sibling 不同，它并不是针对于兄弟节点，而是在所有节点，不分层次。
比如 head 节点为：

```
<head><title>The Dormouse's story</title></head>
```
那么它的下一个节点便是 title，它是不分层次关系的

 - 获取所有前后节点
.next_elements  获取后面的所有节点
 .previous_elements获取前面的所有节点 
 

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
for element in soup.body.next_elements:
    print(repr(element))

```
通过 .next_elements 和 .previous_elements 的迭代器就可以向前或向后访问文档的解析内容,就好像文档正在被解析一样

		

 ## 标准选择器
 

 ## find_all( name , attrs , recursive , text , **kwargs )

可以根据标签名，属性，内容查找文档。

 - name 参数
可以传入一个标签名，会返回该标签名下的内容。
```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
print(soup.find_all('a'))
print(type(soup.find_all('p')))
```
返回结果为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181112143634824.png)

也可以通过层层嵌套的方法获取输出：

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
for p in soup.find_all('p'):
    print(p.find_all('a'))
```
这样就获取到了p标签中a标签的内容。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2018111214393753.png)

 - attrs参数

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
print(soup.find_all(attrs={'name':'dromouse'}))
#也可以使用以下方法，输出结果相同
print(soup.find_all(class_='title'}))

```
输出结果为：
![
](https://img-blog.csdnimg.cn/2018111214442739.png)
可以看到它输出了name属性为dormouse的标签。
注意：如果使用print(soup.find_all(class_='title'}))的class属性是，后面需要加一个'_'，因为class属性是一个特殊的属性。

 - text属性

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
print(soup.find_all(text='Lacie'))
```
返回结果：![在这里插入图片描述](https://img-blog.csdnimg.cn/20181112144843161.png)
他返回的是它里面的内容，但是在做元素查找时没有什么用处，但是做元素匹配时会有用处。

 - limit 参数
find_all() 方法返回全部的搜索结构,如果文档树很大那么搜索会很慢.如果我们不需要全部结果,可以使用 limit 参数限制返回结果的数量.效果与SQL中的limit关键字类似,当搜索到的结果数量达到 limit 的限制时,就停止搜索返回结果.

文档树中有3个tag符合搜索条件,但结果只返回了2个,因为我们限制了返回数量。

```
soup.find_all("a", limit=2)
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

## find( name , attrs , recursive , text , **kwargs )
它与 find_all() 方法唯一的区别是 find_all() 方法的返回结果是值包含一个元素的列表,而 find() 方法直接返回结果，也就是说，find方法只会返回一个元素。

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
print(soup.find('p'))

```
返回结果为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181112145159530.png)
只返回了一条结果。

 ## 其他用法一样的标准选择器
 （1）find_parents()  find_parent()

find_all() 和 find() 只搜索当前节点的所有子节点,孙子节点等. find_parents() 和 find_parent() 用来搜索当前节点的父辈节点,搜索方法与普通tag的搜索方法相同,搜索文档搜索文档包含的内容

（2）find_next_siblings()  find_next_sibling()

这2个方法通过 .next_siblings 属性对当 tag 的所有后面解析的兄弟 tag 节点进行迭代, find_next_siblings() 方法返回所有符合条件的后面的兄弟节点,find_next_sibling() 只返回符合条件的后面的第一个tag节点

（3）find_previous_siblings()  find_previous_sibling()

这2个方法通过 .previous_siblings 属性对当前 tag 的前面解析的兄弟 tag 节点进行迭代, find_previous_siblings() 方法返回所有符合条件的前面的兄弟节点, find_previous_sibling() 方法返回第一个符合条件的前面的兄弟节点

（4）find_all_next()  find_next()

这2个方法通过 .next_elements 属性对当前 tag 的之后的 tag 和字符串进行迭代, find_all_next() 方法返回所有符合条件的节点, find_next() 方法返回第一个符合条件的节点

（5）find_all_previous() 和 find_previous()

这2个方法通过 .previous_elements 属性对当前节点前面的 tag 和字符串进行迭代, find_all_previous() 方法返回所有符合条件的节点, find_previous()方法返回第一个符合条件的节点

## CSS选择器

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
print(soup.select('.story .sister'))
print(soup.select('p a'))
print(soup.select('#link2'))
print(soup.select('p')[0])
```
输出结果为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181112145956535.png)
第一个打印的结果为class属性为‘story’的标签中class属性为‘sister’的元素，使用class进行选择时，名称前需要加一个"."。
第二个打印结果为p标签中a标签的内容，与第一个返回结果相同。
第三个打印结果为id为link2标签的内容，在使用id标签进行选择时需要在名字前面加"#"符号。
第四个打印的是第一个p标签的内容。

 - 进行层层迭代的输出

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
for p in soup.select('p'):
    print(p.select('a'))
```
输出结果为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181112150517918.png)
可以输出p标签中a标签的内容

 - 获取属性

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
for p in soup.select('p'):
    print(p['class'])

```
输出结果为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181112150704814.png)
这样就获取到了p标签中class属性的名称。

 - 获取内容

```
from bs4 import BeautifulSoup
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>

"""

soup = BeautifulSoup(html,'lxml')
for p in soup.select('a'):
    print(p.get_text)
```
输出结果为：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181112150827192.png)
获取到了p标签中a标签的所有内容


## 总结
本篇内容比较多，把 Beautiful Soup 的方法进行了大部分整理和总结，不过这还不算完全，仍然有 Beautiful Soup 的修改删除功能，不过这些功能用得比较少，只整理了查找提取的方法，希望对大家有帮助！小伙伴们加油！
