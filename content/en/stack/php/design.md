---
author: "karson"
title: "php常用设计模式代码"
date: 2020-05-18
description: "常用设计模式：单例模式、工厂模式、观察者模式、适配器模式、装饰器模式"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: karson
authorEmoji: 👻
tags:
- php
categories:

---
## 单例模式
单例模式是一种 <font color='cyan'>**创建唯一实例的设计模式**</font>。它通常用于管理资源，例如数据库连接或者日志记录。

单例模式可以确保 <font color='cyan'>**一个类只有一个实例**</font> ，这样就可以避免多个实例同时访问共享资源。在PHP中，单例模式也被广泛应用于 <font color='cyan'>**缓存管理**</font> 和 <font color='cyan'>**路由器**</font> 等组件。
```php
<?php
class Singleton {
    private static $_instance = null;

    // 构造方法私有化
    private function __construct() {}

    // 提供一个public方法供外部调用，以获取实例对象
    public static function getInstance() {
        if (is_null(self::$_instance)) {
            self::$_instance = new Singleton();
        }
        return self::$_instance;
    }

    // 防止克隆实例
    public function __clone() {
        exit('Clone is not allowed:' . E_USER_ERROR);
    }
}
?>
```

## 工厂模式

### 特点：

<font color='cyan'>**将对象使用与对象创建分离**</font>  ,调用者直接向工厂请求,减少代码的耦合.提高系统的可维护性与可扩展性。
### 应用场景：

提供一种类，具有为您创建对象的某些方法，这样就可以使用工厂类创建对象，而不直接使用new。这样如果想更改创建的对象类型，只需更改该工厂即可

工厂模式是一种 <font color='cyan'>**创建对象的设计模式**</font> 。它提供了一个通用的接口来创建对象，这样就可以使一个类实例化任何具体类。
工厂模式在PHP中有很多应用，例如在创建数据库连接对象时。如果需要在程序中使用不同的数据库连接，可以使用工厂模式来创建连接对象。
```php
<?php
class Button {}
class MacButton extends Button {}
class WinButton extends Button {}

interface AbstractFactory {
    public function createButton();
}

class MacFactory implements AbstractFactory {
    public function createButton() {
        return new MacButton();
    
}

class WinFactory implements AbstractFactory {
    public function createButton() {
        return new WinButton();
    }
}
?>
```

## 观察者模式
### 特点：

观察者模式(Observer)，当 <font color='cyan'>**一个对象状态发生变化时，依赖它的对象全部会收到通知，并自动更新**</font>。观察者模式实现了低耦合，非侵入式的通知与更新机制。

### 应用：

一个事件发生后，要执行一连串更新操作。传统的编程方式，就是在事件的代码之后直接加入处理的逻辑。当更新的逻辑增多之后，代码会变得难以维护。这种方式是耦合的，侵入式的，增加新的逻辑需要修改事件的主体代码。
```php
<?php
# 被观察者接口
interface Observable
{
    // 添加/注册观察者
    public function attach(Observer $observer);
    // 删除观察者
    public function detach(Observer $observer);
    // 触发通知
    public function notify();
}
 
/**
 * 被观察者
 * 职责：添加观察者到$observers属性中，
 * 有变动时通过notify()方法运行通知
 */
class Order implements Observable
{
    // 保存观察者
    private $observers = array();
    // 订单状态
    private $state = 0;
 
    // 添加（注册）观察者
    public function attach(Observer $observer)
    {
        $key = array_search($observer, $this->observers);
        if ($key === false) {
            $this->observers[] = $observer;
        }
    }
 
    // 移除观察者
    public function detach(Observer $observer)
    {
        $key = array_search($observer, $this->observers);
        if ($key !== false) {
            unset($this->observers[$key]);
        }
    }
 
    // 遍历调用观察者的update()方法进行通知，不关心其具体实现方式
    public function notify()
    {
        foreach ($this->observers as $observer) {
            // 把本类对象传给观察者，以便观察者获取当前类对象的信息
            $observer->update($this);
        }
    }
 
    // 订单状态有变化时发送通知
    public function addOrder()
    {
        $this->state = 1;
        $this->notify();
    }
 
    // 获取提供给观察者的状态
    public function getState()
    {
        return $this->state;
    }
}
```

