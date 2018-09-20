# self.crawl详解

## 官方文档

此处要介绍的`PySpider`的获取网络请求的函数是：`self.crawl`，功能很强大。

具体详细的解释，可以参考：

* 官网的英文文档：
  * 比如： [crawl参数 params](http://docs.pyspider.org/en/latest/apis/self.crawl/#params)
* 某热心网友整理的 中文文档：
  * [self.crawl - pyspider中文文档 - pyspider中文网](http://www.pyspider.cn/book/pyspider/self.crawl-16.html)

## 给`GET`的请求添加`查询参数`

给`self.crawl`中给`params`传递对应字典变量，`PySpider`内部会自动把字典编码为url的查询参数 query string.

官方实例：

```python
self.crawl('http://httpbin.org/get', callback=self.callback, params={'a': 123, 'b': 'c'})
```

等价于：

```python
self.crawl('http://httpbin.org/get?a=123&b=c', callback=self.callback)
```

自己之前用的例子有：

```python
  topSignTopParam = {
      "start": 0,
      "rows": 20
  }
  self.crawl(TopSignTopUrl,
      callback=self.getMoreUserCallback,
      params=topSignTopParam,
      save={
          "baseUrl": TopSignTopUrl,
          "isNeedCheckNextPage": True,
          "curPageParam": topSignTopParam
      }
  )
```

## 给callback函数加上额外的参数

使用`self.crawl`的`save`参数即可，然后callback中用`response.save`获取传入的值

举例：

```python
    def getUserDetail(self, userId):
        self.crawl(UserDetailUrl,
            callback=self.userDetailCallback,
            params={"member_id": userId },
            save=userId
        )

    def userDetailCallback(self, response):
        userId = response.save
        print("userId=%s" % userId)
```

## 当请求出错时也执行callback回调函数

需要给callback回调函数加上修饰符`@catch_status_code_error`

举例：

```python
    def picSeriesPage(self, response):
      ...
      self.crawl(curSerieDict["url"], callback=self.carModelSpecPage, save=curSerieDict)

    @catch_status_code_error
    def carModelSpecPage(self, response):
        curSerieDict = response.save
        print("curSerieDict=%s", curSerieDict)
        ...
        return curSerieDict
```
