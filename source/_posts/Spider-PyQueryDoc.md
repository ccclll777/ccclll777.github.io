---
title: Python3中PyQuery的使用
date: 2018-11-18 09:26:18
categories: 
- 爬虫学习
tags:
- 爬虫
- python
- pyquery
---
## 摘要
PyQuery库也是一个非常强大又灵活的网页解析库，如果你有前端开发经验的，都应该接触过jQuery,那么PyQuery就是你非常绝佳的选择，PyQuery 是 Python 仿照 jQuery 的严格实现。语法与 jQuery 几乎完全相同，所以不用再去费心去记一些奇怪的方法了。
<!-- more -->

## 初始化
像Beautiful Soup一样，初始化pyquery的时候，也需要传入HTML文本来初始化一个PyQuery对象。它的初始化方式有多种，比如直接传入字符串，传入URL，传入文件名，等等。下面我们来详细介绍一下。


 - **字符串的初始化**
 

```
html = '''
<div>
    <ul>
         <li class="item-0">first item</li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
         <li class="item-1 active"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a></li>
     </ul>
 </div>
'''
from pyquery import PyQuery as pq
doc = pq(html)
print(doc('li'))
```
和beautifulsoup一样，可以传入一段html代码，声明pq的对象之，将初始化的对象传入CSS选择器。在这个实例中，我们传入li节点，这样就可以选择所有的li节点。
运行结果为：

```
<li class="item-0">first item</li>
<li class="item-1"><a href="link2.html">second item</a></li>
<li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
<li class="item-1 active"><a href="link4.html">fourth item</a></li>
<li class="item-0"><a href="link5.html">fifth item</a></li>
```
提取出了所有的li标签的值。

 - **URL初始化**
初始化的参数不仅可以以字符串的形式传递，还可以传入网页的URL，此时只需要指定参数为url即可：

```
from pyquery import PyQuery as pq
doc = pq(url='https://blog.csdn.net')
print(doc('title'))
```
这样的话，PyQuery对象会首先请求这个URL，然后用得到的HTML内容完成初始化，这其实就相当于用网页的源代码以字符串的形式传递给PyQuery类来初始化。
它与以下的功能是相同的：

```
from pyquery import PyQuery as pq
import requests
doc = pq(requests.get('https://blog.csdn.net').text)
print(doc('title'))
```


 - **文件初始化**
除了传递url外，也可以传入本地的文件进行初始化

```
from pyquery import PyQuery as pq
doc = pq(filename='demo.html')
print(doc('li'))
```
本地HTML文件demo.html，其内容是待解析的HTML字符串。这样它会首先读取本地的文件内容，然后用文件内容以字符串的形式传递给PyQuery类来初始化。


以上3种初始化方式均可，当然最常用的初始化方式还是以字符串形式传递。

## 基本CSS选择器

 - 具体用法

```
html = '''
<div id="container">
    <ul class="list">
         <li class="item-0">first item</li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
         <li class="item-1 active"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a></li>
     </ul>
 </div>
'''
from pyquery import PyQuery as pq
doc = pq(html)
print(doc('#container .list li'))
print(type(doc('#container .list li')))
```

这里我们初始化PyQuery对象之后，传入了一个CSS选择器#container .list li，它的意思是先选取id为container的节点，然后再选取其内部的class为list的节点内部的所有li节点。然后，打印输出。可以看到，我们成功获取到了符合条件的节点。

最后，将它的类型打印输出。可以看到，它的类型依然是PyQuery类型。

 - **查找节点**
 - 子节点
 
查找子节点时，需要用到find()方法，此时传入的参数是CSS选择器。这里还是以前面的HTML为例：

```
from pyquery import PyQuery as pq
doc = pq(html)
items = doc('.list')
print(type(items))
print(items)
lis = items.find('li')
print(type(lis))
print(lis)
```

首先，我们选取class为list的节点，然后调用了find()方法，传入CSS选择器，选取其内部的li节点，最后打印输出。可以发现，find()方法会将符合条件的所有节点选择出来，结果的类型是PyQuery类型。

