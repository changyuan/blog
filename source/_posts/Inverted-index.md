---
title: 倒排索引
date: 2016-08-11 11:19:01
updated: 2016-08-11 11:19:01
tags:
categories:
---

### 正排索引和倒排索引

**倒排索引**（英语：Inverted index），也常被称为**反向索引**、**置入档案**或**反向档案**，被用来存储在全文搜索下某个单词在一个文档或者一组文档中的存储位置的映射。它是文档检索系统中最常用的数据结构。
<!--more-->
有两种不同的反向索引形式：

一条记录的水平反向索引（或者反向档案索引）包含每个引用单词的文档的列表。
一个单词的水平反向索引（或者完全反向索引）又包含每个单词在一个文档中的位置。
后者的形式提供了更多的兼容性（比如短语搜索），但是需要更多的时间和空间来创建。

所谓的正排索引是从索引文档到关键词到内容，倒排索引则是相反从关键词到词频，位置，目录等信息，现在通常用于搜索的。由于互联网上的数据量无限大，不可能存储足够多的文档，所以正排索引用处不大。


有两种不同的反向索引形式：

- 一条记录的水平反向索引（或者反向档案索引）包含每个引用单词的文档的[列表](https://zh.wikipedia.org/wiki/%E5%88%97%E8%A1%A8)。

- 一个单词的水平反向索引（或者完全反向索引）又包含每个单词在一个文档中的位置。


  ​我们就能得到下面的反向文件索引：

- ![{\displaystyleT_{0}=}](https://wikimedia.org/api/rest_v1/media/math/render/svg/1ab3d9202f18a678affe9a339511bef0a7b8b110)`"it is what it is"`
- ![{\displaystyleT_{1}=}](https://wikimedia.org/api/rest_v1/media/math/render/svg/57ac5840245616f79874f4635b3bcb2e3da344dd)`"what is it"`
- ![{\displaystyleT_{2}=}](https://wikimedia.org/api/rest_v1/media/math/render/svg/5713cf5634b3846b4d68c3383b65db289511d8bf)`"it is a banana"`

```
"a":      {2}
"banana": {2}
"is":     {0, 1, 2}
"it":     {0, 1, 2}
"what":   {0, 1}
```
检索的条件"what", "is" 和 "it" 将对应这个集合： ![coll](https://wikimedia.org/api/rest_v1/media/math/render/svg/576f880df65391031d612f3d9130ebd81575f6a1)

对相同的文字，我们得到后面这些完全反向索引，有文档数量和当前查询的单词结果组成的的成对数据。 同样，文档数量和当前查询的单词结果都从零开始。所以，`"banana": {(2, 3)}` 就是说 "banana"在第三个文档里 ( {\displaystyle T_{2}} T_{2})，而且在第三个文档的位置是第四个单词(地址为 3)。

```
"a":      {(2, 2)}
"banana": {(2, 3)}
"is":     {(0, 1), (0, 4), (1, 1), (2, 1)}
"it":     {(0, 0), (0, 3), (1, 2), (2, 0)}
"what":   {(0, 2), (1, 0)}
```

如果我们执行短语搜索"what is it" 我们得到这个短语的全部单词各自的结果所在文档为文档0和文档1。但是这个短语检索的连续的条件仅仅在文档1得到。

反向索引数据结构是典型的搜索引擎检索算法重要的部分。
一个搜索引擎执行的目标就是优化查询的速度：找到某个单词在文档中出现的地方。以前，正向索引开发出来用来存储每个文档的单词的列表，接着掉头来开发了一种反向索引。 正向索引的查询往往满足每个文档有序频繁的全文查询和每个单词在校验文档中的验证这样的查询。
实际上，时间、内存、处理器等等资源的限制，技术上正向索引是不能实现的。
为了替代正向索引的每个文档的单词列表，能列出每个查询的单词所有所在文档的列表的反向索引数据结构开发了出来。
随着反向索引的创建，如今的查询能通过立即的单词标示迅速获取结果（经过随机存储）。随机存储也通常被认为快于顺序存储。

### 构建方法

#### 简单法
  索引的构建[4]  相当于从正排表到倒排表的建立过程。当我们分析完网页时 ,得到的是以网页为主码的索引表。当索引建立完成后 ,应得到倒排表 ,具体流程如图所示：
  流程描述如下：
  1）将文档分析称单词term标记，
  2）使用hash去重单词term
  　　3）对单词生成倒排列表
  　　倒排列表就是文档编号DocID，没有包含其他的信息（如词频，单词位置等），这就是简单的索引。
  　　这个简单索引功能可以用于小数据，例如索引几千个文档。然而它有两点限制：
  　　1）需要有足够的内存来存储倒排表，对于搜索引擎来说， 都是G级别数据，特别是当规模不断扩大时 ,我们根本不可能提供这么多的内存。
  　　2）算法是顺序执行，不便于并行处理。


####  合并法
  归并法[4]  ,即每次将内存中数据写入磁盘时，包括词典在内的所有中间结果信息都被写入磁盘，这样内存所有内容都可以被清空，后续建立索引可以使用全部的定额内存。
  归并索引
  归并索引
  如图 归并示意图：
  合并流程：
  1）页面分析，生成临时倒排数据索引A，B，当临时倒排数据索引A，B占满内存后，将内存索引A，B写入临时文件生成临时倒排文件，
  　　2) 对生成的多个临时倒排文件 ,执行多路归并 ,输出得到最终的倒排文件 ( inverted file)。
  合并流程
  合并流程
  索引创建过程中的页面分析 ,特别是中文分词为主要时间开销。算法的第二步相对很快。这样创建算法的优化集中在中文分词效率上。

>[参考1]: http://blog.csdn.net/hguisu/article/details/7962350
>[baike]: http://baike.baidu.com/view/676861.htm
>[wikipedia]: https://zh.wikipedia.org/wiki/%E5%80%92%E6%8E%92%E7%B4%A2%E5%BC%95
>[参考2]: http://blog.csdn.net/hguisu/article/details/7962350