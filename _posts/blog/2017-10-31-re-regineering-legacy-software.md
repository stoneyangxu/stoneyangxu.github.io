---
layout: post
title:  "遗留系统重建实战"
date:   2017-10-31 23:39:12
categories: 读书笔记
---

> 遗留系统似乎是一个禁忌话题, 开发者想喜鹊一样, 总是寻找下一个闪光的新玩具来娱乐自己, 很多开发人员已经放弃了任何改进遗留软件和让它更易于维护的尝试.

## 目标: 将一个被忽视的老旧代码库转变为一个可维护的, 功能良好的, 能为组织产生价值的软件所需要做的一切事情

## 产生遗留资产的原因有很多, 大多数原因与人有关, 而非技术

- 人员流动导致的信息丢失
- 技术债务累计到不可持续的水平
- ......

## 了解遗留项目中的挑战

> 开发人员在现有代码上花费的时间要远远多于写新代码, 绝大多数开发人员都不得不处理某种形式的遗留项目

### 遗留项目的定义

> 任何已经存在的, 难以维护或难以扩展的项目

遗留项目的特征

- 老旧 - 复杂的项目通常都需要几年的积累, 几代人的维护, 交接过程中遗漏了大量的信息
- 庞大 - 项目越大越难维护, 需要理解的代码越多, 修改的风险越大
- 继承而来 - 从之前的开发或团队继承而来, 只能被迫猜测当时的意图
- 文档不完善 - 没有文档, 即使有也无法完全信任

> 例外: Linux内核, 源于它公开和坦诚沟通的文化, 以及对代码评审的重视

### 遗留代码

#### 没有测试和无法测试的代码

测试往往是我们寻找*系统行为*和*设计假设*的线索, 好的测试套件可以成为*事实上的文档*.

很多遗留项目*没有测试*, 而且从未考虑过如何测试, 后期想要重新添加测试变得十分困难

包括:

- 硬编码: 路径, url等等
- 外部依赖: 数据库, 文件系统, 远程服务等等
- 未知的意图
- 庞大的体积
- ... ...

#### 不灵活的代码

在遗留代码上*实现新功能*或*修改现有行为*会变得非常困难, 每个很小的改动会涉及*很多地方的修改*, 每个修改都要进行测试, 而且往往是*手工测试*

#### 被技术债务拖累的代码

每个开发人员都会写一些*明知不够完善*, 但是*恰好够用*的代码, 这往往是正确的, 但是每次完成一个"刚刚好"的时候, 应该*计划*一下, 何时进行*重新审视和重构*

*债务*通常用来隐喻质量问题的积累, "快速的"解决方案类似贷款, 必须偿还*利息*, 而且在偿还之前, *利息会继续产生利息*. 当需要偿还的利息超过支付能力的时候, 有用的工作就会陷入*停滞*

### 遗留基础设施

#### 开发环境

在一个空白的机器上安装本地开发环境需要多久? 本地能否运行系统? 每次功能调测需要因环境问题花费多少时间?

- 团队的*每个*开发人员都需要这样做, 浪费的时间*累加*来就相当可观
- 开发环境越容易搭建, 愿意参与到项目中的人就越多

#### 过时的依赖

外部依赖的升级往往能带来*功能增强*, *性能改善*, *bug修复*, *安全补丁*, 以及*开发效率提升*等诸多好处.

*定期*进行检查和更新, 只需要很少的投入和代价. 一旦版本差异积累到一定程度, 切换带来的工作量, 风险都会大大增加.

#### 异构环境

项目会在不同的环境上运行: 
- 本地开发环境
- 测试环境
- 预生产环境
- 生产环境

在各个环境上进行功能验证的有效性, 取决于各种环境之间的*差异度*, 环境之间差异度越低, 验证的有效性越高.

### 遗留文化

在开发方式和写作方式上的共同特征

#### 害怕变化

遗留项目都非常复杂, 而且缺乏文档, 当前的项目负责人并不清楚变化所带来的影响和风险. 

- 哪些功能已经不再使用?
- 哪些bug可以被安全的修复?
- 改动软件行为之前需要咨询哪些用户?

安全起见最好是*保持现状*, *任何*变更都被视为纯粹的风险, 而忽略了那些可能带来的好处.

