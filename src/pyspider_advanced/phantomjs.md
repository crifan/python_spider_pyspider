# `Phantomjs`

## 如何解决部分页面内部不显示，无法抓取的问题？

折腾[【已解决】PySpider中页面部分内容不显示 – 在路上](https://www.crifan.com/pyspider_page_content_partial_not_show/)，遇到个问题：

页面中的部分内容不显示，所以无法抓取。

经过研究发现，其实是：

这部分不显示的内容，是原网页中通过后续调用js去生成和获取的，所以可以通过：

给`self.crawl`添加

```python
fetch_type='js'
```

会使得内部调用`phantomjs`，模拟js，渲染生成页面内容。

从而，此处`PySpider`，在这种需要显示js加载的页面内容时，可以利用`phantomjs`。

## 用了`phantomjs`后又出错：`FETCH_ERROR HTTP 599 Connection timed out after milliseconds`

后续继续运行，加了`fetch_type='js'`的代码，去爬取页面数据，结果遇到了：

[【未解决】pyspider运行出错：FETCH_ERROR HTTP 599 Connection timed out after milliseconds](http://www.crifan.com/pyspider_run_error_fetch_error_http_599_connection_timed_out_after_milliseconds)

尝试了多种办法，都无法解决此问题。

所以目前的情况是：

如果加了`phantomjs`，结果在大量爬取页面期间，又会导致出错`FETCH_ERROR HTTP 599 Connection timed out after milliseconds`，而暂时找不到解决办法。

## 给`Phantomjs`添加额外参数

之前折腾过：

[【未解决】pyspider中如何给phantomjs传递额外参数 – 在路上](https://www.crifan.com/pyspider_how_pass_extra_param_to_phantomj)

基本上没有实现想要的效果。但是可供参考。

## phantomjs中的proxy是什么意思

对于pyspider来说，phantomjs-proxy参数指的是：

你另外所运行的`phantomjs的实例` = `host:port`

比如：

在一个终端中运行：

`pyspider phantomjs --port 23450 --auto-restart true`

然后去另外一个终端中运行pyspider：

`pyspider -c config.json all`

其中`config.json`包含了：

`"phantomjs-proxy": "127.0.0.1:23450"`

就可以使得此处的pyspider在启动时不另外启动phantomjs了。

而是去在需要用到phantomjs时，去连接本地电脑127.0.0.1的23450端口中的phantomjs去处理，去加载页面了。

而对于phantomjs本身来说：

proxy，指的是代理，比如翻墙的代理，等等。

具体相关设置，可以参考：

[Command Line Interface | PhantomJS](http://phantomjs.org/api/command-line.html)

中的：

* `–proxy=address:port` specifies the proxy server to use (e.g. –proxy=192.168.1.42:8080)
* `–proxy-type=[http|socks5|none]` specifies the type of the proxy server (default is http).
* `–proxy-auth` specifies the authentication information for the proxy, e.g. –proxy-auth=username:password)

详见：[【已解决】pyspider中phantomjs中的proxy是什么意思 – 在路上](https://www.crifan.com/pyspider_phantomjs_proxy_para_meaning)
