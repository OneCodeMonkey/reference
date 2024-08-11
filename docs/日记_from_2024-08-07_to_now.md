日记流水
===

2024-08
----

### todo

1. 整理 mysql 点
3. rpc 框架及文档
4. websocket 网页demo
5. JVM要点整理
6. 面试题：写两段Java代码，稳定地触发触发一次 Young GC，另一个是稳定地触发一次 Full GC？
7. 如何在web项目中，接入 Github 三方授权登陆的登陆方式？


### Week2

#### 8.7
1. ✅搭建 reference 的 web deploy 的 jenkins 流水线
2. ✅搭建 trading-tool 的 java 项目自动 deploy 的 jenkins 流水线
3. ✅测完意图全集白名单，部署发上线
4. ✅测完DM需求改动，尝试周四上线

#### 8.8 
1. mysql java 中查询分页的 demo
2. kafka 基本用法的 java demo

#### 8.9
1. ✅算法题
2. ✅neitui
3. ✅websocket demo

#### 周末
1. github 三方登陆方式接入
2. ✅java abstract 关键字
3. ✅trading-tool 的波动率报警功能

总结：Java abstract 关键字用法
```text
# Java 中 abstract 关键字用法

## 1.作用
我们一般认为，如果不想某个类被实例化（比如 Animal 类，太抽象了你去实例化它没有意义）时，会在 class 前面加上一个 abstract 关键字。
另外一个重要目的，abstrac 类定义好了以后，我们希望继承此抽象类的 子类，一定去实现 父类即抽象类里定义的某些方法，此时需要用抽象类 abstract 来修饰父类。类似于定义了一个模版类，可以这么理解。
而需要子类必须实现的方法，则在父类中将方法用 abstract 修饰即可。

## 2.使用范围
1. 修饰 class
2. 修饰 method：注意，abstract method 只能定义在 abstract class 中，
    1. 抽象方法注意点：一是抽象类中定义的抽象方法，一定不能有具体实现
    2. 当继承 抽象类的子类 是非抽象类（可实例化）时，必须要强制实现父类中的 abstract 方法

抽象类中不强制要求必须有抽象方法，抽象方法一定出现在抽象类中。

抽象类中 abstract 方法只能是声明，而不能有实现。但非 abstract 方法，还是必须要写具体实现的。

## 3.问题点
1. 抽象类可以继承其他抽象类，或继承一个非抽象类么？
答：可以继承抽象类，这个很好理解。我们可以一直抽象类，一直继承下去，这个没啥问题。

抽象类去继承非抽象类，这个其实也可以。观察 abstract 类的定义，我们会发现它其实是继承自 Object 类的，Object 类是非抽象的，所以一个抽象类去继承一个非抽象类是不违反编译器规则的。

2. 抽象类中的抽象方法，可以重载出多个方法吗？
答：可以的。
比如：`public abstract int sum(int a,int b);`
   `public abstract double sum(double a,int b);`
    `public abstract double sum(int a,int b,double c);`
    `public abstract double sum(double c,int a,int b);`

重载只需要 function 名相同，形参列表不同即可。抽象方法也是可以重载的。

```