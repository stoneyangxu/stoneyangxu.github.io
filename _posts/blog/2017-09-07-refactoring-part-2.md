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


# 简化条件表达式

## Decompose Conditional

有一个复杂的条件表达式，从if／then／else中分别提取为独立函数。

复杂的条件逻辑是最常导致复杂度上升的地点之一

## Consolidate Conditional Expression （合并条件表达式）
有一系列分支拥有相同的结果，将分支进行合并，将条件表达式提取为函数。

*注意：这些表达式没有副作用*

## Consolidate Duplicate Conditional Fragments
多个条件分支拥有相同的代码，提取到if/else之外；让代码清晰表达哪些代码改变，哪些不变

## Remove Control Flag（移除控制标记）
使用break或return取代控制标记

## Replace Nested Conditional with Guard Clauses

条件逻辑难以看清正常的执行路径，使用卫语句表现特殊情况。

*条件分支中，只有一条正确路径，其他都是不常见路径*

## Replace Conditional with Polymorphim
条件式中，根据不同的对象类别提供不同的行为；将每个分支放入subclass中，将原始函数声明为抽象函数。

利用现有的或新建继承结构

## Introduce Null Object

将null value替换为null object，避免再三检查对象是否为null。


## Introduce Assertion
以断言明确表示某种假设

# 简化函数调用

## Rename Mthod
以便更好的表达意图。

## Add Parameter
某个函数需要从调用端得到更多信息，为此函数添加*对象参数*，让对象带进所需信息。

## Rename Parameter
删除不再需要的参数

## Separate Query from Modifier
如果某个函数既返回状态值，又修改对象状态；将其分隔为两个函数，完成各自的职责。

## Parameterize Method
若干函数做了类似的工作，但是在函数本体中包含了不同的值；建立单一函数，以参数表达不同的值；以此去除重复代码并提高灵活性。

## Replace Parameter with Explicit Methods
函数内部的行为完全取决于不同的参数值；将其分离为意图明确的不同函数。

## Preserve Whole Object
如果从对象中取出若干值，将他们作为某一次函数调用时的参数。

该用对象作为参数，当增加或者减少参数时，可以在对象和函数内部处理，避免散弹式修改。

*手法：先传入整个对象，修改引用后移除值参数*

## Replace Parameter with Method
对象调用某个函数，并将结果作为参数，传递给另一个函数。而接受该参数的函数也可以调用前一个函数。

让参数接受者去除该项参数，并直接调用前一个函数。

## 引入参数对象

某些参数总是成组出现，将其提取为一个参数对象。

*手法：新增对象后，先传入对象，再修改引用，然后在删除值参数*

## Remove Setting Method
移除无用的set函数

## Hide Method
没有外部使用的方法，修改为private

## Replace Constructor with Factory Method

如果希望在构造对象时，不仅仅是对他做简单的构造动作；使用工厂函数代替构造函数
*结合状态码进行多态创建*

## Encapsulate Downcase
如果需要进行向下转型动作，将向下转型动作隐藏在函数内，减少向下转型带来的负面影响。

## Replace Error Code with Exception
避免使用错误码表达异常。调用者在发现异常时，可能并不知道如何处理，不如采用异常将错误信息传递上去。

## Replace Exception with Test
面对一个调用者可以预先加以检查的异常，使调用者在调用函数之前进行检查。

异常只是罕见的、难以预料的情况，不应该成为条件检查的替代品。

# 处理概括关系

## Pull Up Field
两个子类拥有相同值域，提取到父类

## Pull Up Method
将子类中重复的方法提取到父类

## Pull Up Constructor Body
子类中的构造函数基本一致，提取到父类

## Pull Down Method
父类中某个方法只于部分子类有关，下沉到相关到子类中

## Pull Down Field
父类中某些值，只被部分子类用到，将其下沉到相关子类中

## Extract Subclass
class中某些特性只被部分子类用到，提炼一个子类。

## Extract Superclass
两个类有相似特性，为两个类建立一个父类，相同特性进行提取

## Extract Interface
若干客户使用class接口中的同一子类，或者有相同的接口
提取为一个独立的接口，使系统的用法更清晰，同时也更容易看清楚系统的责任划分。

## Collapse Hierarchy
父类与子类无太大区别，合并为一体。

避免继承带来的复杂度。

## Form Template Method
子类中的执行顺序相同，只是其中某些步骤不同。
将差异过程提取为函数，将公共部分上移到父类。

## Replace Inheritance with Delegation

父类只使用子类的部分接口，或者根本不需要继承来的数据。
使用组合代替继承。

## Replace Delegation with Inheritance
委托关系过多时，使用继承代替委托

# 大型重构

## Tease Apart Inheritance 梳理并分解继承体系
某个继承体系承担了多项职责，分离为多个，使用委托关联。

## Convert Procedural Design to Objects
将数据记录变成对象，将行为分开，并将行为移动到对象内。

## Separate Domain from Presentation
将业务逻辑与展现分离，建立独立的业务类。

## Extract Hierarchy
某个类承担过多职责，通过继承体系分隔子类的特殊情况

# 安全重构
- 相信你的编码功力
- 相信你的编译器能捕捉遗漏的错误
- 相信测试可以捕捉你和编译器遗漏的错误
- 相信代码审核


![](/images/2017-09-07-23-50-41.jpg)


![](/images/2017-09-07-23-50-57.jpg)
