---
layout: post
title:  "读书笔记：重构——改善既有代码设计（一）"
date:   2017-09-06 02:21:45
categories: 反思
---

# 重构

如果发现需要为程序添加一个特性，而代码结构使你无法很方便的那么做，那就先重构那个程序，使特性的添加比较容易进行，然后再添加特性。

重构之前，首先检查自己是否有一套可靠的测试机制。这些测试必须有自我检验能力。

重构的第一步往往是找出代码的逻辑泥团并运用Extract Method。

提取方法时小心变量，特别是修改过的变量，如果只有一个变量可以作为返回值。

重构技术要以微笑的步伐修改程序，如果你犯下错误，可以很容易的发现它。

任何一个傻瓜都能写出计算机可以理解的代码，唯有写出人类容易理解的代码，才是优秀的程序员。

# 重构概述

- 重构可能为了可读性和可维护性而牺牲性能，除非明确的有性能要求，否则以可读性优先。

- 重构的概念：对软件内部结构的一种调整，目的是在不改变软件可察行为的前提下，提高其可理解性，降低其修改成本

- 重构的目的：使软件更容易被理解和修改

- 重构的时机：事不过三，三则重构
1. 新增特性时
2. 修改功能时
3. 代码检视时

## 计算机科学是这样一门科学，它相信所有问题都可以通过多一个简介层来解决。

间接层的好处：
1. 允许逻辑共享
2. 分开意图和逻辑
3. 将变化隔离
4. 将条件逻辑加以编码

## 重构的困难

学习新技术时，过程中往往处在一个特定环境内，导致难以察觉其不适用场景。

### 数据库

对象与数据库结构紧密耦合

解决方法是在*对象模型*和*数据模型*之间增加一个分隔层（在系统变得不稳定时再加入）

分隔层会增加系统复杂度，但是带来更大的灵活性

### 修改接口

如果修改接口，一切都可能被改变

方法：

- 如果是已发布接口，必须同时维护两个接口，直到所有用户完成切换，旧接口委托到新接口上

- 不要过早发布接口

### 难以通过重构手法完成的设计改动

例如：将某个无安全需求下构建的系统，重构为安全性良好的系统

## 何时不该重构

- 代码无法运作
- 项目已经接近最后期限
## 重构与设计

重构改变了预先过度设计的困难，依然进行设计，但是只需要得到一个足够合理的解决方案就足够了。
重构通过持续改进代码设计，最终演变为合适的设计方案。
降低了设计的难度，减轻了设计的压力，避免了不必要的灵活性和复杂度。

# 代码坏味道

## Duplicated Code 重复代码 
- Extract Method 
- 子类之间重复，在Extract Method之后，Pull Up Method
- 获取可以发现运用Form Template Method获取一个Template Method设计模式
- 如果两个函数以不同算法做相同的事情，考虑Substitute Algorithm，替换为较为简单的算法
- 如果两个不相关的类出现重复，可以考虑Extract Class
## Long Method 过长方法
- 每当感觉需要注释来说明什么的时候，就可以提取为方法
- 提取方法时遇到过多的参数，可以考虑Replace Temp with Query, Introduce Parameter Object, Preserve Whole Object来缩短参数列表
- 条件表达式可以用Decompose Conditional
- 循环可以将循环和其内的代码提取到独立方法

## Large Class 过大的类
- 运用Extract Class或者Extract Subclass提取彼此相关的变量和方法
- 从客户端使用的角度，通过Extract Interface将类按照职责分解
- 涉及ui相关的对象，运用Duplicate Observed Object将数据和行为提取到独立的领域对象

## Long Parameter List 过长参数列

- 传递给他足够的东西，让他能够从中获取到自己需要的东西就可以了
- 如果调用一次方法就可以获取参数，那么Replace Parameter with Method
- 使用Preserve Whole Object将来自同一个对象的一堆数据收集起来，并使用对象替代
- 缺乏对象归属的参数，可以使用Introduce Parameter Object制造出一个对象
## Divergent Change 发散式变化
- 一个类因为多种不同的变化被修改
- 针对每种变化都应该只修改一处
- 根据变化的类型，通过Extract Class提取到另一个class中
## Shortgun Surgery 散弹式修改
- 一个修改引发多个位置的变动
- 针对变化的规律，使用Move Field和Move Method将变化集中
## Feature Envy 依恋情节
- 通过Move Method 将函数移动到合适到位置
## Data Clumps 数据泥团
- 运用Extract Class集中管理值变量
- 提取新对象后，可以动手处理Feature Envy
## Primitive Obsession 基本型别偏执
- 使用Replace Data Value with Object，使用对象表达其含义
- 类型信息可以使用Replace Type Code with Class
- 如果有行为依赖于类型判断，可以使用Replace Type Code with Class或者Replace Type Code with State/Strategy
## Switch Statement
- 优先考虑多态进行替换
- 使用Replace Type Code with Class或者Replace Type Code with State/Strategy
- 进一步使用Replace Conditional with Polymorphism优化调用端的逻辑
- 如果判断条件之一是null，可以考虑Introduce Null Object
## Parallel Inheritance Hierarchies 平行继承结构
- 当为某个class增加一个subclass, 必须为另一个class也增加一个subclass
- 方法是让一个继承体系的实体，引用另一个继承体系的实体
## Lazy Class 冗余类
- 使用Collapse Hierarchy消除多余的继承关系
- 使用Inline Class消除无用的类
## Speculative Generality 夸夸其谈的未来性
- 使用Collapse Hierarchy/Inline Class/Remove Parameter/Remove Method消除多余的类、参数或方法
## Temporary Field 令人迷惑的暂时值域
- 某个变量只为了某种特定情形设定
- 使用Extract Class提取变量和所有相关的处理
- 使用Introduce Null Object避免条件式代码
## Message Chains 过度耦合的消息链
- 先使用Extract method提取消息链的获取，观察他的用途
- 再使用Move Method将方法打入消息链
## Middle Man 中间人转手
- 如果有一半的函数都通过中间人转手，使用Remove Middle Man消除中间人
## Inappropriate Intimacy
- 两个对象过度亲密，彼此探寻对方的private
- 使用Move Method 或者 Move Field 让他们划清界限
- 运用Change Bidirectional Association to Unidirectional
- 可以使用Extract class将密切关系提取到一个class
- 使用Replace Inheritance with Delegation消除过度亲密的继承关系
## Alernative Classes with Different Interfaces 异曲同工的类
- 相同的实现，不同的命名
- 通过Rename Method修正名称，并通过Move Method合并职责
## Incomplete Library Class 不完美的程序库类
- 无法满足需要，又不能修改类库
- 使用Introduce Foreign Method或者Introduce Local Extension
## Data Class 幼稚的数据类
- 尝试使用Move Method将与数据紧密耦合的类移动到Data Class当中
## Refused Bequest 被拒绝的馈赠
- 意味着继承体系的设计错误
- 使用Push Down Method或Push Down Field将多余的函数推送给子类
## Comments 过多的注释
- 当你感觉需要编写注释的时候，先尝试重构，试着让所有的注释都变得多余

# 构筑测试体系
 *编写优良的测试程序，可以极大的提高编程速度*

 *确保所有测试都自动化，让他们检查自己的测试结果*

 *除非你确切体验到这种方法带来的速度提升，否则自我测试就显示不出它的意义*

 *每当你获得一个bug报告，优先编写一个测试来重现它*

 *编写尚未完善的测试并实际运行，好过对完美测试的无尽等待*

 *考虑可能出错的边界条件，把测试火力集中供暖在那*