为了避免风险而拒绝演进, 这会将组织暴露在另一个巨大的风险面前, *被竞争对手抛在身后*. 这是*短期利益与长期利益之间的冲突, 也是个人利益与组织利益之间的矛盾*

极端的做法不可取, 更平衡的做法是评估每个变更的*风险和收益*, 积极寻找有助于做出觉得的*缺失信息*, 最终决策应当采取的*行动,* 保持持续的*演进和适应变化*


#### 知识仓库

在遗留系统上面工作时, 开发人员面对的困难往往是*知识的缺乏*:

- 用户需求与软件功能相关的*领域信息*
- 关于软件设计, 架构, 和内部的*特定项目信息*
- *通用的技术知识*, 如算法, 语言特性, 编码技巧, 类库等等

团队的好处就是拥有队友的帮助, 可以询问他们, 也可以接受他们主动的分享. 但是这种沟通和分享在很多团队中没有发生: 
- 缺乏面对面的沟通
- 代码是我的
- 忙碌的面孔

- - - - -
似乎想要解决所有问题, 就必须*消减工作量*. 好在不必一次性解决所有问题, 可以*一步步的重整*遗留项目
- - - - -

## 找到起点

### 克服恐惧和沮丧

对遗留系统抱有一些负面情绪是正常的, 但是这些负面情绪非常具有*破坏性*, 因为他们会影响我们的*判断*, 阻止我们有效的进行改进.

- 每当修改一行代码, 都会破坏很多不相关的东西, 代码实在太脆弱了
- 如果没有必要, 绝对不要碰任何相关的代码

对于遗留系统, 恐惧会导致我们下意识的进行*防御式编程*, 并对大的更改变得更加*抵制*.

对于遗留系统的恐惧, 往往是源自于*未知*, 大片不能理解的代码, 大量未知的功能场景, 无法预测的影响和风险

对此, 我们需要使用一些*工具和技术*, 帮助我们以*客观*的方法来*接近*重构.

尝试进行*探索性重构*: 

- 尝试重命名
- 在两个类之间移动方法
- 引入新的接口
- 添加注释
- 以及任何让代码更整洁可读的工作

在这个过程中不仅能够增加对代码的理解, 而且会使得代码更容易维护, 在一些帮助下, 是非常安全的:

- 版本控制系统 - 快速的回滚
- IDE - 安全而且快速的执行标准重构
- 编译器 - 编译时获得快速的反馈
- 其他开发人员 - 代码评审或者结对编程

另一个补充方案是添加*特征测试*, 目标是描述系统的*实际行为*, 当涉及遗留代码的时候, *保留现有行为*是最重要的目标.

#### 沮丧

在遗留代码上工作真的让人沮丧, 任何简单的修复都涉及大量的更改, 添加一个简单的功能要耗费几天的时间, 是否继续进行*粘贴复制*, 这些问题会导致真正的*精神压力*. 而且往往导致两种后果: *放弃重构*或者*孤注一掷*

即使选择重构, 并且获得了成功, 又有*多大的差别?* 重构*有价值吗*, 距离目标*还有多远*.

对此, 需要有一种方式*提醒*自己: 我们在重构上的努力是有价值的, 通过选择一个或多个表示代码质量的*指标*:

- 显示软件质量以及质量是如何随时间变化的
- 决定我们下一个重构的目标是什么



### 收集软件有用的数据

- bug以及编码标准违例
- 性能 - 整体的性能状况以及性能热点
- 错误计数 - 生产环境上发生的错误数量
- 常见的任务计时 - 包括搭建开发环境, 发布或部署项目, 修复一个bug的平均时间等
- 常用文件 - 通过版本控制系统找到修改频繁的文件
- 度量可度量的一切 - 拥有过多的信息好过没有信息, 有疑问就度量它

### 用FindBugs, PMD和Checkstyle审查代码库

- FindBugs用来找到潜在的bug
- PMD用来寻找技术上正确, 但是没有遵循最佳实践, 例如耦合度过高的代码
- Checkstyle用来检查编码标准

按照FindBugs -> PMD -> Checkstyle的顺序来执行检查, 按照重要性来修复问题.

在使用多种工具的时候, 需要自定义规则集并消除重复检查项.

### 用Jenkins进行持续审查

