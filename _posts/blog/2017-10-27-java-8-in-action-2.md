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

IntStream, DoubleStream和LongStream, 避免了暗含的装箱成本. 几个接口都带来了常用的*数值归约*方法

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

收集器中的实现, 决定了如何对流进行归约操作.

使用收集器能够更好的*复用和重用*

最经常使用的toList就是一个预定义的收集器: 把流中的元素收集到一个List中

收集器主要包含三大功能:

- 将流元素归约和汇总为一个值
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

### 收集器接口

Collector中包含了一系列的方法, 为进行具体的归约操作提供了范本.

同样的, 可以为Collector接口提供自定义的实现, 创建自己的归约操作.

```java
public interface Collector<T, A, R> {

    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    BinaryOperator<A> combiner();
    Function<A, R> finisher();
    Set<Characteristics> characteristics();
}
```

#### 类型定义

- T 是流中要收集的项目的泛型
- A 是累加器的类型, 累加器是在收集过程中用于收集累加结果的对象
- R 是收集得到的结果对象的类型

例如:

```java
public class ToListCollector<T> implements Collector<T, List<T>, List<T>>
```

#### 接口中的方法

- Supplier<A> supplier(); 建立新的结果容器

```java
@Override
public Supplier<List<T>> supplier() {
    return () -> new ArrayList<T>();
}
```

- BiConsumer<A, T> accumulator(); 将元素添加到结果容器, 执行过程中会有两个参数: 当执行到第n个元素的时候, 得到归约结果的`累加器`和`当前元素`

```java
@Override
public BiConsumer<List<T>, T> accumulator() {
    return (list, item) -> list.add(item);
}
```

- Function<A, R> finisher(); 对结果容器进行最终转换, 返回一个累加过程的最后要调用的函数, 将累加器对象转换为整个集合操作的最终结果

```java
@Override
public Function<List<T>, List<T>> finisher() {
    return i -> i;
}
```

- BinaryOperator<A> combiner(); 合并两个结果容器, 提供一个进行归约操作的函数, 当流进行并行处理时, 各个子部分的累加器如何合并

```java
@Override
public BinaryOperator<List<T>> combiner() {
    return (list1, list2) -> {
        list1.addAll(list2);
        return list1;
    };
}
```

- Set<Characteristics> characteristics(); 返回一个不可变的Characteristics集合, 定义了收集器的行为: 流是否可以进行归并操作, 以及可以使用哪些优化

    - CONCURRENT 累加操作可以并行执行, 收集器可以并行归约流, 如果收集器没有标记为UNORDERED, 则只能在无序流上进行并行归约
    - UNORDERED 归约结果不受流中项目的遍历和累计顺序的影响
    - IDENTITY_FINISH 表明完成器方法返回的函数是恒等函数, 可以跳过. 这种情况下, 累加器对象会直接用作归约过程的最终结果

```java
@Override
public Set<Characteristics> characteristics() {
    return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH, CONCURRENT));
}
```

#### 进行自定义收集而不去实现Collector

对于IDENTITY_FINISH的收集操作, collect方法接受三个参数:

- 供应源 
- 累加器
- 组合器

```java
menu.stream().collect(
    ArrayList::new,
    List::add,
    List::addAll
);
```

### 总结

- collect是一个终端操作, 接受的参数是将流中的元素累计到汇总结果的各种方式
- 预定义收集器可以进行归约, 汇总, 分组等操作
- 可以将多个收集器进行组合复用, 进行多级分组, 分区和归约
- 可以实现Collector接口中定义的方法实现自定义收集器

## 并行数据处理与性能

使用Stream流可以简单的将数据集操作转换为`声明式`并行操作

并行流会被划分为多块执行, 而划分的`方式`往往会导致问题.

### 并行流

并行流就是通过parallelStream方法将一个流分割为多块, 在每个线程中处理各个分块的数据, 充分利用多核CPU的计算能力

#### 将顺序流转换为并行流

```java
public static long parallelSum(long n) {
    return Stream.iterate(1L, i -> i + 1).limit(n).parallel().reduce(Long::sum).get();
}
```

并行执行意味着顺序的不确定性: 

```java
Stream.iterate(1, i -> i + 1).limit(9).parallel().forEach(System.out::print);
// output: 623418759
```
在并行流内部是利用ForkJoinPool执行, 默认的线程数量就是处理器数量.

并不是所有的并行流都能带来性能提升, 比如:

```java

    public static long iterativeSum(long n) {
        long result = 0;
        for (long i = 0; i <= n; i++) {
            result += i;
        }
        return result;
    }

    public static long sequentialSum(long n) {
        return Stream.iterate(1L, i -> i + 1).limit(n).reduce(Long::sum).get();
    }

    public static void main(String[] args) {
        System.out.println("Iterative Sum done in: " + measurePerf(ParallelStreams::iterativeSum, 10_000_000L) + " msecs");
        System.out.println("Sequential Sum done in: " + measurePerf(ParallelStreams::sequentialSum, 10_000_000L) + " msecs");
    }

    public static <T, R> long measurePerf(Function<T, R> f, T input) {
        long fastest = Long.MAX_VALUE;
        for (int i = 0; i < 10; i++) {
            long start = System.nanoTime();
            R result = f.apply(input);
            long duration = (System.nanoTime() - start) / 1_000_000;
            if (duration < fastest) fastest = duration;
        }
        return fastest;
    }
```

