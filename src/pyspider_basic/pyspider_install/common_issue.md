# 安装和启动的常见问题

此处整理在安装和启动`pyspider`期间，经常遇到的问题和其原因及解决办法。

## 启动报错：async=True SyntaxError: invalid syntax

* 运行`pyspider`报错：

  ```bash
    File "/Users/limao/.pyenv/versions/3.8.0/Python.framework/Versions/3.8/lib/python3.8/site-packages/pyspider/run.py", line 231
      async=True, get_object=False, no_input=False):
      ^
  SyntaxError: invalid syntax
  ```

* **原因**：PySpider（很久没继续维护了）最新支持版本是`Python 3.6`，其把`async`作为普通函数参数，是没问题的。
  * 但是`Python 3.7`之后把`async`改为了**系统保留字**，表示`异步`，所以`asyc`不能再作为普通函数参数名
    * 所以当前环境是`Python 3.7+`时，就会报语法错误了
* **解决办法**：2种思路：
  * （自己）把代码（中这类的语法错误）都改掉
  * 把当前Python版本换成旧版本：`Python 3.6`
    * 此处选择换旧版本`Python 3.6`

* **步骤**：

Mac中换Python为3.6

此处以`pyenv`为例：

先去安装Python 3.6的某个版本：

```bash
pyenv install 3.6.8
```

再去设置使用`Python 3.6`：

* 本地
  ```bash
  pyenv local 3.6.8
  ```
* 或全局
  ```bash
  pyenv global 3.6.8
  ```

然后重新安装：

```bash
pip install pyspider
```

之后即可正常运行

```bash
pyspider
```

详见：

【已解决】Mac中给Python3安装PySpider

## 推荐用虚拟环境

在Python的开发中，为了避免不同开发环境的互相影响，一般都会用虚拟环境工具，比如`pipenv`、`virtualenv`。

注意：此处即使用Python虚拟环境，由于前面提到的问题，PySpider也还是要用`Python 3.6`的

### pipenv中安装PySpider

此处是`pipenv`，为了指定使用`Python 3.6`则，可以在`Pipfile`中加上：

```python
[requires]
python_version = "3.6"
```

贴出完整配置：

```python
[[source]]
#url = "https://pypi.python.org/simple"
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
verify_ssl = true
name = "pypi"

[packages]
pymysql = "*"

[dev-packages]

[requires]
python_version = "3.6"
```

创建并安装PySpider：

```bash
pipenv install pyspider
```

或：

先创建虚拟环境，再安装PySpider

```bash
pipenv install
pipenv shell
pip install pyspider
```

### virtualenv中安装PySpider

关于virtualenv：

* 创建虚拟环境：`virtualenv venv`
* 激活激活环境并进入：
  * Mac/Linux：`source venv/bin/activate`
    * 或：
      * `. venv/bin/activate`
    * Win：`venv\Scripts\activate.bat`
* 退出虚拟环境：`deactivate`

在`virtualenv`中安装PySpider：

```bash
pip install pyspider
```

## 启动报错：pycurl报ImportError错 或 Curl报ConfigurationError错

`Mac`中，如果安装期间出错：

> \_\_main\_\_.ConfigurationError: Curl is configured to use SSL, but we have not been able to determine which SSL backend it is using

或运行时报错：

```bash
ImportError: pycurl: libcurl link-time ssl backend (openssl) is different from compile-time ssl backend (none/other)
```

* **原因**：此处导入pycurl时，发现libcurl运行时所依赖的ssl的底层是openssl，和当时编译时的版本不匹配
* **解决办法**：重新编译安装，使得版本一致
* **步骤**：

```bash
pip uninstall -y pycurl
export PYCURL_SSL_LIBRARY=openssl
export LDFLAGS=-L/usr/local/opt/openssl/lib;export CPPFLAGS=-I/usr/local/opt/openssl/include;pip install pycurl --compile --no-cache-dir
```

附上，进入虚拟环境后再操作的例子：

```bash
➜  ChildQuPeiYinApp_downloadDemo pipenv shell
Launching subshell in virtual environment…
 . /Users/crifan/.local/share/virtualenvs/ChildQuPeiYinApp_downloadDemo-IRhFV7xw/bin/activate
➜  ChildQuPeiYinApp_downloadDemo  . /Users/crifan/.local/share/virtualenvs/ChildQuPeiYinApp_downloadDemo-IRhFV7xw/bin/activate
➜  ChildQuPeiYinApp_downloadDemo pip uninstall pycurl
Skipping pycurl as it is not installed.
➜  ChildQuPeiYinApp_downloadDemo export PYCURL_SSL_LIBRARY=openssl
➜  ChildQuPeiYinApp_downloadDemo export LDFLAGS=-L/usr/local/opt/openssl/lib;export CPPFLAGS=-I/usr/local/opt/openssl/include;pip install pycurl --compile --no-cache-dir
Collecting pycurl
  Downloading https://files.pythonhosted.org/packages/e8/e4/0dbb8735407189f00b33d84122b9be52c790c7c3b25286826f4e1bdb7bde/pycurl-7.43.0.2.tar.gz (214kB)
    100% |████████████████████████████████| 215kB 198kB/s
Installing collected packages: pycurl
  Running setup.py install for pycurl ... done
Successfully installed pycurl-7.43.0.2
```

注意：上述的：

