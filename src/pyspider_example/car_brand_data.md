# 汽车之家的品牌等数据

## 文件：`autohomeBrandData.py`

```python
#!/usr/bin/env python
# -*- encoding: utf-8 -*-
# Created on 2018-04-27 21:53:02
# Project: autohomeBrandData

from pyspider.libs.base_handler import *
import string
import re


class Handler(BaseHandler):
    crawl_config = {
    }

    # @every(minutes=24 * 60)
    def on_start(self):
        for eachLetter in list(string.ascii_lowercase):
            self.crawl("https://www.autohome.com.cn/grade/carhtml/%s.html" % eachLetter, callback=self.gradCarHtmlPage)

    @catch_status_code_error
    def gradCarHtmlPage(self, response):
        print("gradCarHtmlPage: response=", response)
        picSeriesItemList = response.doc('.rank-list-ul li div a[href*="/pic/series"]').items()
        print("picSeriesItemList=", picSeriesItemList)
        # print("len(picSeriesItemList)=%s"%(len(picSeriesItemList)))
        for each in picSeriesItemList:
            self.crawl(each.attr.href, callback=self.picSeriesPage)

    @config(priority=2)
    def picSeriesPage(self, response):
        # <a href="/pic/series-t/66.html">查看停产车型&nbsp;&gt;</a>
        # <a class="ckmore" href="/pic/series/588.html">查看在售车型&nbsp;&gt;</a>
        # <span class="fn-right">&nbsp;</span>
        fnRightPicSeries = response.doc('.search-pic-tbar .fn-right a[href*="/pic/series"]')
        print("fnRightPicSeries=", fnRightPicSeries)
        if fnRightPicSeries:
            # hrefValue = fnRightPicSeries.attr.href
            # print("hrefValue=", hrefValue)
            # fullPicSeriesUrl = "https://car.autohome.com.cn" + hrefValue
            fullPicSeriesUrl = fnRightPicSeries.attr.href
            print("fullPicSeriesUrl=", fullPicSeriesUrl)
            self.crawl(fullPicSeriesUrl, callback=self.picSeriesPage)

        # contine parse brand data
        aDictList = []
        # for eachA in response.doc('.breadnav a[href^="/"]').items():
        for eachA in response.doc('.breadnav a[href*="/pic/"]').items():
            eachADict = {
                "text": eachA.text(),
                "href": eachA.attr.href
            }
            print("eachADict=", eachADict)
            aDictList.append(eachADict)

        print("aDictList=", aDictList)

        mainBrandDict = aDictList[-3]
        subBrandDict = aDictList[-2]
        brandSerieDict = aDictList[-1]
        print("mainBrandDict=%s, subBrandDict=%s, brandSerieDict=%s" % (mainBrandDict, subBrandDict, brandSerieDict))

        dtTextList = []
        for eachDt in response.doc("dl.search-pic-cardl dt").items():
            dtTextList.append(eachDt.text())

        print("dtTextList=", dtTextList)

        groupCount = len(dtTextList)
        print("groupCount=", groupCount)

        for eachDt in response.doc("dl.search-pic-cardl dt").items():
            dtTextList.append(eachDt.text())

        ddUlEltList = []
        for eachDdUlElt in response.doc("dl.search-pic-cardl dd ul").items():
            ddUlEltList.append(eachDdUlElt)

        print("ddUlEltList=", ddUlEltList)

        modelDetailDictList = []

        for curIdx in range(groupCount):
            curGroupTitle = dtTextList[curIdx]
            print("------[%d] %s" % (curIdx, curGroupTitle))

            for eachLiAElt in ddUlEltList[curIdx].items("li a"):
                # 1. model name
                # curModelName = eachLiAElt.text()
                curModelName = eachLiAElt.contents()[0]
                curModelName = curModelName.strip()
                print("curModelName=", curModelName)
                curFullModelName = curGroupTitle + " " + curModelName
                print("curFullModelName=", curFullModelName)

                # 2. model id + carSeriesId + spec url
                curModelId = ""
                curSeriesId = ""
                curModelSpecUrl = ""
                modelSpecUrlTemplate = "https://www.autohome.com.cn/spec/%s/#pvareaid=2042128"
                curModelPicUrl = eachLiAElt.attr.href
                print("curModelPicUrl=", curModelPicUrl)
                # https://car.autohome.com.cn/pic/series-s32708/3457.html#pvareaid=2042220
                foundModelSeriesId = re.search("pic/series-s(?P<curModelId>\d+)/(?P<curSeriesId>\d+)\.html",
                                               curModelPicUrl)
                print("foundModelSeriesId=", foundModelSeriesId)
                if foundModelSeriesId:
                    curModelId = foundModelSeriesId.group("curModelId")
                    curSeriesId = foundModelSeriesId.group("curSeriesId")
                    print("curModelId=%s, curSeriesId=%s", curModelId, curSeriesId)
                    curModelSpecUrl = (modelSpecUrlTemplate) % (curModelId)
                    print("curModelSpecUrl=", curModelSpecUrl)

                # 3. model status
                modelStatus = "在售"
                foundStopSale = eachLiAElt.find('i[class*="icon-stopsale"]')
                if foundStopSale:
                    modelStatus = "停售"
                else:
                    foundWseason = eachLiAElt.find('i[class*="icon-wseason"]')
                    if foundWseason:
                        modelStatus = "未上市"

                modelDetailDictList.append({
                    "url": curModelSpecUrl,
                    # "车系ID": curSeriesId,
                    # "车型ID": curModelId,
                    # "车型": curFullModelName,
                    # "状态": modelStatus
                    "brandSerieId": curSeriesId,
                    "modelId": curModelId,
                    "model": curFullModelName,
                    "modelStatus": modelStatus
                })

        print("modelDetailDictList=", modelDetailDictList)

        allSerieDictList = []
        for curIdx, eachModelDetailDict in enumerate(modelDetailDictList):
            """
            defined in mysql

                CREATE TABLE `tbl_autohome_car_info` (
                  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增，主键',
                  `cityDealerPrice` int(11) unsigned NOT NULL DEFAULT '0' COMMENT '经销商参考价',
                  `msrpPrice` int(11) unsigned NOT NULL DEFAULT '0' COMMENT '厂商指导价',
                  `mainBrand` char(20) NOT NULL DEFAULT '' COMMENT '品牌',
                  `subBrand` varchar(20) NOT NULL DEFAULT '' COMMENT '子品牌',
                  `brandSerie` varchar(20) NOT NULL DEFAULT '' COMMENT '车系',
                  `brandSerieId` varchar(15) NOT NULL DEFAULT '' COMMENT '车系ID',
                  `model` varchar(50) NOT NULL DEFAULT '' COMMENT '车型',
                  `modelId` varchar(15) NOT NULL DEFAULT '' COMMENT '车型ID',
                  `modelStatus` char(5) NOT NULL DEFAULT '' COMMENT '车型状态',
                  `url` varchar(200) NOT NULL DEFAULT '' COMMENT '车型url',
                  PRIMARY KEY (`id`)
                ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
            """
            curSerieDict = {
                "url": eachModelDetailDict["url"],
                # "品牌": mainBrandDict["text"],
                # "子品牌": subBrandDict["text"],
                # "车系": brandSerieDict["text"],
                # "车系ID": eachModelDetailDict["车系ID"],
                # "车型": eachModelDetailDict["车型"],
                # "车型ID": eachModelDetailDict["车型ID"],
                # "状态": eachModelDetailDict["状态"]
                "mainBrand": mainBrandDict["text"],
                "subBrand": subBrandDict["text"],
                "brandSerie": brandSerieDict["text"],
                "brandSerieId": eachModelDetailDict["brandSerieId"],
                "model": eachModelDetailDict["model"],
                "modelId": eachModelDetailDict["modelId"],
                "modelStatus": eachModelDetailDict["modelStatus"]
            }
            allSerieDictList.append(curSerieDict)
            # print("before send_message: [%d] curSerieDict=%s" % (curIdx, curSerieDict))
            # self.send_message(self.project_name, curSerieDict, url=eachModelDetailDict["url"])
            print("[%d] curSerieDict=%s" % (curIdx, curSerieDict))
            self.crawl(eachModelDetailDict["url"],
                       callback=self.carModelSpecPage,
                       fetch_type='js',
                       retries=5,
                       connect_timeout=50,
                       timeout=300,
                       save=curSerieDict)

        # print("allSerieDictList=", allSerieDictList)
        # return allSerieDictList

    # def on_message(self, project, msg):
    #    print("on_message: msg=", msg)
    #    return msg

    @catch_status_code_error
    def carModelSpecPage(self, response):
        print("carModelSpecPage: response=", response)
        # https://www.autohome.com.cn/spec/32708/#pvareaid=2042128
        curSerieDict = response.save
        print("curSerieDict", curSerieDict)

        cityDealerPriceInt = 0
        cityDealerPriceElt = response.doc('.cardetail-infor-price #cityDealerPrice span span[class*="price"]')
        print("cityDealerPriceElt=%s" % cityDealerPriceElt)
        if cityDealerPriceElt:
            cityDealerPriceFloatStr = cityDealerPriceElt.text()
            print("cityDealerPriceFloatStr=", cityDealerPriceFloatStr)
            cityDealerPriceFloat = float(cityDealerPriceFloatStr)
            print("cityDealerPriceFloat=", cityDealerPriceFloat)
            cityDealerPriceInt = int(cityDealerPriceFloat * 10000)
            print("cityDealerPriceInt=", cityDealerPriceInt)

        msrpPriceInt = 0
        # body > div.content > div.row > div.column.grid-16 > div.cardetail.fn-clear > div.cardetail-infor > div.cardetail-infor-price.fn-clear > ul > li.li-price.fn-clear > span
        # 厂商指导价=厂商建议零售价格=MSRP=Manufacturer's suggested retail price
        msrpPriceElt = response.doc('.cardetail-infor-price li[class*="li-price"] span[data-price]')
        print("msrpPriceElt=", msrpPriceElt)
        if msrpPriceElt:
            msrpPriceStr = msrpPriceElt.attr("data-price")
            print("msrpPriceStr=", msrpPriceStr)
            foundMsrpPrice = re.search("(?P<msrpPrice>[\d\.]+)万元", msrpPriceStr)
            print("foundMsrpPrice=", foundMsrpPrice)
            if foundMsrpPrice:
                msrpPrice = foundMsrpPrice.group("msrpPrice")
                print("msrpPrice=", msrpPrice)
                msrpPriceFloat = float(msrpPrice)
                print("msrpPriceFloat=", msrpPriceFloat)
                msrpPriceInt = int(msrpPriceFloat * 10000)
                print("msrpPriceInt=", msrpPriceInt)

        # curSerieDict["经销商参考价"] = cityDealerPriceInt
        # curSerieDict["厂商指导价"] = msrpPriceInt
        curSerieDict["cityDealerPrice"] = cityDealerPriceInt
        curSerieDict["msrpPrice"] = msrpPriceInt

        return curSerieDict
```

