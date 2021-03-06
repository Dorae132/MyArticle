# java 8 函数式编程阅读笔记
> Author:Dorae  
> Date:2017年10月16日14:49:30  
> 转载注明出处

----

**<font color="red">Java 8 最主要的新特性就是 Lambda 表达式以及与此相关的特性，如流 (streams)、方法引用 (method references)、功能接口 (functional interfaces)</font>**  
**<font color="red">接口变动、函数式接口、lambda表达式、流操作（外部迭代到内部迭代、并行操作）、Optional</font>**  
**<font color="red">阅读java.util.function(Function)与java.util.stream(Collector)</font>**  
**<font color="red">第八章 设计架构原则、使用Lambda编写并发程序，暂未看，后续跟进</font>**
**<font color="red">JVM内部是通过invokedynamic指令来实现Lambda表达式的</font>**  
**Stream的两个核心流水线、自动并行**  
**Stream如何并行、Lambda的执行原理**
**Lambda要尽量避免副作用**
**java7的fork/join框架待跟进**  
**<span style="color:red">测试时注意即时编译与gc的影响</span>**

+ Lambda表达式
+ 函数接口
+ 自定义函数接口
+ 常用流操作
+ 流与集合的区别
+ 集合类流操作的实现原理，内部迭代器，spliterator（并行迭代器）
+ 建造者模式
+ 尽量避免副作用，唯一例外的是foreach，因为其表示一个终结方法(无论何时，将Lambda 表达式传给Stream 上的高阶函数，都应该尽量避免副作用。唯一的
例外是forEach 方法，它是一个终结方法。)
+ Lambda表达式重载的工作原理，本质上上一个匿名方法
+ <font color="red">为什么Supplier<T>可以和匿名内部类一起工作，实现向后兼容？</font>
+ Java中的泛型基于对其是Object的假设，所以有了装箱类型
+ Java8中为了减少集合中频繁的装箱与拆箱操作，提供了相应的方法，Integer、Long、Double，应该尽量减少对装箱类型集合的操作（<font color="red">好像更慢了20倍左右，目前未知原因</font>）
+ Collectors.toMap()方法的merge只会保存一个元素(k1, k2) -> k1,那么只有k1会被保存
+ <font color="red">如何调试java8风格的代码</font>
+ Jva8提供了向后兼容（二进制兼容性，通过默认方法实现，默认方法与普通方法在继承规则上的区别）
	+ 默认方法只能通过调用子类的方法来修改子类本身，避免了对子类的各种假设(通过this与工具类实现)
	+ 默认方法重写规则不同，可以说没有重写
	+ 一旦与类中定义的方法产生冲突，优先选择类中的方法（<font color="red">因为默认方法的主要目的是为了实现向后兼容</font>）
	+ 默认方法的多重继承会编译报错，可以使用interface.super来实现类似父类的操作
	+ 接口中的静态属性可以被继承，但是静态方法不能被继承，静态属性属于类而非实例
	+ 三定律
	+ 对于类而言，静态方法可以继承，不可以重写，不能实现多态（可以隐藏）；静态方法可以被自身引用调用，不能被父类引用调用（即使父类也有此方法），父类引用通过多态 （A b1 = new B()） 调用的是自己的静态方法。
	+ java中静态属性和静态方法可以被继承，但是没有被重写(overwrite)而是被隐藏. 
	+ 接口中的静态属性与类中的类似，但是静态方法有区别(静态属性不能在实例中获得引用)
	+ 对于类中的非静态属性不能实现多态，而非静态方法可以实现多态，是因为静态绑定（对属性，编译期绑定），动态绑定（方法，运行期），对于属性可以通过get方法来解决，但是子类与父类要同时声明
	+ 接口中的静态方法是为了更好的编写类库，组织代码结构
	+ <span style="color:red">关于静态属性与静态方法
		+ 类中
			+ 静态属性可以被继承；可以被实例获得引用；因为属性是静态绑定，所以没有多态
			+ 静态方法可以被继承；可以被实例获得引用；但是不能实现多态
		+ 接口中
			+ 静态属性可以被继承；但是实例不能获得引用;(其实可以获得引用)
			+ 静态方法不能被类、接口继承；</span>
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
+ 一些操作在有序的流上开销更大，调用unordered 方法消除这种顺序就能解决该问题。大多数操作都是在有序流上效率更高，比如filter、map 和reduce 等。

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

## [深入理解java函数式编程笔记](http://www.cnblogs.com/CarpenterLee/p/5936664.html)

