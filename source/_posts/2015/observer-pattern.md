---
title: 观察者模式
date: 2015-09-22
tags: 
- design pattern
---

观察者模式有时也被称**作发布/订阅模式**，该模式用于为对象实现发布/订阅功能：一旦**主体对象**状态发生改变，与之关联的**观察者对象**会收到通知，并进行相应操作。

将一个系统分割成一个一些类相互协作的类有一个不好的副作用，那就是需要维护相关对象间的一致性。我们不希望为了维持一致性而使各类紧密耦合，这样会给维护、扩展和重用都带来不便。观察者就是解决这类的耦合关系的。


**消息队列系统、事件**都使用了观察者模式。

PHP 为观察者模式定义了两个接口：`SplSubject` 和 `SplObserver`。`SplSubject` 可以看做**主体对象**的抽象，`SplObserver` 可以看做**观察者对象**的抽象，要实现观察者模式，只需**让主体对象实现 SplSubject** ，**观察者对象实现 SplObserver**，并实现相应方法即可。

## 示例1
```php
//下面两个接口，在php已经有了，直接implements即可，
interface IObserver
{
    function onChanged( $sender, $args );
}

interface IObservable
{
    function addObserver( $observer );
}

class UserList implements IObservable
{
    private $_observers = array();

    public function addCustomer( $name )
    {
        foreach( $this->_observers as $obs )
            $obs->onChanged( $this, $name );
    }

    public function addObserver( $observer )
    {
        $this->_observers []= $observer;
    }
}

class UserListLogger implements IObserver
{
    public function onChanged( $sender, $args )
    {
        echo( "'$args' added to user list\n" );
    }
}

$ul = new UserList();
$ul->addObserver( new UserListLogger());
$ul->addCustomer( "Jack" );

```


#### 示例2 
使用了php中的 SplSubject,SplObserver 主体对象和观察者对象类
```php

// User.php 被观察主体对象

<?php

namespace DesignPatterns\Behavioral\Observer;

/**
 * 观察者模式 : 被观察对象 (主体对象)
 *
 * 主体对象维护观察者列表并发送通知
 *
 */
class User implements \SplSubject
{
    /**
     * user data
     *
     * @var array
     */
    protected $data = array();

    /**
     * observers
     *
     * @var \SplObjectStorage
     */
    protected $observers;
    
    public function __construct()
    {
        $this->observers = new \SplObjectStorage();
    }

    /**
     * 附加观察者
     *
     * @param \SplObserver $observer
     *
     * @return void
     */
    public function attach(\SplObserver $observer)
    {
        $this->observers->attach($observer);
    }

    /**
     * 取消观察者
     *
     * @param \SplObserver $observer
     *
     * @return void
     */
    public function detach(\SplObserver $observer)
    {
        $this->observers->detach($observer);
    }

    /**
     * 通知观察者方法
     *
     * @return void
     */
    public function notify()
    {
        /** @var \SplObserver $observer */
        foreach ($this->observers as $observer) {
            $observer->update($this);
        }
    }

    /**
     *
     * @param string $name
     * @param mixed  $value
     *
     * @return void
     */
    public function __set($name, $value)
    {
        $this->data[$name] = $value;

        // 通知观察者用户被改变
        $this->notify();
    }
}

// UserObserver.php  // 观察者

namespace DesignPatterns\Behavioral\Observer;

/**
 * UserObserver 类（观察者对象）
 */
class UserObserver implements \SplObserver
{
    /**
     * 观察者要实现的唯一方法
     * 也是被 Subject 调用的方法
     *
     * @param \SplSubject $subject
     */
    public function update(\SplSubject $subject)
    {
        echo get_class($subject) . ' has been updated';
    }
}

// 测试

namespace DesignPatterns\Behavioral\Observer\Tests;

use DesignPatterns\Behavioral\Observer\UserObserver;
use DesignPatterns\Behavioral\Observer\User;

/**
 * ObserverTest 测试观察者模式
 */
class ObserverTest extends \PHPUnit_Framework_TestCase
{

    protected $observer;

    protected function setUp()
    {
        $this->observer = new UserObserver();
    }

    /**
     * 测试通知
     */
    public function testNotify()
    {
        $this->expectOutputString('DesignPatterns\Behavioral\Observer\User has been updated');
        $subject = new User();

        $subject->attach($this->observer);
        $subject->property = 123;
    }

    /**
     * 测试订阅
     */
    public function testAttachDetach()
    {
        $subject = new User();
        $reflection = new \ReflectionProperty($subject, 'observers');

        $reflection->setAccessible(true);
        /** @var \SplObjectStorage $observers */
        $observers = $reflection->getValue($subject);

        $this->assertInstanceOf('SplObjectStorage', $observers);
        $this->assertFalse($observers->contains($this->observer));

        $subject->attach($this->observer);
        $this->assertTrue($observers->contains($this->observer));

        $subject->detach($this->observer);
        $this->assertFalse($observers->contains($this->observer));
    }

    /**
     * 测试 update() 调用
     */
    public function testUpdateCalling()
    {
        $subject = new User();
        $observer = $this->getMock('SplObserver');
        $subject->attach($observer);

        $observer->expects($this->once())
            ->method('update')
            ->with($subject);

        $subject->notify();
    }
}

```

