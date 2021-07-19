# JVM调优

## 1. JDK命令行工具

### 1. jps

`jps(Java Process Status)`

**作用**：查看正在运行的Java进程

**基本情况**：显示指定系统内所有的HotSpot虚拟机进程（查看虚拟机进程信息），可用于查询正在进行的虚拟机进程。

说明：对于本地虚拟机进程来说，进程的本地虚拟机ID与操作系统的进程ID是一致的，是唯一的。

**基本语法**：

`jps [options] [hostid]`

```
haipenghou@192 ~ % jps -help
usage: jps [-help]
       jps [-q] [-mlvV] [<hostid>]

Definitions:
    <hostid>:      <hostname>[:<port>]
```

1. `options`参数：

	- `-q`:仅仅显示`LVMID(local virtual machine id).`即本地虚拟机唯一id.不显示主类的名称等
	- `-l`:输出应用程序主类的全类名 或 如果进程执行的是jar包，则输出jar完整路径
	- `-m`:输出虚拟机进程启动时传递给主类main()的参数
	- `-v`:列出虚拟机进程启动时的JVM参数。比如：-Xms20m -Xmx50m是启动程序制定的jvm参数。

	说明：以上参数可以综合使用。

	补充：如果某Java进程关闭了默认开启的`UserPerfData`参数(即使用参数`-XX:-UserPerfData`),那么jps命令（以及下面介绍的jstat）将无法探知该Java进程。

2. `hostid`参数:

	RMI注册表中的注册的主机名。

	如果想要远程监控主机上的java程序，需要安装`jstatd`.

### 2. jstat

`jstat(JVM Statistics Monitoring Tool)`

**作用**：查看JVM统计信息

**基本情况**：用于监视虚拟机各种运行状态信息的命令行工具。它可以显示本地或者远程虚拟机进程中的类装载，内存，垃圾收集，JIT编译等运行数据。

在没有GUI图形界面，只提供了纯文本控制台环境的服务器上，它将是运行期定位虚拟机性能问题的首选工具。常用于检测**垃圾回收问题**以及**内存泄漏问题**。

官方文档

```
https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html
```

**基本语法**：

`jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]`

1. `option`参数：

  **类装载相关的**：

  - `-class`:显示`ClassLoader`的相关信息：类的加载，卸载数量，总空间，类装载所消耗的时间等。

  ```
  haipenghou@192 ~ % jstat -class 611
  加载类的个数		类占用的字节数					卸载的类的个数					卸载类占用字节数      花费时间
  Loaded  					Bytes  							Unloaded  						Bytes     				Time   
     692  					1381.0       				0     								0.0       				0.12
  ```

  **垃圾回收相关的**

  - `-gc`:显示与GC相关的堆信息。包括Eden区，两个Survivor区，老年代，永久代等的容量，已用空间，GC时间合计等信息。

  ```
  haipenghou@192 ~ % jstat -gc 611      
   S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT   
  10752.0 10752.0  0.0    0.0   65536.0   7864.9   175104.0     0.0     4480.0 780.7  384.0   76.6       0    0.000   0      0.000    0.000
  ```

  ​			参数说明：

  1. 新生代相关：

  	- `S0C`:第一个幸存者区的大小（字节，下同）
  	- `S1C`:第二个幸存者区的大小
  	- `S0U`:第一个幸存者区已使用的大小
  	- `S1U`:第二个幸存者区已使用的大小
  	- `EC`:Eden空间的大小
  	- `EU`:Eden空间已使用的大小
  2. 老年代相关：

  	- `OC`:老年代的大小
  	- `OU`:老年代已使用的大小
  3. 方法区（元空间）相关：

  	- `MC`:方法区的大小
  	- `MU`:方法区已使用的大小
  	- `CCSC`:压缩类空间的大小
  	- `CCSU`:压缩类空间已使用的大小
  4. 其他：

  	- `YGC`:从应用程序启动到采样时young gc次数
  	- `YGCT`:从应用程序启动到采样时young gc消耗的时间（秒）
  	- `FGC`:从应用程序启动到采样时full gc次数
  	- `FGCT`:从应用程序启动到采样时full gc消耗时间（秒）
  	- `GCT`:从应用程序启动到采样时gc的总时间,=YGCT+FGCT;

  - `-gccapacity`:显示内容与`-gc`基本相同，但输出主要关注Java堆各个区域使用到的最大，最小空间。
  - `-gcutil`:显示内容与`-gc`基本相同，但输出主要关注已使用空间占总空间的百分比
  - `-gccause`:与`-gcutil`功能一样，但是会额外输出导致最后一次或当前正在发生的GC产生的原因
  - `-gcnew`:显示新生代GC状况
  - `-gcnewcapacity`:显示内容与`-gcnew`基本相同，输出主要关注使用到的最大，最小空间。
  - `-geold:`显示老年代GC状况

  **JIT相关的**

  - `-compiler`:显示JIT编译过的方法，耗时等信息

  ```
  haipenghou@192 ~ % jstat -compiler 611
  Compiled Failed Invalid   Time   FailedType FailedMethod
        89      0       0     0.03          0 
  ```

  - `-printcompilation`:输出已经被JIT编译的方法

