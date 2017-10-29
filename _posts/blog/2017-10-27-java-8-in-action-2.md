---
layout: post
title:  "JAVA8实战 - 函数式数据处理"
date:   2017-10-27 22:47:37
categories: 读书笔记
---

# 引入流

流可以让程序员轻松的处理集合, 分组, 过滤, 并行处理等等.

## 流是什么

`流`是Java API的新成员, 允许以`声明性`方式处理数据集合. 另外流还可以透明的进行`并行处理`.

看看如下复杂的处理:

```java
List<Dish> lowCaloricDishes = new ArrayList<>();

for (Dish dish : menus) {
    if (dish.getCalories() < 400) {
        lowCaloricDishes.add(dish);
    }
}

Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
    @Override
    public int compare(Dish o1, Dish o2) {
        return Integer.compare(o1.getCalories(), o2.getCalories());
    }
});

List<String> lowCaloricDishesName = new ArrayList<>();
for (Dish d : lowCaloricDishes) {
    lowCaloricDishesName.add(d.getName());
}

```

上述复杂的代码, 只是进行了过滤, 排序和映射, 在Java 8中可以简单的写作:

```java
List<String> lowCaloricDishesName = menus.stream()
        .filter(d -> d.getCalories() < 400)
        .sorted(comparing(Dish::getCalories))
        .map(Dish::getName)
        .collect(toList());
```

如果想利用透明的多线程能力,只需要把stream方法替换为parallelStream: 

```java
List<String> lowCaloricDishesName = menus.parallelStream()
        .filter(d -> d.getCalories() < 400)
        .sorted(comparing(Dish::getCalories))
        .map(Dish::getName)
        .collect(toList());
```

新的方式有几个明显的好处:

- 代码简洁, 逻辑清晰
- 使用链式语法组装数据处理流水线
- filter, sorted, map都是与具体线程模型无关的高层次构建, 可以透明的利用并发能力

## 流简介

流 - *从支持数据处理操作的源,生成的元素序列*

- 元素序列 - 与集合一样, 流提供接口访问特定元素类型的一组有序值. 与集合不同的是, 集合讲的是`数据`, 流讲的是`计算`.
- 源 - 提供数据的源, 例如集合, 数组, 或者输入/输出资源. 从有序集合生成的流会保留原有顺序.
- 数据处理操作 - 流的数据处理功能类似与数据库的操作, 以及函数式编程语言的常用操作

此外, 流有两个重要特点:

- 流水线 - 很多流操作都会返回`一个流`, 多个操作可以串联起来, 形成一个流水线
- 内部迭代 - 流的迭代操作是在内部进行的

```java
List<String> lowCaloricDishesName = 
        menus.parallelStream() // 调用stream方法获得流
        .filter(d -> d.getCalories() < 400) // 建立流水线操作, 先进行过滤
        .sorted(comparing(Dish::getCalories)) // 继续流水线操作, 进行排序
        .limit(3) // 只选取前三个
        .map(Dish::getName) // 映射操作
        .collect(toList()); // 将结果保存在另一个list里面
```

- filter - 接受lambda, 从流中排除某些元素
- map - 接受lambdb, 将元素转换成其他形式
- limit - 截断流
- collect - 将流转换为其他形式, 在调用collect之前的操作并没有做任何事, 直到collect调用才执行

## 流与集合

简单来说, 流与集合的差别是*什么时候进行计算*

- 集合是内存中的数据结构, 包含*所有*需要计算的值, 所有元素都要经过计算之后加入到集合中
- 流是在概念上固定的数据结构, 其元素是*按需*计算的, 只有消费者真正使用的时候才进行计算. 

### 只能遍历一次

与迭代器类似, 只能*遍历一次*, 如果需要再次遍历, 只能从源重新获取.

### 外部迭代与内部迭代

使用迭代接口, 例如for-each进行迭代的叫做*外部迭代*

流是在内部进行迭代, 秩序要提供函数告诉他应该做什么即可.

- 在内部迭代时, 可以透明的进行并行处理, 或者更优化的顺序进行处理
- 外部迭代中, 需要自己手动管理迭代顺序和并行操作

### 流操作

分为两大类:

- 中间操作 - 连接起来构成流水线的操作, 在终端操作之前不会做任何动作, 因为中间操作很多都可以*合并*起来, 在终端操作中一次性处理
- 终端操作 - 终止流水线并收集数据的操作, 生成任何不是流的结果

### 总结

- 流: 从支持数据处理操作的源生成一系列元素
- 流使用内部迭代
- 流操作有两类: 中间操作和终端操作
- 流中的元素是按需计算的

## 使用流

使用内部迭代来便利流, 使得内部可以进行各种*优化*动作, 包括合并, 调整顺序, 或者并发执行. 

### 筛选和切片

