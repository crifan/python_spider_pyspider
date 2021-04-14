# 项目操作举例

此处就以一个简单的项目为例来说明，从头到尾是如何运行和操作的：

首次运行`pyspider`，会提示是否运行使用网络，点击`允许`

![pyspider_steps_allow_network](../../assets/img/pyspider_steps_allow_network.png)

输出：

```bash
➜  ChildQuPeiYinApp_downloadDemo pyspider
phantomjs fetcher running on port 25555
[I 180925 11:25:51 result_worker:49] result_worker starting...
[I 180925 11:25:51 processor:211] processor starting...
[I 180925 11:25:51 tornado_fetcher:638] fetcher starting...
[I 180925 11:25:52 scheduler:647] scheduler starting...
[I 180925 11:25:52 scheduler:782] scheduler.xmlrpc listening on 127.0.0.1:23333
[I 180925 11:25:52 scheduler:586] in 5m: new:0,success:0,retry:0,failed:0
[I 180925 11:25:52 app:76] webui running on 0.0.0.0:5000
```

打开页面：

http://0.0.0.0:5000

![pyspider_5000_open_page](../../assets/img/pyspider_5000_open_page.png)

去`Create New Project`新建项目

![pyspider_new_project](../../assets/img/pyspider_new_project.png)

进入项目调试界面：

![pyspider_debug_main](../../assets/img/pyspider_debug_main.png)

编写代码，或者已写好代码后去粘贴代码，再点击保存：

![pyspider_code_save](../../assets/img/pyspider_code_save.png)

接着点击`Run`，开始运行。

会出现`Follow`，点击`Follow`

![pyspider_follow_3](../../assets/img/pyspider_follow_3.png)

其中Follow后的`3`，指的是有产生了3条请求链接，可供后续继续访问

![pyspider_show_links](../../assets/img/pyspider_show_links.png)

点击第一个的`三个点`，去展开：

![pyspider_click_three_point_expand](../../assets/img/pyspider_click_three_point_expand.png)

可以看到当前请求的详细参数

点击`右箭头`=`>`

![pyspider_right_to_new](../../assets/img/pyspider_right_to_new.png)

![pyspider_right_to_new_2](../../assets/img/pyspider_right_to_new_2.png)

可以看到输出

然后再点击后续的链接，去运行：

![pyspider_run_more_link](../../assets/img/pyspider_run_more_link.png)

![pyspider_run_more_link_2](../../assets/img/pyspider_run_more_link_2.png)

点击 `左箭头`=`<` 返回上一级：

![pyspider_left_previous_level](../../assets/img/pyspider_left_previous_level.png)

再点击`Run`

![pyspider_up_level_run](../../assets/img/pyspider_up_level_run.png)

回到**上一级**的输出了：

![pyspider_returned_up_level](../../assets/img/pyspider_returned_up_level.png)

如此，即可，根据需要去，反复的：

* 点击某个请求的`Run`，进入下一级
* 点击`<`返回上一级

去调试，直到得到你需要的结果，即可完成。
