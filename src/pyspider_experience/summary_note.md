# PySpider的心得

## 对于加载更多内容，除了想办法找js或api，也可以换个其他的思路

**问题**：想要获取单个页面的更多的内容，一般页面都是向下滚动，加载更多。内部往往是js实现，调用额外的api获取更多数据，加载更多数据。

**思路**：所以一般往往会去研究和抓包，搞清楚调用的api。但是其实有思路多去看看网页中与之相关的其他内容，往往可以通过其他途径，比如另外有个单独的页面，可以获取我所需要的所有的车型车系的数据。就可以避免非要去研究和抓包api了。

详见：[【已解决】pyspider中如何加载汽车之家页面中的更多内容](http://www.crifan.com/pyspider_how_load_more_content_data_from_current_page)

## 调试界面中的`enable css selector helper`

点击web后可以看到html页面内容

再点击`enable css selector helper`后

之后点击某个页面元素，则可以直接显示出对应的css的selector

![PySpider中的css选择器帮助](../assets/img/pyspider_css_selector_help.png)

不过话说我个人调试页面期间，很少用到。

都是直接去Chrome浏览器中调试页面，查看html源码，寻找合适的css selector。