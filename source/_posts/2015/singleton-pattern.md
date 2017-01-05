---
title: 门面模式(singleton pattern)
date: 2015-09-23
tags: 
- design pattern
---

- 必须拥有一个访问级别为 `private` 的构造函数，有效防止类被随意实例化； 
- 必须拥有一个保存类的实例的静态变量； 
- 必须拥有一个访问这个实例的公共的静态方法，该方法通常被命名为 getInstance()； 
- 必须拥有一个私有的空的__clone方法，防止实例被克隆复制

```php
class Singleton
{

    private $name;

    private static $instance;

    private function __construct()
    {

    }

    private function __clone()
    {
        // TODO: Implement __clone() method.
    }


    public static function getInstance()
    {

        if (!self::$instance instanceof self) {
            self::$instance = new self();
        }
        return self::$instance;
    }


    public function getName()
    {
        return $this->name;
    }

    public function setName($name)
    {
        $this->name = $name;
    }

    public function sayHi()
    {
        echo "hi";
    }

}


// 调用demo

$single = Singleton::getInstance();
//$c_single= clone  $single;
$single->setName("TOM");
echo $single->getName();
```