2. `interval`参数：

	用于指定输出统计数据的周期，单位为毫秒。即：查询间隔。

3. `count`参数：

	用于指定查询的总次数。

4. `-t`参数：

  可以在输出信息前加上一个Timestamp列，显示程序的运行时间。单位s

  **经验**

  1. 我们可以比较Java进程的启动时间以及总GC时间(GCT列)，或者两次测量的间隔时间以及总GC时间的增量，来得出GC时间占运行时间的比例。

  2. 如果该比例超过20%,则说明目前堆的压力较大；如果超过90%，则说明堆里几乎没有可用空间，随时都可能抛出OOM异常

5. `-h`参数：

	可以在周期性数据输出时，输出多少行后输出一个表头信息

### 3. jinfo

`jinfo(Configuration Info for java)`

**作用**：查看虚拟机配置参数信息，也可用于调整虚拟机的配置参数

**基本情况**：在很多情况下，Java应用程序不会指定所有的Java虚拟机参数。而此时，开发人员可能不知道某一个具体的Java虚拟机参数的默认值。开发人员可以很方便得使用jinfo找到Java虚拟机参数的当前值。

**基本语法**

`jinfo [option] <pid>`

1. `option`参数：
	- `-sysprops`:
	- `-flags`：

**拓展**：

1. `java -XX:+PrintFlagsInitial`:查看所有JVM参数启动的初始值
2. `java -XX:+PrintFlagsFinal`:查看所有JVM参数的最终值
3. `java -XX:+PrintCommandLineFlags`:查看那些已经被用户或者JVM设置过的详细的XX参数的名称和值

### 4. jmap

`jmap（JVM Memory Map）`

**作用**：导出内存映射文件，显示堆内存相关信息

**基本情况**：一方面获取dump文件（堆转储快照文件，二进制文件），它还可以获取目标Java进程的内存相关信息，包括Java堆各区域的使用情况，堆中对象的统计信息，类加载信息等。

**基本语法**：

1. 导出内存映射文件

  - 手动的方式：

  	- `jmap -dump:format=b,file=/Users/haipenghou/Documents/1.hprof pid`
  	- `jmap -dump:live,format=b,file=/Users/haipenghou/Documents/1.hprof pid`

  - 自动的方式：

  	当程序发生OOM退出系统时，一些瞬时信息都随着程序的终止而消失，而重现OOM问题往往比较困难或者耗时，此时若能在OOM时，自动导出dump文件就显得非常迫切。
  	
  	比较常见的取得堆快照文件的方法，即使用
  	
  	- `-XX:+HeapDumpOnOutOfMemoryError`:在程序发生OOM时，导出应用程序的当前堆快照
  	- `-XX:HeapDumpPath`:可以指定堆快照的保存位置。比如：
  	
  	`-XX:HeapDumpPath=/Users/haipenghou/Documents/JavaDocument/5.hprof`

2. 显示堆内存相关信息

  - `jmap -heap pid`:查看某一时间点的堆内存情况，与`jstat -interval`连续记录有区别。
  - `jmap -histo pid`：也是某一时间点

**小结**

1. 由于jmap将访问堆中的所有对象，为了保证在此过程中不被应用线程干扰，jmap需要借助安全点机制，让所有线程停留在不改变堆中数据的状态。也就是说，由jmap导出的堆快照必定是安全点位置的。这可能导致基于该堆快照的分析结果存在偏差。
2. 举个例子，假设在编译生成的机器码中，某些对象的生命周期在两个安全点之间，那么:live选项将无法探知到这些对象
3. 另外，如果某个线程长时间无法跑到安全点，jmap将一直等下去。与前面讲的jstat则不同，垃圾回收器会主动将jstat所需要的摘要数据保存至固定位置之中，而jstat只需直接读取。