执行结果:

```
Iterative Sum done in: 5 msecs
Sequential Sum done in: 148 msecs
```

第二个方法要慢很多, 原因在于:

- 方法内部的装箱, 拆箱操作消耗性能
- iterate每次执行都需要依赖前一次的执行结果, 很难真正拆分为独立的小块

如果使用其他API来避免拆箱和装箱操作, 性能会得到很大提升, 这说明合适的`数据结构`可以很大的影响到执行效率: 

```java
    public static long rangedSum(long n) {
        return LongStream.rangeClosed(1, n).reduce(Long::sum).getAsLong();
    }
    
    System.out.println("Range forkJoinSum done in: " + measurePerf(ParallelStreams::rangedSum, 10_000_000L) + " msecs");

    // output: Range forkJoinSum done in: 14 msecs
```

#### 正确的使用并行流

错用并行流的首要原因是算法改变了某些`共享状态`

```java
    public static class Accumulator {
        private long total = 0;

        public void add(long value) {
            total += value;
        }
    }
    public static long sideEffectParallelSum(long n) {
        Accumulator accumulator = new Accumulator();
        LongStream.rangeClosed(1, n).parallel().forEach(accumulator::add);
        return accumulator.total;
    }
```

上述代码进行累加操作, 但是每次执行的结果都不相同, 原因在于forEach过程中, 始终在`竞争`total这个共享状态

```
Result: 24126846328568
Result: 18359895440871
Result: 19559025945863
Result: 20770663312463
Result: 26318400284968
Result: 17731221333150
Result: 15349897897796
Result: 28819573961056
Result: 26025413522058
Result: 20379630589112
```

#### 高效使用并行流

- 如果有疑问, `测量`
- 留意`装箱` - 使用原始类型流来避免这种操作
- 有些操作本身在并行流上的操作就更慢, 比如findFirst和limit
- 考虑流的操作流水线的总计算成本, 单个元素的计算成本越高, 则并行流性能好的可能性越大
- 对于数据量较少的流, 不适合并行流
- 考虑流背后的数据结构是否易于拆解, 比如ArrayList就要比LinkedList更容易拆解, 因为前者不需要遍历就可以平均拆分; 使用range工厂创建的原始类型流也更容易拆解.
- 流自身的特点, 以及流水线中的中间操作修改流的方式
- 考虑终端操作中合并步骤的成本

### 分支/合并框架

分支/合并框架的目的是以`递归`的方式将可以并行的任务拆分为更小的任务, 然后将每个子任务的结果合并起来生成整体结果.

它是ExecutorService接口的一个实现, 把任务分配给线程池中的工作线程.

#### 使用RecursiveTask

要把任务提交到这个池, 必须创建RecursiveTask<T>的一个子类, 其中R是并行化任务(以及所有子任务)产生的**结果类型**

```java
public class ForkJoinSum extends RecursiveTask<Long> {
    @Override
    protected Long compute() {
        return null;
    }
}
```
compute是唯一需要实现的方法, 当中包含**任务拆分逻辑**, 以及无法拆分之后的**单个子任务计算逻辑**


![](/images/2017-11-02-23-07-06.jpg)


```java

public class ForkJoinSum extends RecursiveTask<Long> {

    private static final long THRESHOLD = 10_000;

    // 要求和的数组
    private final long[] numbers;
    private final int start;
    private final int end;

    public ForkJoinSum(long[] numbers) {
        this(numbers, 0, numbers.length);
    }

    private ForkJoinSum(long[] numbers, int start, int end) {
        this.numbers = numbers;
        this.start = 0;
        this.end = numbers.length;
    }

    @Override
    protected Long compute() {

        int length = end - start;

        // 小于拆分阈值时, 执行顺序计算
        if (length <= THRESHOLD) {
            return computeSequentially();
        }
        
        // 拆分任务
        ForkJoinSum leftTask = new ForkJoinSum(numbers, start, start + length / 2);
        // 异步执行新创建的子任务
        leftTask.fork();

        // 创建任务为后一半数组求和
        ForkJoinSum rightTask = new ForkJoinSum(numbers, start + length / 2, end);
        
        // 同步执行第二部分任务
        Long rightResult = rightTask.compute();
        
        // 获取第一个子任务的结果
        Long leftResult = leftTask.join();
        
        return leftResult + rightResult;
    }

    private long computeSequentially() {
        long sum = 0;
        for (int i = start; i < end; i++) {
            sum += numbers[i];
        }

        return sum;
    }
}
```

使用的时候:

```java
long[] numbers = LongStream.rangeClosed(1, n).toArray();
ForkJoinTask<Long> task = new ForkJoinSumCalculator(numbers);
return new ForkJoinPool().invoke(task);
```