- 检查过程不应该是手动进行的, 开发人员难以保证定期执行检查
- 通过Jenkins这类工具自动跟踪跟踪代码检入, 自动运行检视工具, 在仪表盘上显示结果
- 使用Jenkins自动化进行其他的一些工作:
    - 单元测试
    - 端到端测试
    - 生成文档
    - 软件部署
    - 发布到maven仓库
    - 多级性能测试
    - 构建和发布自动生成的API客户端
- 将Jenkins作为团队成员*工作流程代码化*的地方


## 准备重构

> 重构时要始终记得组织目标

### 达成团队共识

*整个团队*应该共同进行代码更改, 评审彼此的工作并分享重构中收获的信息.

每个人都需要理解项目意义, 重构的目标和计划

#### 传统主义者 - 反对任何形式的变化

也许是因为在遗留系统上的错误修改让他们频繁吃亏, 也许是他们认为"真正"的工作只有修改bug和添加功能, 传统主义者比任何人更不喜欢在遗留系统上工作, 但是认为重构会带来不可预知的风险.

对于传统主义者:

- 结对编程 - 让传统主义者看到重构带来的好处, 以及重构过程中如何控制风险
- 解释技术债务 - 每次快速而取巧的实现, 每次复制粘贴代码, 每次只能满足当前要求的bug修复, 都是一个新的技术债务, 增加团队所需要支付的"利息"; 使用数据来证明消除技术债务能够带来的好处, bug修复时长以及功能的可扩展性

#### 反传统主义者 - 进行流氓重构的破坏者

厌恶遗留代码, 无法忍受不整洁的代码, 热衷于提高代码质量, 到处重写代码. 不仅引入大量的bug, 也会伤害现有的代码拥有者.

需要将他们引导到更有用的活动当中:
- 代码评审 - 拒绝过大的改动, 拆分为更容易管理的代码块
- 自动化测试 - 改动必须有自动化测试
- 结对编程 - 引导他们到真正需要的地方
- 划定代码区域 - 由团队来提供最有价值, 风险最小的代码区域

#### 一切都在于沟通

良好的沟通有助于团队内达成一致的共识, 并且有效的进行知识传递

- 代码评审 - 避免风险和分享知识的机会
- 结对编程 - 实时交流是更加不受限制的交流机会, 但是效果因人而异, 最好将决定权交给具体的开发人员
- 特殊活动 - 技术讲座, 黑客马拉松, 下午茶......

### 获得组织批准

#### 使它变得正式

重构很容易让人觉得是可以轻松完成的: 只是在调整已经写好的代码而已, 不应该花费那么多时间, 一次只需要做一点, 任何有一个小时的空闲时间都可以.

*实际并不是这样*

虽然每天进行了一点调整, 让代码保持良好的状态, 但是这种做法*很难扩展到更大规模的改进*, 进行重大的改进是需要*专门的时间和资源*的.

需要让组织者理解, 重构是能够带来价值的, 它符合组织的利益.

需要让利益相关方理解, 重构时能够带来*长远效益*的, 而且从中识别出*合理的业务价值*, 例如性能优化, 技术能力, 可扩展性等等.

#### 备用计划: 神秘的20%

如果重构可以由一个开发在一周时间内完成, 说明他足够小, 如果走正式的流程会太过繁琐, 可以使用*神秘的20%计划*:

- 开始重构, 无需授权
- 一次只做一点点, 不超过20%的工作时间
- 一旦有了值得分享的结果, 就可以揭开秘密
- 根据组织的反馈改进工作, 知道团队对质量满意


### 选择重构目标

重构目标的选择需要考虑三个维度:

- 价值 - 能够给产品或组织带来多大的价值
- 难度 - 执行重构的需要花费多大的代价
- 风险 - 重构带来的连锁反应有多大


### 重构还是重写?

用重构的方法能将代码质量恢复到合理的水平么?

抛弃现有代码重新实现会更快, 更容易吗?

以及, 第三种解决方案: 用第三方解决方案, 商业的或者开源的

#### 不应该重写的情况

完全重写*一定是个坏主意*