### 5. jhat

`jhat(JVM Heap Analysis Tool)`

**作用**：JDK自带堆分析工具

**基本情况**：Sun JDK提供的`jhat`命令与`jmap`命令搭配使用，用于分析jmap生成的heap dump文件（堆转储快照）。jhat内置了一个微型的HTTP/HTML服务器，生成dump文件的分析结果后，用户可以在浏览器中查看结果(分析虚拟机转储快照信息)。

使用了jhat命令，就启动了一个http服务，端口号是7000，即http://localhost:7000/

jhat命令在JDK9,JDK10中已经被删除，官方建议用`VisualVM`代替

### 6. jstack

`jstack(JVM Stack Trace)`

**作用**：打印JVM中线程快照

**基本情况**：用于生成虚拟机指定进程当前时刻的线程快照（虚拟机堆栈跟踪）。线程快照就是当前虚拟机内指定进程的每一条线程正在执行的方法堆栈的集合。

生成线程快照的作用：可用于定位线程出现长时间停顿的原因，如线程间死锁，死循环，请求外部资源导致的长时间等待等问题。这些都是导致线程长时间停顿的常见原因。当线程出现停顿时，就可以用jstack显示各个线程调用的堆栈情况。

在thread dump中，要留意下面几种状态

1. 死锁，Deadlock(重点关注)
2. 等待资源，Waiting on condition（重点关注）
3. 等待获取监视器，Waiting on monitor entry（重点关注）
4. 阻塞，Blocked（重点关注）
5. 执行中，Runnable
6. 暂停，Suspended

**基本语法**

`jstack [option] <pid>`

`option`参数：

1. `-F`:当正常输出的请求不被响应时，强制输出线程堆栈。
2. `-l`：除堆栈外，显示关于锁的附加信息
3. `-m`:如果调用到本地方法的话，可以显示C/C++的堆栈
4. `-h`:帮助操作

## 2. JDK可视化工具

### 1. Visual VM

**基本概述**：

1. Visual VM是一个功能强大的多合一故障诊断和性能监控的可视化工具。

2. 他集成了多个JDK命令行工具，使用Visual VM可用于显示虚拟机进程及进程的配置和环境信息(`jps`,`jinfo`),监视应用程序的CPU,GC,堆，方法区及线程的信息(`jstat`,`jstack`)等,甚至代替JConsole.

3. 地址：

	```
	https://visualvm.github.io/index.html
	```

**主要功能**

1. 生成/读取堆内存快照
2. 查看JVM参数和系统属性
3. 查看运行中的虚拟机进程
4. 生成/读取线程快照
5. 程序资源实时监控

## 3. 内存泄漏

### 1. 内存泄漏

可达性分析算法来判断对象是否是不再使用的对象，本质都是判断一个对象是否还被引用，那么对于这种情况下，由于代码的实现不同就会出现很多种内存泄漏问题（**让JVM误以为此对象还在引用中，无法回收，造成内存泄漏**）

- 是否还被使用？
- 是否还被需要？

**补充**

严格来说，只有对象不再被程序用到了，但是GC又不能回收它们的情况，才叫内存泄漏

但实际情况很多时候一些不太好的实践（或疏忽）会导致对象的生命周期变得很长甚至导致OOM,也可以叫做宽泛意义上的"内存泄漏"

### 2. 内存泄漏和内存溢出的关系

**内存泄漏的增多，最终会导致内存溢出**

### 3. Java中内存泄漏的8种情况

1. 静态集合类

	静态集合类，如HashMap,LinkedList等等。如果这些容器为静态的，那么它们的生命周期与JVM程序一致，则容器中的对象在程序结束前将不能被释放，从而造成内存泄漏。简单而言，长生命周期对象持有短生命周期对象的引用，尽管短生命周期的对象不再被使用，但是长生命周期对象持有它的引用而导致不能被回收。

	```
	public class MemoryLeak {
		static List list = new ArrayList();
		
		public void oomTests() {
			Object obj = new Object();//局部变量
			list.add(obj);
		}
	}
	```

