---
layout: post
title:  "读书笔记：重构——改善既有代码设计（二）"
date:   2017-09-07 00:56:02
categories: 读书笔记
---


# 重新组织函数

## Extract Method

*将代码提取到独立到函数，用名称来解释用途*

*长度不是关键，重要到是函数名称和函数本体之间到语义差距*

### 优点： 
- 更容易复用
- 名称能够表明意图
- 函数更容易被重写

### 难点
- 变量的处理，如果只存在于被提炼的代码，移动到方法内
- 是否有变量的值被改变，如果存在一个值被修改，作为参数传入
- 如果返回的变量不止一个，可以考虑封装为值对象

## Inline Method
如果函数的本体于其名称一样清晰，应该使用函数本体内容，以避免不必要的复杂度

### 场景
- 内部代码和函数名同样清晰
- 有一群不合理的函数，需要移动到其他类中，内联之后移动更为容易
- 使用的太多的间接层

## Inline Temp
如果存在一个变量，植被简单表达式赋值了一次，而他妨碍了其他重构手法
- Inline Temp 经常作为 Replace Temp with Query的前置动作

*做法：先将变量声明为final，借助编译器检查是否真的只赋值了一次*

## Replace Temp with Query

### 动机
- 避免重复的计算表达式
- 可以复用
- 更容易进行内部优化
### 难点
如果变量被赋值多次，首先尝试Split Temporary Variable和Separate Query from Modifier

## Introduce Explaining Variable
使用变量保存复杂的表达式，用来说明表达式的意图
*优先进行Extract Method，如果有困难（存在大量局部变量或变量赋值）再考虑Introduce Explaining Variable*

## Split Temporary Variabal

同一个变量被赋值两次，用作不同的用途，针对每次赋值，使用不同的临时变量

两个特例：
- 循环变量 例如：for循环中的 i,j,k
- 集用变量 例如：sum

## Remove Assignments to Parameters

如果方法内对传入参数进行操作，用一个临时变量代替该参数的操作

动机：避免值修改、引用修改带来的歧义

## Replace Method with Method Object

如果一个大型方法中包含太多局部变量，导致无法进行方法提取；将这个函数放进一个新建对象中，所有局部变量作为内部属性，然后再内部进行Extract Method

## Substitute Algorihm 替换你的算法

把某个算法替换为更清晰的算法

# 在对象之间搬迁特性

## Move Method

某个方法更多的依赖另一个类的内容，将其移动到另一个类中

### 难点
对于方法内依赖的field，存在几种方案：
1. 将这个field也搬迁过去
2. 建立一个目标类到源类的引用
3. 将源对象作为参数传递
4. 将这个变量作为参数传递

## Move Field
某个field被另一个类更多的用到，将其移动到更关心他的类，将源对象中的引用，修改为目标对象的方法调用
如果源类中存在很多对field的引用，使用Self Encapsulate Field提取为方法，然后将在方法内切换到目标类的调用

## Extract Class
某个类职责过多，根据职责拆分为多个

根据功能创建一个新类，通过Move Field和Move Method进行搬迁，如果搬迁之后存在过多的双向引用，尝试修改为单向

## Inline Class
如果类集合没有太多的功能，尝试进行合并，减少不必要的复杂度

### 做法
- 目标类中的复制所有接口，首先委托到另一个类
- 客户端代码搬迁到目标类的接口调用
- 移动方法内部实现

## Hide Delegate
根据迪米特法则，客户端应该了解尽可能少的内容

### 做法
- 对于委托关系中的函数，建立一个简单的委托函数
- 调整客户端，让他只调用server的函数（不要跳过委托调用下一层）

## Remove Middle Man

与上一个手法想法，如果存在太多的委托函数，不妨直接调用

## Introduce Foreign Method

如果需要使用一个额外的函数，但是又无法修改这个class的内容。在调用端建立一个新函数，封装值得封装的使用方式。

## Introduce Local Extension

上一个手法的扩展，如果存在大量的Foreign Method，不妨新建一个扩展类，组织起这些封装或扩展

存在两种方式：
- 继承
- 组合（或装饰）

# 重新组织数据

## Self Encapsulate Field 
直接访问值域，但是与值域之间的耦合关系变得笨拙；将其切换到getter/setter函数的调用

- 好处是子类可以通过复写来修改获取数据的方式
- 缺点是代码不易读

解决方式是：优先使用直接访问，直到需要改变的时候

## Replace Data Value with Object

有一笔数据项带有额外的数据和行为，将这笔数据项变成一个对象，可以有更多的空间保存额外的信息和行为。

## Change Value to Reference

一个类衍生出很多相等实体，可以替换为单一对象。

### 方法
1. 创建一个简单工厂方法，将new实例化切换到方法的调用
2. 将目标类的构造函数私有化
3. 在工厂内部缓存对象

## Change Reference to Value
在引用对象和值对象之间切换，会随时发生变化，需要回头路

Value Object 的优势是不可变，在分布式系统和并发系统中非常有优势。

## Replace Array with Object
数组中的内容各自代表了不同的东西，则将其替换为对象，并加上相关的行为。

## Duplicate Observed Data
当ui界面和业务逻辑混在一起时，可以分离为两个类，使用观察者模式监听业务对象的变化，保持ui界面的同步。

## Change Unidirectional Association to Bidirectional

两个class都需要使用对方的数据，则在两者之间建立相互的引用。
- 在一对多场景下，使用单一的一方来控制关联
- 如果存在明显的组成关系，容器类负责控制关联
- 多对多关系下，任意哪个都可以
*在setter中需要更新反向指针，需要添加getter函数的测试*

## Change Bidirectional Association to Unidirectional
去掉不必要的关联

*双向关联的代价：需要维护双向连接，确保对象被正确的创建和删除*
*移除时，不仅要考察直接读取点，还要考察直接读取点的调用函数*

## Replace Magic Number with Symbolic Constant
*使用敞亮，根据其意义进行命名*

## Encapsulate Field
将不必要的public值域修改为private，使用getter/setter函数进行代替

## Encapsulate Collection
如果有函数直接返回Collection；让函数返回一个只读副本，添加维护函数进行操作，避免客户端代码的错误修改。
*不应该提供setter函数*

## Replace Record with Data Class

新建一个Class，代替record，提供getter/setter方法

## Replace Type Code with Class
将数值型的状态码替换为class/enum

- 先在Class中支持整数形式，切换引用端代码
- 测试通过后，再将整数类型修改为对象类型的引用
## Replace Type Code with Subclasses
类型会影响行为时，使用子类

- 是Replace Conditional with Polymorphism的基础
- 为每一种类型创建一个子类
- 使用简单工厂返回合适的子类

## 两种情况不能进行：
1. type code值在创建之后发生了变化
2. 已经有其他子类
此时需要使用Replace Type Code with State/Strategy

## Replace Type Code with State/Strategy
无法使用subclass时，以state object（专门用来描述状态的对象）取代type code

### 过程
- 将type code在内部封装
- 新建一个class，根据用途命名
- 针对每种子类型，新建一个子类，在子类中覆盖getType方法
- 在source class中新增对类型对象的引用
- 将typecode相关的逻辑，搬迁到state object
- 调整为type code复制的函数，将一个恰当的subclass赋值进去

### 选择哪种模式取决于后续动作
- 重构后，简化一个算法，使用strategy
- 重构后，搬迁状态数据，使用state

## Replace Subclass with Fields
如果subclass的差别只是常量数据，销毁subclass，将数据作为值域
