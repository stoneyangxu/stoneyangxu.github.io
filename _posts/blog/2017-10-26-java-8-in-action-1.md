---
layout: post
title:  "JAVA8实战 - 基础知识"
date:   2017-10-26 22:47:37
categories: 读书笔记
---

# 概要

- JAVA8是发布以来变化最大的一次
- 完全向下兼容
- 引入新的语法和设计模式，帮助开发者编写更清楚、更简洁的代码
- 引入函数式编程方式
- 其他的一些优化改进

# 基础知识

- 总结java的主要变化:`Lambda、方法引用、流和默认方法`
- 了解行为`参数化`，这是一种`软件开发模式`，也是引入Lambda的主要原因
- 全面解释Lambda和方法引用

## 为什么要关心Java8

- Java8让开发者使用起来更容易、更简洁

```java
    // before java8
    Collections.sort(appleList, new Comparator<Apple>() {
        @Override
        public int compare(Apple o1, Apple o2) {
            return o1.getWeight() .compareTo(o2);
        }
    });

    // Java8
    appleList.sort(comparing(Apple::getWeight));
```

- 如果想利用计算机的多核能力必须使用多线程，Java8借助Stream简化了多线程的使用

- 函数(方法)作为一等公民，代替原本复杂的匿名类

### Java怎么还在变

- 编程语言就像生态系统一样，新的语言出现，旧语言就被取代，除非他们不断演变
- 在大数据背景下，开发者需要更方便的并行处理机制
- 语言需要不断改进, 以跟进硬件的更新或满足程序员的期望

### 流处理

`流`是一系列数据项，一次只生成一项。

程序可以从输入流中一个一个的读取数据，然后以同样的方式写入输出流。

一个程序的输出流很可能是另一个程序的输入流。

例如linux中的管道:

```shell
$ cat file1 file2 | tr "[A-Z]" "[a-z]" | sort | tail -3
```

Java8在`java.util.stream`中添加了`Stream API`，Stream<T>就是一系列T类型的项目，Java8通过内置方法，将流`连接`起来，形成复杂的`流水线`。这带来两个好处:

- 更高层次的抽象，面向流进行设计
- 免费、透明的并行处理

```java
// 例如下面的方法: 按value过滤后,再按照currency分组;
    
Map<Currency, List<Transaction>> transactionsByCurrencies = new HashMap<>();
for (Transaction transaction : transactions) {
if(transaction.getValue() > 1000) {
    Currency currency = transaction.getCurrency();
    List<Transaction> transactionsForCurrency = transactionsByCurrencies.get(currency);
    if (transactionsForCurrency == null) {
        transactionsForCurrency = new ArrayList<>();
        transactionsByCurrencies.put(currency, transactionsForCurrency);
    }
    transactionsForCurrency.add(transaction);
}
}


// 使用流API实现的话:
Map<Currency, List<Transaction>> transactionsByCurrencies = transactions.stream()
        .filter((Transaction t) -> t.getValue() > 1000)
        .collect(groupingBy(Transaction::getCurrency));

```

### 用行为参数化把代码传递给方法

把方法作为参数传递给另一个方法.

在函数式编程语言中, 这是一个基础概念, 将函数作为一等公民, 可以作为参数传入, 也可以作为结果返回.

Scala和Groovy已经证明, 将函数作为一等公民可以极大的`扩充程序员的工具库`, 让编程变得更加容易.

```java

// 列举当前目录下的隐藏文件
File[] hiddenFile = new File(".").listFiles(new FileFilter() {
    @Override
    public boolean accept(File file) {
        return file.isHidden();
    }
});

// java8
File[] hiddenFileInJava8 = new File(".").listFiles(File::isHidden);

```

- `File::isHidden`会创建一个`方法引用`, 可以作为值进行传递