- 风险 - 遗留代码中包含了多年积累下来的业务场景和bug修复, 重写很可能遗漏; 重写的时间难以估计, 而且可能引入大量的bug, 甚至在进行到一半的时候才发现方案不可行
- 开销 - 团队经常*低估*重写带来的工作量, 除代码本身之外, 还有很多周边的工作需要完成. 作为一个*折中*的方案, 可以从遗留系统中进行部分迁移, 通过技术手段进行适配或兼容, 最重要的是一定要有*计划去跟踪和清除*这些临时措施
- 超期 - 重写经常会超期完成, 除了已经提过的风险之外, 还需要在重写过程中始终保持新老代码的同步, 而且重写一旦开始便不能停止, 否则一切努力都会白费

#### 从头重写的好处
- 自由 - 在遗留代码上工作, 很容易陷入到现有的复杂逻辑当中, 重写的过程中, 更容易避免被现有代码过度影响
- 可测试性 - 可以在一开始就将可测试性考虑到设计当中

#### 重写的必要条件

将重构作为*默认手段*, 除非:

- 尝试过重构但是失败了 - 即使重构失败, 在这个过程中能够加深理解, 而且重构的结果也是能够被充分利用的
- 编程范性的转变 - 新的技术框架, 新的编程语言

### 第三种方式: 增量重写

重构伴随着太多的负担, 完全重写承担了太多的风险.

折中的方式是将重写划分为*若干小的阶段*: 
- 每个阶段都能提供*业务价值*
- 可以在任何阶段之后*停止项目*, 而且仍然能带来一些好处

> "绞杀者模式": 新系统围绕着旧系统建立, 并且截取旧系统的输入和输出, 逐步承担更多的职责, 直到旧系统静静的消亡


## 重构

### 有纪律的重构

当记录被正确执行的时候, 重构应该是*完全安全*的. 

- 将重构与其他工作分开 - 不要假设自己可以安全的完成修改和重构, 先修改再重构, 或者反过来
- 依靠IDE - 它们可以安全快速的执行重构, 但是仍然不能放弃人工的核查
- 依靠版本控制系统 - 重构往往是探索性和持续性工作, 利用回退和分支功能来保证安全
- Mikado方法 - 将大型任务拆分, 通过*探索*的方式构建一个*依赖图*, 包含所有的小任务, 以最佳的*顺序*来执行这些任务

> Mikado方法提出了简单的解决方案。当你进行重构时，一旦发现某些依赖的部分出错了，则画一张图表来表示这些错误，并标明真正去修复错误之前，需要先做什么事情。然后还原你的改动，开始观察那张依赖图中的某个叶子结点。修复那个错误，看它是否会引起更多的问题——如果不会，重复这个过程——继续对剩余部分进行重构并在图中画出更多的叶子、还原你的改动，并再从某个叶子结点开始修复工作。

