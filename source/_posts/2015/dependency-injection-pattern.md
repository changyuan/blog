---
title: 依赖注入(dependency injection pattern)
date: 2015-09-28
tags: 
- design pattern
---

依赖注入模式需要在调用者外部完成容器创建以及容器中接口与实现类的运行时绑定工作，在 Laravel 中该容器就是服务容器，而接口与实现类的运行时绑定则在服务提供者中完成。此外，除了在调用者的构造函数中进行依赖注入外，还可以通过在调用者的方法中进行依赖注入。

[三种常见注入方式：](https://en.wikipedia.org/wiki/Dependency_injection)
1. 构造注入
2. setter注入
3. 接口注入


```php
interface Shape
{
    public function getCircum();

    public function getArea();
}

//矩形

class Rectangle implements Shape
{

    private $width, $height;

    public function __construct($width, $height)
    {
        $this->width = $width;
        $this->height = $height;
    }

    public function getCircum()
    {

        return ($this->width + $this->height) * 2;
    }

    public function getArea()
    {

        return $this->width * $this->height;
    }
}

class Circle implements Shape
{

    private $radii;

    public function __construct($radii)
    {
        $this->radii = $radii;
    }

    public function getCircum()
    {
        // TODO: Implement getCircum() method.
        return 2 * M_PI * $this->radii;

    }

    public function getArea()
    {
        // TODO: Implement getArea() method.
        return M_PI * pow($this->radii, 2);
    }
}

```

## 构造注入
```php

class DependencyInjection
{
    public $shape;

    public function __construct(Shape $shape)
    {
        $this->shape = $shape;
    }
}


// 调用示例：

$rectangle = new Rectangle(4, 2);
$r_di = new DependencyInjection($rectangle);
echo $r_di->shape->getCircum();
echo $r_di->shape->getArea();

$circle = new Circle(3);
$c_di = new DependencyInjection($circle);
echo $c_di->shape->getCircum();
echo $c_di->shape->getArea();
```

## setter 注入

```php

class DependencyInjectionSetter
{

    public $shape;

    public function setter(Shape $shape)
    {

        $this->shape = $shape;
    }
}

//调用示例：

$rectangle = new Rectangle(4, 2);
$setter_di = new DependencyInjectionSetter();
$setter_di->setter($rectangle);
echo $setter_di->shape->getCircum();

```

## interface 注入

```php
interface ServiceSetter
{
    public function setService(Shape $shape);
}

class ServiceClient implements ServiceSetter
{

    public $shape;

    public function setService(Shape $shape)
    {
        // TODO: Implement setService() method.
        $this->shape = $shape;
    }
}

//调用示例：

$sc = new ServiceClient();
$sc->setService($rectangle);
echo $sc->shape->getArea();
```