```java


    public List<Apple> filterGreenApples(List<Apple> apples) {
        List<Apple> result = new ArrayList<>();
        
        for (Apple apple : apples) {
            if ("green".equals(apple.getColor())) {
                result.add(apple);
            }
        }
        
        return result;
    }
    
    public List<Apple> filterHeavyApples(List<Apple> apples) {
        List<Apple> result = new ArrayList<>();

        for (Apple apple : apples) {
            if (apple.getWeight() > 150) {
                result.add(apple);
            }
        }

        return result;
    }
    
    // 上述代码编写麻烦, 而且带来了重复代码, 区别只是过滤规则不同
    
    // java8
    
    public static boolean isGreenApple(Apple apple) {
        return "green".equals(apple.getColor()); 
    }

    public static boolean isHeavyApple(Apple apple) {
        return apple.getWeight() > 150;
    }

    public static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p){
        List<Apple> result = new ArrayList<>();
        for(Apple apple : inventory){
            if(p.test(apple)){
                result.add(apple);
            }
        }
        return result;
    }       
    
    // 使用:
    List<Apple> greenApples = filterApples(inventory, FilteringApples::isGreenApple);
    List<Apple> heavyApples = filterApples(inventory, FilteringApples::isHeavyApple);

    // 或者直接使用lambda表达式
    filterApples(apples, (Apple a) -> "green".equals(a.getColor()));
    filterApples(apples, (Apple a) -> a.getWeight() > 150);

```

### 并行与共享的可变数据

想要免费使用流的并行能力,需要保证方法不能`有副作用`, 即:不能修改共享变量,也可以被叫做`纯函数`或者`无状态函数`

```java
// 使用parallelStream将列表转化为并行流, 使用免费的并行能力
List<Apple> heavyApples = apples.parallelStream().filter((Apple a) -> a.getWeight() > 150)
        .collect(toList());
```

- 内部库会进行自动分区
- 使用纯函数来利用免费的并行计算

### 默认方法

为了支持`库设计师`, 让他们能够写出`更容易改进`的接口, 避免修改接口时, 破坏已有的实现类.

```java
    
// java.util.Collection    
default Stream<E> stream() {
    return StreamSupport.stream(spliterator(), false);
}
```

### 其他函数式编程思想

- Option<T> - 避免出现NullPointer异常
- 模式匹配 - 更简洁的表达逻辑判断
- ......

## 通过行为参数化传递代码

例如:

```java
public static List<Apple> filterGreenApples(List<Apple> inventory){
	List<Apple> result = new ArrayList<>();
	for(Apple apple: inventory){
		if("green".equals(apple.getColor())){
			result.add(apple);
		}
	}
	return result;
}

public static List<Apple> filterApplesByColor(List<Apple> inventory, String color){
	List<Apple> result = new ArrayList<>();
	for(Apple apple: inventory){
		if(apple.getColor().equals(color)){
			result.add(apple);
		}
	}
	return result;
}

public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight){
	List<Apple> result = new ArrayList<>();
	for(Apple apple: inventory){
		if(apple.getWeight() > weight){
			result.add(apple);
		}
	}
	return result;
}
```

过滤一个列表,只是规则不同,却产生了大量重复的逻辑

在调用时,传入的参数也不能很好的表明意图:

```java
List<Apple> greenApples = filterGreenApples(inventory);
List<Apple> redApples = filterApplesByColor(inventory, "red");
List<Apple> heavyApples = filterApplesByWeight(inventory, 150);

```

*变化的内容是我们的选择标准, 我们称他为`谓词`*

通过接口对`谓词`进行建模, 并且实现多个选择标准, 类似于设计模式中的`策略模式`: 

```java
interface ApplePredicate{
	boolean test(Apple a);
}

static class AppleWeightPredicate implements ApplePredicate{
	public boolean test(Apple apple){
		return apple.getWeight() > 150; 
	}
}
static class AppleColorPredicate implements ApplePredicate{
	public boolean test(Apple apple){
		return "green".equals(apple.getColor());
	}
}
```

让filter方法接受原始列表和一个`谓词`, 对列表项进行测试, 这就是`行为参数化`: 

```java
public static List<Apple> filter(List<Apple> inventory, ApplePredicate p){
	List<Apple> result = new ArrayList<>();
	for(Apple apple : inventory){
		if(p.test(apple)){
			result.add(apple);
		}
	}
	return result;
}       
```

当需要添加其他筛选规则时, 可以直接简单的定义新的规则: 