2. 单例模式

	单例模式，和静态集合导致内存的原因相似，因为单例的静态特性，它的生命周期和JVM的生命周期一样长，所以如果单例对象持有外部对象的引用，那么这个外部对象也不会被回收，那么就会造成内存泄漏。

3. 内部类持有外部类

	内部类持有外部类，如果一个外部类的实例对象的方法返回了一个内部类的实例对象

	这个内部类对象被长期引用了，即使那个外部类实例对象不再被使用，但由于内部类持有外部类的实例对象，这个外部类对象将不会被垃圾回收，这也会造成内存泄漏

4. 各种连接，如数据库连接，网络连接和IO连接等。

	在对数据库进行操作的过程中，首先需要建立与数据库的连接，当不再使用时，需要调用close方法来释放与数据库的连接，只有连接被关闭后，垃圾回收器才会回收对应的对象。

	否则，如果在访问数据库的过程中，对Connection，Statement或ResultSet不显性地关闭，将会造成大量的对象无法被回收，从而引起内存泄漏

5. 变量不合理的作用域

	一般而言，一个变量的定义的作用范围大于其使用范围，很有可能会造成内存泄漏。另一方面，如果没有及时地把对象设置为null,很有可能导致内存泄漏的发生。

6. 改变哈希值

	当一个对象被存储进HashSet集合中以后，就不能修改这个对象中的那些参与计算哈希值的字段了 

	否则，对象修改后的哈希值与最初存储进HashSet集合中时的哈希值就不同了，在这种情况下，即使在contains方法使用该对象的当前引用作为的参数去HashSet集合中检索对象，也将返回不到对象的结果，这也会导致无法从HashSet集合中单独删除当前对象，造成内存泄漏。

7. 缓存泄漏

	内存泄漏的另一个常见来源是缓存，一旦你把对象引用加入到缓存中，他就容易遗忘

8. 监听器和回调

## 4. JVM运行时参数

### 4.1 JVM参数选项类型

- 类型一：标准参数选项

	**特点**：比较稳定，后续版本基本不会变化，`以-开头`

	各种选项：运行`java`或者`java -help`可以看到所有的标准选项

- 类型二：-X参数选项

	**特点**：非标准化参数，`以-X开头`

	各种选项：运行`java -X`命令可以看到所有的X选项

	JVM的JIT编译模式相关的选项：

	- `-Xint`:禁用JIT,所有字节码都被解释执行，这个模式的速度最慢

	- `-Xcomp`:所有字节码第一次使用就都被编译成本地代码，然后再执行

	- `-Xmixed`:混合模式，默认，让JIT根据程序运行的情况，有选择地将某些代码

	特别的：

	- `-Xms<size>`：设置初始 Java 堆大小，等价于`-XX:InitialHeapSize`

	- `-Xmx<size>`：设置最大 Java 堆大小，等价于`-XX:MaxHeapSize`
	- `-Xss<size>`：设置 Java 线程堆栈大小， 等价于`-XX:ThreadStackSize`

- 类型三：-XX参数选项

	**特点**：非标准化参数，是使用最多的参数类型，这类选项属于实验性，不稳定，`以-XX开头`

	**作用**：用于开发和调试JVM

	**分类**：

	1. Boolean类型：

		- `-XX:+<option>`:表示启用option属性
			1. `-XX:+UseParallelGC`:选择垃圾收集器为并行收集器
			2. `-XX:+UseG1GC`：启用G1收集器
			3. `-XX:+UseAdaptiveSizePolicy`:自动选择年轻代区大小和相应的Survivor区比例。
		- `-XX:-<option>`:表示禁用option属性

	2. 非Boolean类型（key-value类型）：

		- 子类型1:数值型格式`-XX:<option>=<number>`

			number表示数值，number可以带上单位，比如：'m','M'表示兆，'k','K'表示kb,'g','G'表示g

			1. `-XX:NewSize=1024m`:表示设置新生代初始大小为1024兆
			2. `-XX:MaxGCPauseMillis=500`:表示设置GC停顿时间：500毫秒
			3. `-XX:GCTimeRatio=19`:表示设置吞吐量
			4. `-XX:NewRatio=2`:表示新生代与老年代的比例

		- 子类型2:非数值型格式`-XX:<name>=<string>`

			1. `-XX:HeapDumpPath=/Users/haipenghou/Documents/heapdump.hprof`:用于指定heap转存文件的存储路径

	**特别的**：

	`-XX:+PrintFlagsFinal`:输出所有参数的名称和默认值

