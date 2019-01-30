---
title: Elastic Search
date: 2019-01-30 16:36:34
updated: 2019-01-30 16:36:34
tags:
categories:
---

# ES 命令

delete index

> curl -X DELETE 'http://localhost:9200/samples'

list all indexes

> curl -X GET 'http://localhost:9200/_cat/indices?v'

list all docs in index

> curl -X GET 'http://localhost:9200/sample/_search'


<!--more-->

query using URL parameters

> curl -X GET http://localhost:9200/samples/_search?q=school:Harvard

> curl -XGET --header 'Content-Type: application/json' http://localhost:9200/samples/_search -d '{
      "query" : {
        "match" : { "school": "Harvard" }
    }
}'



list index mapping

> curl -X GET http://localhost:9200/samples

> http://10.16.73.100:12801/_mapping/

Add Data

> curl -XPUT --header 'Content-Type: application/json' http://localhost:9200/samples/_doc/1 -d '{
   "school" : "Harvard"			
}'


Update Doc

> curl -XPUT --header 'Content-Type: application/json' http://localhost:9200/samples/_doc/2 -d '
{
    "school": "Clemson"
}'

> curl -XPOST --header 'Content-Type: application/json' http://localhost:9200/samples/_doc/2/_update -d '{
"doc" : {
               "students": 50000}
}'


backup index

> curl -XPOST --header 'Content-Type: application/json' http://localhost:9200/_reindex -d '{
  "source": {
    "index": "samples"
  },
  "dest": {
    "index": "samples_backup"
  }
}'

# ES DSL查询

query DSL ES中索引的数据都会存储一个_score分值，分值越高就代表越匹配

filter DSL 是或者不是,它不会去计算任何分值，也不会关心返回的排序问题，因此效率会高一点。

![查询DSL（query DSL）和过滤DSL（filter DSL](http://images2015.cnblogs.com/blog/120296/201603/120296-20160318164439303-146810557.png)



## 基本查询demo

```json
{
	"query": {
		"bool": {
			"must": [{
				"term": {
					"jz_post-operate_label": "rpo"
				}
			}, {
				"range": {
					"jz_post-id": {
						"lt": 5193035
					}
				}
			}]
		}
	},
	"sort": [{
			"jz_post-id": {
				"order": "desc"
			}
		}]

		,
	"_source": [
		"jz_post-id"
	]
}
```

## [geo 距离查询](http://www.dczou.com/viemall/740.html)

### ES 版本实现

需要将经纬度设置字符串或者数据，对象类型。数据类型设置为geo_point才能查询，底层是[Geohash](https://en.wikipedia.org/wiki/Geohash) 实现

位置过滤：

1. geo_distance 查找距离某个中心点距离在一定范围内的位置
2. geo_bounding_box 查找某个长方形区域内的位置
3. geo_distance_range 查找距离某个中心的距离在min和max之间的位置
4. geo_polygon 查找位于多边形内的地点


```json
{
  "query": {
    "bool": {
      "must_not": [
        {
          "terms": {
            "jz_post-id": [
              5433682
            ]
          }
        }
      ],
      "filter": [
        {
          "term": {
            "jz_post-source": 91
          }
        },
        {
          "term": {
            "jz_post-listing_status": 5
          }
        },
        {
          "nested": {
            "path": "jz_post_address",
            "query": {
              "constant_score": {
                "filter": {
                  "geo_distance": {
                    "distance": "2km",
                    "jz_post_address.location": {
                      "lat": "30.26977011",
                      "lon": "120.15865222"
                    }
                  }
                },
                "boost": 1
              }
            }
          }
        }
      ]
    }
  },
  "sort": [
    {
      "_geo_distance": {
        "jz_post_address.location": {
          "lat": "30.26977011",
          "lon": "120.15865222"
        },
        "unit": "km",
        "order": "asc"
      }
    }
  ]
}

```

### php+mysql实现

1. 计算范围，可以做搜索用户

```php
function GetRange($lat,$lon,$raidus){
  //计算纬度
  $degree = (24901 * 1609) / 360.0;
  $dpmLat = 1 / $degree;
  $radiusLat = $dpmLat * $raidus;
  $minLat = $lat - $radiusLat; //得到最小纬度
  $maxLat = $lat + $radiusLat; //得到最大纬度
  //计算经度
  $mpdLng = $degree * cos($lat * (PI / 180));
  $dpmLng = 1 / $mpdLng;
  $radiusLng = $dpmLng * $raidus;
  $minLng = $lon - $radiusLng; //得到最小经度
  $maxLng = $lon + $radiusLng; //得到最大经度
  //范围
  $range = array(
    'minLat' => $minLat,
    'maxLat' => $maxLat,
    'minLon' => $minLng,
    'maxLon' => $maxLng
  );
  return $range;
}
```
2. 获取范围内的所有数据


```php
$result = GetRange(110.325945,20.031541,5000);
 
$where = " (`jingdu` between ".$result['minLat']." and ".$result['maxLat'].") and ( `weidu` between ".$result['minLon']." and ".$result['maxLon']." ) ";
$query = $db->query("select * from distance_table where $where order BY id DESC ");
while ( $row = $db->fetch_array($query) ) {
    $list[] = $row['all_name'];
}
print_r($list);
```


### PHP计算经纬度坐标之间的距离

```php 
function getDistance($lat1, $lng1, $lat2, $lng2, $len_type = 1, $decimal = 2) {
    $radLat1 = $lat1 * PI / 180.0;
    $radLat2 = $lat2 * PI / 180.0;
    $a = $radLat1 - $radLat2;
    $b = ($lng1 * PI / 180.0) - ($lng2 * PI / 180.0);
    $s = 2 * asin(sqrt(pow(sin($a/2),2) + cos($radLat1) * cos($radLat2) * pow(sin($b/2),2)));
    $s = $s * EARTH_RADIUS;
    $s = round($s * 1000);
    if ($len_type > 1)
    {
    $s /= 1000;
    }
    return round($s, $decimal);
}
```


## 精确匹配查询(match_phrase)
```
{
  "query": {
    "constant_score": {
      "filter": {
        "bool": {
          "must": [
            {
              "term": {
                "jz_post-city_id": 289
              }
            },
            {
              "term": {
                "jz_post-job_date_type": 0
              }
            },
            {
              "term": {
                "jz_post-company_id": 1836671
              }
            },
            {
              "terms": {
                "jz_post-listing_status": [
                  0,
                  5
                ]
              }
            }
          ],
          "should": [
            {
              "match_phrase": {
                "jz_post-title": "诚心招聘业务员"
              }
            }
          ]
        }
      }
    }
  }
}
```

## 短语前缀匹配

它与短语匹配的区别为它能匹配的方位更广，它可以命中到短语+其他内容的内容

```json
{
  "query": {
    "match_phrase_prefix": {
      "FIELD": "PREFIX"
    }
  }
}
```

# ES文档参考

- [es-dsl-cheatsheet](http://moliware.com/es-dsl-cheatsheet/)
- [DSL语法查询文档](https://blog.csdn.net/g1969119894/article/details/80111067#%E6%9F%A5%E8%AF%A2)
- [ES5.6文档](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-filter-context.html)