## 官方demo
使用了`SplObjectStorage` 这个对象，这个对象本身有attach ,detach 的方法，如果不用，可以参考第二个
```php

class MyObserver1 implements SplObserver {
    public function update(SplSubject $subject) {
        echo __CLASS__ . ' - ' . $subject->getName();
    }
}

class MyObserver2 implements SplObserver {
    public function update(SplSubject $subject) {
        echo __CLASS__ . ' - ' . $subject->getName();
    }
}

class MySubject implements SplSubject {
    private $_observers;
    private $_name;

    public function __construct($name) {
        $this->_observers = new SplObjectStorage();
        $this->_name = $name;
    }

    public function attach(SplObserver $observer) {
        $this->_observers->attach($observer);
    }

    public function detach(SplObserver $observer) {
        $this->_observers->detach($observer);
    }

    public function notify() {
        foreach ($this->_observers as $observer) {
            $observer->update($this);
        }
    }

    public function getName() {
        return $this->_name;
    }
}

$observer1 = new MyObserver1();
$observer2 = new MyObserver2();

$subject = new MySubject("test");

$subject->attach($observer1);
$subject->attach($observer2);
$subject->notify();

```


## 一个报纸和订阅者的例子，很像发布和订阅

```php

/**
* Subject,that who makes news
*/
class Newspaper implements \SplSubject{
    private $name;
    private $observers = array();
    private $content;
    
    public function __construct($name) {
        $this->name = $name;
    }

    //add observer
    public function attach(\SplObserver $observer) {
        $this->observers[] = $observer;
    }
    
    //remove observer
    public function detach(\SplObserver $observer) {
        
        $key = array_search($observer,$this->observers, true);
        if($key){
            unset($this->observers[$key]);
        }
    }
    
    //set breakouts news
    public function breakOutNews($content) {
        $this->content = $content;
        $this->notify();
    }
    
    public function getContent() {
        return $this->content." ({$this->name})";
    }
    
    //notify observers(or some of them)
    public function notify() {
        foreach ($this->observers as $value) {
            $value->update($this);
        }
    }
}

/**
* Observer,that who recieves news
*/
class Reader implements SplObserver{
    private $name;
    
    public function __construct($name) {
        $this->name = $name;
    }
    
    public function update(\SplSubject $subject) {
        echo $this->name.' is reading breakout news <b>'.$subject->getContent().'</b><br>';
    }
}

$newspaper = new Newspaper('Newyork Times');

$allen = new Reader('Allen');
$jim = new Reader('Jim');
$linda = new Reader('Linda');

//add reader
$newspaper->attach($allen);
$newspaper->attach($jim);
$newspaper->attach($linda);

//remove reader
$newspaper->detach($linda);

//set break outs
$newspaper->breakOutNews('USA break down!');

```