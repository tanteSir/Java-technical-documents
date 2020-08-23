### JDK1.8新特性

#### 一、接口的默认和静态方法

```java
	我们可以给接口添加一个非抽象的方法实现，只需要使用default关键字即可。这个特征又叫扩展方法
	
如：
	public interface JDK8Interface{
        //静态方法
        static void staticMethod(){
            system.out.println("静态方法");
        }
        
        //default定义默认方法
        default void defaultMethod(){
            system.out.println("默认方法");
        }
	}
```

#### 二、Lambda表达式

##### 1、什么是Lambda表达式？

```
lambda表达式本质上是一个匿名方法
lambda表达式由三部分组成： 参数列表、箭头、表达式或语句块
```

```java
平时的
	public int add(int x, int y){
        return x+y;
	}
转换成lambda表达式是这样的：
	(int x, int y) -> x+y;
类型可省略，如：
	(x,y) -> x+y;
或
	(x，y) -> {return x+y;} //显示指明返回值
	
注意：
	() -> {system.out.println("Hello Lambda");}
	这种没参，没返回值的，相当于 Runnable中run方法的一个实现

	c - > {return c.size();}
	如果只有一个参数且可以被java识别类型，那参数的括号也可以省略
```

##### 2、lambda表达式的类型（它是Object吗?）

```java
	lambda表达式可以被看做是一个Object，但不代表它就是Object。
	lambda表达式的类型，叫“目标类型”，它的目标类型是“函数接口”。
	这是JDK1.8引入的概念，它的定义是：一个接口，如果只有一个显示声明的抽象方法，那它就是一个函数接口，一般用 @FunctionalInterface标出来（也可不标）
如：
	@FunctionalInterface
	public interface Runnable{ void run();}
	
	你可以用lambda表达式为一个函数接口赋值
如：
	Runnable r1 = () -> {system.out.println("Hello Lambda");};
然后在赋值给一个Object：
	Object obj = r1;
但不能这样做：	
	//Error Object is not a functional interface!
	Object obj = () - > {system.out.println("Hello Lambda");};
必须显示转换成一个接口才可以：
	Object obj = （Runnable）() -> {system.out.println("Hello Lambda");};
	
Lambda表达式只有转型成一个函数接口后才能被当做Object使用，所以下面这句也会报错：
	system.out.println(() -> {});//错误！目标类型不明
必须先转型：
	system.out.println((Runnable)() -> {});//正确
```

#### 三、允许使用 : : 关键字传递方法或构造函数引用

```

```

#### 四、函数式接口

```
接口只能有一个抽象方法
```

#### 五、支持多重注释

```
满足很多时候一个注解需要在某一位置使用多次的场景
```

#### 六、提供新的时间API

```
提供了 java.time包
```

#### 七、Base64编码

```
Base64编码成为java类库的标准
```

#### 八、JS引擎Nashorn

```
Nashorn允许在JVM上开发允许JavaScript代码，允许Java与JavaScript代码互相调用
```

#### 九、Stream

```
把真正的函数式编程风格引入Java代码中
```

#### 十、Optional

```java
	JDK1.8中引入Optional类来防止空指针异常。Optional类实际上是个容器，它可以保存T的值，或保存null。使用Optional类我们可不用显式进行空指针检查了
```

#### 十一、扩展注解的支持

```
	JDK1.8扩展了注解的上下文，几乎可以为任何东西添加注解，包括局部变量、泛型类、父类和接口的实现，连方法的异常也能添加注解
```

#### 十二、并行(Parallel)数组

```
	支持对数组并行处理，主要是parallelSort()方法，它可以在多核机器上极高的提高数组排序的顺序
```

#### 十三、编译器优化

```
	JDK1.8将方法参数名加到字节码中，这样在运行时就能通过反射获取参数名，只需要在编译时使用-parameters参数
```

#### 十四、IO升级