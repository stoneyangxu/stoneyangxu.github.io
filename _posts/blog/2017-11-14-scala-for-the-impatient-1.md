---
layout: post
title:  "快学Scala - 基础"
date:   2017-11-14 23:32:03
categories: Scala
---

Scala是一门非常有趣的语言, 它以**JVM为目标环境**, 将**面向对象**和**函数式**编程有机的结合, 带来独特的编程体验

- 有动态语言那样**灵活简洁**
- 有**静态类型检查**来保证安全和执行效率
- 有强大的**抽象能力**
- 能处理**脚本化**的临时任务
- 能处理**高并发**场景下的分布式应用

# 基础 A1

## Scala解释器

安装Scala之后, 在命令行输入`scala`可以开启REPL解释器:

```shell
$ scala
Welcome to Scala 2.12.3 (Java HotSpot(TM) 64-Bit Server VM, Java 9.0.1).
Type in expressions for evaluation. Or try :help.

scala> 

```

可以直接在解释器中输入指令并执行计算:

```shell
scala> 8 * 5 + 2
res0: Int = 42

scala> res0 * 0.5
res1: Double = 21.0

scala> "Hello," + res0
res2: String = Hello,42

scala> 
```

在命令行中可以通过**制表符补全**功能来输入方法名:

```shell
scala> res2.to
to          toBuffer      toDouble       toInt        toList        toMap   toShort    toTraversable      
toArray     toByte        toFloat        toIterable   toLong        toSeq   toStream   toUpperCase        
toBoolean   toCharArray   toIndexedSeq   toIterator   toLowerCase   toSet   toString   toVector           
```

> REPL: read - eval - print loop
> 实际上它并不是一个解释器, 输入的内容被快速编译为字节码, 然后交给Java虚拟机执行

在REPL中输入`:help`可以列出有用的**命令清单**

```shell
scala> :help
All commands can be abbreviated, e.g., :he instead of :help.
:edit <id>|<line>        edit history
:help [command]          print this summary or command-specific help
:history [num]           show the history (optional num is commands to show)
:h? <string>             search the history
:imports [name name ...] show import history, identifying sources of names
:implicits [-v]          show the implicits in scope
:javap <path|class>      disassemble a file or class name
:line <id>|<line>        place line(s) at the end of history
:load <path>             interpret lines in a file
:paste [-raw] [path]     enter paste mode or paste a file
:power                   enable power user mode
:quit                    exit the interpreter
:replay [options]        reset the repl and replay all previous commands
:require <path>          add a jar to the classpath
:reset [options]         reset the repl to its initial state, forgetting all session entries
:save <path>             save replayable session to a file
:sh <command line>       run a shell command (result is implicitly => List[String])
:settings <options>      update compiler options, if possible; see reset
:silent                  disable/enable automatic printing of results
:type [-v] <expr>        display the type of an expression without evaluating it
:kind [-v] <type>        display the kind of a type. see also :help kind
:warnings                show the suppressed warnings from the most recent line which had any

scala> 
```

## 声明值和变量

```scala
val answer = 8 * 5 + 2 // 常亮
var counter = 0 // 变量
```

在声明的时候, 不需要指定类型, Scala能够通过初始化表达式**推断出来**

也可以将多个值或变量放在一起声明:

```shell
scala> val xmax, ymax = 100
xmax: Int = 100
ymax: Int = 100
```

## 常用类型

Scala定义了七种类型: Byte, Char, Short, Int, Long, Float, Double和Boolean

Scala不特意**区分**基础类型和引用类型, 可以直接对数字执行方法

```shell
scala> 1.toString
res3: String = 1

scala> 1.to(10)
res4: scala.collection.immutable.Range.Inclusive = Range 1 to 10
```

Scala底层用java.lang.String来表示字符串, 在实际使用的时候, 会被**隐式转换**为StringOps类, 给字符串追加了上百种操作.

同样, 还提供了RichInt, RichDouble, RichChar等, 为它们对应的数值提供了**便捷方法**

以及BigInt和BigDecimal类用于**任意大小(但有穷)**的数字

## 算术和操作符重载

在Scala中, 类似`a + b`的运算, 实际上都是**方法调用**, 它是`a.+(b)`的简写

Scala并没有提供++和--操作符, 需要使用`+=1`和`-=1`

> Scala允许定义操作符, 在适当的情况下有分寸的使用

## 关于方法调用

```scala
"hello".intersect("world")
```

在方法调用的时候, 如果方法并没有**参数**, 则不需要使用**括号**

```shell
scala> "hello".sorted
res5: String = ehllo
```

在Java中的静态方法, 对应到Scala中是定义在**单例对象**之上的; 包也可以有**包对象**, 通过引入这个包, 就可以不带前缀的使用包对象里的方法

## apply方法

```shell
scala> val s = "Hello"
s: String = Hello

scala> s(4)
res6: Char = o
```

这种将()操作符重载的形式, 其原理是一个名为`apply`的方法, 可以将元素类型为T的序列s想象为一个**函数**, 这个函数将i**映射**为s(i)

```scala
BigInt("123456")
// 等价于
BigInt.apply("123456")
```

## Scaladoc

> https://docs.scala-lang.org/style/scaladoc.html

- 类名旁边的**C和O**分别链接到对应的类和伴生对象
- 特质会显示**t和O**标记
- RichInt, RighDouble, StringOps
- 方法可以以函数作为参数: def count(p: (char) => Boolean): Int
- Scala中使用方括号[]来表示类型参数
- implicit表示**隐式参数**

## 练习

1. REPL中输入`3.`, 然后使用Tab自动补全:

```shell
scala> 3.
!=   /    >=          ceil          getClass        isPosInfinity   isWhole     shortValue       toDegrees     toOctalString   underlying   
%    <    >>          compare       intValue        isValidByte     longValue   signum           toDouble      toRadians       until        
&    <<   >>>         compareTo     isInfinite      isValidChar     max         to               toFloat       toShort         |            
*    <=   ^           doubleValue   isInfinity      isValidInt      min         toBinaryString   toHexString   unary_+                      
+    ==   abs         floatValue    isNaN           isValidLong     round       toByte           toInt         unary_-                      
-    >    byteValue   floor         isNegInfinity   isValidShort    self        toChar           toLong        unary_~                      
```

2. 计算3的平方根, 然后对值求平方

```shell
scala> math.pow(math.sqrt(3), 2)
res2: Double = 2.9999999999999996
```

3. "crazy" * 3

```shell
scala> "crazy" * 3
res3: String = crazycrazycrazy
```

4. 10 max 2

```shell
scala> 10 max 2
res4: Int = 10
```


![](/images/2017-11-15-00-45-45.jpg)


5. 用BigInt计算2的1024次方

```shell
scala> val i = BigInt(2)
i: scala.math.BigInt = 2
scala> i.pow(1024)
res5: scala.math.BigInt = 179769313486231590772930519078902473361797697894230657273430081157732675805500963132708477322407536021120113879871393357658789768814416622492847430639474124377767893424865485276302219601246094119453082952085005768838150682342462881473913110540827237163350510684586298239947245938479716304835356329624224137216
```

6. 生成一个随机的BitInt并将它转换为三十六进制

```shell
scala> scala.math.BigInt(scala.util.Random.nextInt).toString(36)
res13: String = -rsr59j
```

7. 获取字符串的首尾

```shell
scala> val s = "hello"
s: String = hello

scala> s(0)
res14: Char = h
scala> s(s.length - 1)
res15: Char = o

```