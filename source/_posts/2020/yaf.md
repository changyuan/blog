---
title: Yaf Yar使用
date: 2020-05-02 14:51:38
updated: 2020-05-02 14:51:38
tags:
- yaf 
- rewrite
categories:
---

# Yar (Yet Another RPC Framework) 一个rpc框架

Yar 是一个轻量级, 高效的RPC框架, 它提供了一种简单方法来让PHP项目之间可以互相远程调用对方的本地方法. 并且Yar也提供了并行调用的能力. 可以支持同时调用多个远程服务的方法.


### 范例

```php


//服务端
<?php

/* 假设这个页面的访问路径是: http://example.com/operator.php */

class Operator {

    /**
     * Add two operands
     * @param interge 
     * @return interge
     */
    public function add($a, $b) {
        return $this->_add($a, $b);
    }

    /**
     * Sub 
     */
    public function sub($a, $b) {
        return $a - $b;
    }

    /**
     * Mul
     */
    public function mul($a, $b) {
        return $a * $b;
    }

    /**
     * Protected methods will not be exposed
     * @param interge 
     * @return interge
     */
    protected function _add($a, $b) {
        return $a + $b;
    }
}

$server = new Yar_Server(new Operator());
$server->handle();
?>


// 客户端

<?php
$client = new yar_client("http://example.com/operator.php");

/* call directly */
var_dump($client->add(1, 2));

/* call via call */
var_dump($client->call("add", array(3, 2)));


/* __add can not be called */
var_dump($client->_add(1, 2));
?>
```



## 多个客户端异步调用

```php

<?php
function callback($ret, $callinfo) {
    echo $callinfo['method'] , " result: ", $ret , "\n";
}

/* 注册一个异步调用 */
Yar_Concurrent_Client::call("http://example.com/operator.php", "add", array(1, 2), "callback");
Yar_Concurrent_Client::call("http://example.com/operator.php", "sub", array(2, 1), "callback");
Yar_Concurrent_Client::call("http://example.com/operator.php", "mul", array(2, 2), "callback");

/* 发送所有注册的调用, 等待返回, 返回后Yar会调用callback回掉函数 */
Yar_Concurrent_Client::loop();





function callback($ret, $callinfo) {
    echo $callinfo['method'] , " result: ", $ret , "\n";
}

/* 注册一个异步调用 */
Yar_Concurrent_Client::call("http://example.com/operator.php", "add", array(1, 2), "callback");
/* 发送所有注册的调用, 等待返回, 返回后Yar会调用callback回掉函数 */
Yar_Concurrent_Client::loop();
/* 重置call ,否则上面的call会调用*/
Yar_Concurrent_Client::reset();
Yar_Concurrent_Client::call("http://example.com/operator.php", "sub", array(2, 1), "callback");
Yar_Concurrent_Client::loop();


?>

```
# Yaf官方文档

[php net yaf官网](https://www.php.net/manual/zh/book.yaf.php)
[中文文档](https://www.laruence.com/manual/index.html)

### Yaf的优点

- 用C语言开发的PHP框架, 相比原生的PHP, 几乎不会带来额外的性能开销.
- 所有的框架类, 不需要编译, 在PHP启动的时候加载, 并常驻内存.
- 更短的内存周转周期, 提高内存利用率, 降低内存占用率.
- 灵巧的自动加载. 支持全局和局部两种加载规则, 方便类库共享.
- 高性能的视图引擎.
- 高度灵活可扩展的框架, 支持自定义视图引擎, 支持插件, 支持自定义路由等等.
- 内建多种路由, 可以兼容目前常见的各种路由协议.
- 强大而又高度灵活的配置文件支持. 并支持缓存配置文件, 避免复杂的配置结构带来的性能损失.
- 在框架本身,对危险的操作习惯做了禁止.
- 更快的执行速度, 更少的内存占用.


## 安装

```php

Yaf can be installed from source code by:

cd /path/to/yaf-src/
phpize
./configure
make
sudo make install

```

## 伪静态

```php

http://dev-all.ekeguan.com/test/1/1.html

在 Bootstrap.php 中 _initRoute 增加：
//http://dev-all.ekeguan.com/product/list/1/2
 $router = $dispatcher->getRouter();
$route  = new Yaf_Route_Rewrite(
        "/product/list/:id/:name",
        array(
            "controller" => "index",
            "action"     => "index",
        )
    ); 
    
    $router->addRoute('dummy', $route);
// http://dev-all.ekeguan.com/user-list/1

// http://dev-all.ekeguan.com/test/1/1.html

    $config = array(
     "user-list" => array(
        "type"  => "rewrite",        //Yaf_Route_Rewrite route
        "match" => "/user-list/:id", //match only /user/list/?/
        "route" => array(
            'controller' => "index",  //route to user controller,
            'action'     => "index",  //route to list action
        ),
     ),
     "test" => array(
        "type"  => "regex",          //Yaf_Route_Regex route
        "match" => "#^/test/([^/]+)/([^/])(.*)\.html+#",         //match arbitrary request uri
        "route" => array(
            'controller' => "index",  //route to product controller,
            'action'     => "index",    //route to dummy action
        ),
        "map" => array(
           1 => "uri",   // now you can call $request->getParam("uri")
        ),
     ),
 );

 $router->addConfig(new Yaf_Config_Simple($config));
 
 
 //分页
 $router = $dispatcher->getRouter();
$config = array(
   "detail" => array(
      "type"  => "regex",          //Yaf_Route_Regex route
      "match" => "#^/details/(\d+)\/?(\d?)(\.html)?#",         //match arbitrary request uri
      "route" => array(
          'controller' => "index",  //route to product controller,
          'action'     => "index",    //route to dummy action
      ),
      "map" => array(
         1 => "id",   // now you can call $request->getParam("uri")
         2 => "page",   // now you can call $request->getParam("uri")
      ),
   ),
);

$router->addConfig(new Yaf_Config_Simple($config));

```