```java
static class AppleRedAndHeavyPredicate implements ApplePredicate{
	public boolean test(Apple apple){
		return "red".equals(apple.getColor()) 
				&& apple.getWeight() > 150; 
	}
}
```

上述结果在使用时依然存在缺陷, 必须构造一个谓词类, 并且new一个实例进行传递:

```java
List<Apple> redAndHeavyApples = filter(inventory, new AppleRedAndHeavyPredicate());
```

一种方案是使用Java早起的匿名类:

```java
List<Apple> redApples = filter(inventory, new ApplePredicate() {
	public boolean test(Apple a){
		return a.getColor().equals("red"); 
	}
});
```

但是依然不够简洁.

使用lambda表达式进一步简化:

```java

// Predicate 是 Java8 内置的谓词类
public List<Apple> filter(List<Apple> apples, Predicate<Apple> predicate) {
    List<Apple> result = new ArrayList<>();

    for (Apple apple : apples) {
        if (predicate.test(apple)) {
            result.add(apple);
        }
    }

    return result;
}

// 使用lambda表达式
filter(apples, (Apple a) -> "green".equals(a.getColor()));
```

再继续往下进行, filter方法可以泛化, 使得过滤逻辑应用在不同的类上:

```java
public List<T> filter(List<T> list, Predicate<T> predicate) {
    List<T> result = new ArrayList<>();

    for (T e : list) {
        if (predicate.test(e)) {
            result.add(e);
        }
    }

    return result;
}

List<Integer> even = filter(numbers, (Integer n) -> n % 2 == 0);
```

### Java8的内置支持

- sort方法(Collections.sort)

```java
apples.sort((Apple a, Apple b) -> a.getWeight().compareTo(b.getWeight()));
```

- 用Runnable执行代码块

```java
Thread t = new Thread(() -> System.out.println("thread"));
```

### GUI事件处理

```java
button.setOnAction((ActionEvent e) -> label.setText("Sent!"));
```

### 总结

- 行为参数化, 就是一个方法接受`不同的行为`作为`参数`, 在内部使用它们, 完成不同的能力, 类似于`策略模式`
- 行为化参数可以让代码具有更好的`灵活性`, 更`简洁的语法`
- 用lambda语法来代替以往复杂的匿名类语法
- Java API包含了很多不同行为进行参数化的方法

## Lambda表达式

### 简洁

Lambda表达式可以理解为`简洁`的表示`可传递`的`匿名函数`的一种方式.
包含`参数列表`, `函数主题`, `返回类型`, 以及`可以抛出的异常列表`

```java
List<Apple> redApples = filter(inventory, new ApplePredicate() {
	public boolean test(Apple a){
		return a.getColor().equals("red"); 
	}
});

// 可以使用lamdba简化为: 

// 备注: Lambda表达式`隐含`了return语句
List<Apple> redApples = filter(inventory, (Apple a) -> a.getColor().equals("red"));

```

### Lambda表达式的语法为

```java
(parameters) -> exception
// 或者

(parameters) -> { exception }
```

### 函数式接口

只定义`一个`抽象方法的接口(不计算`默认方法`)

```java
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

Lambda允许直接以`内联`的形式为`函数式接口`的抽象方法提供实现, 并把整个`表达式`作为函数式接口的实例.

> 可以使用@FunctionalInterface声明函数式接口, 提示编译器对接口形式进行校验

### 函数描述符

函数式接口的抽象方法的`签名`, 同样也是lambda表达式的签名

例如: (Apple a) -> a.getWeight()的签名就是(Apple) -> Integer

### 实例: 简化模板方法

```java
    // 函数式接口
	public interface BufferedReaderProcessor{
		public String process(BufferedReader b) throws IOException;
	}
	
	// 模板方法
	public static String processFile(BufferedReaderProcessor p) throws IOException {
		try(BufferedReader br = new BufferedReader(new FileReader("data.txt"))){
			return p.process(br);
		}
	}
	
    // lambda表达式省略匿名类语法
	String oneLine = processFile((BufferedReader b) -> b.readLine());
```
### 内置的函数式接口

- Predicate - 定义用来`判断`的抽象接口tsest, 接受`泛型T对象`, 返回`boolean`

![](/images/2017-10-27-01-01-11.jpg)

```java
package java.util.function;