#### 使用Fork/Join框架的最佳做法

- 对一个任务调用join方法会**阻塞调用方**, 直到任务作出结果, 所以要在两个任务都**开始之后**才调用
- 不应该在RecursiveTask内部使用invoke方法, 应该始终使用**compute和fork**方法, 只有顺序代码才使用invoke来启动并行计算
- 对子任务调用fork方法可以把它排进ForkJoinPool, 同时调用左右任务的join方法, 效率要**更低**, 因为这样做可以为其中一个子任务**复用当前线程**
- 因为启动了异步任务, 调试Fork/Join框架会比较困难
- 和并行流一样, 不应该理所当然的认为并行更快, 可以考虑: 将IO密集型和CPU密集型操作分为不同任务; Fork/Join框架需要"预热"之后才能被JIT编译器优化(性能测试前跑几遍程序);
- 工作窃取 - Fork/Join框架会使用"工作窃取"技术来分摊多个线程的工作负担, 当线程池中的一个线程处于闲置状态时, 会随机的选择另一个线程, 并从它的工作队列末尾**窃取**一个线程, 以此来达到整体上的平衡.

### Spliterator

Spliterator叫做**可分迭代器**, 和Iterator一样用来遍历数据源, 但是他是为了**并行计算**而设计的

```java
public interface Spliterator<T> {
    boolean tryAdvance(Consumer<? super T> action);
    Spliterator<T> trySplit();
    long estimateSize();
    int characteristics();
}
```

- tryAdvance 遍历数据源, 并执行action方法, 如果还有其他元素则返回true
- trySplit 可以把一些元素**划分出去**, 给另一个Spliterator执行, 二者并行执行
- estimateSize 用来估算剩余元素个数, 有助于均匀划分
- characteristics 方法返回特性, 代表本身特征集的编码
    - ORDERED - 元素有既定的顺序, 子任务的遍历和划分也遵循这一顺序
    - DISTINCT - 元素都是唯一的(equals)
    - SORTED - 元素按照一个预定义的顺序排序
    - SIZED - 由一个已知大小的源建立, 所以estimateSize返回的是精确值
    - NONNULL - 元素不会为null
    - IMMUTABL - 数据源不能修改, 不能添加, 删除或修改任何元素
    - CONCURRE - 可以被其他线程同时修改而无需同步
    - SUBSIZED - 拆分出来的所有自己都是SIZED

#### 拆分过程

拆分的算法是一个**递归过程**, 递归调用trySplit直到他返回null


```java
    class WordCounter {
        private final int counter;
        private final boolean lastSpace;

        public WordCounter(int counter, boolean lastSpace) {
            this.counter = counter;
            this.lastSpace = lastSpace;
        }

        public WordCounter accumulate(Character c) {
            if (Character.isWhitespace(c)) {
                return lastSpace ? this : new WordCounter(counter, true);
            } else {
                return lastSpace ? new WordCounter(counter+1, false) : this;
            }
        }

        public WordCounter combine(WordCounter wordCounter) {
            return new WordCounter(counter + wordCounter.counter, wordCounter.lastSpace);
        }

        public int getCounter() {
            return counter;
        }
    }

    class WordCounterSpliterator implements Spliterator<Character> {

        private final String string;
        private int currentChar = 0;

        private WordCounterSpliterator(String string) {
            this.string = string;
        }

        @Override
        public boolean tryAdvance(Consumer<? super Character> action) {
            // 迭代并执行计算            
            action.accept(string.charAt(currentChar++)); 
            return currentChar < string.length();
        }

        @Override
        public Spliterator<Character> trySplit() {
            int currentSize = string.length() - currentChar;
            
            // 如果剩余字符小于十个, 不进行拆分
            if (currentSize < 10) {
                return null;
            }
            for (int splitPos = currentSize / 2 + currentChar; splitPos < string.length(); splitPos++) {
                if (Character.isWhitespace(string.charAt(splitPos))) {
                    
                    // 拆分一个新的Spliterator
                    Spliterator<Character> spliterator = new WordCounterSpliterator(string.substring(currentChar, splitPos));
                    currentChar = splitPos;
                    return spliterator;
                }
            }
            return null;
        }

        @Override
        public long estimateSize() {
            return string.length() - currentChar;
        }

        @Override
        public int characteristics() {
            
            // 累加当前流的特性
            return ORDERED + SIZED + SUBSIZED + NONNULL + IMMUTABLE;
        }
    }
```

### 总结

- **内部迭代**可以并行的处理一个流, 而无需显示的使用线程
- 选择并行的时候一定要进行**测量**, 并行执行的效率很可能会违反直觉
- **使用原始流**而不是一般化的流, 可能比并行或修改算法带来的收益更大
- Fork/Join框架使用**递归方式**将并行任务拆分为更小的任务, 在**不同线程**上执行并最终将各个任务的结果**合并**
- Spliterator定义了流**如何拆分**他要遍历的数据