---
layout: information_retrieval
title: ElasticSearch快速搭建
date: 2017-07-31 12:57:41
tags: [ElasticSearch]
---

## 下载工具

- ElasticSearch5.5.0 https://www.elastic.co/downloads/elasticsearch
- Kibana https://www.elastic.co/downloads/kibana

## 配置环境

- MacOS
- Windows

<!-- more -->

## 配置步骤

1. 将ElasticSearch与Kibana解压完成,并且进入各自的bin文件分别启动elasticsearch与kibana.
2. 根据启动kibana终端的信息提示,进入Kibana.
3. 进入Kibana中的Dev Tools中的Console,开始写mapping.这里的analyzer原本应该写standard,但是我们安装了中文分词器.

```json
PUT /legal_document
{
  "mappings": {
    "article":{
      "properties": {
        "title":{
          "type":"text"
          , "analyzer": "ik_smart"
          , "search_analyzer": "ik_smart"
        },
        "content":{
          "type": "text"
          , "analyzer": "ik_smart"
          , "search_analyzer": "ik_smart"
        }
      }
    }
  }
}
```

4.使用python将所有的法律文本写入elasticSearch中,并让它生成检索.

```python
# -*- coding:utf-8 -*-
from elasticsearch import Elasticsearch
import os
# 这里的path对应法律文本的路径
path = "C:/Users/Administrator/Desktop/20170713王宁抓取法律文本"
es = Elasticsearch()
files = os.listdir(path)
number = 1
for file in files:
    if not os.path.isdir(file):
        f = open(path + "/" + file, encoding="gb18030")
        iter_f = iter(f)
        str = ""
        for line in iter_f:
            str = str + line
        es.index(index="legal_document", doc_type="article", body={"title": file, "content": str})
        print('开始导入第')
        print(number)
        print("条\n")
        number += 1
```

5.使用python进行搜索并高亮目标

```python
from elasticsearch import Elasticsearch
es = Elasticsearch()
search_field = input("请输入查询字段名:")
search = es.search(
    index='legal_document',
    doc_type='article',
    body={
        "query": {
            "match": {
                "content": search_field
            }
        },
        "highlight": {
            "fields": {
                "content": {}
            }
        }
    }
)
print(search)
```

## 安装分词器

https://github.com/medcl/elasticsearch-analysis-ik/

```shell
git clone https://github.com/medcl/elasticsearch-analysis-ik
cd elasticsearch-analysis-ik
git checkout tags/{version}
mvn clean
mvn compile
mvn package
```

## 其他问题

**分词器有两种模式,我们需要找到自己最需要的模式进行分词处理.**

1. ik_max_word: 会将文本做最细粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,中华人民,中华,华人,人民共和国,人民,人,民,共和国,共和,和,国国,国歌”，会穷尽各种可能的组合；
2. ik_smart: 会做最粗粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,国歌”。

####  