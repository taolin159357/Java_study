一个java文件只能有一个public class，public class的名字必须和文件名字一样；



每个子类的构造函数的第一句话，都默认调用父类的无参构造函数super()，super语句必须放在第一条；



派生类继承抽象类，若没有实现基类中的所有abstract方法，那么派生类也必须被定义为抽象类；实现接口，就必须实现所有未实现的方法，如果没有全部实现，那么只能成为一个抽象类；接口没有构造函数和main函数，抽象类可以有main函数且能运行；



静态方法需要调用才会执行，执行顺序：static块>匿名块>构造函数；



常量池：相同的值只存取一份，节省内存，共享访问；![](1.png)

![](2.png)

![](3.png)

![](4.png)

![](5.png)



不可变对象一旦创建，这个对象(状态/值)不能被更改了，如八个基本型别的包装类的对象，String,BigInteger和BigDecimal等的对象；

![](6.png)



访问规则

![](7.png)



异常规则

![](8.png)

![](9.png)

![](10.png)



​		使用try-catch-finally处理编译时异常，是得程序在编译时就不再报错，但是运行时仍可能报错，相当于我们使用try-catch-finally将一个编译时可能出现的异常，延迟到运行时出现；也即是说try必须有，catch和finally至少要有一个，finally必执行，catch块的异常匹配是从上而下执行的，并且只能进入一个catch块；finally中声明的是一定会被执行的代码，即使catch中又出现异常了；针对于编译时异常，我们说一定要考虑异常的处理。

异常打印写法：

e.printStackTrace();

System.out.println(e.getMessage());

