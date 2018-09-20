# PySpider安装

下面介绍，如何安装`PySpider`。

## Mac中安装PySpider

此处以Mac中为例，去介绍如何安装PySpider。

如果只是简单的直接安装的话，则可以去：

```shell
pip install pyspider
```

即可。

不过，Python的开发中，一般避免不同开发环境的影响，推荐使用虚拟环境工具，比如`pipenv`。

此处使用`pipenv`虚拟环境去安装`PySpider`：

安装（虚拟环境同时安装）PySpider：`pipenv install pyspider`

即可。

## 安装期间出现SSL或pycurl错误

如果安装期间出错：

> \_\_main\_\_.ConfigurationError: Curl is configured to use SSL, but we have not been able to determine which SSL backend it is using

或运行期间出错：

> ImportError pycurl libcurl link-time ssl backend (openssl) is different from compile-time ssl backend (none/other)

则解决办法是：

```shell
pip uninstall pycurl
export PYCURL_SSL_LIBRARY=openssl
export LDFLAGS=-L/usr/local/opt/openssl/lib;export CPPFLAGS=-I/usr/local/opt/openssl/include;pip install pycurl --compile --no-cache-dir
```

详见：

* [【记录】Mac中安装和运行pyspider](http://www.crifan.com/mac_install_and_run_pyspider)
* 【已解决】pipenv虚拟环境中用pip安装pyspider出错：、\_\_main\_\_.ConfigurationError: Curl is configured to use SSL, but we have not been able to determine which SSL backend it is using
* [【已解决】pyspider运行出错：ImportError pycurl libcurl link-time ssl backend (openssl) is different from compile-time ssl backend (none/other)](http://www.crifan.com/pyspider_run_importerror_pycurl_libcurl_link_time_ssl_backend_openssl_is_different_from_compile_time_ssl_backend_none_other)

> #### warning:: 上述命令要在对应虚拟环境下运行
> 如果本身你前面是用虚拟环境，比如`pipenv`中去安装的pyspider，则上述命令要先去进入虚拟环境：
> `pipenv shell`
> 之后再去运行，否则是无效的。