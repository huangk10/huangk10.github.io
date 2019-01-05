---
layout:     post
title:      BeautifulSoup教程
subtitle:   
date:       2019-01-05
author:     huangk
header-img: 
catalog: true
tags:
    - BeautifulSoup
    - python
---

## BeautifulSoup教程

### 如何使用

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(open('index.html'))
soup = BeautifulSoup("<html>data</html>")
```

### 对象的种类

Beautiful Soup将复杂HTML文档转换成一个复杂的树形结构,每个节点都是Python对象,所有对象可以归纳为4种: `Tag` , `NavigableString` , `BeautifulSoup` , `Comment` .

#### Tag

```python
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')
tag = soup.b
type(tag)
# <class 'bs4.element.Tag'>
```

现在介绍一下tag中最重要的属性: name和attributes

##### Name

每个tag都有自己的名字,通过 `.name` 来获取:

```python
tag.name
# u'b'
```

如果改变了tag的name,那将影响所有通过当前Beautiful Soup对象生成的HTML文档:

```python
tag.name = "blockquote"
tag
# <blockquote class="boldest">Extremely bold</blockquote>
```

##### Attributes

一个tag可能有很多个属性. tag `<b class="boldest">` 有一个 “class” 的属性,值为 “boldest” . tag的属性的操作方法与字典相同:

```python
tag['class']
# u'boldest'
```

也可以直接”点”取属性, 比如: `.attrs` :

```python
tag.attrs
# {u'class': u'boldest'}
```

#### 可以遍历的字符串（NavigableString)

字符串常被包含在tag内.Beautiful Soup用 `NavigableString` 类来包装tag中的字符串:

```python
tag.string
# u'Extremely bold'
type(tag.string)
# <class 'bs4.element.NavigableString'>
```

tag中包含的字符串不能编辑,但是可以被替换成其它的字符串,用 [replace_with()](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#replace-with) 方法:

```python
tag.string.replace_with("No longer bold")
tag
# <blockquote>No longer bold</blockquote>
```

#### BeautifulSoup

`BeautifulSoup` 对象表示的是一个文档的全部内容.大部分时候,可以把它当作 `Tag` 对象

因为 `BeautifulSoup` 对象并不是真正的HTML或XML的tag,所以它没有name和attribute属性.但有时查看它的 `.name` 属性是很方便的,所以 `BeautifulSoup` 对象包含了一个值为 “[document]” 的特殊属性 `.name`

```python
soup.name
# u'[document]'
```

#### 注释及特殊字符串

`Tag` , `NavigableString` , `BeautifulSoup` 几乎覆盖了html和xml中的所有内容,但是还有一些特殊对象.容易让人担心的内容是文档的注释部分:

```python
markup = "<b><!--Hey, buddy. Want to buy a used parser?--></b>"
soup = BeautifulSoup(markup)
comment = soup.b.string
type(comment)
# <class 'bs4.element.Comment'>
```

`Comment` 对象是一个特殊类型的 `NavigableString` 对象:

```python
comment
# u'Hey, buddy. Want to buy a used parser'
```

### 遍历文档树

```python
html_doc = """
<html><head><title>The Dormouse's story</title></head>

<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc)
```

拿这个文档举例怎么从文档的一段内容找到另一段内容

#### 子节点

##### tag的名字

操作文档树最简单的方法就是告诉它你想获取的tag的name.如果想获取 <head> 标签,只要用 `soup.head` :

```python
soup.head
# <head><title>The Dormouse's story</title></head>

soup.title
# <title>The Dormouse's story</title>
```

这是个获取tag的小窍门,可以在文档树的tag中多次调用这个方法.下面的代码可以获取<body>标签中的第一个<b>标签:

```python
soup.body.b
# <b>The Dormouse's story</b>
```

通过点取属性的方式只能获得当前名字的第一个tag:

```python
soup.a
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
```

如果想要得到所有的<a>标签,或是通过名字得到比一个tag更多的内容的时候,就需要用到 Searching the tree 中描述的方法,比如: find_all()

```python
soup.find_all('a')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

##### .contents和.children

ag的 `.contents` 属性可以将tag的子节点以列表的方式输出:

```python
head_tag = soup.head
head_tag
# <head><title>The Dormouse's story</title></head>

head_tag.contents
[<title>The Dormouse's story</title>]

title_tag = head_tag.contents[0]
title_tag
# <title>The Dormouse's story</title>
title_tag.contents
# [u'The Dormouse's story']
```

通过tag的 `.children` 生成器,可以对tag的子节点进行循环:

```python
for child in title_tag.children:
    print(child)
    # The Dormouse's story
```

##### .descendants

但是<title>标签也包含一个子节点:字符串 “The Dormouse’s story”,这种情况下字符串 “The Dormouse’s story”也属于<head>标签的子孙节点. `.descendants` 属性可以对所有tag的子孙节点进行递归循环:

```python
for child in head_tag.descendants:
    print(child)
    # <title>The Dormouse's story</title>
    # The Dormouse's story
```

##### .string, .strings和stripped_strings

如果tag中包含多个字符串可以使用 `.strings` 来循环获取:

```python
for string in soup.strings:
    print(repr(string))
    # u"The Dormouse's story"
    # u'\n\n'
    # u"The Dormouse's story"
    # u'\n\n'
    # u'Once upon a time there were three little sisters; and their names were\n'
    # u'Elsie'
    # u',\n'
    # u'Lacie'
    # u' and\n'
    # u'Tillie'
    # u';\nand they lived at the bottom of a well.'
    # u'\n\n'
    # u'...'
    # u'\n'
```

输出的字符串中可能包含了很多空格或空行,使用 `.stripped_strings` 可以去除多余空白内容:

```python
for string in soup.stripped_strings:
    print(repr(string))
    # u"The Dormouse's story"
    # u"The Dormouse's story"
    # u'Once upon a time there were three little sisters; and their names were'
    # u'Elsie'
    # u','
    # u'Lacie'
    # u'and'
    # u'Tillie'
    # u';\nand they lived at the bottom of a well.'
    # u'...'
```

全部是空格的行会被忽略掉,段首和段末的空白会被删除

#### 父节点

##### .parent

通过 `.parent` 属性来获取某个元素的父节点.在例子“爱丽丝”的文档中,<head>标签是<title>标签的父节点:

```python
title_tag = soup.title
title_tag
# <title>The Dormouse's story</title>
title_tag.parent
# <head><title>The Dormouse's story</title></head>
```

##### .parents

通过元素的 `.parents` 属性可以递归得到元素的所有父辈节点,下面的例子使用了 `.parents` 方法遍历了<a>标签到根节点的所有节点.

```python
link = soup.a
link
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
for parent in link.parents:
    if parent is None:
        print(parent)
    else:
        print(parent.name)
# p
# body
# html
# [document]
# None
```

#### 兄弟节点

看一段简单的例子:

```python
sibling_soup = BeautifulSoup("<a><b>text1</b><c>text2</c></b></a>")
print(sibling_soup.prettify())
# <html>
#  <body>
#   <a>
#    <b>
#     text1
#    </b>
#    <c>
#     text2
#    </c>
#   </a>
#  </body>
# </html>
```

因为<b>标签和<c>标签是同一层:他们是同一个元素的子节点,所以<b>和<c>可以被称为兄弟节点.一段文档以标准格式输出时,兄弟节点有相同的缩进级别.在代码中也可以使用这种关系.

#### 回退和前进

### 搜索文档树

#### 过滤器

##### 字符串

最简单的过滤器是字符串.在搜索方法中传入一个字符串参数,Beautiful Soup会查找与字符串完整匹配的内容,下面的例子用于查找文档中所有的<b>标签:

```python
soup.find_all('b')
# [<b>The Dormouse's story</b>]
```

如果传入字节码参数,Beautiful Soup会当作UTF-8编码,可以传入一段Unicode 编码来避免Beautiful Soup解析编码出错

##### 正则表达式

如果传入正则表达式作为参数,Beautiful Soup会通过正则表达式的 `match()` 来匹配内容.下面例子中找出所有以b开头的标签,这表示<body>和<b>标签都应该被找到:

```python
import re
for tag in soup.find_all(re.compile("^b")):
    print(tag.name)
# body
# b
```

下面代码找出所有名字中包含”t”的标签:

```python
for tag in soup.find_all(re.compile("t")):
    print(tag.name)
# html
# title
```

##### 列表

如果传入列表参数,Beautiful Soup会将与列表中任一元素匹配的内容返回.下面代码找到文档中所有<a>标签和<b>标签:

```python
soup.find_all(["a", "b"])
# [<b>The Dormouse's story</b>,
#  <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

##### True

##### 方法

如果没有合适过滤器,那么还可以定义一个方法,方法只接受一个元素参数 ,如果这个方法返回 `True` 表示当前元素匹配并且被找到,如果不是则反回 `False`

下面方法校验了当前元素,如果包含 `class` 属性却不包含 `id` 属性,那么将返回 `True`:

```python
def has_class_but_no_id(tag):
    return tag.has_attr('class') and not tag.has_attr('id')
```

将这个方法作为参数传入 `find_all()` 方法,将得到所有<p>标签:

```python
soup.find_all(has_class_but_no_id)
# [<p class="title"><b>The Dormouse's story</b></p>,
#  <p class="story">Once upon a time there were...</p>,
#  <p class="story">...</p>]
```

返回结果中只有<p>标签没有<a>标签,因为<a>标签还定义了”id”,没有返回<html>和<head>,因为<html>和<head>中没有定义”class”属性.

#### find_all()

find_all( name, attrs, recursive , text, **kwargs)

`find_all()` 方法搜索当前tag的所有tag子节点,并判断是否符合过滤器的条件.这里有几个例子:

```python
soup.find_all("title")
# [<title>The Dormouse's story</title>]

soup.find_all("p", "title")
# [<p class="title"><b>The Dormouse's story</b></p>]

soup.find_all("a")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.find_all(id="link2")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

import re
soup.find(text=re.compile("sisters"))
# u'Once upon a time there were three little sisters; and their names were\n'
```

##### name参数

`name` 参数可以查找所有名字为 `name` 的tag,字符串对象会被自动忽略掉.

简单的用法如下:

```python
soup.find_all("title")
# [<title>The Dormouse's story</title>]
```

重申: 搜索 `name` 参数的值可以使任一类型的 [过滤器](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#id25) ,字符窜,正则表达式,列表,方法或是 `True` .

##### keyword参数

如果一个指定名字的参数不是搜索内置的参数名,搜索时会把该参数当作指定名字tag的属性来搜索,如果包含一个名字为 `id` 的参数,Beautiful Soup会搜索每个tag的”id”属性.

```python
soup.find_all(id='link2')
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

如果传入 `href` 参数,Beautiful Soup会搜索每个tag的”href”属性:

```python
soup.find_all(href=re.compile("elsie"))
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]
```

搜索指定名字的属性时可以使用的参数值包括 [字符串](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#id27) , [正则表达式](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#id28) , [列表](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#id29), [True](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#true) .

使用多个指定名字的参数可以同时过滤tag的多个属性:

```python
soup.find_all(href=re.compile("elsie"), id='link1')
# [<a class="sister" href="http://example.com/elsie" id="link1">three</a>]
```

#### find()

find( [name](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#id32) , [attrs](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#css) , [recursive](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#recursive) , [text](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#text) , [**kwargs](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#keyword) )

`find_all()` 方法将返回文档中符合条件的所有tag,尽管有时候我们只想得到一个结果.比如文档中只有一个<body>标签,那么使用 `find_all()` 方法来查找<body>标签就不太合适, 使用 `find_all` 方法并设置 `limit=1` 参数不如直接使用 `find()` 方法.下面两行代码是等价的:

```python
soup.find_all('title', limit=1)
# [<title>The Dormouse's story</title>]

soup.find('title')
# <title>The Dormouse's story</title>
```

唯一的区别是 `find_all()` 方法的返回结果是值包含一个元素的列表,而 `find()` 方法直接返回结果.

`find_all()` 方法没有找到目标是返回空列表, `find()` 方法找不到目标时,返回 `None` .

```python
print(soup.find("nosuchtag"))
# None
```

`soup.head.title` 是 [tag的名字](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#id17) 方法的简写.这个简写的原理就是多次调用当前tag的 `find()` 方法:

```python
soup.head.title
# <title>The Dormouse's story</title>

soup.find("head").find("title")
# <title>The Dormouse's story</title>
```