import java.util.Objects;

/**
 * Represents a predicate (boolean-valued function) of one argument.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #test(Object)}.
 *
 * @param <T> the type of the input to the predicate
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);

    /**
     * Returns a composed predicate that represents a short-circuiting logical
     * AND of this predicate and another.  When evaluating the composed
     * predicate, if this predicate is {@code false}, then the {@code other}
     * predicate is not evaluated.
     *
     * <p>Any exceptions thrown during evaluation of either predicate are relayed
     * to the caller; if evaluation of this predicate throws an exception, the
     * {@code other} predicate will not be evaluated.
     *
     * @param other a predicate that will be logically-ANDed with this
     *              predicate
     * @return a composed predicate that represents the short-circuiting logical
     * AND of this predicate and the {@code other} predicate
     * @throws NullPointerException if other is null
     */
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    /**
     * Returns a predicate that represents the logical negation of this
     * predicate.
     *
     * @return a predicate that represents the logical negation of this
     * predicate
     */
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    /**
     * Returns a composed predicate that represents a short-circuiting logical
     * OR of this predicate and another.  When evaluating the composed
     * predicate, if this predicate is {@code true}, then the {@code other}
     * predicate is not evaluated.
     *
     * <p>Any exceptions thrown during evaluation of either predicate are relayed
     * to the caller; if evaluation of this predicate throws an exception, the
     * {@code other} predicate will not be evaluated.
     *
     * @param other a predicate that will be logically-ORed with this
     *              predicate
     * @return a composed predicate that represents the short-circuiting logical
     * OR of this predicate and the {@code other} predicate
     * @throws NullPointerException if other is null
     */
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    /**
     * Returns a predicate that tests if two arguments are equal according
     * to {@link Objects#equals(Object, Object)}.
     *
     * @param <T> the type of arguments to the predicate
     * @param targetRef the object reference with which to compare for equality,
     *               which may be {@code null}
     * @return a predicate that tests if two arguments are equal according
     * to {@link Objects#equals(Object, Object)}
     */
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}
```

- Consumer - 定义抽象方法accept, 接受泛型对象T, `返回void`; 访问对象T, 并`执行某些操作`

```java
package java.util.function;

import java.util.Objects;

/**
 * Represents an operation that accepts a single input argument and returns no
 * result. Unlike most other functional interfaces, {@code Consumer} is expected
 * to operate via side-effects.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #accept(Object)}.
 *
 * @param <T> the type of the input to the operation
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);

    /**
     * Returns a composed {@code Consumer} that performs, in sequence, this
     * operation followed by the {@code after} operation. If performing either
     * operation throws an exception, it is relayed to the caller of the
     * composed operation.  If performing this operation throws an exception,
     * the {@code after} operation will not be performed.
     *
     * @param after the operation to perform after this operation
     * @return a composed {@code Consumer} that performs in sequence this
     * operation followed by the {@code after} operation
     * @throws NullPointerException if {@code after} is null
     */
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}

```

- Function - 定义抽象方法apply, 接受泛型T的对象, 返回泛型R的对象; 用于将对象信息`映射到输出`

```java
package java.util.function;

import java.util.Objects;

/**
 * Represents a function that accepts one argument and produces a result.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #apply(Object)}.
 *
 * @param <T> the type of the input to the function
 * @param <R> the type of the result of the function
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);

    /**
     * Returns a composed function that first applies the {@code before}
     * function to its input, and then applies this function to the result.
     * If evaluation of either function throws an exception, it is relayed to
     * the caller of the composed function.
     *
     * @param <V> the type of input to the {@code before} function, and to the
     *           composed function
     * @param before the function to apply before this function is applied
     * @return a composed function that first applies the {@code before}
     * function and then applies this function
     * @throws NullPointerException if before is null
     *
     * @see #andThen(Function)
     */
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    /**
     * Returns a composed function that first applies this function to
     * its input, and then applies the {@code after} function to the result.
     * If evaluation of either function throws an exception, it is relayed to
     * the caller of the composed function.
     *
     * @param <V> the type of output of the {@code after} function, and of the
     *           composed function
     * @param after the function to apply after this function is applied
     * @return a composed function that first applies this function and then
     * applies the {@code after} function
     * @throws NullPointerException if after is null
     *
     * @see #compose(Function)
     */
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    /**
     * Returns a function that always returns its input argument.
     *
     * @param <T> the type of the input and output objects to the function
     * @return a function that always returns its input argument
     */
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}

