---
title: Mysql 5.7 以上版本对JSON字段的操作
date: 2020-05-02 15:10:23
updated: 2020-05-02 15:10:23
tags:
- mysql
categories:
---




[官方文档mysql 5.7 json](https://dev.mysql.com/doc/refman/5.7/en/json.html)
[官方文档mysql 5.7 json functions](https://dev.mysql.com/doc/refman/5.7/en/json-functions.html)

[官方文档mysql 8.0 json](https://dev.mysql.com/doc/refman/8.0/en/json.html)


<!-- more -->

Name | Description
---|---
->|Return value from JSON column after evaluating path; equivalent to JSON_EXTRACT().
->>?(introduced 5.7.13)|Return value from JSON column after evaluating path and unquoting the result; equivalent to JSON_UNQUOTE(JSON_EXTRACT()).
JSON_APPEND()?(deprecated)|Append data to JSON document
JSON_ARRAY()|Create JSON array
JSON_ARRAY_APPEND()|Append data to JSON document
JSON_ARRAY_INSERT()|Insert into JSON array
JSON_CONTAINS()|Whether JSON document contains specific object at path
JSON_CONTAINS_PATH()|Whether JSON document contains any data at path
JSON_DEPTH()|Maximum depth of JSON document
JSON_EXTRACT()|Return data from JSON document
JSON_INSERT()|Insert data into JSON document
JSON_KEYS()|Array of keys from JSON document
JSON_LENGTH()|Number of elements in JSON document
JSON_MERGE()?(deprecated 5.7.22)|"Merge JSON documents| preserving duplicate keys. Deprecated synonym for JSON_MERGE_PRESERVE()"
JSON_MERGE_PATCH()?(introduced 5.7.22)|"Merge JSON documents| replacing values of duplicate keys"
JSON_MERGE_PRESERVE()?(introduced 5.7.22)|"Merge JSON documents| preserving duplicate keys"
JSON_OBJECT()|Create JSON object
JSON_PRETTY()?(introduced 5.7.22)|Print a JSON document in human-readable format
JSON_QUOTE()|Quote JSON document
JSON_REMOVE()|Remove data from JSON document
JSON_REPLACE()|Replace values in JSON document
JSON_SEARCH()|Path to value within JSON document
JSON_SET()|Insert data into JSON document
JSON_STORAGE_SIZE()?(introduced 5.7.22)|Space used for storage of binary representation of a JSON document
JSON_TYPE()|Type of JSON value
JSON_UNQUOTE()|Unquote JSON value
JSON_VALID()|Whether JSON value is valid





JSON_ARRAY  json数组

```sql
mysql> SELECT JSON_ARRAY(1, "abc", NULL, TRUE, CURTIME());
+---------------------------------------------+
| JSON_ARRAY(1, "abc", NULL, TRUE, CURTIME()) |
+---------------------------------------------+
| [1, "abc", null, true, "11:30:24.000000"]   |



-- write table
create table t1(jdoc json);

insert into t1(jdoc) values(json_array('a','b',now()));
```


JSON_OBJECT

```sql
mysql> SELECT JSON_OBJECT('key1', 1, 'key2', 'abc');
+---------------------------------------+
| JSON_OBJECT('key1', 1, 'key2', 'abc') |
+---------------------------------------+
| {"key1": 1, "key2": "abc"}            |
+---------------------------------------+
```


JSON_EXTRACT 查询json列  同 column->path

```sql
mysql> SELECT JSON_EXTRACT('{"id": 14, "name": "Aztalan"}', '$.name');
+---------------------------------------------------------+
| JSON_EXTRACT('{"id": 14, "name": "Aztalan"}', '$.name') |
+---------------------------------------------------------+
| "Aztalan"                                               |
+---------------------------------------------------------+



-- demo
-- 对于某一列是json而言，需要使用内置的函数 json_extract(列名,'$.key') --这个函数有2个参数，第一个参数是json列的列名，第二个参数$.key 其中key为json字符串中某一个key。

> SELECT uid,json_extract(info,'$.mail') AS 'mail',json_extract(info,'$.name') AS 'name' FROM USER;


mysql> SELECT c, JSON_EXTRACT(c, "$.id"), g
     > FROM jemp
     > WHERE JSON_EXTRACT(c, "$.id") > 1
     > ORDER BY JSON_EXTRACT(c, "$.name");
+-------------------------------+-----------+------+
| c                             | c->"$.id" | g    |
+-------------------------------+-----------+------+
| {"id": "3", "name": "Barney"} | "3"       |    3 |
| {"id": "4", "name": "Betty"}  | "4"       |    4 |
| {"id": "2", "name": "Wilma"}  | "2"       |    2 |
+-------------------------------+-----------+------+
3 rows in set (0.00 sec)

mysql> SELECT c, c->"$.id", g
     > FROM jemp
     > WHERE c->"$.id" > 1
     > ORDER BY c->"$.name";
+-------------------------------+-----------+------+
| c                             | c->"$.id" | g    |
+-------------------------------+-----------+------+
| {"id": "3", "name": "Barney"} | "3"       |    3 |
| {"id": "4", "name": "Betty"}  | "4"       |    4 |
| {"id": "2", "name": "Wilma"}  | "2"       |    2 |
+-------------------------------+-----------+------+


```

column->>path

等价于
> JSON_UNQUOTE( JSON_EXTRACT(column, path) )

> JSON_UNQUOTE(column -> path)

> column->>path


```
mysql> SELECT * FROM jemp WHERE g > 2;
+-------------------------------+------+
| c                             | g    |
+-------------------------------+------+
| {"id": "3", "name": "Barney"} |    3 |
| {"id": "4", "name": "Betty"}  |    4 |
+-------------------------------+------+
2 rows in set (0.01 sec)

mysql> SELECT c->'$.name' AS name
    ->     FROM jemp WHERE g > 2;
+----------+
| name     |
+----------+
| "Barney" |
| "Betty"  |
+----------+
2 rows in set (0.00 sec)

mysql> SELECT JSON_UNQUOTE(c->'$.name') AS name
    ->     FROM jemp WHERE g > 2;
+--------+
| name   |
+--------+
| Barney |
| Betty  |
+--------+
2 rows in set (0.00 sec)

mysql> SELECT c->>'$.name' AS name
    ->     FROM jemp WHERE g > 2;
+--------+
| name   |
+--------+
| Barney |
| Betty  |
+--------+
2 rows in set (0.00 sec)
```


JSON_TYPE

```sql
mysql> SELECT JSON_TYPE('["a", "b", 1]');
+----------------------------+
| JSON_TYPE('["a", "b", 1]') |
+----------------------------+
| ARRAY                      |
+----------------------------+

mysql> SELECT JSON_TYPE('"hello"');
+----------------------+
| JSON_TYPE('"hello"') |
+----------------------+
| STRING               |
+----------------------+

-- error
mysql> SELECT JSON_TYPE('hello');
ERROR 3146 (22032): Invalid data type for JSON data in argument 1
to function json_type; a JSON string or JSON type is required.
```



JSON_MERGE

```sql
mysql> SELECT JSON_MERGE('["a", 1]', '{"key": "value"}');
+--------------------------------------------+
| JSON_MERGE('["a", 1]', '{"key": "value"}') |
+--------------------------------------------+
| ["a", 1, {"key": "value"}]                 |
+--------------------------------------------+
```


JSON_SEARCH(json_doc, one_or_all, search_str[, escape_char[, path] ...])

如果json_doc，search_str或path参数中的任何一个为NULL，则返回NULL；否则返回NULL。文档内没有路径；或找不到search_str。
如果json_doc参数不是有效的JSON文档，任何路径参数不是有效的路径表达式，one_or_all不是'one'或'all'或escape_char不是常量表达式，则会发生错误。

要在搜索字符串中指定文字％或_字符，请在其前面加上转义字符。如果escape_char参数丢失或为NULL，则默认值为\。否则，scape_char必须为空或一个字符的常量。
search_str:  % 匹配任意的字符（含0）, _  完全匹配一个字符


如果在准备好的语句中使用了JSON_SEARCH（），并且使用？提供了escape_char参数？参数，参数值在执行时可能是恒定的，但在编译时却不是。

```sql
mysql> SET @j = '["abc", [{"k": "10"}, "def"], {"x":"abc"}, {"y":"bcd"}]';

mysql> SELECT JSON_SEARCH(@j, 'one', 'abc');
+-------------------------------+
| JSON_SEARCH(@j, 'one', 'abc') |
+-------------------------------+
| "$[0]"                        |
+-------------------------------+

mysql> SELECT JSON_SEARCH(@j, 'all', 'abc');
+-------------------------------+
| JSON_SEARCH(@j, 'all', 'abc') |
+-------------------------------+
| ["$[0]", "$[2].x"]            |
+-------------------------------+

mysql> SELECT JSON_SEARCH(@j, 'all', 'ghi');
+-------------------------------+
| JSON_SEARCH(@j, 'all', 'ghi') |
+-------------------------------+
| NULL                          |
+-------------------------------+
```


JSON_CONTAINS(target, candidate[, path])

```sql
mysql> SET @j = '{"a": 1, "b": 2, "c": {"d": 4}}';
mysql> SET @j2 = '1';
mysql> SELECT JSON_CONTAINS(@j, @j2, '$.a');
+-------------------------------+
| JSON_CONTAINS(@j, @j2, '$.a') |
+-------------------------------+
|                             1 |
+-------------------------------+
mysql> SELECT JSON_CONTAINS(@j, @j2, '$.b');
+-------------------------------+
| JSON_CONTAINS(@j, @j2, '$.b') |
+-------------------------------+
|                             0 |
+-------------------------------+

mysql> SET @j2 = '{"d": 4}';
mysql> SELECT JSON_CONTAINS(@j, @j2, '$.a');
+-------------------------------+
| JSON_CONTAINS(@j, @j2, '$.a') |
+-------------------------------+
|                             0 |
+-------------------------------+
mysql> SELECT JSON_CONTAINS(@j, @j2, '$.c');
+-------------------------------+
| JSON_CONTAINS(@j, @j2, '$.c') |
+-------------------------------+
|                             1 |
+-------------------------------+
```

JSON_CONTAINS_PATH(json_doc, one_or_all, path[, path] ...)

one': 1 if at least one path exists within the document, 0 otherwise.
'all': 1 if all paths exist within the document, 0 otherwise.



```sql
mysql> SET @j = '{"a": 1, "b": 2, "c": {"d": 4}}';
mysql> SELECT JSON_CONTAINS_PATH(@j, 'one', '$.a', '$.e');
+---------------------------------------------+
| JSON_CONTAINS_PATH(@j, 'one', '$.a', '$.e') |
+---------------------------------------------+
|                                           1 |
+---------------------------------------------+
mysql> SELECT JSON_CONTAINS_PATH(@j, 'all', '$.a', '$.e');
+---------------------------------------------+
| JSON_CONTAINS_PATH(@j, 'all', '$.a', '$.e') |
+---------------------------------------------+
|                                           0 |
+---------------------------------------------+
mysql> SELECT JSON_CONTAINS_PATH(@j, 'one', '$.c.d');
+----------------------------------------+
| JSON_CONTAINS_PATH(@j, 'one', '$.c.d') |
+----------------------------------------+
|                                      1 |
+----------------------------------------+
mysql> SELECT JSON_CONTAINS_PATH(@j, 'one', '$.a.d');
+----------------------------------------+
| JSON_CONTAINS_PATH(@j, 'one', '$.a.d') |
+----------------------------------------+
|                                      0 |
+----------------------------------------+
```

JSON_SET() replaces existing values and adds nonexisting values.

JSON_INSERT() inserts values without replacing existing values.

JSON_REPLACE() replaces only existing values.



```sql
mysql> SET @j = '{ "a": 1, "b": [2, 3]}';
mysql> SELECT JSON_SET(@j, '$.a', 10, '$.c', '[true, false]');
+-------------------------------------------------+
| JSON_SET(@j, '$.a', 10, '$.c', '[true, false]') |
+-------------------------------------------------+
| {"a": 10, "b": [2, 3], "c": "[true, false]"}    |
+-------------------------------------------------+
mysql> SELECT JSON_INSERT(@j, '$.a', 10, '$.c', '[true, false]');
+----------------------------------------------------+
| JSON_INSERT(@j, '$.a', 10, '$.c', '[true, false]') |
+----------------------------------------------------+
| {"a": 1, "b": [2, 3], "c": "[true, false]"}        |
+----------------------------------------------------+
mysql> SELECT JSON_REPLACE(@j, '$.a', 10, '$.c', '[true, false]');
+-----------------------------------------------------+
| JSON_REPLACE(@j, '$.a', 10, '$.c', '[true, false]') |
+-----------------------------------------------------+
| {"a": 10, "b": [2, 3]}                              |
+-----------------------------------------------------+
```