## 适配器模式
### 目的
适配器模式是一种 <font color='cyan'>**将不兼容的对象或接口转换为兼容的对象或接口**</font> 的设计模式。它适用于使用不同的库或框架的程序，或者在API升级时需要调整现有代码的情况。

在PHP中，适配器模式的一个使用例子是在将数据从不同的数据源导入数据库时。例如，如果需要从一个XML文件中导入数据并将其插入到一个MySQL数据库中，适配器可以将XML数据源转换为MySQL数据源，然后插入到数据库中。

### 应用场景：

1. 封装有缺陷的接口设计
2. 统一多个类的接口设计，比如一个支付系统，有三种不同的支付方式，微信支付、支付宝支付、网银支付，这三种支付的实现方法都不一样，那么我们可以用适配器模式，让他们对外具有统一的方法，这样，我们在调用的时候就非常的方便。
3. 兼容老版本的接口

```php
<?php
/**
 * 适配器模式演示代码
 * Target适配目标： IDataBase接口
 * Adaptee被适配者： mysql和mysqli、postgresql的数据库操作函数
 * Adapter适配器 ：mysql类和mysqli、postgresql类
 */

/**
 * Interface IDatabase 适配目标，规定的接口将被适配对象实现
 * 约定好统一的api行为
 */
interface IDatabase {
    // 定义数据库连接方法
    public function connect($host, $username, $password, $database);
    // 定义数据库查询方法
    public function query($sql);
    // 关闭数据库
    public function close();
}

/**
 * Class Mysql 适配器
 */
class Mysql implements IDatabase {
    protected $connect; // 连接资源

    /**
     * 实现连接方法
     *
     * @param $host host
     * @param $username 用户名
     * @param $password 密码
     * @param $database 数据库名
     */
    public function connect($host, $username, $password, $database)
    {
        $connect = mysql_connect($host, $username, $password);
        mysql_select_db($database, $connect);
        $this->connect = $connect;
        //其他操作
    }

    /**
     * 实现查询方法
     *
     * @param $sql 需要被查询的sql语句
     * @return mi
     */
    public function query($sql)
    {
        return mysql_query($sql);
    }

    // 实现关闭方法
    public function close()
    {
        mysql_close();
    }
}

/**
 * Class Mysqli 适配器
 */
class Mysqli implements IDatabase {
    protected $connect; // 连接资源

    /**
     * 实现连接方法
     *
     * @param $host host
     * @param $username 用户名
     * @param $password 密码
     * @param $database 数据库名
     */
    public function connect($host, $username, $password, $database)
    {
        $connect = mysqli_connect($host, $username, $password, $database);
        $this->connect = $connect;
        //其他操作
    }

    /**
     * 实现查询方法
     *
     * @param $sql 需要被查询的sql语句
     */
    public function query($sql)
    {
        return mysqli_query($this->connect, $sql);
    }

    // 实现关闭
    public function close()
    {
        mysqli_close($this->connect);
    }
}

/**
 * Class Postgresql 适配器
 */
class Postgresql implements IDatabase
{
    protected $connect; // 连接资源

    /**
     * 实现连接方法
     *
     * @param $host
     * @param $username
     * @param $password
     * @param $database
     */
    public function connect($host, $username, $password, $database)
    {
        $this->connect = pg_connect("host=$host dbname=$database user=$username password=$password");
        //其他操作
    }

    /**
     * 实现查询方法
     *
     * @param $sql 需要被查询的sql语句
     */
    public function query($sql)
    {
        // 其他操作
    }

    // 实现关闭方法
    public function close()
    {

    }
}


/**
 * 客户端使用演示
 * 这里以mysql为例
 * 只要模式设计好，不论有多少种数据库，实现和调用方式都是一样的
 * 因为都是实现的同一个接口，所以都是可以随意切换的
 */

$host = 'localhost';
$username = 'root';
$password = 'root';
$database = 'mysql';

//$client = new Postgresql();
//$client = new Mysql();
$client = new Mysqli();
$client->connect($host, $username, $password, $database);
$result = $client->query("select * from db");
while ($rows = mysqli_fetch_array($result)) {
    var_dump($rows);
}
$client->close();
```

