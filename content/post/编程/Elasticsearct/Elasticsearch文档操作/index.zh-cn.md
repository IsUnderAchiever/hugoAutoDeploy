---
title: Elasticsearch基础文档操作
description: Elasticsearch基础文档操作
date: 2023-02-25
slug: Elasticsearch
image: 202412212115512.png
categories:
    - Elasticsearch
---

## Elasticsearch基础
#### 环境配置
![image-20230224221154537](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302242212916.png)

### 创建、查询、删除、修改 索引库
```cmd
# Click the Variables button, above, to create your own variables.
GET ${exampleVariable1} // _search
{
  "query": {
    "${exampleVariable2}": {} // match_all
  }
}
GET _analyze
{
  "analyzer": "pinyin",
  "text": "刘德华"
}
GET _analyze
{
  "analyzer": "ik_smart",
  "text": "我的字典里没有白嫖"
}
GET _analyze
{
  "analyzer": "ik_max_word",
  "text": "我的字典里没有白嫖"
}
# 创建索引库
PUT /user
{
  "mappings": {
    "properties": {
      "id":{
        "type":"keyword",
        "index":false
      },
      "nick_name": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "username": {
        "type": "keyword",
        "index": false
      },
      "password": {
        "type": "keyword",
        "index": false
      }
    }
  }
}
# 查找索引库
GET /user
# 删除索引库
DELETE /user
# 修改索引库
# 事实上，索引库创建后是不允许修改的
# 修改字段时会导致原有的倒排索引失效
# 禁止修改原有的字段，但可以添加新的字段
# 一定要是一个全新的字段名，不能与原有的字段名相同
# 修改索引库的本质是，添加新字段
PUT /user/_mapping
{
  "properties":{
    "新字段名":{
      "type":"integer"
    }
  }
}
```
### 文档的CRUD操作
```cmd
# 插入文档
POST /user/_doc/1
{
  "nick_name":"小美"
}
# 查询文档
GET /user/_doc/1
# 删除文档
DELETE /user/_doc/1
```
#### 修改文档
![image-20230224224312857](https://raw.githubusercontent.com/IsUnderAchiever/markdown-img/master/PicGo01/202302242243003.png)
```cmd
# 全量修改文档
PUT /user/_doc/1
{
  "nick_name":"小美"
}
# 新增
# 由于不存在id为2的文档
PUT /user/_doc/2
{
  "nick_name":"妹妹"
}
# 局部修改字段
POST /user/_update/1
{
  "doc":{
    "nick_name":"小美"
  }
}
```