其实find()的查找范围是节点的所有子孙节点，而如果我们只想查找子节点，那么可以用children()方法：

```
lis = items.children()
print(type(lis))
print(lis)
```

如果要筛选所有子节点中符合条件的节点，比如想筛选出子节点中class为active的节点，可以向children()方法传入CSS选择器.active：

```
lis = items.children('.active')
print(lis)
```
结果将会作出筛选，留下了class为active的节点。

 -**父节点**
我们可以用parent()方法来获取某个节点的父节点，示例如下：

```
html = '''
<div class="wrap">
    <div id="container">
        <ul class="list">
             <li class="item-0">first item</li>
             <li class="item-1"><a href="link2.html">second item</a></li>
             <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
             <li class="item-1 active"><a href="link4.html">fourth item</a></li>
             <li class="item-0"><a href="link5.html">fifth item</a></li>
         </ul>
     </div>
 </div>
'''
from pyquery import PyQuery as pq
doc = pq(html)
items = doc('.list')
container = items.parent()
print(type(container))
print(container)
```
这里我们首先用.list选取class为list的节点，然后调用parent()方法得到其父节点，其类型依然是PyQuery类型。

这里的父节点是该节点的直接父节点，也就是说，它不会再去查找父节点的父节点，即祖先节点。

如果想获取某个祖先节点，需要使用parents()方法：

```
from pyquery import PyQuery as pq
doc = pq(html)
items = doc('.list')
parents = items.parents()
print(type(parents))
print(parents)
```
输出结果有两个：一个是class为wrap的节点，一个是id为container的节点。也就是说，parents()方法会返回所有的祖先节点。

如果想要筛选某个祖先节点的话，可以向parents()方法传入CSS选择器，这样就会返回祖先节点中符合CSS选择器的节点：

```
parent = items.parents('.wrap')
print(parent)
```

 - **兄弟节点**
如果要获取兄弟节点，可以使用siblings()方法。这里还是以上面的HTML代码为例：

```
from pyquery import PyQuery as pq
doc = pq(html)
li = doc('.list .item-0.active')
print(li.siblings())
```
这里首先选择class为list的节点内部class为item-0和active的节点，也就是第三个li节点。那么，很明显，它的兄弟节点有4个，那就是第一、二、四、五个li节点。这里将会输出这几个兄弟节点。

如果要筛选某个兄弟节点，我们依然可以向siblings方法传入CSS选择器，这样就会从所有兄弟节点中挑选出符合条件的节点了：

```
from pyquery import PyQuery as pq
doc = pq(html)
li = doc('.list .item-0.active')
print(li.siblings('.active'))
```
这里我们筛选了class为active的节点，通过刚才的结果可以观察到，class为active的兄弟节点只有第四个li节点，所以结果应该是一个。

## 遍历
刚才可以观察到，pyquery的选择结果可能是多个节点，也可能是单个节点，类型都是PyQuery类型，并没有返回像Beautiful Soup那样的列表。

对于单个节点来说，可以直接打印输出，也可以直接转成字符串：

```
from pyquery import PyQuery as pq
doc = pq(html)
li = doc('.item-0.active')
print(li)
print(str(li))
```
这两种处理方式的输出结果是一样的。


对于多个节点的结果，我们就需要遍历来获取了。例如，这里把每一个li节点进行遍历，需要调用items()方法：

```
from pyquery import PyQuery as pq
doc = pq(html)
lis = doc('li').items()
print(type(lis))
for li in lis:
    print(li, type(li))
```
调用items()方法后，会得到一个生成器，遍历一下，就可以逐个得到li节点对象了，它的类型也是PyQuery类型。每个li节点还可以调用前面所说的方法进行选择，比如继续查询子节点，寻找某个祖先节点等，非常灵活。

## 获取信息

提取到节点之后，我们的最终目的当然是提取节点所包含的信息了。比较重要的信息有两类，一是获取属性，二是获取文本，下面分别进行说明。

 - **获取属性**