```

- 自动装箱接口, 自动将原始类型转换为包装类型, 例如: IntPredicate, LongPredicate等

```java
package java.util.function;

import java.util.Objects;

/**
 * Represents a predicate (boolean-valued function) of one {@code int}-valued
 * argument. This is the {@code int}-consuming primitive type specialization of
 * {@link Predicate}.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #test(int)}.
 *
 * @see Predicate
 * @since 1.8
 */
@FunctionalInterface
public interface IntPredicate {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param value the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(int value);

    /**
     * Returns a composed predicate that represents a short-circuiting logical
     * AND of this predicate and another.  When evaluating the composed
     * predicate, if this predicate is {@code false}, then the {@code other}
     * predicate is not evaluated.
     *
     * <p>Any exceptions thrown during evaluation of either predicate are relayed
     * to the caller; if evaluation of this predicate throws an exception, the
     * {@code other} predicate will not be evaluated.
     *
     * @param other a predicate that will be logically-ANDed with this
     *              predicate
     * @return a composed predicate that represents the short-circuiting logical
     * AND of this predicate and the {@code other} predicate
     * @throws NullPointerException if other is null
     */
    default IntPredicate and(IntPredicate other) {
        Objects.requireNonNull(other);
        return (value) -> test(value) && other.test(value);
    }

    /**
     * Returns a predicate that represents the logical negation of this
     * predicate.
     *
     * @return a predicate that represents the logical negation of this
     * predicate
     */
    default IntPredicate negate() {
        return (value) -> !test(value);
    }

    /**
     * Returns a composed predicate that represents a short-circuiting logical
     * OR of this predicate and another.  When evaluating the composed
     * predicate, if this predicate is {@code true}, then the {@code other}
     * predicate is not evaluated.
     *
     * <p>Any exceptions thrown during evaluation of either predicate are relayed
     * to the caller; if evaluation of this predicate throws an exception, the
     * {@code other} predicate will not be evaluated.
     *
     * @param other a predicate that will be logically-ORed with this
     *              predicate
     * @return a composed predicate that represents the short-circuiting logical
     * OR of this predicate and the {@code other} predicate
     * @throws NullPointerException if other is null
     */
    default IntPredicate or(IntPredicate other) {
        Objects.requireNonNull(other);
        return (value) -> test(value) || other.test(value);
    }
}

```

### 函数式接口的一些例子

- 布尔表达式 - Predicate
- 创建对象 - Supplier
- 消费对象 - Consumer
- 选择/提取 - Function或ToIntFunction
- 合并两个值 - IntBinaryOperator
- 比较两个对象 - Comparator或BiFunction或ToIntBiFunction

### 类型检查/类型推断/限制

#### 类型检查

Lambda表达式自身并不知道自己所代表的`函数式接口`, 编译器根据`上下文`进行`推断`. 所需要的类型称为`目标类型`

当使用Lambda表达式的时候, 根据所在方法的接口声明找到`函数式接口`, 判断出实际的`目标类型`并进行`类型检查`

同样的表达式可以匹配到`不同`的函数式接口上, 只要方法签名能够兼容. 

```java
Callable<Integer> c = () -> 42;
PrivilegedAction<Integer> p = () -> 42;
```

不可以将Lambda表达式复制给Object对象, 因为Object不是一个函数式接口

```java
Object o = () -> {} // error!
```

#### 推断

可以在表达式中省略参数的类型, 让编译器自己去推断.

```java
List<Apple> greenApples = filter(apples, a -> "green".equals(a.getColor());
```

### 使用局部变量

Lambda表达式允许使用`自由变量`, 即表达式外部的变量.

```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
```

但是, 变量必须显示声明为`final`或者是`实时上的final`

```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber); // compile error!
portNumber = 7331; 
```

原因在于,局部变量是保存在`栈`上, 表达式是在一个`线程`中使用, 如果变量被改变, 可能导致线程的不安全. 

### 方法引用

调用特定方法的Lambda的一种`快捷写法`, 根据`已有`的方法来创建表达式

```java
apples.sort(comparing(Apple::getWeight));
```

主要包含三类:

- 指向`静态方法` - Integer::parseInt

```java
    public List<Integer> toInts(List<String> stringList, ToIntFunction<String> t) {
        List<Integer> result = new ArrayList<>();
        for (String s : stringList) {
            result.add(t.applyAsInt(s));
        }
        return result;
    }

    toInts(stringList, Integer::parseInt);
