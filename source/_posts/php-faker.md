---
title: php-faker 
date: 2016-08-19 14:46:49
updated: 2016-08-19 14:46:49
tags: php
categories:
---

Faker 是一个用来生成数据库实例数据的工具。Faker is a PHP library that generates fake data for you. Whether you need to bootstrap your database, create good-looking XML documents, fill-in your persistence to stress test it, or anonymize data taken from a production service, Faker is for you.

### installation

`composer require fzaninotto/faker`

### basic usage



Use `Faker\Factory::create()` to create and initialize a faker generator, which can generate data by accessing properties named after the type of data you want.

```php
<?php

require_once '/path/to/Faker/src/autoload.php';
$faker = Faker\Factory::create();
echo $faker->name;
echo $faker->address;
echo $faker->text;
echo $faker->email;

```

在`src/Faker/Generator.php` 中狠多种类型可以参考，包括file，image，uuid 各种数据类型。

[GIthub]: https://github.com/fzaninotto/Faker

### 在laravel 中的使用

Laravel artisan 的 tinker 是一个 [REPL (read-eval-print-loop)](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) ，REPL 是指 交互式命令行界面，它可以让你输入一段代码去执行，并把执行结果直接打印到命令行界面里。通过`php artisan tinker`  和faker结合之后，可以直接用来测试。Tinker 是 Laravel 自带的 REPL，基于 [PsySH](http://psysh.org/) 构建而来。同样需要`psysh`的支持。

使用示例如下：

```php
php artisan tinker
//这是是使用faker的工厂模式生成数据，其中App\User需要在database\factories\ModelFactories 文件中提前写好需要设置的数据类型和字段
factory("App\User::class",10)->create(); 
//查询
APP\User::all(); 
App\User::count();
//增加数据
$user = new App\User;
$user->name = "Wruce Bayne";
$user->email = "iambatman@savegotham.com";
$user->save();
// 删除数据
$user = App\User::find(1);
$user->delete();

// 还可以查看doc帮助
doc dd
doc app
  
// 显示源代码
 show <functionName>
```


