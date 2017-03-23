---
title: Mysql MRG_MyISAM引擎分表法
date: 2017-01-25 12:08:15
updated: 2017-01-25 12:08:15
tags:
categories:
---

一般来说，当我们的数据库的数据超过了100w记录的时候就应该考虑分表或者分区了，这次我来详细说说分表的一些方法。目前我所知道的方法都是MYISAM的，INNODB如何做分表并且保留事务和外键，我还不是很了解。

首先，我们需要想好到底分多少个表，前提当然是满足应用。这里我使用了一个比较简单的分表方法，就是根据自增id的尾数来分，也就是说分0-9一共10个表，其取值也很好做，就是对10进行取模。另外，还可以根据某一字段的md5值取其中几位进行分表，这样的话，可以分的表就很多了。

好了，先来创建表吧，代码如下

 
 

```sql
CREATE TABLE `test`.`article_0` (  
`id` BIGINT( 20 ) NOT NULL ,  
`subject` VARCHAR( 200 ) NOT NULL ,  
`content` TEXT NOT NULL ,  
PRIMARY KEY ( `id` )  
) ENGINE = MYISAM CHARACTER SET utf8 COLLATE utf8_general_ci  
  
CREATE TABLE `test`.`article_1` (  
`id` BIGINT( 20 ) NOT NULL ,  
`subject` VARCHAR( 200 ) NOT NULL ,  
`content` TEXT NOT NULL ,  
PRIMARY KEY ( `id` )  
) ENGINE = MYISAM CHARACTER SET utf8 COLLATE utf8_general_ci  
  
CREATE TABLE `test`.`article_2` (  
`id` BIGINT( 20 ) NOT NULL ,  
`subject` VARCHAR( 200 ) NOT NULL ,  
`content` TEXT NOT NULL ,  
PRIMARY KEY ( `id` )  
) ENGINE = MYISAM CHARACTER SET utf8 COLLATE utf8_general_ci  
  
CREATE TABLE `test`.`article_3` (  
`id` BIGINT( 20 ) NOT NULL ,  
`subject` VARCHAR( 200 ) NOT NULL ,  
`content` TEXT NOT NULL ,  
PRIMARY KEY ( `id` )  
) ENGINE = MYISAM CHARACTER SET utf8 COLLATE utf8_general_ci  
  
CREATE TABLE `test`.`article_4` (  
`id` BIGINT( 20 ) NOT NULL ,  
`subject` VARCHAR( 200 ) NOT NULL ,  
`content` TEXT NOT NULL ,  
PRIMARY KEY ( `id` )  
) ENGINE = MYISAM CHARACTER SET utf8 COLLATE utf8_general_ci  
  
CREATE TABLE `test`.`article_5` (  
`id` BIGINT( 20 ) NOT NULL ,  
`subject` VARCHAR( 200 ) NOT NULL ,  
`content` TEXT NOT NULL ,  
PRIMARY KEY ( `id` )  
) ENGINE = MYISAM CHARACTER SET utf8 COLLATE utf8_general_ci  
  
CREATE TABLE `test`.`article_6` (  
`id` BIGINT( 20 ) NOT NULL ,  
`subject` VARCHAR( 200 ) NOT NULL ,  
`content` TEXT NOT NULL ,  
PRIMARY KEY ( `id` )  
) ENGINE = MYISAM CHARACTER SET utf8 COLLATE utf8_general_ci  
  
CREATE TABLE `test`.`article_7` (  
`id` BIGINT( 20 ) NOT NULL ,  
`subject` VARCHAR( 200 ) NOT NULL ,  
`content` TEXT NOT NULL ,  
PRIMARY KEY ( `id` )  
) ENGINE = MYISAM CHARACTER SET utf8 COLLATE utf8_general_ci  
  
CREATE TABLE `test`.`article_8` (  
`id` BIGINT( 20 ) NOT NULL ,  
`subject` VARCHAR( 200 ) NOT NULL ,  
`content` TEXT NOT NULL ,  
PRIMARY KEY ( `id` )  
) ENGINE = MYISAM CHARACTER SET utf8 COLLATE utf8_general_ci  
  
CREATE TABLE `test`.`article_9` (  
`id` BIGINT( 20 ) NOT NULL ,  
`subject` VARCHAR( 200 ) NOT NULL ,  
`content` TEXT NOT NULL ,  
PRIMARY KEY ( `id` )  
) ENGINE = MYISAM CHARACTER SET utf8 COLLATE utf8_general_ci   
```
 

 好了10个表创建完毕了，需要注意的是，这里的id不能设为自增，而且所有的表结构必须一致，包括结构，类型，长度，字段的顺序都必须一致那么对于这个id如何取得呢？后面我会详细说明。现在，我们需要一个合并表，用于查询，创建合并表的代码如下

```sql
CREATE TABLE `test`.`article` (  
`id` BIGINT( 20 ) NOT NULL ,  
`subject` VARCHAR( 200 ) NOT NULL ,  
`content` TEXT NOT NULL ,  
PRIMARY KEY ( `id` )  
) ENGINE=MRG_MyISAM DEFAULT CHARSET=utf8 INSERT_METHOD=0 UNION=(`article_0`,`article_1`,`article_2`,`article_3`,`article_4`,`article_5`,`article_6`,`article_7`,`article_8`,`article_9`); 
```
    这里INSERT_METHOD=0在某些版本可能不工作，需要改成INSERT_METHOD=NO

注意，合并表也必须和前面的表有相同的结构，类型，长度，包括字段的顺序都必须一致这里的`INSERT_METHOD=0`表示不允许对本表进行insert操作。好了，当需要查询的时候，我们可以只对article这个表进行操作就可以了，也就是说这个表仅仅只能进行select操作

那么对于插入也就是insert操作应该如何来搞呢，首先就是获取唯一的id了，这里就还需要一个表来专门创建id，代码如下
```
CREATE TABLE `test`.`create_id` (  
`id` BIGINT( 20 ) NOT NULL AUTO_INCREMENT PRIMARY KEY  
) ENGINE = MYISAM   
  也 就是说，当我们需要插入数据的时候，必须由这个表来产生id值，我的php代码的方法如下

function get_AI_ID() {  
    $sql  = "insert into create_id (id) values('')";  
    $this->db->query($sql);  
    return $this->db->insertID();  
}   
```
  好了，现在假设我们要插入一条数据了，应该怎么操作呢？还是继续看代码吧

```sql
function new_Article() {  
    $id  = $this->get_AI_ID();  
    $table_name = $this->get_Table_Name($id);  
    $sql = "insert into {$table_name} (id,subject,content) values('{$id}','测试标题','测试内容')";  
    $this->db->query($sql);  
}  


/** 
 * 用于根据id获取表名 
 */  
function get_Table_Name($id) {  
    return 'article_'.intval($id)%10;  
}   
```
其实很简单的，对吧，就是先获取id，然后根据id获取应该插入到哪个表，然后就很简单了。

对于update的操作我想应该不需要再说了吧，无非是有了id，然后获取表名，然后进行update操作就好了。