### 4.2 常用的JVM参数选项

1. 打印设置的XX选项及值
	- `-XX:+PrintCommandLineFlags`:可以让程序运行前打印出用户手动设置或者JVM自动设置的XX选项
	- `-XX:+PrintFlagsInitial`:打印出所有XX选项的默认值
	- `-XX:+PrintFlagsFinal`:打印出XX选项在运行程序时生效的值
	- `-XX:+PrintVMOptions`:打印JVM的参数
	
2. 堆，栈，方法区等内存的大小
	-  栈
		- `-Xss<size>`：设置每个线程的栈大小为size
	- 堆内存
		- `-Xms<size>`:初始堆内存
		- `-Xmx<size>`:最大堆内存
		- `-Xmn<size>`:年轻代大小，官方推荐配置为整个堆大小的3/8
		- `-XX:NewSize=<size>`:年轻代初始值
		- `-XX:MaxNewSize=<size>`:年轻代最大值
		- `-XX:SurvivorRatio=<ratio>`:设置年轻代中`Eden`区与一个`Survivor`区的比值，默认为8
		- `-XX:+UseAdaptiveSizePolicy`:自动选择各区大小比例
		- `-XX:NewRatio=<ratio>`:设置老年代与年轻代（包括1个Eden和2个Survivor区）的比值
		- `-XX:PretenureSizeThreadshold=<size>`:设置让大于此阈值的对象直接分配在老年代，单位为字节，只对Serial,ParNew收集器有效
		- `-XX:MaxTenuringThreshold=<count>`:默认值为15,新生代每次MinorGC后，还存活的对象年龄+1，当对象的年龄大于设置的这个值时就进入老年代
		- `-XX:+PrintTenuringDistribution`:让JVM在每次MinorGC后打印出当前使用的Survivor中对象的年龄分布
		- `-XX:TargetSurvivorRatio`:表示MinorGC结束后Survivor区域中占用空间的期望比例
	- 方法区
		1. 永久代（JDK1.7及以前）：
			- `-XX:PermSize=<size>`:设置永久代初始值
			- `-XX:MaxPermSize=<size>`:设置永久代最大值
		2. 元空间（JDK1.8以后）:
			- `-XX:MetaspaceSize`:初始空间大小
			- `-XX:MaxMetaspaceSize`:最大空间，默认没有限制
			- `-XX:+UseCompressedOops`:压缩对象指针
			- `-XX:+UseCompressedClassPointers`:压缩类指针
			- `-XX:CompressedClassSpaceSize`:设置Klass Metaspace的大小，默认为1G
		3. 直接内存
			- `-XX:MaxDirectMemorySize`:指定DirectMemory容量，若未指定，则默认与Java堆最大值一样。
	
3. OutOfMemory相关的选项：

	- `-XX:+HeapDumpOnOutOfMemoryError`:在内存出现OOM的时候，把Heap转存(Dump)到文件以便后续分析

	- `-XX:+HeapDumpBeforeFullGC`:在出现FullGC之前，生成Heap转储文件
	- `-XX:HeapDumpPath=<path>`:指定heap转存文件的存储路径
	- `-XX:OnOutOfMemoryError=<path>/xx.sh`:指定一个可执行程序或者脚本的路径，当发生OOM的时候，去执行这个脚本

