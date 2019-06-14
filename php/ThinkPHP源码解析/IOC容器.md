在上一篇文章中我们讲到了 ThinkPHP 如何实现自动加载，如果想看的话可以看
 [ThinkPHP5.1 源码浅析（二）自动加载机制][1]




在阅读本篇文章 之前，我希望你掌握了 IOC 、DI 、Facade的基本知识，如果不了解，请先查看着几篇文章。

[深入理解控制反转（IoC）和依赖注入（DI）][2]



那么步入正题。

# 服务调用

基于分析框架的 入口脚本文件`index.php`

```php
// 加载基础文件
require __DIR__ . '/../thinkphp/base.php';

// 支持事先使用静态方法设置Request对象和Config对象

// 执行应用并响应
Container::get('app')->run()->send();

```

上面 `base.php` 中的作用是载入自动加载机制，和异常处理，以及开启日志功能。

```php
// 执行应用并响应
Container::get('app')->run()->send();
```

在这里才是使用了 IOC 容器功能，获取 app 这个容器[^注1]

进入 Container 中之后我们先介绍他的类属性

```php
protected static $instance; // 定义我们的容器类实例，采用单例模式，只实例化一次
protected $instances = [];	// 容器中的对象实例
protected $bind = [];		// 容器绑定标识
protected $name = [];		// 容器标识别名
```

`$instances` 是实现了 注册树模式[^注2]，存储值

```php
array (
  'think\\App' => App实例,
  'think\\Env' => Env实例,
  'think\\Config' => Config实例,
   ...
)
```



`bind` 在初始化时，会载入一堆初始数据，记录一堆类别名和 类名的映射关系。

```php
protected $bind = [
      'app' => 'think\\App',
      'build' => 'think\\Build',
      'cache' => 'think\\Cache',
      'config' => 'think\\Config',
      'cookie' => 'think\\Cookie',
		...
    ]
```

`name`和 `bind` 属性记录的值都是很类似的，都是 类别名和 类名的映射关系。区别是，`name` 记录的是 **已经实例化后的** 映射关系。



进入get方法

```php
public static function get($abstract, $vars = [], $newInstance = false)
{
    return static::getInstance()->make($abstract, $vars, $newInstance);
}
```

 这一段代码没什么好讲的，就是先获取当前容器的实例（单例），并实例化。



进入 make 方法

```php
public function make($abstract, $vars = [], $newInstance = false)
{
    if (true === $vars) {
        // 总是创建新的实例化对象
        $newInstance = true;
        $vars        = [];
    }
	// 如果已经存在并且实例化的类，就用别名拿到他的类
    $abstract = isset($this->name[$abstract]) ? $this->name[$abstract] : $abstract;
	// 如果已经实例化，并且不用每次创建新的实例的话，就直接返回注册树上的实例
    if (isset($this->instances[$abstract]) && !$newInstance) {
        return $this->instances[$abstract];
    }
    // 如果我们绑定过这个类，例如 'app' => 'think\\App',
    if (isset($this->bind[$abstract])) {
        $concrete = $this->bind[$abstract];
		// 因为ThinkPHP 实现可以绑定一个闭包或者匿名函数进入，这里是对闭包的处理
        if ($concrete instanceof Closure) {
            $object = $this->invokeFunction($concrete, $vars);
        } else {
            // 记录 映射关系，并按照 类名来实例化，如 think\\App
            $this->name[$abstract] = $concrete;
            return $this->make($concrete, $vars, $newInstance);
        }
    } else {
        // 按照类名调用该类
        $object = $this->invokeClass($abstract, $vars);
    }

    if (!$newInstance) {
        $this->instances[$abstract] = $object;
    }
	// 返回制作出来的该类
    return $object;
}
```

我们拆分一下，

```php
if (true === $vars) {
        // 总是创建新的实例化对象
        $newInstance = true;
        $vars        = [];
}
```

这段代码是 让我们函数可以 使用 `make($abstract, true)`的方式调用此函数，使我们每次得到的都是新的实例。（我觉得这种方式不是很好，每个变量的造成含义不明确）

```php
// 如果已经存在并且实例化的类，就用别名拿到他的类
    $abstract = isset($this->name[$abstract]) ? $this->name[$abstract] : $abstract;
	// 如果已经实例化，并且不用每次创建新的实例的话，就直接返回注册树上的实例
    if (isset($this->instances[$abstract]) && !$newInstance) {
        return $this->instances[$abstract];
    }
```

前面说过，`name` 中存放的是已经实例化的 **别名=> 类名** 的映射关系，我们在这里尝试取出 类名，如果该类实例化，就直接返回。

```php
// 如果我们绑定过这个类，例如 'app' => 'think\\App',
    if (isset($this->bind[$abstract])) {
        $concrete = $this->bind[$abstract];
		// 因为ThinkPHP 实现可以绑定一个闭包或者匿名函数进入，这里是对闭包的处理
        if ($concrete instanceof Closure) {
            $object = $this->invokeFunction($concrete, $vars);
        } else {
            // 记录 映射关系，并按照 类名来实例化，如 think\\App
            $this->name[$abstract] = $concrete;
            return $this->make($concrete, $vars, $newInstance);
        }
    } else {
        // 按照类名调用该类
        $object = $this->invokeClass($abstract, $vars);
    }
```



这里是看我们需要容器加载的类是否以前绑定过别名（我们也可以直接 `bind('classNickName')` 来设置一个）

1. 如果绑定过，那么就来实例化它。
2. 如果没有，那么就认定他是一个类名，直接调用。[^注3]

# 服务绑定

5/28日完成



[^注1]: 在这里系统找不到 Container 类的位置，所以会执行自动加载机制去寻找 Container 的位置，并加载它
[^注2]: 把一堆实例挂在树上，需要的时候在拿来用。
[^注3]: 直接调用是使用了反射后的结果，关于反射的知识点在自行查看

[1]: https://segmentfault.com/a/1190000019280079
[2]: https://segmentfault.com/a/1190000019280079

# 门面模式