## 文件：`AutohomeResultWorker.py`

```python
#!/usr/bin/env python
# -*- encoding: utf-8 -*-
# Project: autohomeBrandData
# Function: implement custom result worker for autohome car data
# Author: Crifan Li
# Date: 20180512
# Note:
#   If you want to modify to your mysql and table, you need:
#   (1) change change MysqlDb config to your mysql config
#   (2) change CurrentTableName to your table name
#   (3) change CreateTableSqlTemplate to your sql to create new mysql table fields
#   (4) before use this ResultWorker, run py file to execute testMysqlDb, to init db and create table
#   (5) if your table field contain more type, edit insert to add more type for "TODO: add more type formatting if necessary"


import pymysql
import pymysql.cursors
from pyspider.result import ResultWorker

CurrentTableName = "tbl_autohome_car_info"
CreateTableSqlTemplate = """CREATE TABLE IF NOT EXISTS `%s` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增，主键',
  `cityDealerPrice` int(11) unsigned NOT NULL DEFAULT '0' COMMENT '经销商参考价',
  `msrpPrice` int(11) unsigned NOT NULL DEFAULT '0' COMMENT '厂商指导价',
  `mainBrand` char(20) NOT NULL DEFAULT '' COMMENT '品牌',
  `subBrand` varchar(20) NOT NULL DEFAULT '' COMMENT '子品牌',
  `brandSerie` varchar(20) NOT NULL DEFAULT '' COMMENT '车系',
  `brandSerieId` varchar(15) NOT NULL DEFAULT '' COMMENT '车系ID',
  `model` varchar(50) NOT NULL DEFAULT '' COMMENT '车型',
  `modelId` varchar(15) NOT NULL DEFAULT '' COMMENT '车型ID',
  `modelStatus` char(5) NOT NULL DEFAULT '' COMMENT '车型状态',
  `url` varchar(200) NOT NULL DEFAULT '' COMMENT '车型url',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;"""

class AutohomeResultWorker(ResultWorker):

    def __init__(self, resultdb, inqueue):
        """init mysql db"""
        print("AutohomeResultWorker init: resultdb=%s, inqueue=%s" % (resultdb, inqueue))
        ResultWorker.__init__(self, resultdb, inqueue)

        self.mysqlDb = MysqlDb()
        print("self.mysqlDb=%s" % self.mysqlDb)

    def on_result(self, task, result):
        """override pyspider on_result to save data into mysql"""
        # assert task['taskid']
        # assert task['project']
        # assert task['url']
        # assert result
        print("AutohomeResultWorker on_result: task=%s, result=%s" % (task, result))
        insertOk = self.mysqlDb.insert(result)
        print("insertOk=%s" % insertOk)

class MysqlDb:
    config = {
        'host': '127.0.0.1',
        'port': 3306,
        'user': 'root',
        'password': 'crifan_mysql',
        'database': 'AutohomeResultdb',
        'charset': "utf8"
    }

    defaultTableName = CurrentTableName
    connection = None

    def __init__(self):
        """init mysql"""
        # 1. connect db first
        if self.connection is None:
            isConnected = self.connect()
            print("Connect mysql return %s" % isConnected)

        # 2. create table for db
        createTableOk = self.createTable(self.defaultTableName)
        print("Create table %s return %s" %(self.defaultTableName, createTableOk))

    def connect(self):
        try:
            self.connection = pymysql.connect(**self.config, cursorclass=pymysql.cursors.DictCursor)
            print("connect mysql ok, self.connection=", self.connection)
            return True
        except pymysql.Error as err:
            print("Connect mysql with config=", self.config, " error=", err)
            return False

    def quoteIdentifier(self, identifier):
        """
            for mysql, it better to quote identifier xxx using backticks to `xxx`
            in case, identifier:
                contain special char, such as space
                or same with system reserved words, like select
        """
        quotedIdentifier = "`%s`" % identifier
        # print("quotedIdentifier=", quotedIdentifier)
        return quotedIdentifier

    def executeSql(self, sqlStr, actionDescription=""):
        print("executeSql: sqlStr=%s, actionDescription=%s" % (sqlStr, actionDescription))

        if self.connection is None:
            print("Please connect mysql first before %s" % actionDescription)
            return False

        cursor = self.connection.cursor()
        print("cursor=", cursor)

        try:
            cursor.execute(sqlStr)
            self.connection.commit()
            print("+++ Ok to execute sql %s for %s" % (sqlStr, actionDescription))
            return True
        except pymysql.Error as err:
            print("!!! %s when execute sql %s for %s" % (err, sqlStr, actionDescription))
            return False

    def createTable(self, newTablename):
        print("createTable: newTablename=", newTablename)

        createTableSql = CreateTableSqlTemplate % (newTablename)
        print("createTableSql=", createTableSql)

        return self.executeSql(sqlStr=createTableSql, actionDescription=("Create table %s" % newTablename))

    def dropTable(self, existedTablename):
        print("dropTable: existedTablename=", existedTablename)

        dropTableSql = "DROP TABLE IF EXISTS %s" % (existedTablename)
        print("dropTableSql=", dropTableSql)

        return self.executeSql(sqlStr=dropTableSql, actionDescription=("Drop table %s" % existedTablename))

    # def insert(self, **valueDict):
    def insert(self, valueDict, tablename=defaultTableName):
        """
            inset dict value into mysql table
            makesure the value is dict, and its keys is the key in the table
        """
        print("insert: valueDict=%s, tablename=%s" % (valueDict, tablename))

        dictKeyList = valueDict.keys()
        dictValueList = valueDict.values()
        print("dictKeyList=", dictKeyList, "dictValueList=", dictValueList)

        keyListSql = ", ".join(self.quoteIdentifier(eachKey) for eachKey in dictKeyList)
        print("keyListSql=", keyListSql)
        # valueListSql = ", ".join(eachValue for eachValue in dictValueList)
        valueListSql = ""
        formattedDictValueList = []
        for eachValue in dictValueList:
            # print("eachValue=", eachValue)
            eachValueInSql = ""
            valueType = type(eachValue)
            # print("valueType=", valueType)
            if valueType is str:
                eachValueInSql = '"%s"' % eachValue
            elif valueType is int:
                eachValueInSql = '%d' % eachValue
            # TODO: add more type formatting if necessary
            print("eachValueInSql=", eachValueInSql)
            formattedDictValueList.append(eachValueInSql)

        valueListSql = ", ".join(eachValue for eachValue in formattedDictValueList)
        print("valueListSql=", valueListSql)

        insertSql = """INSERT INTO %s (%s) VALUES (%s)""" % (tablename, keyListSql, valueListSql)
        print("insertSql=", insertSql)
        # INSERT INTO tbl_car_info_test (`url`, `mainBrand`, `subBrand`, `brandSerie`, `brandSerieId`, `model`, `modelId`, `modelStatus`, `cityDealerPrice`, `msrpPrice`) VALUES ("https://www.autohome.com.cn/spec/5872/#pvareaid=2042128", "宝马", "华晨宝马", "宝马3系", "66", "2010款 320i 豪华型", "5872", "停售", 325000, 375000)

        return self.executeSql(sqlStr=insertSql, actionDescription=("Insert value to table %s" % tablename))

    def delete(self, modelId, tablename=defaultTableName):
        """
            delete item from car model id for existing table of autohome car info
        """
        print("delete: modelId=%s, tablename=%s" % (modelId, tablename))

        deleteSql = """DELETE FROM %s WHERE modelId = %s""" % (tablename, modelId)
        print("deleteSql=", deleteSql)

        return self.executeSql(sqlStr=deleteSql, actionDescription=("Delete value from table %s by model id %s" % (tablename, modelId)))

def testMysqlDb():
    """test mysql"""

    testDropTable = True
    testCreateTable = True
    testInsertValue = True
    testDeleteValue = True

    # 1.test connect mysql
    mysqlObj = MysqlDb()
    print("mysqlObj=", mysqlObj)

    # testTablename = "autohome_car_info"
    # testTablename = "tbl_car_info_test"
    testTablename = CurrentTableName
    print("testTablename=", testTablename)

    if testDropTable:
        # 2. test drop table
        dropTableOk = mysqlObj.dropTable(testTablename)
        print("dropTable", testTablename, "return", dropTableOk)

    if testCreateTable:
        # 3. test create table
        createTableOk = mysqlObj.createTable(testTablename)
        print("createTable", testTablename, "return", createTableOk)

    if testInsertValue:
        # 4. test insert value dict
        valueDict = {
            "url": "https://www.autohome.com.cn/spec/5872/#pvareaid=2042128", #车型url
            "mainBrand": "宝马", #品牌
            "subBrand": "华晨宝马", #子品牌
            "brandSerie": "宝马3系", #车系
            "brandSerieId": "66", #车系ID
            "model": "2010款 320i 豪华型", #车型
            "modelId": "5872", #车型ID
            "modelStatus": "停售", #车型状态
            "cityDealerPrice": 325000, #经销商参考价
            "msrpPrice": 375000 # 厂商指导价
        }
        print("valueDict=", valueDict)
        insertOk = mysqlObj.insert(valueDict=valueDict, tablename=testTablename)
        print("insertOk=", insertOk)

    if testDeleteValue:
        toDeleteModelId = "5872"
        deleteOk = mysqlObj.delete(modelId=toDeleteModelId, tablename=testTablename)
        print("deleteOk=", deleteOk)

def testAutohomeResultWorker():
    """just test for create mysql db is ok or not"""
    autohomeResultWorker = AutohomeResultWorker(None, None)
    print("autohomeResultWorker=%s" % autohomeResultWorker)

if __name__ == '__main__':
    testMysqlDb()
    # testAutohomeResultWorker()
```

### 配置文件：`config.json`

```json
{
  "resultdb":   "mysql+resultdb://root:crifan_mysql@127.0.0.1:3306/AutohomeResultdb",
  "result_worker":{
      "result_cls": "AutohomeResultWorker.AutohomeResultWorker"
   },
  "phantomjs-proxy": "127.0.0.1:23450",
  "phantomjs" : {
    "port": 23450,
    "auto-restart": true,
    "load-images": false,
    "debug": true
  }
}
```