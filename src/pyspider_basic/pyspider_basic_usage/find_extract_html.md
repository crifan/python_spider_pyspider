# PySpider中查找提取元素

PySpider中内置的用于查找和定位html网页中元素的库是：`PyQuery`

`PyQuery`算是一个`css选择器`，模拟`JS`领域的`jQuery`，所以叫做`PyQuery`。

具体细节是，PySpider针对html的响应`response`，默认提供了一个`doc`属性，其内置了`PyQuery`解析后结果，所以你可以用`response.doc("your_css_selector")`去选择你要的html中的内容了。

而具体的your_css_selector的写法，则就变成`PyQuery`的写法了。

## 举例：提取汽车之家车型车系相关数据

下面通过想象的例子来解释，`PyQuery`的常见用法：

比如想要提取：

```html
<ul class="rank-list-ul" 0="">
  <li id="s3170">
...
    <div>
      ...
      <a id="atk_3170" href="//car.autohome.com.cn/pic/series/3170.html#pvareaid=103448">图库</a>
    ...
    </div>
  </li>
...
</ul>
```

中的`href`的值

* 语言描述可以是：`class`是`rank-list-ul`的`ul`元素，下面的`li`，下面的`div`，下面的`a`，且`href`值是包含`/pic/series`的

* 转换成PyQuery的语法是
  * `.rank-list-ul li div a[href*="/pic/series"]`
    * 也可以换成另外的写法：
      * `ul[class='rank-list-ul'] li div a[href*="/pic/series"]`
      * `ul[class='rank-list-ul'] a[href*="/pic/series"]`
        * 如果你确定此规则不会误匹配其他元素，也可以省略中间的节点的查找

* 对应PySpider的代码是：
  * 获取匹配到的第一个元素
    ```python
      firstMatchADoc = response.doc('.rank-list-ul li div a[href*="/pic/series"]')
    ```
  * 获取到所有匹配到的元素
    ```python
      for eachADoc in response.doc('.rank-list-ul li div a[href*="/pic/series"]').items():
        print("eachADoc=%s" % eachADoc)
    ```

以及对于：

```html
<dl id="33" olr="6">
  <dt><a href="//car.autohome.com.cn/price/brand-33.html#pvareaid=2042362"><img width="50" height="50"
        src="//car2.autoimg.cn/cardfs/series/g26/M0B/AE/B3/100x100_f40_autohomecar__wKgHEVs9u5WAV441AAAKdxZGE4U148.png"></a>
    <div><a href="//car.autohome.com.cn/price/brand-33.html#pvareaid=2042362">奥迪</a></div>
  </dt>
  <dd>

    <div class="h3-tit"><a href="//car.autohome.com.cn/price/brand-33-9.html#pvareaid=2042363">一汽-大众奥迪</a></div>
    <ul class="rank-list-ul" 0>

      <li id="s3170">
        <h4><a href="//www.autohome.com.cn/3170/#levelsource=000000000_0&pvareaid=101594">奥迪A3</a></h4>
        <div>指导价：<a class="red" href="//www.autohome.com.cn/3170/price.html#pvareaid=101446">19.32-23.46万</a></div>
        <div><a href="//car.autohome.com.cn/price/series-3170.html#pvareaid=103446">报价</a> <a id="atk_3170"
            href="//car.autohome.com.cn/pic/series/3170.html#pvareaid=103448">图库</a> <a data-value="3170"
            class="js-che168link" href="//www.che168.com/china/series3170/">二手车</a> <a
            href="//club.autohome.com.cn/bbs/forum-c-3170-1.html#pvareaid=103447">论坛</a> <a
            href="//k.autohome.com.cn/3170/#pvareaid=103459">口碑</a></div>

      </li>

      <li id="s692">
        <h4><a href="//www.autohome.com.cn/692/#levelsource=000000000_0&pvareaid=101594">奥迪A4L</a></h4>
        <div>指导价：<a class="red" href="//www.autohome.com.cn/692/price.html#pvareaid=101446">30.58-39.68万</a></div>
        <div><a href="//car.autohome.com.cn/price/series-692.html#pvareaid=103446">报价</a> <a id="atk_692"
            href="//car.autohome.com.cn/pic/series/692.html#pvareaid=103448">图库</a> <a data-value="692"
            class="js-che168link" href="//www.che168.com/china/series692/">二手车</a> <a
            href="//club.autohome.com.cn/bbs/forum-c-692-1.html#pvareaid=103447">论坛</a> <a
            href="//k.autohome.com.cn/692/#pvareaid=103459">口碑</a></div>

      </li>
...
...
...
```