- 用谓词筛选 - 使用filter方法和一个谓词函数, 返回符合谓词条件的流
- 筛选各异元素 - 使用distinct方法, 返回元素各异的流(使用equals和hashCode对比)
- 截断流 - 使用limit限制元素个数
- 跳过元素 - 使用skip跳过前n个元素, 如果不足n个则返回空的流

### 映射

- map - 流中的每个元素执行函数并生成一个新的元素
- flatMap - 扁平化一个流, 各个元素并不是分别映射, 而是映射元素的内容; 或者说是将流中的每个值映射成另一个流, 并将流连接起来

```java
String[] arrayOfWords = {"Hello", "World"};

Arrays.stream(arrayOfWords)
        .map(w -> w.split(""))
        .flatMap(Arrays::stream) // 扁平化两个流的内容
        .distinct()
        .forEach(System.out::println);
```

```
H
e
l
o
W
r
d
```

### 查找和匹配

#### 匹配

- anyMatch - 终端操作, 返回最终判断结果, 判断流中是否有*至少一个*元素满足谓词条件

```java
boolean anyMatch = Arrays.stream(arrayOfWords)
        .anyMatch(s -> s.contains("llo"));
```

- allMatch - 终端操作, 返回最终判断结果, 判断流中是否*所有*元素都满足谓词条件
- noneMatch - 终端操作, 返回最终判断结果, 判断流中是否*没有*元素满足谓词条件
- anyMatch, allMatch, noneMatch 三个操作都使用了*短路*方式

#### 查找

- findAny - 找到任意一个满足条件的元素, 可以与filter配合使用并返回Optional对象

```java
Optional<String> worlds = Arrays.stream(arrayOfWords)
        .filter(s -> s.contains("o"))
        .findAny();
```

- Optional - 表示值*可能*存在或*不存在*

```java
Arrays.stream(arrayOfWords)
        .filter(s -> s.contains("o"))
        .findAny()
        .ifPresent(System.out::println);
```

- findFirst - 在有序流中, 返回第一个匹配的元素

### 归约

将流中所有元素结合起来, 在函数式编程语言中, 叫做*折叠*

#### 求和

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int sum = numbers.stream()
        .reduce(0, (s, n) -> s + n); // 15
```

reduce的第一个参数0表示*初始值*, 用lambda表达式表示*折叠操作*, 并返回最终结果

或者使用Java内置的api来进一步简化:

```java
int sumWithAPI = numbers.stream()
        .reduce(0, Integer::sum);
```

#### 最大值和最小值

```java
Optional<Integer> max = numbers.stream()
        .reduce(Integer::max);

Optional<Integer> min = numbers.stream()
        .reduce(Integer::min);
```

#### 使用技巧

```java
String allNames = transactions.stream()
        .map(t -> t.getTrader().getName())
        .sorted()
        .collect(joining()); // 使用joining代替reduce提高效率, 其内部使用StringBuilder实现
        
Optional<Transaction> minTransaction = transactions.stream()
        .min(comparing(Transaction::getValue));

```

### 数值流

在进行求和计算的时候, 需要使用reduce方法来进行处理. Stream上不存在sum接口, 原因是对于Stream<Dish>这样的流, sum方法是没有任何意义的.

#### 原始类型流特化

IntStream, DoubleStream和LongStream, 避免了暗含的装箱成本. 几个接口都带来了常用的*数值规约*方法

特化的原因在于装箱带来的成本, 即int和Integer之间的*效率差异*

- 映射到数值流

```java
int calories = menu.stream()
                   .mapToInt(Dish::getCalories)
                   .sum();
```

- 转换回对象流

```java
Stream<Integer> stream = menu.stream()
                   .mapToInt(Dish::getCalories)
                   .boxed();
```

- 默认值OptionalInt

```java
OptionalInt maxCalories = menu.stream()                                                      
                              .mapToInt(Dish::getCalories)
                              .max();
int max = maxCalories.orElse(0);
```

#### 数值范围

生成指定范围的数值流:

```java
IntStream evenNumbers = IntStream.rangeClosed(1, 100)
                         .filter(n -> n % 2 == 0);
```

### 构建流

- 由值创建流

```java
Stream<String> stream = Stream.of("Java 8", "Lambdas", "In", "Action");
```

- 创建空的流

```java
Stream<String> emptyStream = Stream.empty();
```

- 由数组创建流

```java
Arrays.stream(numbers);
```

- 由文件生成流

```java
 long uniqueWords = Files.lines(Paths.get("data.txt"), Charset.defaultCharset())
                         .flatMap(line -> Arrays.stream(line.split(" ")))
                         .distinct()
                         .count();
```

- 由函数创建流 - 无限流
    - iterator - 有状态的无限流

```java
Stream.iterate(0, n -> n + 2)
      .limit(10)
      .forEach(System.out::println);
```

    - generator - 无状态的无限流

```java
Stream.generate(Math::random)
      .limit(10)
      .forEach(System.out::println);

// stream of 1s with Stream.generate
IntStream.generate(() -> 1)
         .limit(5)
         .forEach(System.out::println);