* `/usr/local/opt/openssl`是你的openssl安装路径
  * 如果你的不是这个路径， 要换成你的实际路径
  * 对应的
    * `/usr/local/opt/openssl/lib`是`lib`库的路径
    * `/usr/local/opt/openssl/include`是`include`头文件的路径

详见：

* [【记录】Mac中安装和运行pyspider](http://www.crifan.com/mac_install_and_run_pyspider)
* 【已解决】pipenv虚拟环境中用pip安装pyspider出错：、\_\_main\_\_.ConfigurationError: Curl is configured to use SSL, but we have not been able to determine which SSL backend it is using
* [【已解决】pyspider运行出错：ImportError pycurl libcurl link-time ssl backend (openssl) is different from compile-time ssl backend (none/other)](http://www.crifan.com/pyspider_run_importerror_pycurl_libcurl_link_time_ssl_backend_openssl_is_different_from_compile_time_ssl_backend_none_other)

### 启动报错：fatal error openssl/ssl.h file not found

如果上面步骤：

`export LDFLAGS=-L/usr/local/opt/openssl/lib;export CPPFLAGS=-I/usr/local/opt/openssl/include;pip install pycurl --compile --no-cache-dir`

报错：

```bash
    In file included from src/docstrings.c:4:
    src/pycurl.h:165:13: fatal error: 'openssl/ssl.h' file not found
    #   include <openssl/ssl.h>
    1 error generated.
    error: command 'gcc' failed with exit status 1
```

* **直接原因**：找不到`openssl/ssl.h`
* **多种可能**
  * 之前没安装过`openssl`
    * **解决办法**：Mac中去安装：
      ```bash
      brew install openssl
      ```
      * 然后再重试即可
  * Mac中已安装过`openssl`
    * 所以此处Mac中是有`openssl/ssl.h`的，只是传入的路径不对
    * **解决办法**：找到已安装的`openssl`的实际路径，传入正确的路径。
    * **步骤**：

找到已安装的openssl的实际安装路径

```bash
brew info openssl
```

可以看到有：

`/usr/local/Cellar/openssl@1.1/1.1.1d (7,983 files, 17.9MB)`

其中的：

`/usr/local/Cellar/openssl@1.1/1.1.1d`

就是我们要的，此处openssl的实际安装路径

通过

```bash
open /usr/local/Cellar/openssl@1.1/1.1.1d
```

确认是有对应的：

`/usr/local/Cellar/openssl@1.1/1.1.1d/include/openssl/ssl.h`

这个文件的。所以传入路径应该改为：

`/usr/local/Cellar/openssl@1.1/1.1.1d/include`

完整命令是：

```bash
export LDFLAGS=-L/usr/local/opt/openssl/lib;export CPPFLAGS=-I/usr/local/Cellar/openssl@1.1/1.1.1d/include;pip install pycurl --compile --no-cache-dir
```

详见：

【已解决】Mac中pip安装pycurl报错：fatal error openssl/ssl.h file not found

### 启动报错：Error Could not create web server listening on port 25555

* **现象**：运行`pyspider`时能看到有错误
  * `Error: Could not create web server listening on port 25555`
* **原因**：之前已启动过`pyspider`，其内部会默认启动`phantomjs`，而虽然之前虽然已关闭掉`pyspider`，但是没有杀掉`phantomjs`的进程，导致端口`25555`被占用，而报错
* **解决办法**：杀掉端口是`25555`的`phantomjs`进程即可
* **步骤**：
  * 找`phantomjs`进程ID：
    * `ps aux | grep 25555`
  * 杀掉对应进程
    * `kill -9 xxx`
* **举例**

```bash
✘ limao@xxx  ~/dev/crifan/python/demo_spider  ps aux | grep 25555
limao            35620   0.0  0.0  4277272    820 s002  R+   10:27上午   0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn 25555
limao            33983   0.0  0.4  6130968  34128 s002  S    10:17上午   0:30.45 phantomjs --ssl-protocol=any --disk-cache=true /Users/limao/.pyenv/versions/3.6.8/lib/python3.6/site-packages/pyspider/fetcher/phantomjs_fetcher.js 25555
 limao@xxx  ~/dev/crifan/python/demo_spider  kill -9 33983
```

之后即可正常启动`pyspider`，且能看到`phantomjs`可以正常启动了：

`phantomjs fetcher running on port 25555`

## 启动报错：Deprecated option domaincontroller use http_authenticator.domain_controller instead

启动报错：

```bash
  File "/Users/limao/.pyenv/versions/3.6.8/lib/python3.6/site-packages/wsgidav/wsgidav_app.py", line 118, in _check_config
    raise ValueError("Invalid configuration:\n  - " + "\n  - ".join(errors))
ValueError: Invalid configuration:
  - Deprecated option 'domaincontroller': use 'http_authenticator.domain_controller' instead.
```

* **原因**：`wsgidav`版本兼容问题
* **解决办法**：换兼容的没问题的旧版本`2.4.1`
* **步骤**：

```bash
pip install wsgidav==2.4.1
```

## 启动报错：ImportError cannot import name DispatcherMiddleware

启动报错：

```bash
  File "/Users/limao/.pyenv/versions/3.6.8/lib/python3.6/site-packages/pyspider/webui/app.py", line 64, in run
    from werkzeug.wsgi import DispatcherMiddleware
ImportError: cannot import name 'DispatcherMiddleware'
```

* **原因**：`werkzeug`版本兼容问题
* **解决办法**：换兼容的没问题的旧版本`0.16.1`
* **步骤**：

```bash
pip install werkzeug==0.16.1
```
