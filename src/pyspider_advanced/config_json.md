
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