## 装饰器模式
装饰器模式是一种通过在运行时动态添加功能的设计模式。它通过包装目标对象来实现这一点，从而扩展或修改其行为。
在PHP中，装饰器模式可以用于单元测试、日志记录、和调试等方面。通过使用装饰器模式，可以在目标类中添加或修改方法，而不会改变目标类本身。

### 结构
MilkTea：原本的对象和装饰共同的接口 示例中指：奶茶

Oolong、Latte： 原本的对象 示例中指：声声乌龙、幽兰拿铁

Decorator： 实现接口的装饰抽象类

Cream、…：具体的装饰 示例中：奶油、碧根果、开心果


奶茶基类
```php
/**
 * 奶茶
 */
interface MilkTea
{
    /**
     * 名称
     * @return mixed
     */
    public function name();

    /**
     * 价格
     * @return mixed
     */
    public function price();
}
```

具体奶茶类：
```php
/**
 * 声声乌龙
 */
class Oolong implements MilkTea
{
    public function name()
    {
        return '声声乌龙';
    }

    public function price()
    {
        return 16;
    }
}
```

小白拿铁
```php
/**
 * 幽兰拿铁
 */
class Latte implements MilkTea
{
    public function name()
    {
        return '幽兰拿铁';
    }

    public function price()
    {
        return 17;
    }
}
```

加料类基类
```php
class Decorator implements MilkTea
{
    protected $milkTea;

    public function __construct(MilkTea $milkTea)
    {
        $this->milkTea = $milkTea;
    }

    public function name()
    {
        if ($this->milkTea != null) {
            return $this->milkTea->name();
        }
        return '';
    }

    public function price()
    {
        if ($this->milkTea != null) {
            return $this->milkTea->price();
        }
        return 0;
    }
}
```

加料具体类
奶油


```php
/**
 * 奶油
 */
class Cream extends Decorator
{
    public function name()
    {
        return $this->milkTea->name() . '+ 奶油';
    }

    public function price()
    {
        return $this->milkTea->price() + 4;
    }
}
```

碧根果
```php
/**
 * 碧根果
 */
class Pecan extends Decorator
{
    public function name()
    {
        return $this->milkTea->name() . '+ 碧根果';
    }

    public function price()
    {
        return $this->milkTea->price() + 2;
    }
}
```


开心果
```php
/**
 * 开心果
 */
class Pistachio extends Decorator
{
    public function name()
    {
        return $this->milkTea->name() . '+ 开心果';
    }

    public function price()
    {
        return $this->milkTea->price() + 4;
    }
}
```

客户端使用：
```php
/**
 * 我点一杯 幽兰拿铁+ 奶油+ 开心果
 */
$latte = new Latte();
$cream = new Cream($latte);
$pistachio = new Pistachio($cream);

echo $pistachio->name();
echo '   ' . $pistachio->price() . '元' . PHP_EOL;

/**
 * 点一杯加三个奶油的声声乌龙（因为我比较喜欢喝奶油）
 */

$oolong = new Oolong();
$cream = new Cream($oolong);
$cream = new Cream($cream);
$cream = new Cream($cream);

echo $cream->name();
echo '   ' . $cream->price() . '元' . PHP_EOL;
```

输出：
```shell
幽兰拿铁+ 奶油+ 开心果   25元
声声乌龙+ 奶油+ 奶油+ 奶油   28元
```