提取到某个PyQuery类型的节点后，就可以调用attr()方法来获取属性：

```
html = '''
<div class="wrap">
    <div id="container">
        <ul class="list">
             <li class="item-0">first item</li>
             <li class="item-1"><a href="link2.html">second item</a></li>
             <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
             <li class="item-1 active"><a href="link4.html">fourth item</a></li>
             <li class="item-0"><a href="link5.html">fifth item</a></li>
         </ul>
     </div>
 </div>
'''
from pyquery import PyQuery as pq
doc = pq(html)
a = doc('.item-0.active a')
print(a, type(a))
print(a.attr('href'))
```
这里首先选中class为item-0和active的li节点内的a节点，它的类型是PyQuery类型。

然后调用attr()方法。在这个方法中传入属性的名称，就可以得到这个属性值了。

此外，也可以通过调用attr属性来获取属性，用法如下：

```
print(a.attr.href)
```
两种方法的结果相同。

如果选中的是多个元素，然后调用attr()方法，只会输出一个节点的结果。

```
a = doc('a')
print(a, type(a))
print(a.attr('href'))
print(a.attr.href)
```

这是因为，当返回结果包含多个节点时，调用attr()方法，只会得到第一个节点的属性。
如果入到这种情况，需要对结果进行遍历：

```
from pyquery import PyQuery as pq
doc = pq(html)
a = doc('a')
for item in a.items():
    print(item.attr('href'))
```
这样就能正常返回所有的结果了。

 - **获取文本**
获取节点之后的另一个主要操作就是获取其内部的文本了，此时可以调用text()方法来实现：

```
html = '''
<div class="wrap">
    <div id="container">
        <ul class="list">
             <li class="item-0">first item</li>
             <li class="item-1"><a href="link2.html">second item</a></li>
             <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
             <li class="item-1 active"><a href="link4.html">fourth item</a></li>
             <li class="item-0"><a href="link5.html">fifth item</a></li>
         </ul>
     </div>
 </div>
'''
from pyquery import PyQuery as pq
doc = pq(html)
a = doc('.item-0.active a')
print(a)
print(a.text())
```
这里首先选中一个a节点，然后调用text()方法，就可以获取其内部的文本信息。此时它会忽略掉节点内部包含的所有HTML，只返回纯文字内容。

但如果想要获取这个节点内部的HTML文本，就要用html()方法了：

```
from pyquery import PyQuery as pq
doc = pq(html)
li = doc('.item-0.active')
print(li)
print(li.html())
```
这里我们选中了第三个li节点，然后调用了html()方法，它返回的结果应该是li节点内的所有HTML文本。

这里同样有一个问题，如果我们选中的结果是多个节点，text()或html()会返回什么内容？我们用实例来看一下：

```
html = '''
<div class="wrap">
    <div id="container">
        <ul class="list">
             <li class="item-1"><a href="link2.html">second item</a></li>
             <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
             <li class="item-1 active"><a href="link4.html">fourth item</a></li>
             <li class="item-0"><a href="link5.html">fifth item</a></li>
         </ul>
     </div>
 </div>
'''
from pyquery import PyQuery as pq
doc = pq(html)
li = doc('li')
print(li.html())
print(li.text())
print(type(li.text())
```
运行结果如下：

```
<a href="link2.html">second item</a>
second item third item fourth item fifth item
<class 'str'>
```
结果可能比较出乎意料，html()方法返回的是第一个li节点的内部HTML文本，而text()则返回了所有的li节点内部的纯文本，中间用一个空格分割开，即返回结果是一个字符串。

所以这个地方值得注意，如果得到的结果是多个节点，并且想要获取每个节点的内部HTML文本，则需要遍历每个节点。而text()方法不需要遍历就可以获取，它将所有节点取文本之后合并成一个字符串。

 - **节点操作**
pyquery提供了一系列方法来对节点进行动态修改，比如为某个节点添加一个class，移除某个节点等，这些操作有时候会为提取信息带来极大的便利。
 - addClass和removeClass

