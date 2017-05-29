# elasticsearch-note
关于搜索引擎elasticsearch的使用笔记

安装[JDK8](http://www.cnblogs.com/duanluan/p/5791726.html)<br>
安装[elasticsearch-rtf](https://github.com/medcl/elasticsearch-rtf)，注意安装路径不能有中文，不然会安装不成功<br>
安装[elasticsearch-head](https://github.com/mobz/elasticsearch-head)<br>
安装`node.js`, `npm`<br>
修改elasticsearch-5.1.1/config路径的elasticsearch.yml文件，最后添加上以下代码<br>
  	
	http.cors.enabled : true 
	http.cors.allow-origin : "*" 
	http.cors.allow-methods : OPTIONS, HEAD, GET, POST, PUT, DELETE 
	http.cors.allow-headers : "X-Requested-With, Content-Type, Content-Length, X-User" 
	
安装[Kibana 5.1.1](https://www.elastic.co/downloads/past-releases/kibana-5-1-1)，注意安装的版本要与elasticsearch一致 <br>

启动elasticsearch:

    cd elasticsearch-5.11/bin
    ./elasticsearch   
启动elasticsearch-head:

    cd elasticsearch-head 
    npm run start
启动kibana:

    cd kibana-5.1.1/bin 
    ./kibana



## es的文档、索引的增删改差增删改查的操作
```sql
#索引初始化操作
#制定指定分片和副本的数量
#sharps一旦设置不能修改
PUT lagou
{
  "settings": {
    "index": {
      "number_of_shards":5,
      "number_of_replicas":1
    }
  }
}

GET lagou/_settings
GET _all/_settings
GET .kibana,lagou/_settings

#修改settings
PUT lagou/_settings
{
  "number_of_replicas": 1
}

#获取索引信息
GET _all
GET lagou

#保存文档
PUT lagou/job/1
{
  "title":"python分布式爬虫开发",
  "salary_min":15000,
  "city":"北京",
  "company":{
    "name":"百度",
    "company_addr":"北京市软件园"
  },
  "publish_date":"2017-4-16",
  "comments":15
}

POST lagou/job/
{
  "title":"python django 开发工程师",
  "salary_min":30000,
  "city":"上海",
  "company":{
    "name":"美团科技",
    "company_addr":"北京市软件园A区"
  },
  "publish_date":"2017-4-16",
  "comments":20
}

GET lagou/job/1
GET lagou/job/1/_source
GET lagou/job/1?_source=title,city

#修改文章
PUT lagou/job/1
{
  "title":"python分布式爬虫开发",
  "salary_min":15000,
  "company":{
    "name":"百度",
    "company_addr":"北京市软件园"
  },
  "publish_date":"2017-4-16",
  "comments":15
}

POST lagou/job/1/_update
{
  "doc":{
    "comments":20
  }
}
GET lagou/job/1?_source=comments

#删除
DELETE lagou/job/1
DELETE lagou

#批量操作
#GET _mget

#bulk批量操作
POST _bulk
{"index":{"_index":"lagou", "_type":"job", "_id":"1"}}
{"title":"python分布式爬虫开发", "salary_min":15000, "city":"北京", "company":{"name":"百度", "company_addr":"北京市软件园"}}
{"index":{"_index":"lagou", "_type":"job", "_id":"2"}}
{"title":"python django开发", "salary_min":30000, "city":"北京", "company":{"name":"美团", "company_addr":"北京市软件园A区"}}  


#创建索引
PUT lagou
{
  "mappings":{
    "job":{
      "properties":{
        "title":{
          "type":"text"
        },
        "salary_min":{
          "type":"integer"
        },
        "city":{
          "type":"keyword"
        },
        "company":{
          "properties":{
            "name":{
              "type":"text"
            },
            "company_addr":{
              "type":"text"
            },
            "employee":{
              "type":"integer"
            }
          }
        },
        "publish_date":{
          "type":"date",
          "format":"yyyy-MM-dd"
        },
        "comments":{
          "type":"integer"
        }
      }
    }
  }
}

PUT lagou/job/1
{
  "title":"python分布式爬虫开发",
  "salary_min":15000,
  "company":{
    "name":"百度",
    "company_addr":"北京市软件园"
  },
  "publish_date":"2017-4-16",
  "comments":15
}

GET lagou/_mapping/job 

#添加映射
PUT lagou
{
  "mappings":{
    "job":{
      "properties":{
        "title":{
          "store":true,
          "type":"text",
          "analyzer":"ik_max_word"
        },
        "company_name":{
          "store":true,
          "type":"keyword"
        },
        "desc":{
          "type":"text"
        },
        "comments":{
          "type":"integer"
        },
        "add_time":{
          "type":"date",
          "format":"yyyy-MM-dd"
        }
      }
    }  
  }
}

#match查询
PUT lagou/job/1
{
  "title":"python工程师",
  "company_name":"去哪儿",
  "desc":"五险一金,包住宿",
  "comments":50,
  "add_time":"2017-05-21"
}

PUT lagou/job/2
{
  "title":"python爬虫工程师",
  "company_name":"艺龙网",
  "desc":"五险一金,包住宿",
  "comments":40,
  "add_time":"2017-05-21"
}

PUT lagou/job/3
{
  "title":"爬虫工程师",
  "company_name":"美团网",
  "desc":"五险一金,包住宿",
  "comments":30,
  "add_time":"2017-05-21"
}

PUT lagou/job/4
{
  "title":"数据分析师",
  "company_name":"去哪儿",
  "desc":"五险一金,包住宿",
  "comments":20,
  "add_time":"2017-05-21"
}

#match查询
GET lagou/job/_search
{
  "query":{
    "match":{
      "title":"python数据"
    }
}
}

#term查询
GET lagou/job/_search
{
  "query":{
    "term":{
      "title":"python"
    }
  }
}

#terms查询
GET lagou/job/_search
{
  "query": {
    "terms":{
      "title":["工程师","python","金融"]
    }
  }
}

#控制查询的返回数量
GET lagou/job/_search
{
  "query":{
    "match":{
      "title":"python"
    }
  },
  "from":0,
  "size":2
}

#match_all 查询
GET lagou/_search
{
  "query":{
    "match_all":{}
  }
}

#match_phrase查询
#短语查询
GET /lagou/_search
{
  "query":{
    "match_phrase": {
      "title": {
        "query": "python工程",
        "slop":2
      }
    }
  }
}

#mutil_match和查询
#比如可以指定多个字段
#比如查询title和desc这两个字段里包含python的关键词文档
GET lagou/_search
{
  "query":{
    "multi_match": {
      "query": "python五先险一金",
      "fields": ["title^3","desc"]
    }
  }
}

#返回指定字段
GET lagou/job/_search
{
  "stored_fields": ["title", "company_name"],
  "query":{
    "match":{
      "title":"python"
    }
  }
}

#通过sort把结果增序排列
GET lagou/job/_search
{
  "query":{
    "match_all":{}
  },
  "sort":[{
    "comments":{
      "order":"asc" 
    }  
  }
  ]
}

#通过sort把结果降序排列
GET lagou/job/_search
{
  "query":{
    "match_all":{}
  },
  "sort":[{
    "comments":{
      "order":"desc" 
    }  
  }
  ]
}

#查询范围
#range查询 评论数大于等于20小于等于30
GET lagou/job/_search
{
  "query":{
    "range":{
      "comments":{
        "gte":20,
        "lte":30,
        "boost":2
      }
    }
  }
}

GET lagou/job/_search
{
  "query":{
    "range":{
      "add_time":{
        "gte":"2017-04-01",
        "lte":"now"
      }
    }
  }
}

#wildcard查询
GET lagou/job/_search
{
  "query":{
    "wildcard": {
      "title": {
        "value": "pyth*n",
        "boost": 2
      }
    }
  }
}


#bool查询
#老版本的filter已经被bool替代
#bool查询包括must should must_not来完成,格式如下:

#{
#  "filter":[],
#  "must":[],
#  "should":[],
#  "must_not":[]
#}

#建立测试数据
POST lagou/testjob/_bulk
{"index":{"_id":1}}
{"salary":10, "title":"Python"}
{"index":{"_id":2}}
{"salary":20, "title":"Scrapy"}
{"index":{"_id":3}}
{"salary":30, "title":"Django"}
{"index":{"_id":4}}
{"salary":30, "title":"Elasticsearch"}

DELETE lagou/testjob
#简单过滤查询


#最简单的filter查询
#selete * from testjob where salary=20
#薪水为20k的工作
GET lagou/testjob/_search
{
  "query":{
    "bool":{
      "must":{
        "match_all":{}
      },
      "filter":{
        "term": {
          "salary":20
        }
      }
    }
  }
}

#也可以指定多个值
GET lagou/testjob/_search
{
  "query":{
    "bool":{
      "must":{
        "match_all":{}
      },
      "filter":{
        "terms": {
          "salary":[10,20]
        }
      }
    }
  }
}

#select * from testjob where title="Python"
GET lagou/testjob/_search
{
  "query":{
    "bool":{
      "must":{
        "match_all":{}
      },
      "filter":{
        "term":{
          "title":"python"
        }
      }
    }
  }
}

#过滤查询,可以组合过滤查询
#select * from testjob where (salary=20 OR title=Python) AND (salary != 30)
#查询薪资等于20k或者工作为python的工作,排除价格为30k的
GET lagou/testjob/_search
{
  "query":{
    "bool":{
      "should": [
        {"term":{"salary":20}},
        {"term":{"title":"python"}}
      ],
      "must_not":{
        "term":{"salary":30}
      }
    }
  }
}

#嵌套查询
#select * from testjob where title="elasticsearch" or (title="django" AND salary=30)
GET lagou/testjob/_search
{
  "query":{
    "bool":{
      "should":[
        {"term":{"title":"elasticsearch"}},
        {"bool":{
          "must":[
          {"term":{"title":"django"}},
          {"term":{"salary":30}}
          ]
          }
        }
      ]
    }
  }
}

#过滤空和非空数据
#建立测试数据
POST lagou/testjob2/_bulk
{"index":{"_id":1}}
{"tags":["search"]}
{"index":{"_id":2}}
{"tags":["search", "python"]}
{"index":{"_id":3}}
{"other_field":["some_data"]}
{"index":{"_id":4}}
{"tags":null}
{"index":{"_id":5}}
{"tags":["search",null]}

#处理null空止值的的方法
#select tags from testjob2 where tags is not NULL
GET lagou/testjob2/_search
{
  "query":{
    "bool":{
      "filter": {
        "exists": {
          "field": "tags"
        }
      }
    }
  }
}

#查看分析器结果
GET _analyze
{
  "analyzer":"ik_max_word",
  "text":"python金融数据分析师"
}
```