IntStream.generate(new IntSupplier(){ // 与表达式的区别是可以处理状态保存
    public int getAsInt(){
        return 2;
    }
}).limit(5)
  .forEach(System.out::println);

```


### 总结

- 使用Stream API可以处理复杂的数据处理
- 使用filter, distinct, limit, skip对数据流进行筛选和切片
- 使用map和flatMap对流进行转换和映射
- 使用findAny, findFirst来查找流中的元素
- 使用anyMatch, noneMatch, allMatch结合谓词判断进行匹配
- 使用reduce对流进行合并
- 流操作分为有状态和无状态, filter, map等是无状态的; sorted和distinct是有状态的.
- 流有三种基本的原始类型特化流: IntStream, DoubleStream, LongStream
- 流可以从集合创建, 也可以从值, 数组, 文件以及iterate和generate等特定操作生成

## 使用流收集数据

使用函数式的收集比集合式的更为简洁和明确, 只需要指定*做什么*, 而不需要关心*怎么做*

```java
// before java8

Map<Currency, List<Transaction>> transactionsByCurrencies = new HashMap<>();
for (Transaction transaction : transactions) {
    Currency currency = transaction.getCurrency();
    List<Transaction> transactionsForCurrency = transactionsByCurrencies.get(currency);
    if (transactionsForCurrency == null) {
            transactionsForCurrency = new ArrayList<>();
        transactionsByCurrencies.put(currency, transactionsForCurrency);
    }
    transactionsForCurrency.add(transaction);
}

// java8

Map<Currency, List<Transaction>> transactionsByCurrencies = transactions.stream()
    .collect(groupingBy(Transaction::getCurrency));
```

### 收集器

收集器中的实现, 决定了如何对流进行规约操作.

使用收集器能够更好的*复用和重用*

最经常使用的toList就是一个预定义的收集器: 把流中的元素收集到一个List中

收集器主要包含三大功能:

- 将流元素规约和汇总为一个值
- 元素分组
- 元素分区

#### 归约和汇总

- count - 收集流中的总个数

```java
menu.stream().count();
menu.stream().collect(counting());
```
- maxBy, minBy

```java
Comparator<Dish> comparator = Comparator.comparingInt(Dish::getCalories);
Optional<Dish> mostCalorieDish = menu.stream().collect(maxBy(comparator));
// or
Optional<Dish> mostCalorieDish = menu.stream().max(comparator);
```

- 汇总

```java
int total = menu.stream().collect(summingInt(Dish::getCalories)); // 求和
double avg = menu.stream().collect(averagingInt(Dish::getCalories)); // 平均值
String names = menu.stream().map(Dish::getName).collect(joining(",")); // 连接字符串
int total = menu.stream().collect(reducing(0, Dish::getCalories, (i, j) -> i + j)); // 通用汇总
```

- reduce和collect的区别
    - reduce方法语义上在于合并为一个值, 修改同一个数据结构, 难以并发
    - collect方法语义上是要改变容器, 从而累计要输出的结果, 适合与并行操作
#### 分组

根据一个或多个属性对集合中的项目进行分组

```java
menu.stream().collect(groupingBy(Dish::getType));
```

给groupingBy方法传递一个方法, 这个方法叫做*分类函数*


- 按逻辑条件分类, 而不是简单的属性值

```java
menu.stream().collect(
        groupingBy(dish -> {
            if (dish.getCalories() <= 400) return CaloricLevel.DIET;
            else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
            else return CaloricLevel.FAT;
        } ));
```

- 多级分组

```java
menu.stream().collect(
        groupingBy(Dish::getType,
                groupingBy((Dish dish) -> {
                    if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                    else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                    else return CaloricLevel.FAT;
                } )
        )
);
```

- 按子组收集数据 - 传递给第一个groupingBy的收集器可以是任意类型

```java
menu.stream().collect(groupingBy(Dish::getType, counting())); // {MEAT=2, FISH=3, OTHER=4}

// 将收集器中的结果转换为另一种映射
menu.stream().collect(
        groupingBy(Dish::getType,
                collectingAndThen(
                        reducing((d1, d2) -> d1.getCalories() > d2.getCalories() ? d1 : d2),
                        Optional::get))); // 收集之后调用get方法

// 对子收集器结果进行映射
menu.stream().collect(
        groupingBy(Dish::getType, mapping(
                dish -> { if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
                else return CaloricLevel.FAT; },
                toSet() )));

```

#### 分区

分区是分组的特殊情况, 用一个谓词作为分组条件

```java
Map<Boolean, List<Dish>> partitioned = menu.stream().collect(partitioningBy(Dish::isVegetarian));
```

相对于filter, 分区的好处是可以保留true和false两套列表.

- 多级收集器

```java
Map<Boolean, Map<Dish.Type, List<Dish>>> partitioned = menu.stream()
    .collect(partitioningBy(Dish::isVegetarian, groupingBy(Dish::getType)));
```