```
html = '''
<div class="wrap">
    <div id="container">
        <ul class="list">
             <li class="item-0">first item</li>
             <li class="item-1"><a href="link2.html">second item</a></li>
             <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
             <li class="item-1 active"><a href="link4.html">fourth item</a></li>
             <li class="item-0"><a href="link5.html">fifth item</a></li>
         </ul>
     </div>
 </div>
'''
from pyquery import PyQuery as pq
doc = pq(html)
li = doc('.item-0.active')
print(li)
li.removeClass('active')
print(li)
li.addClass('active')
print(li)
```
首先选中了第三个li节点，然后调用removeClass()方法，将li节点的active这个class移除，后来又调用addClass()方法，将class添加回来。每执行一次操作，就打印输出当前li节点的内容。
运行结果如下：

```
<li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
<li class="item-0"><a href="link3.html"><span class="bold">third item</span></a></li>
<li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
```
可以看到，一共输出了3次。第二次输出时，li节点的active这个class被移除了，第三次class又添加回来了。

所以说，addClass()和removeClass()这些方法可以动态改变节点的class属性。

 - attr、text和html
当然，除了操作class这个属性外，也可以用attr()方法对属性进行操作。此外，还可以用text()和html()方法来改变节点内部的内容。示例如下：

```
html = '''
<ul class="list">
     <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
</ul>
'''
from pyquery import PyQuery as pq
doc = pq(html)
li = doc('.item-0.active')
print(li)
li.attr('name', 'link')
print(li)
li.text('changed item')
print(li)
li.html('<span>changed item</span>')
print(li)
```
这里我们首先选中li节点，然后调用attr()方法来修改属性，其中该方法的第一个参数为属性名，第二个参数为属性值。接着，调用text()和html()方法来改变节点内部的内容。三次操作后，分别打印输出当前的li节点。
运行结果如下：

```
<li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
<li class="item-0 active" name="link"><a href="link3.html"><span class="bold">third item</span></a></li>
<li class="item-0 active" name="link">changed item</li>
<li class="item-0 active" name="link"><span>changed item</span></li>
```
可以发现，调用attr()方法后，li节点多了一个原本不存在的属性name，其值为link。接着调用text()方法，传入文本之后，li节点内部的文本全被改为传入的字符串文本了。最后，调用html()方法传入HTML文本后，li节点内部又变为传入的HTML文本了。

所以说，如果attr()方法只传入第一个参数的属性名，则是获取这个属性值；如果传入第二个参数，可以用来修改属性值。text()和html()方法如果不传参数，则是获取节点内纯文本和HTML文本；如果传入参数，则进行赋值。

 - remove()
remove()方法就是移除，它有时会为信息的提取带来非常大的便利。下面有一段HTML文本：

```
html = '''
<div class="wrap">
    Hello, World
    <p>This is a paragraph.</p>
 </div>
'''
from pyquery import PyQuery as pq
doc = pq(html)
wrap = doc('.wrap')
print(wrap.text())
```
现在想提取Hello, World这个字符串，而不要p节点内部的字符串，需要怎样操作呢？

这里直接先尝试提取class为wrap的节点的内容，看看是不是我们想要的。运行结果如下：

```
Hello, World This is a paragraph.
```
这个结果还包含了内部的p节点的内容，也就是说text()把所有的纯文本全提取出来了。如果我们想去掉p节点内部的文本，可以选择再把p节点内的文本提取一遍，然后从整个结果中移除这个子串，但这个做法明显比较烦琐。

这时remove()方法就可以派上用场了，我们可以接着这么做:

```
wrap.find('p').remove()
print(wrap.text())
```
首先选中p节点，然后调用了remove()方法将其移除，然后这时wrap内部就只剩下Hello, World这句话了，然后再利用text()方法提取即可。

另外，其实还有很多节点操作的方法，比如append()、empty()和prepend()等方法，它们和jQuery的用法完全一致，详细的用法可以参考官方文档：http://pyquery.readthedocs.io/en/latest/api.html。

