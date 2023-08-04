- 创建索引

```
PUT /book
```

- 插入数据

```
PUT /book/_doc/1
{
  "name": "Bootstrap开发修改测试",
  "description":"Bootstrap是一个前端页面开发框架",
  "studymodel": "201002",
  "price": "38.6",
  "timestamp": "2019-08-25 19:11:12",
  "pic": "group1/M00/00/00/abc.jpg",
  "tags": ["bootstrap", "dev"]
}

PUT /book/_doc/2
{
  "name": "java编程思想",
  "description":"java编程思想是一本java书籍",
  "studymodel": "201003",
  "price": "99.8",
  "timestamp": "2019-08-30 19:11:12",
  "pic": "group1/M00/00/00/def.jpg",
  "tags": ["java", "dev"]
}
```

- 搜索数据

```
GET /book/_doc/1
```

- 修改数据

```
POST /book/_doc/1/_update
{
  "doc":{
    "description": "修改描述3"
  }
}

POST /book/_update/1
{
  "doc": {
    "studymodel": "12345"
  }
}
```

- 删除数据

```
DELETE /book/_doc/1
```





```bash
PUT /hotel
{
  "mappings": {
    "properties": {
      "id":{
        "type": "keyword"
      },
      "name":{
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "address":{
        "type": "keyword",
        "index": false
      },
      "price":{
        "type": "integer"
      },
      "score":{
        "type": "integer"
      },
      "brand":{
        "type": "keyword",
        "copy_to": "all"
      },
      "city":{
        "type": "keyword",
        "copy_to": "all"
      },
      "starName":{
        "type": "keyword",
        "copy_to": "all"
      },
      "bussiness":{
        "type": "keyword",
        "copy_to": "all"
      },
      "location":{
        "type": "geo_point"
      },
      "pic":{
        "type": "keyword",
        "index": false
      },
      # 通过all字段可以将若干的字段组合到一起，提升查询效率
      "all":{
        "type": "keyword"
      }
    }
  }
}
```