对应的代码是：

想要从

```html
<a href="//car.autohome.com.cn/price/brand-33.html#pvareaid=2042362"><img width="50" height="50" src="//car2.autoimg.cn/cardfs/series/g26/M0B/AE/B3/100x100_f40_autohomecar__wKgHEVs9u5WAV441AAAKdxZGE4U148.png"></a>
```

获取brand的logo的`img`的代码：

```python
brandDoc = response.doc('dl dt')
brandLogoDoc = brandDoc.find('a img')
brandLogoUrl = brandLogoDoc.attr["src"]
```

从：

```html
<div><a href="//car.autohome.com.cn/price/brand-33.html#pvareaid=2042362">奥迪</a></div>
```

中获取brand的name的`a`的代码：

```python
brandNameDoc = brandDoc.find('div a')
brandName = brandNameDoc.text()
```

从：

```html
<div class="h3-tit"><a href="//car.autohome.com.cn/price/brand-33-9.html#pvareaid=2042363">一汽-大众奥迪</a></div>
```

获取merchant的所有的`a`的代码：

```python
merchantDocGenerator = response.doc("dd div[class='h3-tit'] a").items()
merchantDocList = list(merchantDocGenerator)
merchantDocLen = len(merchantDocList)
```

注意：`.items()`返回的是`generator`，想要得到`list`，需要用`list(yourGenerator)`去转换得到

从：

```html
<ul class="rank-list-ul" 0>
...
```

获取`rank-list-ul`的`class`的`dd`下面的`ul`的merchant的代码：

```python
merchantRankDocGenerator = response.doc("dd ul[class='rank-list-ul']")
merchantRankDocList = list(merchantRankDocGenerator)
merchantRankDocListLen = len(merchantRankDocList)
```

以及获取每个元素：

* 属性值：用`attr`
  * 类型是：`dict`
* 字符串值：用`text()`
  * 类型是：`str`

举例：

```python
for curIdx, merchantItem  in enumerate(merchantDocList):
    merchantName = merchantItem.text()
    merchantItemAttr = merchantItem.attr
    merchantUrl = merchantItemAttr["href"]
```

## PyQuery资料

`response.doc`返回后的`PyQuery对象`，之后可以继续用PyQuery去操作

此处列出PyQuery的一些典型的操作函数：

* `PyQuery.filter(selector)`
* `PyQuery.find(selector)`
* `PyQuery.items(selector=None)`
* `PyQuery.siblings(selector=None)`

另外，常见的一些属性来说：

* `PyQuery.text(value=<NoDefault>)`：当前节点的text文本值
* `PyQuery.html(value=<NoDefault>, **kwargs)`：当前节点的html值

详见：

* 官网文档
  * [pyquery – PyQuery complete API — pyquery 1.2.4 documentation](https://pythonhosted.org/pyquery/api.html#pyquery.pyquery.PyQuery.find)
  * [Traversing — pyquery 1.2.4 documentation](https://pythonhosted.org/pyquery/traversing.html)
  * [Attributes — pyquery 1.2.4 documentation](https://pythonhosted.org/pyquery/attributes.html)
  * [CSS — pyquery 1.2.4 documentation](https://pythonhosted.org/pyquery/css.html)
* 独立教程
  * [HTML解析库Python版jQuery：PyQuery](http://book.crifan.com/books/python_html_parse_pyquery/website)
