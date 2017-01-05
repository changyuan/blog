---
title: 门面模式(facade pattern)
date: 2015-09-22
tags: 
- design pattern
---

门面模式（Facade）或 外观模式,定义了一个高层接口，这个接口使得子系统更加容易使用：引入门面角色之后，用户只需要直接与门面角色交互，*用户与子系统之间的复杂关系由门面角色来实现*，从而降低了系统的耦合度。Laravel 中门面模式的使用也很广泛，**基本上每个服务容器中注册的服务提供者类**都对应一个门面类。(接口封装场景)


```php
// OsInterface.php

namespace DesignPatterns\Structural\Facade;

/**
 * OsInterface接口
 */
interface OsInterface
{
    /**
     * halt the OS
     */
    public function halt();
}


// BiosInterface.php

namespace DesignPatterns\Structural\Facade;

/**
 * BiosInterface接口
 */
interface BiosInterface
{
    /**
     * execute the BIOS
     */
    public function execute();

    /**
     * wait for halt
     */
    public function waitForKeyPress();

    /**
     * launches the OS
     *
     * @param OsInterface $os
     */
    public function launch(OsInterface $os);

    /**
     * power down BIOS
     */
    public function powerDown();
}


//实现

namespace DesignPatterns\Structural\Facade;

/**
 * 门面类
 */
class Facade
{
    /**
     * @var OsInterface
     */
    protected $os;

    /**
     * @var BiosInterface
     */
    protected $bios;

    /**
     * This is the perfect time to use a dependency injection container
     * to create an instance of this class
     *
     * @param BiosInterface $bios
     * @param OsInterface   $os
     */
    public function __construct(BiosInterface $bios, OsInterface $os)
    {
        $this->bios = $bios;
        $this->os = $os;
    }

    /**
     * turn on the system
     */
    public function turnOn()
    {
        $this->bios->execute();
        $this->bios->waitForKeyPress();
        $this->bios->launch($this->os);
    }

    /**
     * turn off the system
     */
    public function turnOff()
    {
        $this->os->halt();
        $this->bios->powerDown();
    }
}

// 测试

namespace DesignPatterns\Structural\Facade\Tests;

use DesignPatterns\Structural\Facade\Facade as Computer;
use DesignPatterns\Structural\Facade\OsInterface;

/**
 * FacadeTest用于测试门面模式
 */
class FacadeTest extends \PHPUnit_Framework_TestCase
{

    public function getComputer()
    {
        $bios = $this->getMockBuilder('DesignPatterns\Structural\Facade\BiosInterface')
                ->setMethods(array('launch', 'execute', 'waitForKeyPress'))
                ->disableAutoload()
                ->getMock();
        $os = $this->getMockBuilder('DesignPatterns\Structural\Facade\OsInterface')
                ->setMethods(array('getName'))
                ->disableAutoload()
                ->getMock();
        $bios->expects($this->once())
                ->method('launch')
                ->with($os);
        $os->expects($this->once())
                ->method('getName')
                ->will($this->returnValue('Linux'));

        $facade = new Computer($bios, $os);
        return array(array($facade, $os));
    }

    /**
     * @dataProvider getComputer
     */
    public function testComputerOn(Computer $facade, OsInterface $os)
    {
        // interface is simpler :
        $facade->turnOn();
        // but I can access to lower component
        $this->assertEquals('Linux', $os->getName());
    }
}
```
