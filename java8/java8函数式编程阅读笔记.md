# java 8
>author:Dorae  
>date:2017年10月16日14:49:30

----

**<font color="red">Java 8 最主要的新特性就是 Lambda 表达式以及与此相关的特性，如流 (streams)、方法引用 (method references)、功能接口 (functional interfaces)</font>**  
**<font color="red">接口变动、函数式接口、lambda表达式、流操作（外部迭代到内部迭代、并行操作）、Optional</font>**  
**<font color="red">阅读java.util.function(Function)与java.util.stream(Collector)</font>**  
**<font color="red">第八章 设计架构原则、使用Lambda编写并发程序，暂未看，后续跟进</font>**

+ Lambda表达式
+ 函数接口
+ 自定义函数接口
+ 常用流操作
+ 流与集合的区别
+ 集合类流操作的实现原理，内部迭代器，spliterator（并行迭代器）
+ 建造者模式
+ 尽量避免副作用，唯一例外的是foreach，因为其表示一个终结方法
+ Lambda表达式重载的工作原理，本质上上一个匿名方法
+ <font color="red">为什么Supplier<T>可以和匿名内部类一起工作，实现向后兼容？</font>
+ Java中的泛型基于对其是Object的假设，所以有了装箱类型
+ Java8中为了减少集合中频繁的装箱与拆箱操作，提供了相应的方法，Integer、Long、Double，应该尽量减少对装箱类型集合的操作（<font color="red">好像更慢了20倍左右，目前未知原因</font>）
+ Collectors.toMap()方法的merge只会保存一个元素(k1, k2) -> k1,那么只有k1会被保存
+ <font color="red">如何调试java8风格的代码</font>
+ Jva8提供了向后兼容（二进制兼容性，通过默认方法实现，默认方法与普通方法在继承规则上的区别）
	+ 默认方法只能通过调用子类的方法来修改子类本身，避免了对子类的各种假设
	+ 默认方法重写规则不同，可以说没有重写
	+ 一旦与类中定义的方法产生冲突，优先选择类中的方法（<font color="red">因为默认方法的主要目的是为了实现向后兼容</font>）
	+ 默认方法的多重继承会编译报错，可以使用interface.super来实现类似父类的操作
	+ 接口中的静态属性可以被继承，但是静态方法不能被继承，静态属性属于类而非实例
	+ 三定律
	+ 对于类而言，静态方法可以继承，不可以重写，不能实现多态（可以隐藏）；静态方法可以被自身引用调用，不能被父类引用调用（即使父类也有此方法），父类引用通过多态 （A b1 = new B()） 调用的是自己的静态方法。
	+ java中静态属性和静态方法可以被继承，但是没有被重写(overwrite)而是被隐藏. 
	+ 接口中的静态属性与类中的类似，但是静态方法有区别
	+ 对于类中的非静态属性不能实现多态，而非静态方法可以实现多态，是因为静态绑定（对属性，编译期绑定），动态绑定（方法，运行期），对于属性可以通过get方法来解决，但是子类与父类要同时声明
	+ 接口中的静态方法是为了更好的编写类库，组织代码结构
+ @FunctionalInterface接口的作用，方便以后重构发现问题，另外有些只有一个抽象方法的接口可能只是巧合为函数接口，例如持有某种状态的接口
+ 方法引用
+ 流中的元素顺序
	+出现顺序依赖于数据源和对流的操作 
+ 流的并行操作
+ 数据分块（partitioningBy）与分组(groupBy)，Collectors.joining
+ <font color="red">自定义组合收集器（有助于理解收集器的设计思路）</font>
+ 理解重构与定制收集器的思路
+ <font color="red">理解Collector与Function接口的意图</font>
+ reduce的1、2、3参数参数接口
	1. 返回Optional对象
	2. 需要提供一个初始值，并且两个参数的类型必须相同
	3. 第三个参数是为了处理并发操作，所以在使用时需要比较小心，对于其中的参数2为二元函数，其两个参数可以不是同种类型，但是对于参数3，由于其是为了处理并行操作的（对不同线程的结果进行汇聚），所以其函数类型必须与参数2兼容。
+ <font color="red">Spliterator并行迭代器的作用</font>
+ map的computeIfAbsent操作，foreach操作
+ reduce的限制
	+ 初始值必须为组合函数的恒等值
	+ 组合操作必须符合结合定律
	+ 避免持有锁
	+ 同时调用paraller与sequential，则最后调用的生效
+ 性能（fork/join框架）
	+ 数据大小
	+ 源数据结构（分割越容易越好），arraylist、数组比较好，hashSet、treeSet次之，linked最差
	+ 装箱
	+ 核的数量（可使用）
	+ 单元处理开销（越大越好）
+ Arrays提供的parallelPrefix、parallelSetAll、parallelSort会改变原有数组
+ <font color="red">如何使用Lambda表达式提高非集合代码的质量</font>
	+ 代码不断的查询和操作某对象，目的是为了最后给该对象设一个值
	+ 继承的目的只是为了覆盖一个方法（匿名内部类runable）
	+ 如果有一个整体上大概相似的模式，只是行为上有所不同（lambda传递的是模式），就可以试着加入一个Lambda 表达式（重复的东西写两遍）
+ Debug
	+ peek能查看流中每个元素的值，同时继续操作流，可以在其间设置断点

----

## Java 8 被动迭代式特性介绍

+ 迭代器的机制：对集合中的元素进行顺序访问，并能进行某些操作。其本质是在封装的对象集合上做循环，根据控制权限可以分为主动迭代、被动迭代。Spliterator并行迭代器（fork/join框架）
+ map提供了default merge函数。
+ <font color="red">Java 8 提供了全新的遍历对象集合方式，该方式主要包含被动迭代、流、并行流三种方法，它的可读性更好、不易出错、更容易并行化，foreach其实就是内部迭代，重写了collection的foreach</font>
+ foresach多线程环境下线程安全
+ 集合和数组都可以用来产生流，因此称作数据流。流不存储集合中的元素，相反，流是通过管道操作来自数据源的值序列的一种机制
+ Stream 表面上看与一个集合很类似，允许你改变和获取数据，但是实际上它与集合是有很大区别的：
	+ Stream 自己不会存储元素，元素可能被存储在底层的集合中，或者根据需要产生出来
	+ Stream 操作符不会改变源对象，相反，它们会返回一个持有结果的新 Stream
	+ Stream 操作符可能是延迟执行的，这意味着它们会等到需要结果的时候才执行
+ <font color="red">Spliterator接口的设计思想，可以使用其实现并行迭代</font>
+ Stream是一次循环完成所有操作，stream.generate()生成自己的流，stream.iterate

----

## 深入理解java函数式编程笔记
> http://www.cnblogs.com/CarpenterLee/p/5936664.html

+ Lambda的类型就是对应函数接口的类型