4. 垃圾收集器相关选项：

  - 查看默认垃圾回收器

  	- `-XX:+PrintCommandLineFlags`:查看命令行相关参数（包括使用的垃圾收集器）

  	- 使用命令行指令：`jinfo -flag 相关垃圾回收器参数 进程ID`

  - Serial回收器:

  	Serial收集器作为HotSpot中Client模式下的默认新生代垃圾收集器。Serial Old是运行在Client模式下默认的老年代垃圾回收器

  	`-XX:+UseSerialGC`:指定年轻代和老年代都使用串行收集器。等价于新生代用Serial GC,且老年代用Serial Old GC。可以获得最高的单线程收集效率。

  - ParNew回收器：

  	- `-XX:+UseParNewGC`:手动指定使用ParNew收集器执行内存回收任务。它表示年轻代使用并行收集器，不影响老年代
  	- `-XX:ParallelGCThreads=N`:限制线程数量，默认开启和CPU数量相同的线程数。

  - `Parallel`回收器(JDK8默认)：

  	- `-XX:+UseParallelGC`:手动指定年轻代使用Parallel并行收集器执行内存回收任务
  	- `-XX:+UseParallelOldGC`:手动指定老年代使用
  		1. 分别适用于新生代和老年代。默认JDK8是开启的
  		2. 上面两个参数，默认开启一个，另一个也会被开启。（互相激活）
  	- `-XX:ParallelGCThreads`:设置年轻代并行收集器的线程数。一般最好与CPU数量相等，以避免过多的线程数影响垃圾收集性能。
  		1. 在默认情况下，当CPU数量小于8个，ParallelGCThreads的值等于CPU数量。
  		2. 当CPU数量大于8个，ParallelGCThreads的值等于`3+[5*CPU_Count/8]`.
  	- `-XX:MaxGCPauseMillis`:设置垃圾收集器最大停顿时间（即STW的时间）。单位是毫秒
  		1. 为了**尽可能地**把停顿时间控制在MaxGCPauseMillis以内，收集器在工作时会调整Java堆大小或者其他一些参数
  		2. 对用户来讲，停顿时间越短越好。但是在服务器端，我们注重高并发，整体的吞吐量。所以服务器适合Parallel，进行控制。
  		3. 该参数适用需谨慎
  	- `-XX:GCTimeRatio`:垃圾收集时间占总时间的比例(`=1/(N+1)`).用于衡量吞吐量的大小。
  		1. 取值范围（0，100）。默认值99，也就是垃圾回收时间不超过1%。
  		2. 与`-XX:MaxGCPauseMillis`参数有一定矛盾性。暂停时间越长，Radiocan参数就容易超过设定的比例。
  	- `-XX:+UseAdaptiveSizePolicy`:设置Parallel Scavenge收集器具有自适应调节策略
  		1. 在这种模式下，年轻代的大小，Eden和Survivor的比例，晋升老年代对象年龄等参数会被自动调整，以达到在堆大小，吞吐量和停顿时间之间的平衡点。
  		2. 在手动调优比较困难的场合，可以直接使用这种自适应的方式，仅指定虚拟机的最大堆，目标的吞吐量(GCTimeRatio)和停顿时间（MaxGCPauseMillis）.让虚拟机自己完成调优工作。

  - CMS回收器

  	- `-XX:+UseConcMarkSweepGC`:手动指定使用CMS收集器执行内存回收任务

  		开启该参数后会自动将`-XX:+UseParNewGC`打开。即：`ParNew`(Young区用)+`CMS`(Old区用)+`Serial Old`的组合。

  	- `-XX:CMSInitiatingOccupanyFraction`:设置堆内存使用率的阈值，一旦达到该阈值，便开始进行回收

  		1. JDK5及以前版本的默认值为68，即当老年代的空间使用率达到68%时，会执行一次CMS回收。JDK6及以上版本默认值为92%。
  		2. 如果内存增长缓慢，则可以设置一个稍大的值，大的阈值可以有效降低CMS的触发频率，减少老年代回收的次数可以较为明显地改善应用程序的性能。反之，如果应用程序内存使用率增长很快，则应该降低这个阈值，以避免频繁出发老年代传串行收集器。因此通过该选项便可以有效降低Full GC的执行次数。

  	- `-XX:+UseCMSCompactAtFullCollection`:用于指定在执行完Full GC后对内存空间进行压缩整理，以避免内存碎片的产生，不过由于内存压缩整理过程无法并发执行，所带来的问题就是停顿时间变得更长了。

  	- `-XX:CMSFullGCsBeforeCompaction`:设置在执行多少次Full GC后对内存空间进行压缩。

  	- `-XX:ParallelCMSThreads`:设置CMS的线程数量。

  		1. CMS默认启动的线程数是`(ParallelGCThreads+3)/4`.ParallelGCThreads是年轻代并行收集的线程数。当CPU资源比较紧张时，受到CMS收集器线程的影响，应用程序的性能在垃圾回收阶段可能会非常糟糕。

  	**特别说明**

  	1. JDK9新特性：CMS被标记为Deprecate。
  	2. JDK14新特性：删除CMS垃圾回收器

  - G1回收器

  	- `-XX:MaxGCPauseMillis`:设置期望达到的最大GC停顿时间指标（JVM会尽力实现，但不保证达到）。默认值是200ms
  	- `-XX:ParallelGCThread`:设置STW时GC线程数的值。最多设置为8.
  	- `-XX:ConcGCThreads`:设置并发标记的线程数。将n设置为
  	- `-XX:InitiatingHeapOccupancyPercent`:
  	- `-XX:G1NewSizePercent`:新生代占用整个堆内存的最小百分比（默认5%）
  	- `-XX:G1MaxNewSizePercent`:新生代占用整个堆内存的最大百分比（默60%）

