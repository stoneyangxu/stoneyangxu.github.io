---
layout: post
title:  "JAVA8实战 - 高效编程"
date:   2017-10-28 22:47:37
categories: 读书笔记
---

# 高效Java8编程

## 重构/测试和调试

在现有代码上重构代码, **适配和使用Lambda表达式**, 使得代码具有更好的**可读性和灵活性**

一些常见的**设计模式**在使用Lambda之后会变得更简洁:

- 策略模式
- 模板方法模式
- 观察者模式
- 责任链模式
- 工厂模式

### 为改善可读性和灵活性重构代码

使用Lambda表达式来代替匿名类或者将现有的方法以参数形式传递, 能写出更简洁的代码.

#### 改善代码的可读性

- 减少冗长代码, 让其更容易理解
- 通过方法引用和Stream API, 让代码更直观

包括三个方法:

- 重构代码, 使用Lambda代替匿名类
- 用方法引用重构Lambda表达式
- 用Stream API重构命令式的数据处理


#### 从匿名类到Lambda表达式

```java
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Task");
    }
};

// 简化为

Runnable r = () -> System.out.println("Task");
```

在某些情况下, 使用表达式可能会比较复杂:

- 匿名类和Lambda表达式中的**this和super含义不同**, 匿名类中, this是自身, 而在Lambda中代表的是包含类
- 匿名类可以屏蔽包含类的变量, 而Lambda不能, 会导致编译错误
- 在设计方法重载的时候, 使用Lambda表达式可能会使代码更晦涩, 因为表达式是根据上下文来推断类型的, 可能会导致两者都匹配的情况, 需要利用显示转换才能选择正确的方法.

```java
public void doSomething(Runnable r) { r.run(); };
public void doSomething(Task t) { t.execute(); };

// 错误, 两者都会匹配
// doSomething(() -> System.out.println("Task")); 

// 需要使用显示转换
doSomething((Task)() -> System.out.println("Task"));
```

#### 从Lambda表达式到方法引用

- Lambda表达式非常适合**传递代码片段**, 尽量使用**方法引用**, 因为方法名能够更好的表明意图.
- 尽量考虑使用静态辅助方法, 比如comparing和maxBy

```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

inventory.sort(comparing(Apple::getWeight));
```

#### 从命令式的数据处理切换到Stream

Stream API能够更清晰的表达数据处理管道的意图, 还能够通过短路, 延迟载入和并行进行优化.

```java
List<String> dishNames = new ArrayList<>();
for (Dish dish : menu) {
    if (dish.getCalories() > 300) {
        disNames.add(dish.getNames());
    }
}


menu.parallelStream()
    .filter(d -> d.getCalories() > 300)
    .map(Dish::getName)
    .collect(toList())
```

## 默认方法

## 使用Optional取代null

## CompletableFuture: 组合式异步编程

## 新的日期和时间API

