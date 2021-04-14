
# 独立的配置文件: `config.json`

如果需要用到复杂一点的配置，比如`result worker`，则是需要单独写配置文件。

PySpider的配置文件一般叫做`config.json`

比如用如下内容：

```json
{
  "taskdb":     "mysql+taskdb://root:crifan_mysql@127.0.0.1:3306/AutohomeTaskdb",
  "projectdb":  "mysql+projectdb://root:crifan_mysql@127.0.0.1:3306/AutohomeProjectdb",
  "resultdb":   "mysql+resultdb://root:crifan_mysql@127.0.0.1:3306/AutohomeResultdb",
  "result_worker":{
      "result_cls": "AutohomeResultWorker.AutohomeResultWorker"
   }
}
```

将`config.json`保存在`pyspider`命令运行所在的当前目录下：

![config.json保存在pyspider当前目录下](../assets/img/config_json_in_pyspider_folder.png)

然后去`-c`指定配置文件：

```bash
pyspider -c config.json
```

## 举例

一个配置更简单的`config.json`

```json
{
    "webui": {
        "port": 5000,
        "need-auth": false
    },
    "scheduler": {
        "delete_time": 30
    }
}
```

运行调用效果：

```bash
 pyspider -c use_python_spider_framework/config.json 
[I 200731 14:48:06 result_worker:49] result_worker starting...
phantomjs fetcher running on port 25555
[I 200731 14:48:06 processor:211] processor starting...
[I 200731 14:48:06 scheduler:647] scheduler starting...
[I 200731 14:48:06 tornado_fetcher:638] fetcher starting...
[I 200731 14:48:06 scheduler:782] scheduler.xmlrpc listening on 127.0.0.1:23333
[I 200731 14:48:06 scheduler:126] project crawlBaiduHotList_PySpider_0731_1436 updated, status:DEBUG, paused:False, 0 tasks
[I 200731 14:48:06 scheduler:965] select crawlBaiduHotList_PySpider_0731_1436:_on_get_info data:,_on_get_info
[I 200731 14:48:06 scheduler:586] in 5m: new:0,success:0,retry:0,failed:0
[I 200731 14:48:06 tornado_fetcher:188] [200] crawlBaiduHotList_PySpider_0731_1436:_on_get_info data:,_on_get_info 0s
[D 200731 14:48:06 project_module:145] project: crawlBaiduHotList_PySpider_0731_1436 updated.
[I 200731 14:48:06 processor:202] process crawlBaiduHotList_PySpider_0731_1436:_on_get_info data:,_on_get_info -> [200] len:12 -> result:None fol:0 msg:0 err:None
[I 200731 14:48:06 scheduler:360] crawlBaiduHotList_PySpider_0731_1436 on_get_info {'min_tick': 0, 'retry_delay': {}, 'crawl_config': {}}
[I 200731 14:48:06 app:76] webui running on 0.0.0.0:5000
```

![pyspider_config_data_path](../assets/img/pyspider_config_data_path.png)