5. GC日志相关选项

	**常用参数**

	- `-verbose:gc`:输出gc日志信息，默认输出到标准输出

		```
		[GC (Allocation Failure)  16301K->13850K(59392K), 0.0132293 secs]
		[GC (Allocation Failure)  30234K->30124K(59392K), 0.0168536 secs]
		[Full GC (Ergonomics)  30124K->29948K(59392K), 0.0112070 secs]
		[Full GC (Ergonomics)  46291K->45951K(59392K), 0.0149830 secs]
		```

	- `-XX:+PrintGC`:等同于`-verbose:gc`,表示打开简化的GC日志

	- **`-XX:+PrintGCDetails`**:**在发生垃圾回收时打印内存回收详细的日志，并在进程退出时输出当前内存各区域分配情况。**

		```
		[GC (Allocation Failure) [PSYoungGen: 16301K->2032K(18432K)] 16301K->13850K(59392K), 0.0124433 secs] [Times: user=0.02 sys=0.05, real=0.01 secs] 
		[GC (Allocation Failure) [PSYoungGen: 18416K->1964K(18432K)] 30234K->30092K(59392K), 0.0163215 secs] [Times: user=0.02 sys=0.07, real=0.02 secs] 
		[Full GC (Ergonomics) [PSYoungGen: 1964K->0K(18432K)] [ParOldGen: 28128K->29948K(40960K)] 30092K->29948K(59392K), [Metaspace: 3699K->3699K(1056768K)], 0.0101056 secs] [Times: user=0.06 sys=0.00, real=0.01 secs] 
		[Full GC (Ergonomics) [PSYoungGen: 16342K->5000K(18432K)] [ParOldGen: 29948K->40950K(40960K)] 46291K->45951K(59392K), [Metaspace: 3699K->3699K(1056768K)], 0.0131390 secs] [Times: user=0.03 sys=0.04, real=0.02 secs] 
		Heap
		 PSYoungGen      total 18432K, used 9978K [0x00000007bec00000, 0x00000007c0000000, 0x00000007c0000000)
		  eden space 16384K, 60% used [0x00000007bec00000,0x00000007bf5be8a8,0x00000007bfc00000)
		  from space 2048K, 0% used [0x00000007bfe00000,0x00000007bfe00000,0x00000007c0000000)
		  to   space 2048K, 0% used [0x00000007bfc00000,0x00000007bfc00000,0x00000007bfe00000)
		 ParOldGen       total 40960K, used 40950K [0x00000007bc400000, 0x00000007bec00000, 0x00000007bec00000)
		  object space 40960K, 99% used [0x00000007bc400000,0x00000007bebfdb50,0x00000007bec00000)
		 Metaspace       used 3705K, capacity 4536K, committed 4864K, reserved 1056768K
		  class space    used 415K, capacity 428K, committed 512K, reserved 1048576K
		```

	- `-XX:+PrintGCTimeStamps`:输出GC发生时的时间戳,**不可以独立使用**，需要配合`-XX:+PrintGCDetails`

	- `-XX:+PrintGCDateStamps`:输出GC发生时的时间戳（以日期的形式）,**不可以独立使用**，

	- `-XX:+PrintHeapAtGC`:每一次GC前和GC后，都打印堆信息

	- `-Xloggc:<file>`:把GC日志写入到一个文件中，而不是打印到标准输出中。

6. 其他参数

	- `-XX:DisbaleExplicitGC`:禁止hotspot执行System.gc().默认禁用
	- `-XX:+UseTLAB`:使用TLAB,默认打开
	- `-XX:+DoEscapeAnalysis`:开启逃逸分析
	- `-XX:+PrintTLAB`:打印TLAB的使用情况
	- `-XX:TLABSize`: 设置TLAB的大小

### 4.3 通过Java代码获取JVM参数

**Runtime获取**

## 5. GC日志

### 5.1 GC日志参数

1. 

### 5.2 GC日志格式

### 5.3 GC日志分析工具































