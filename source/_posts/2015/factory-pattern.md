---
title: 工厂模式(factory pattern)
date: 2015-09-27
tags: 
- design pattern
---

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



// 创建工厂
class Factory
{

    public static function create()
    {
        //判断穿过来的参数个数
        switch (func_num_args()) {
            case 1:
                return new Circle(func_get_arg(0));
                break;
            case 2:
                return new Rectangle(func_get_arg(0), func_get_arg(1));
                break;
        }

    }
}


$rectangle = Factory::create(4,2);
echo $rectangle->getCircum()." ";
echo $rectangle->getArea()." ";

$circle = Factory::create(3);
echo $circle->getCircum()." ";
echo $circle->getArea();
```