```

- 指向`任意类型实例方法` - String::length

```java
    public void print(List<Apple> apples, Consumer<Apple> c) {
        for (Apple apple : apples) {
            c.accept(apple);
        }
    }

    print(appleList, Apple::printColor)
```

- 指向`现有对象的实例方法` 

```java
    List<String> list = Arrays.asList("stone", "yang", "xu", "others");
    String fullName = "stoneyangxu";
    List<String> result = list.stream().filter(fullName::contains).collect(Collectors.toList());

```

- 方式二和方式三的区别: 方式二是引用一个`对象`的方法, 这个对象本身是`表达式的参数`; 方式三是在表达式`主体`调用对象的方法

#### 构造函数的引用

```java
ClassName::new
```

- 无参数的构造函数

```java
Supplier<Apple> s = Apple::new;
Apple apple = s.get();

// 等价于

Supplier<Apple> s = () -> new Apple();
Apple apple = s.get();
```
- 有一个参数的构造函数

```java
Function<Integer, Apple> f = Apple::new;
Apple apple = f.apply(100);

// 等价于

Function<Integer, Apple> f = weight -> new Apple(weight);
Apple apple = f.apply(100);

```

- 有多个参数的构造函数

```java
BiFunction<String, Integer, Apple> b = Apple::new;
Apple apple = b.apply("green", 100);

// 等价于

BiFunction<String, Integer, Apple> b = (color, weight) -> new Apple(color, weight);
Apple apple = b.apply("green", 100);
```


### 复合Lambda表达式的有用方法

将简单的表达式`复合`成为复杂的表达式. 

借助函数式接口的`默认方法`

#### 比较器复合


```java

// 基本
Comparator<Apple> c = comparing(Apple::getWeight);

// 逆序
Comparator<Apple> c = comparing(Apple::getWeight).reversed();

// 比较器链
Comparator<Apple> c = comparing(Apple::getWeight).thenComparing(Apple::getColor)

```

#### 谓词复合

```java
// 基本
Predicate<Apple> redApple = (Apple a) -> "red".equals(a.getColor());
Predicate<Apple> heavyApple = (Apple a) -> a.getWeight() > 150;

// 否定
Predicate<Apple> notRedApple = redApple.negate();

// 而且
Predicate<Apple> redAndHeavyApple = redApple.and(heavyApple);

// 或者
Predicate<Apple> redOrGreenApple = redApple.and((Apple a) -> "green".equals(a.getColor()));
```

#### 函数复合

```java
// 基本
Function<Integer, Integer> increase = x -> x + 1;
Function<Integer, Integer> multiply = x -> x * 2;

// 链式
Function<Integer, Integer> f = increase.andThen(multiply);
f.apply(1); // multiply(increase(1)) => 4

// 组合
Function<Integer, Integer> g = increase.compose(multiply);
g.apply(1); // increase(multiply(1)) => 3
```

### 总结

- Lambda表达式可以理解为匿名函数的简化
- `函数式接口`是只声明一个抽象接口(不计算默认方法)的接口
- 只有在接受函数式接口的地方,才能够使用lambda表达式
- Java8自带的一些函数式接口,位于java.uitl.function内
- Java8提供了自动装箱的函数式接口
- Lambda表达式需要代表的类型成为`目标类型`
- `方法引用`允许使用现有方法的实现并直接传递它们
- 利用函数式接口的`默认方法`, 复合成更为复杂的表达式