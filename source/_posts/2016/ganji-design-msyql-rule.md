---
title: 【转】赶集Mysql军规
date: 2016-01-28 13:52:43
updated: 2016-01-28 13:52:43
tags:
categories:
---

写在前面:

++ 总是在灾难发生后，才想起容灾的重要性；总是在吃过亏后，才记得曾经有人提醒过。++

# 核心军规
1. 不在数据库做运算：cpu计算务必移至业务层
2. 控制单表数据量：单表记录控制在1000w
3. 控制列数量：字段数控制在20以内
4. 平衡范式与冗余：为提高效率牺牲范式设计，冗余数据
5. 拒绝3B：拒绝大sql，大事物，大批量

<!-- more -->

# 字段类军规

7. 用好数值类型
- tinyint(1Byte)
- smallint(2Byte)
- mediumint(3Byte)
- int(4Byte)
- bigint(8Byte)
- bad case：int(1)/int(11)

8. 字符转化为数字
- 用int而不是char(15)存储ip
- 优先使用enum或set,例如：`sex` enum (‘F’, ‘M’)
- 避免使用NULL字段；NULL字段很难查询优化；NULL字段的索引需要额外空间；NULL字段的复合索引无效
```sql
bad case：
`name` char(32) default null
`age` int not null
good case：
`age` int not null default 0
```
9. 少用text/blob
varchar的性能会比text高很多；实在避免不了blob，请拆表;不在数据库里存图片：是否需要解释？

# 索引类军规
12. 谨慎合理使用索引；改善查询、减慢更新；索引一定不是越多越好（能不加就不加，要加的一定得加）；覆盖记录条数过多不适合建索引，例如“性别”
13. 字符字段必须建前缀索引
14. 不在索引做列运算：`select id where age +1 = 10;`
15. innodb主键推荐使用自增列（SK：博主不认可）;主键建立聚簇索引;主键不应该被修改；字符串不应该做主键；如果不指定主键，innodb会使用唯一且非空值索引代替
16. 不用外键；请由程序保证约束

# sql类军规
17. sql语句尽可能简单；一条sql只能在一个cpu运算；大语句拆小语句，减少锁时间；一条大sql可以堵死整个库
18. 简单的事务,事务时间尽可能短。不好的例子：上传图片事务
19. 避免使用trig/func；触发器、函数不用；客户端程序取而代之
20. 不用select * ： 消耗cpu，io，内存，带宽，这种程序不具有扩展性
21. OR改写为IN()；`or`的效率是n级别；`in`的消息时log(n)级别；in的个数建议控制在200以内
```sql
select id from t where phone=’159′ or phone=’136′;
=>
select id from t where phone in (’159′, ’136′);
```
22. OR改写为UNION
mysql的索引合并很弱智
```sql
select id from t where phone = ’159′ or name = ‘john’;
=>
select id from t where phone=’159′
union
select id from t where name=’jonh’
```
23. 避免负向%
24. 慎用count(*)
25. limit高效分页
limit越大，效率越低
```sql
select id from t limit 10000, 10;
=>
select id from t where id > 10000 limit 10;
```
26. 使用`union all`替代`union`, `union`有去重开销
27. 少用连接join
28. 使用group by,分组,自动排序
29. 请使用同类型比较
30. 使用`load data`导数据
load data比insert快约20倍；
31. 打散批量更新
32. 新能分析工具
```sql
show profile;
mysqlsla;
mysqldumpslow;
explain;
show slow log;
show processlist;
show query_response_time(percona)
```