> [用Mikado方法重构遗留软件](http://www.infoq.com/cn/news/2012/03/mikado-method/)

> [The.Mikado.Method(2014.2)](https://pan.baidu.com/share/link?uk=2214641459&shareid=1783280597#list/path=%2F)


### 常见的遗留代码的特征和重构

- 陈旧代码 - 已经不再需要的东西, 移除它们
- 有毒的测试 - 没有测试任何东西, 很容易失败的测试, 随机失败的测试; 修复 -> 禁用 -> 删除或重写
- 过多的null - 利用编程语言的Optional机制, 或者用注解来标注可能为null的位置
- 不必要的可变状态 - 将不可变作为默认设计
    - 所有字段标记为final
    - 使用构造器来初始化所有字段
    - setter方法构造一个新对象并返回它
    - 更新客户端代码, 以不可变的方式来使用类的实例
- 错综复杂的业务逻辑 - 利用设计原则, 模式等手段来管理业务逻辑
- 视图层中的复杂性 - 引入一个转换层(ViewModel)来缓解视图层的复杂度, 减少外部依赖, 使其变得可测试

### 测试遗留代码

#### 测试不可测试的代码

*重构需要自动测试保障, 编写测试之前需要先重构, 让代码可以测试......*

用一点力来*打破表面*, 利用*代码评审*来弥补缺少的测试

首先要做的就是将它*与依赖隔离开*, 创建一个*间接类*, 将外部依赖包装起来

或者使用Mockito之类的mock工具来模拟外部依赖.

#### 没有单元测试的回归测试

> 在重构之前编写单元测试有时是不可能的, 而且往往是没有意义的

- 遗留代码如果在设计之初就没有考虑可测试性, 补充测试时很难的
- 重构一旦设计多个单元的修改, 单元测试可能会变得没有意义

可以:

- 让测试的模块化级别比重构影响的代码高一个级别
- 别过度追求覆盖率
- 自动化所有测试, 重构时快速而且频繁的运行所有测试

### 让用户为你工作

- 渐进发布新版本, 同时监控错误和回归问题
- 收集真实的用户数据, 利用他来使得测试更高效
- 执行新版本的隐藏发布


## 重撘架构

### 什么是重搭架构

重构在源码级别帮助我们做出改善

重搭架构在更高级别进行重构, 包级别, 模块级别, 库级别, 主要目标包括:

- 通过模块化内建质量
- 良好的设计保障可维护性
- 通过独立达到自治

但是, 这种方式使得*构建脚本*和*工作流程*变得复杂, 以及*分布式*服务下的复杂性.

### 将单体应用分解为模块

目标是:

- 引入显式接口 - 使得测试更容易被mock
- 将源代码拆分为模块 - 容易复用且依赖明确
- 改善依赖管理 - 使得模块更容易管理
- 清理并简化构建脚本

具体的措施包括:

- 定义模块的接口 - 保持功能的单一性
- 构建脚本和依赖管理 - 定义模块的骨架, 引入依赖管理工具, 使得模块间的依赖只限于接口
- 分拆模块 - 借助工具分析依赖关系, 耐心的划分模块边界
- 引入依赖注入框架 - 仅通过接口来进行模块之间的组合
- 使用Maven和Gradle这些为多模块项目设计的工具
- 在开发分支上进行重构, 并且定期同步分支

### 选择一种架构

|    架构    |                          技术优势                          |                                       技术挑战                                      |             组织优势            |                       组织挑战                       |
|:----------:|:----------------------------------------------------------:|:-----------------------------------------------------------------------------------:|:-------------------------------:|:----------------------------------------------------:|
| 单体架构   | 低延时 开发简单 没有重复的模型                             | 伸缩 代码库大小带来的复杂度 意想不到的交互危险                                      | 特性内沟通开销低                | 失败的恐惧 特性间沟通开销大                          |
| 前后端分离 | 能够独立扩展前后端 业务逻辑与表示分离 复用后端构建多个前端 | 网络调用引起的复杂度                                                                | 专业性 更快的迭代前端 向SOA演进 | 沟通开销 知识壁垒 前后端开发相互阻塞                 |
| SOA        | 细粒度的伸缩性 隔离 封装                                   | 运维开销 延时 服务发现 跟踪,调试,日志记录 热点服务 API文档,客户端 集成测试 数据碎片 | 自治                            | 自治程度的困境 重复工作的风险                        |
| 微服务     | 与SOA类似的目标,但是做到物理隔离                           | 与SOA相同,只会更多                                                                  | 界限上下文带来更多的自治        | 意味着需要DevOps 需要一个平台团队 需要思维方式的切换 |

### 增量式的演进

保持整体, 实验性的从*非关键服务*开始, 逐步同化其他功能, 并且在过程中获取必须的*工具, 经验和流程*

给予团队足够的*决定权*, 自行判断是否演进以及如何演进

## 大规模重写

在大规模重写前, 是已经*用尽*其他所有选项了.

- 项目会无休止的拖延
- 投入巨大的精力, 只能为客户带来有限的价值

### 决定项目范围

重写的三种形式, 并且伴随着不同的*目标*:

- 黑盒式重写 - 目标是软件行为保持不变, 更新技术栈或者让软件更容易维护
- 温习式重写 - 额外的目标是使用重写来*获取记录*, *更新和标准化规范*
- 补偿式重写 - 额外的目标是开发一些*新功能*作为重写的一部分, 支付我们薪水的人需要一些激励

使用明确的*文档*来记录项目范围:

- 新功能
- 现有功能
- 及时性以及功能完整性
- 分阶段发布

### 从过去学习

避免在重写时完全放弃旧代码, 对现有代码采取*平衡的态度*, 现有的代码中:

- 包含了多年积累的**bug修复**, **性能优化**, **极端情况处理**
- 精确的定义了软件的**现有行为**

### 如何处理数据库

#### 共享现有数据库

**优点**:

- 简单
- 无需更新其他应用程序和脚本

**缺点**:
- 无法选择存储技术
- 无法重搭架构 - 服务拆分难以达到预期效果
- 无法重构模式 - 数据结构难以重新规划
- 有损坏数据风险 

**建议**:

- 在持久层大力投资 - 在应用程序中做一个转换层, 将新规划的领域模型转换为现有的数据模型
- 为获取数据库的控制权限制定计划 - 在彻底关闭遗留系统之前, *记录*所有的"临时方案", 并在重构数据库之后进行清理

#### 创建新的数据库

可以避免共享数据库的缺点, 但是需要负担*同步数据*的复杂度

实现方式包括:

- 实时同步
    - 数据库到数据库(触发器推送)
    - 数据库到应用程序(触发器推送到队列) - 队列保证新系统未启动时数据不会丢失
    - 应用程序到应用程序(遗留系统推送) - 缺陷是要修改遗留系统
    - 定时器轮询(拉取) - 数据稍有滞后, 避免了遗留系统的大量修改
- 批量同步 - 包括**全库复制**和**根据查询匹配进行复制**, 用于全量或快速的修复数据
- 监控 - 使用工具来监控数据的一致性和正确性, 发送警告或自动修复数据


![](/images/2017-11-02-00-36-11.jpg)


- 简化方案: 复制流量 - 复制所有来自遗留系统的流量, 但是要确保:
    - 不共享一个数据库
    - 新系统相对稳定

#### 应用程序间通信

在设计软件时,通常有一个**假设** : 自己是数据的唯一写入者, 一旦这个假设无效, 会出现很多问题.

问题的根源是程序无法获知**其他程序写入了数据**

这就需要在写入数据的时候, **通知**目标程序, 让其进行相应的处理

**注意**: 这只是当前情况下的**临时方案**, 需要以一种**容易移除**的方式来实现它, 例如**事件总线**

- - - - -
如上已将包含了重整软件的理念和方法, 后续会介绍如何**构建和维护**软件, 并且防止**新写的代码变为遗留代码**
- - - - -

## 开发环境的自动化

让软件运行起来是一件很复杂的事情, **自动化脚本**不仅能够让软件快速运行起来, 还能充当**补充文档**的角色

- 使用根目录下的**README**文档记录信息, 并且让评审者能够检查文档的同步更新
- 使用Vagant令多台虚拟机的管理过程自动化
- 使用Ansible令应用程序的配置自动化
- 让新加入的开发人员开始的工作容易一些, 就更有可能让他们做出贡献
- 尽可能的**移除**外部依赖, 虚拟机包含所需要的一切, 让开发人员能够掌控一切

## 将自动化扩展到测试环境, 预生产环境以及生产环境

软件运行的环境通常包括开发, 测试, 生产环境, 使用工具将自动化配置从开发环境扩展到所有环境.

**好处**:

- 保证环境的一致性
- 易于更新软件
- 易于搭建新环境
- 支持追踪配置更改

**方法**:

- 使用Ansible脚本来处理多种环境
- 让Jenkins负责管理



## 对遗留软件的开发, 构建以及部署过程进行现代化

- 自动化 - 避免繁琐的操作和人为失误
- 更新工具链
- 用Jenkins实现持续集成和自动化
- 自动发布和部署

## 停止编写遗留代码

如何让代码停止变为遗留代码, 除了之前我们讨论的一切, 还包括:

- 源代码并不是项目的全部 - 技术上还包括工具链, 自动化, 持续集成和持续审查; 组织上还包括文档, 沟通, 外部人员的贡献, 质量文化, 开发人员在维护软件时的自由时间等等
- 信息不能自由传递 - 积极促进开发人员之间的沟通并营造一个信息自由流动的环境
    - 文档 - 内容丰富, 易发现, 易编写, 易阅读, 可信赖
    - 促进沟通 - 代码评审, 结对编程, 技术访谈, 项目展示, 黑客日
- 重构工作是永无止境的 - 需要时刻警惕质量问题;需要在团队中培养共同承担代码质量的文化;定期进行代码评审;警惕"破窗效应", 定期清理破碎的窗户
- 自动化一切 - 始终留心观察可以自动化的的那部分开发工作流程
- 小型为佳 - 软件应该设计成可遗弃的, 让它小到能被丢弃, 并能在几周或者几天甚至几小时内被重写