+ Lambda的类型就是对应函数接口的类型
+ **无存储：**stream不是一种数据结构，它只是某种数据源的一个视图，数据源可以是一个数组，Java容器或I/O channel等。
+ **为函数式编程而生：**对stream的任何修改都不会修改背后的数据源，比如对stream执行过滤操作并不会删除被过滤的元素，而是会产生一个不包含被过滤元素的新stream。
+ **惰式执行：**stream上的操作并不会立即执行，只有等到用户真正需要结果的时候才会执行。
+ **可消费性：**stream只能被“消费”一次，一旦遍历过就会失效，就像容器的迭代器那样，想要再次遍历必须重新生成。
+ 通俗的讲flatMap()的作用就相当于把原stream中的所有元素都"摊平"之后组成的Stream，转换前后元素的个数和类型都可能会改变。
+ [收集器](http://www.cnblogs.com/CarpenterLee/p/6550212.html)挺好。
+ [<font color="red">深入理解Stream流水线</font>](http://www.cnblogs.com/CarpenterLee/p/6637118.html)。
+ Lambda相当于回调
+ **[Stream的Pipeline、Spliterator](http://www.cnblogs.com/CarpenterLee/p/6637118.html)**
+ 实际上Stream API内部实现的的本质，就是如何重载Sink的这四个接口方法
+ **事实上，除了打印之外其他场景都应避免使用副作用**  
特别说明：副作用不应该被滥用，也许你会觉得在Stream.forEach()里进行元素收集是个不错的选择，就像下面代码中那样，但遗憾的是这样使用的正确性和效率都无法保证，因为Stream可能会并行执行。大多数使用副作用的地方都可以使用归约操作更安全和有效的完成

## [java.util.stream库](https://www.ibm.com/developerworks/cn/java/j-java-streams-1-brian-goetz/index.html?ca=drs-)

+ 流的关注点是计算，而不是数据。
+ 流没有为它们处理的元素提供存储空间，而且流的生命周期更像一个时间点 — 调用终止操作。不同于集合，流也可以是无限的；相应地，一些操作（limit()、findFirst()）是短路，而且可在无限流上运行有限的计算。
+ 大多数流操作都要求传递给它们的拉姆达表达是互不干扰和无状态的。
+ 所有并发性风险的根源是共享可变状态。共享可变状态的一种可能来源是流来源
+ 互不干扰要求不仅不包括在流操作期间被其他操作突变的来源，而且传递给流操作的拉姆达表达式本身也应避免突变来源
+ **<font color="red">最好的情况是，如果传递给流操作的拉姆达表达式完全没有副作用，也就是说，它们不会突变任何基于堆的状态或在执行过程中执行任何 I/O。如果有副作用，它们应负责执行任何需要的协调，以确保这些副作用是线程安全的。</font>**
+ **除了不修改流来源之外，传递给流操作的拉姆达表达式也应是无状态的**
+ 在大多数时候，该库使我们的代码能够运行得更快，而且不需要我们投入精力。能够执行这样的优化的代价是，我们必须接受对我们传递给流操作的拉姆达表达式的操作的一些限制，以及我们对副作用的一定的依赖。（总之，这是一次很划算的交易）
+ **大多数流操作都要求传递给它们的拉姆达表达是互不干扰和无状态的。互不干扰意味着它们不会修改流来源（可能会多线程）；无状态意味着它们不会访问（读或写）任何可能在流操作寿命内改变的状态，还应该没有副作用（可能会被优化掉）。**
+ 从某种程度讲，这些要求源于以下事实：如果管道并行执行，流库可能从多个线程访问数据源，或并发地调用这些拉姆达表达式。需要这些限制才能确保计算保持正确。

### 使用流进行聚合

+ 通过将累加描述为缩减，而不使用累加器反模式，可以采用更抽象、更紧凑、更并行化 的方式来描述计算 — 只要您的二元运算符满足一个简单条件：结合性
+ 结合性意味着分组无关紧要

### Stream的幕后原理
+ **<font color="red">一个流管道 包含一个流来源、0 或多个中间操作，以及一个终止操作</font>**
+ 即使忽略了并行分解的可能性，Spliterator 抽象也是一个 “更好的迭代器” — 更容易编写，更容易使用，而且通常具有更低的每元素访问开销。但 Spliterator 抽象还扩展到了并行分解领域。一个 spliterator 描述剩余元素的序列，调用 tryAdvance() 或 forEachRemaining() 元素访问方法来在该序列中前进。
+ 流管道是通过构造流来源及其中间操作的链接列表表示来构建的。在内部表示中，管道的每个阶段都通过一个流标志 位图来描述，该位图描述了在流管道的这一阶段已知的元素信息。流使用这些标志优化流的构造和执行。
+ 高质量的 spliterator 实现不仅提供了高效的元素访问和拆分，还会描述元素的特征。（例如，一个 HashSet 的 spliterator 报告 DISTINCT 特征，因为已知一个 Set 的元素是不同的。）
+ **<font color="red">在某些情况下，Streams 可以使用来源和之前的操作的知识来完全省略某个操作。</font>**
+ 并行执行将会执行类似的操作，但不会创建单个机器，每个工作线程将会获取自己的机器副本并将其数据节提供给它，然后将每个线程机器的结果与其他机器的结果合并，生成最终结果
+ 如果流没有遇到顺序，limit() 操作可以自由选择任何 N 个元素，这让执行效率变得高得多。知道元素后可立即将其发往下游，无需任何缓存，而且线程之间唯一需要执行的协调是发送一个信号来确保未超出目标流长度
+ 遇到顺序成本的另一个不太常见的示例是排序。如果遇到顺序有意义，那么 sorted() 操作会实现一种稳定 排序（相同的元素按照它们进入输入时的相同顺序出现在输出中），而对于无序的流，稳定性（具有成本）不是必需的。distinct() 具有类似的情况：如果流有一个遇到顺序，那么对于多个相同的输入元素，distinct() 必须发出其中的第一个，而对于无序的流，它可以发出任何元素 — 同样可以获得高效得多的并行实现。
+ 在使用 collect() 聚合时会遇到类似的情形。如果在无序流上执行 collect(groupingBy()) 操作，与任何键对应的元素都必须按它们在输入中出现的顺序提供给下游收集器。此顺序对应用程序通常没有什么意义，而且任何顺序都没有意义。在这些情况下，可能最好选择一个并发 收集器（比如 groupingByConcurrent()），它可以忽略遇到顺序，并让所有线程直接收集到一个共享的并发数据结构中（比如 ConcurrentHashMap），而不是让每个线程收集到它自己的中间映射中，然后再合并中间映射（这可能产生很高的成本）
+ 随着可用核心数超过任务数，另一个性能目标就变得很诱人：使用多个核心更快完成单个任务来减少延迟。不是所有任务都能通过这种分解来轻松处理；最适合的任务是数据密集型的查询，其中的同一个操作会在大型的数据集上完成。
+ **<font color="red">更多现代课程将并发性 描述为正确、高效地控制对共享资源的访问，而并行性 指的是使用更多资源更快地解决问题。构造线程安全的数据结构属于并发性范畴，通过锁、事件、旗语、协同程序或软件事务内存 (STM) 等原语实现。另一方面，并行性使用分区或分片等技术来使多个活动处理一项任务，而没有协调过程。  
&emsp;&emsp;并发性和并行性的目标都是同时完成多件事。但是，应用这两种技术的轻松程度有着很大差别。使用锁等协调原语正确创建并发代码很难、容易出错且不自然。通过让每个工作者拥有自己的问题部分来处理问题，从而正确创建并行代码，这样做相对更简单、更安全且更自然。</font>**
+ 并行执行完成的工作始终比顺序执行的多，因为除了解决问题之外，它还必须分解问题，创建和管理描述子问题的任务，分派和等待这些任务，合并它们的结果。
+ 它能够使用一个核心或许多核心高效地处理数据，**而且它不需要担心协调对共享可变状态的访问带来复杂性 — 因为没有共享可变状态。**将问题分解为若干个子问题，安排每个问题仅访问来自某个特定子问题的数据，这通常不需要进行线程间协调。
+ **<font color="red">分而治之与性能，数据集的大小和它的内存位置</font>**
+ 影响并行性有效性的因素包括问题本身、解决问题所使用的算法、对任务分解和调度的运行时支持，以及数据集的大小和内存位置。
+ 可能影响并行化有效性的因素。这些因素包括
	+ 问题的特征
	+ 用于实现解决方案的算法
	+ 用于调入任务来并行执行的运行时框架
	+ 以及数据集的大小和内存布局
+ 可能导致并行执行效率降低的多个因素：[点这里](https://www.ibm.com/developerworks/cn/java/j-java-streams-5-brian-goetz/index.html?ca=drs-)
	+ 来源的拆分成本很高或拆分不均。
	+ 合并部分结果的成本很高。
	+ 问题不允许足够的可利用并行性。
	+ 数据布局导致糟糕的访问位置。
	+ 没有足够的数据来克服并行性的启动成本。
+ **在操作与遇到顺序紧密关联时，您可能对并行执行的时间和空间成本感到很奇怪。对于顺序执行，在显明实现中，我们通常会按遇到顺序遍历输入，所以对遇到顺序的依赖很少很明显或成本很高。在并行执行中，这种依赖性可能具有非常高的成本**
+ **最终，数据结构中的指针越多，遍历这种数据结构给内存抓取单元带来的压力就越大，这可能对计算时间（CPU 会花时间等待数据时）和并行性（因为许多核心同时从内存抓取会给可用来将数据从内存传输到缓存的带宽造成压力）产生负面影响。**
+ opWrapSink其实是保留了上游操作的指针

## 总结

&emsp;&emsp;**<span style="color:red">因为并行性通常仅提供了加快运行时间的潜力，所以仅在它能带来实际提速时才应使用它。使用顺序流开发和测试您的代码；然后，如果您的性能需求暗示需要进一步改进，可考虑采用并行性作为可能的优化战略。尽管度量对确保您的优化工作不会适得其反至关重要，但对于许多流管道，您可以通过检查来确定它们不太适合并行性。损害潜在并行提速的因素包括不良或不均匀的拆分来源、高组合成本、对遇到顺序的依赖性、位置不佳或没有足够的数据。另一方面，每个元素的大计算量 (Q) 可以弥补其中一些缺陷。</span>**

**评估类与接口的static属性与方法的继承规则**

**并行化的限制**