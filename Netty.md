# Netty

## 1. NIO

>
>
>

### 1.1 概述

1. NIO有三大核心部分：**Channel(通道)**,**Buffer(缓冲区)**,**Selector(选择器)**
2. NIO是面向缓冲区，或者面向块的。
3. NIO与BIO的比较
	- BIO以流的方式处理数据，而NIO以块的方式处理数据，块I/O的效率比流I/O高很多
	- BIO是阻塞的，NIO是非阻塞的
	- BIO基于字节流和字符流进行操作，而NIO基于Channel和Buffer进行操作，Selector用于监听多个通道的事件（比如：连接请求，数据到达等
4. Selector,Channel和Buffer的关系
	- 每个Channel都会对应一个Buffer
	- Selector会对应一个线程，一个线程对应多个Channel（连接）
	- 程序切换到哪个Channel是由事件决定的，Event就是一个重要概念
	- Selector会根据不同的事件，在各个不同的通道上切换
	- Buffer就是一个内存块，底层是有一个数组。
	- 数据的读取写入通过Buffer,BIO中要么是输入流，或者是输出流，不能双向。但是NIO的Buffer既可以读也可以写，需要filp()切换
	- Channel是双向的

### 1.2 Buffer

`Buffer`类

```
public abstract class Buffer {
		//标记
	  private int mark = -1;
	  //位置，下一个要被读或写的元素索引，每次读写缓冲区数据时都会改变值，为下次读写做准备
    private int position = 0;
    //表示缓冲区的当前终点，不能对缓冲区超过极限的位置进行读写操作，且极限是可以修改的
    private int limit;
    //容量，即可以容纳的最大数据量；在缓冲区创建时被设定并且不能改变
    private int capacity;
    
    //反转此缓冲区
    public final Buffer flip() {
        limit = position;//读数据
        position = 0;
        mark = -1;
        return this;
    }
    
    //返回此缓冲区的容量
    public final int capacity() {
        return capacity;
    }
    //返回此缓冲区的位置
    public final int position() {
        return position;
    }
    //设置此缓冲区的位置
    public final Buffer position(int newPosition) {
        if ((newPosition > limit) || (newPosition < 0))
            throw new IllegalArgumentException();
        position = newPosition;
        if (mark > position) mark = -1;
        return this;
    }
    //返回此缓冲区的限制
    public final int limit() {
        return limit;
    }
    //设置此缓冲区的限制
    public final Buffer limit(int newLimit) {
        if ((newLimit > capacity) || (newLimit < 0))
            throw new IllegalArgumentException();
        limit = newLimit;
        if (position > newLimit) position = newLimit;
        if (mark > newLimit) mark = -1;
        return this;
    }
    //清除此缓冲区，即将各个标记恢复到初始状态，但是数据并没有真正擦除
    public final Buffer clear() {
        position = 0;
        limit = capacity;
        mark = -1;
        return this;
    }
    //告知position和limit之间是否有元素
    public final boolean hasRemaining() {
        return position < limit;
    }
    
    //===JDK1.6时引入的api
    //告知此缓冲区是否具有可访问的底层实现数组
    public abstract boolean hasArray();
    
    //返回此缓冲区的底层实现数组
    public abstract Object array();
    
}      
```

它们的关系：`mark<=position<=limit<=capacity`

`Buffer`的子类中，对Java中的基本数据类型（boolean除外），都有一个`Buffer`类型与之对应，最常用的自然是`ByteBuffer`类（二进制数据），以`ByteBuffer`为例，

```
public abstract class ByteBuffer extends Buffer implements Comparable<ByteBuffer> {
		//实际存放数据的位置，即缓冲区
		final byte[] hb;  
		
		//创建直接缓冲区
		public static ByteBuffer allocateDirect(int capacity) {
        return new DirectByteBuffer(capacity);
    }
    //创建缓冲区，并设置初始容量,它是一个静态方法
    public static ByteBuffer allocate(int capacity) {
        if (capacity < 0)
            throw new IllegalArgumentException();
        return new HeapByteBuffer(capacity, capacity);
    }
    
    //从当前位置position上get，get之后，position会自动+1
    public abstract byte get();
    
    //从指定位置get
    public abstract byte get(int index);
    
    //从当前position上put,put之后，position会自动+1
    public abstract ByteBuffer put(byte b);
    
    //从指定位置上put
    public abstract ByteBuffer put(int index, byte b);
}      
```

### 1.3 Channel

1. NIO的通道类似于流，但有以下区别：

	- 通道可以同时进行读写，而流只能读或写

	- 通道可以实现异步读写数据

	- 通道可以从缓冲读数据，也可以写数据到缓冲

2. Channel是NIO中的一个接口
3. 常见的Channel类有：
	- `FileChannel`:用于文件的数据读写
	- `DatagramChannel`:用于UDP的数据读写
	- `ServerSocketChannel`:用于TCP的数据读写,类似`ServerSocket`
	- `SocketChannel`:用于TCP的数据读写,类似`Socket`

`Channel`接口

```
public interface Channel extends Closeable {

	public boolean isOpen();
	
	public void close() throws IOException;
}
```

**`FileChannel`类**常用API

```
public abstract class FileChannel extends AbstractInterruptibleChannel
    implements SeekableByteChannel, GatheringByteChannel, ScatteringByteChannel {
    
    //从通道读取数据并放到缓冲区中，通道->缓冲区
    public abstract int read(ByteBuffer dst) throws IOException;
    
    //把缓冲区的数据写到通道中,缓冲区->通道
    public abstract int write(ByteBuffer src) throws IOException;
    
    //从目标通道中复制数据到当前通道,目标通道->当前通道
    public abstract long transferFrom(ReadableByteChannel src,long position, long count)
				throws IOException;
				
		//把数据从当前通道复制给目标通道,当前通道->目标通道
		public abstract long transferTo(long position, long count, WritableByteChannel target)
        throws IOException;
        
}
```

`FileOutputStream`内置了`FileChannel`

```
public class FileOutputStream extends OutputStream {
	private FileChannel channel;
}
```

同样，`FileInputStream`内置了`FileChannel`

```
public class FileInputStream extends InputStream {
	private FileChannel channel = null;
}
```

**`ServerSocketChannel`类**，在服务器端监听新的客户端Socket连接，产生`SocketChannel`

```
public abstract class ServerSocketChannel extends AbstractSelectableChannel
    implements NetworkChannel {
  //得到一个ServerSocketChannel通道
  public static ServerSocketChannel open();
  
  //接受一个连接，返回代表这个连接的通道对象
  public abstract SocketChannel accept();
  
  //设置服务器端端口号
  public final ServerSocketChannel bind(SocketAddress local)；
  
  //设置阻塞或非阻塞模式，取值false表示采用非阻塞模式
  public final SelectableChannel configureBlocking(boolean block)；
  
  //注册一个选择器并设置监听事件
  public final SelectionKey register(Selector sel, int ops)；
}
```

ServerSocketChannel和SocketChannel的关系如下：

![](../../Pictures/F9B2E5DA-1F90-45C6-AB55-709DA1672908.png)

**`SocketChannel`类**,具体负责进行读写操作。NIO把缓冲中的数据写入到通道，或者把通道里的数据读到缓冲区。

```
public abstract class SocketChannel extends AbstractSelectableChannel
    implements ByteChannel, ScatteringByteChannel, GatheringByteChannel, NetworkChannel {
  //得到一个SocketChannel通道
  public static SocketChannel open();
  //连接服务器
  public abstract boolean connect(SocketAddress remote);
  //往通道写数据
  public abstract int read(ByteBuffer dst);
  //往通道读数据
  public abstract int write(ByteBuffer src);
  //关闭通道
  public void close();
}
```

### 1.4 Buffer和Channel注意事项

1. ByteBuffer支持类型化的put和get，put放入的是什么数据类型，get就应该使用相应的数据类型来取出，否则可能有`BufferUnderflowException`异常。
2. 可以将一个普通Buffer转成只读Buffer,如果放入数据会抛出`ReadOnlyBufferException`异常
3. NIO还提供了`MappedByteBuffer`,它继承自`ByteBuffer`,可以让文件直接在内存中（堆外的内存）中进行修改，而如何同步到文件由NIO来完成。
4. NIO还支持通过多个Buffer(即Buffer数组)完成读写操作，即Scattering和Gatering

### 1.5 Selector

1. **`Selector`（选择器，也叫多路复用器）能够检测多个注册的通道上是否有事件发生**(注意：多个Channel以事件的方式可以注册到同一个Selecotr),如果有时间然后针对每个事件进行相应的处理。这样就可以只用一个单线程去管理多个通道，也就是管理多个连接和请求
2. 只有在连接真正有读写事件发生时，才会进行读写。
3. 避免了多线程之间的上下文切换导致的开销

`Selector`类

```
public abstract class Selector implements Closeable {

	//得到一个选择器对象
	public static Selector open();
	
	//监控所有注册的通道，当其中有IO操作可以进行时，将对应的SelectionKey加入到内部集合中并返回
	//参数用于设置超时时间
	public abstract int select(long timeout)；
	
	//selector中所有注册的SelectionKey
	public abstract Set<SelectionKey> keys();
	
	//从内部集合中得到所有的SelectionKey,selector中有事件发生的SelectionKey
	public abstract Set<SelectionKey> selectedKeys();
}
```

它的实现类`SelectorImpl`

```
public abstract class SelectorImpl extends AbstractSelector {

		protected Set<SelectionKey> selectedKeys = new HashSet();
}
```

`SelectionKey`，表示Selector和网络通道的注册关系，共`OP_READ`，`OP_WRITE`，`OP_CONNECT`，`OP_ACCEPT`四种。

```
public abstract class SelectionKey{
	//代表读操作
	public static final int OP_READ = 1 << 0;
	//代表写操作
	public static final int OP_WRITE = 1 << 2;
	//代表连接已经建成
	public static final int OP_CONNECT = 1 << 3;
	//有新的网络连接可以accept
	public static final int OP_ACCEPT = 1 << 4;
	
	//通过SelectionKey反向获取channel,得到与之关联的通道
	public abstract SelectableChannel channel();
	//得到与之关联的Selector对象
	public abstract Selector selector();
	//得到与之关联的共享数据
	public final Object attachment()；
	//设置或改变监听事件
	public abstract SelectionKey interestOps(int ops);
	//是否可以accept
	public final boolean isAcceptable() {
        return (readyOps() & OP_ACCEPT) != 0;
    }
  //是否可以读
  public final boolean isReadable() {
        return (readyOps() & OP_READ) != 0;
    }
  //是否可以写
  public final boolean isWritable() {
        return (readyOps() & OP_WRITE) != 0;
    }
}
```

**Selector,SelectionKey,ServerSocketChannel和SocketChannel的关系**

- 当客户端连接时，会通过`ServerSocketChannel`得到`SocketChannel`
- 将`SocketChannel`注册到`Selector`上,使用`register(Selector sel, int ops)`,一个`Selector`上可以注册多个`SocketChannel`
- 注册后返回一个`SelectionKey`，会和该`Selector`关联（集合）
- `Selector`进行监听`selecct`方法,返回有事件发生的通道的个数
- 进一步得到各个`SelectionKey(有事件发生)`
- 再通过`SelectionKey`反向获取`SocketChannel`
- 可以通过得到的`SocketChannel`，完成业务处理

### 1.6 零拷贝

1. 在Java程序中，常见的零拷贝有mmap(内存映射)和sendFile。
2. DMA(direct memory access):直接内存拷贝，不使用CPU。
3. 零拷贝从操作系统看，指的是没有CPU拷贝
4. 零拷贝不仅带来更少的数据复制，还能带来其他的性能优势，例如更少的上下文切换。**是进行网络数据传输的非常重要的优化手段。**

**mmap优化**

1. mmap通过内存映射，将文件映射到内核缓冲区，同时，用户空间可以共享内核空间的数据。这样，在进行网络传输时，就可以减少内核空间到用户控件的拷贝次数。
2. 三次拷贝，三次状态切换

**sendFile优化**

1. Linux2.1版本提供了sendFile函数，其基本原理如下：数据根本不经过用户态，直接从内核缓冲区进入到SocketBuffer，同时，由于和用户态完全无关，就减少了一次上下文切换。三次拷贝，两次状态切换
2. Linux在2.4版本中，做了一些修改，避免了从内核缓冲区拷贝到SocketBuffer的操作，直接拷贝到协议栈，从而再一次减少了数据拷贝。这里其实有一次cpu拷贝kernel buffer->socket buffer.但是拷贝的信息很少，比如length,offset,消耗低，可以忽略。两次拷贝，两次状态切换。

**mmap和sendFile的区别**

- mmap适合小数据量的读写，sendFile适合大文件传输。
- mmap需要3次上下文切换，3次数据拷贝；sendFile需要2次上下文切换，最少2次数据拷贝。
- senfFile可以利用DMA方式，减少CPU拷贝，mmap则不能（必须从内核拷贝到Socket缓冲区）

## 2. Netty模型

### 2.1 

1. Netty抽象出两组线程池，BossGroup专门负责接收客户端连接，WorkGroup专门负责网络读写操作
2. NioEventLoop表示一个不断循环执行处理任务的线程，每个NioEventLoop都有一个selector,用于监听绑定在其上的socket网络通道
3. NioEventLoop内部采用串行化设计，从消息的读取->解码->处理->发送，始终由IO线程NioEventLoop负责

**组件梳理**

- NioEventLoopGroup下包含多个NioEventLoop
- 每个NioEventLoop中包含一个Selector,一个taskQueue
- 每个NioEventLoop的Selector上可以注册监听多个NioChannel
- 每个NioChannel只会绑定在唯一的NioEventLoop上
- 每个NioChannel都会绑定一个自己的ChannelPipeline

### 2.2 任务队列

任务队列中的Task有3种典型使用场景

1. 用户程序自定义的普通任务。
2. 用户自定义定时任务
3. 非当前Reactor线程调用Channel的各种方法

### 2.3 异步模型

1. 
2. Netty中的I/O操作是异步的，包括Bind,Write,Connect等操作会简单的返回一个ChannelFuture
3. 调用者并不能立刻获得结果，而是通过Future-Listener机制，用户可以方便的主动获取或者通过通知机制获得IO操作结果。
4. Netty的异步模型是建立在future和callback的基础之上。callback就是回调。Future的核心思想是：假设一个方法fun,执行过程可能非常耗时，等待fun返回显然不合适。那么可以在调用fun的时候，立马返回一个Future，后续可以通过Future去监控方法fun的处理结果（即：Future-Listener机制）

**Future-Listener机制**

1. 当Future对象刚刚创建时，处于非完成状态，调用者可以通过返回的ChannelFuture来获取操作执行的状态，注册监听函数来执行完成后的操作。 
2. 常见操作如下：
	- 通过isDone方法来判断当前操作是否完成
	- 通过isSuccess方法来判断已完成的当前操作是否成功。
	- 通过getCause方法来获取已完成的当前操作失败的原因
	- 通过isCancelled方法来判断已完成的当前操作是否被取消
	- 通过addListener方法来注册监听器，当操作已完成(isDone方法返回完成),将会通知指定的监听期；如果Future对象已完成，则通知指定的监听器。

### 2.4 Netty核心组件

#### 2.4.1 Bootstrap,ServerBootstrap

1. Bootstrap是引导的意思，一个Netty应用通常由一个Bootstrap开始，主要作用是配置整个Netty程序，串联各个组件，Netty中Bootstrap类是客户端程序的启动引导类，ServerBootstrap是服务器端启动引导类
2. 常用方法，链式编程

```
public class Bootstrap/ServerBootstrap extends AbstractBootstrap<Bootstrap/ServerBootstrap, ServerChannel> {
	
	//该方法用于服务器端，用来设置两个EventLoop
	public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup);
	
	//该方法用于客户端，用来设置一个EventLoopGroup
	public B group(EventLoopGroup group);
	
	//该方法用来设置一个服务器端的通道实现
	public B channel(Class<? extends C> channelClass);
	
	//用来给ServerChannel添加配置
	public <T> B option(ChannelOption<T> option, T value);
	
	//用来给接收到的通道添加配置
	public <T> ServerBootstrap childOption(ChannelOption<T> childOption, T value);
	
	//用来设置业务处理类（自定义的handler）
	public ServerBootstrap childHandler(ChannelHandler childHandler);
	
	//用于服务器端，设置占用的端口号
	public ChannelFuture bind(int inetPort);
	
	//用于客户端，用来连接服务器
	public ChannelFuture connect(InetAddress inetHost, int inetPort)
}
```

#### 2.4.2 Future,ChannelFuture

1. Netty中所有的IO操作都是异步的，不能立刻得知消息是否被正确处理。但是可以过一会等它执行完成或者直接注册一个监听，具体的实现就是通过Future和ChannelFuture,它们可以注册一个监听，当操作执行成功或失败时监听会自动触发注册的监听事件
2. 常见的方法

```
public interface ChannelFuture extends Future<Void> {

		//返回当前正在进行IO操作的通道
    Channel channel();
    //等待异步操作执行完毕
    ChannelFuture sync() throws InterruptedException;
}
```

#### 2.4.3 Channel

1. Channel提供异步的网络I/O操作（比如建立连接，读写，绑定端口），异步调用意味着任何I/O调用都将立即返回，并且不保证在调用结束时所请求的I/O操作已完成。
2. 调用立即返回一个ChannelFuture实例，通过注册监听器到ChannelFuture上，可以I/O操作成功，失败或取消时回调通知调用方
3. 不同协议，不同阻塞类型的连接都有不同的Channel类型与之对应，常用的Channel类型：
	- NioSocketChannel:异步的客户端TCP Socket连接
	- NioServerSocketChannel:异步的服务器端TCP Socket连接
	- NioDatagramChannel:异步的UDP连接
	- NioSctpChannel:异步的客户端Sctp连接
	- NioSctpServerChannel:异步的Sctp服务器端连接，这些通道涵盖了UDP和TCP网络IO 以及文件IO

#### 2.4.4 Selector

1. Netty基于Selector对象实现I/O多路复用，通过Selector一个线程可以监听多个连接的Channel事件
2. 当向一个Selector中注册Channel后，Selector内部的机制就可以自动不断地查询(Select)这些注册的Channel是否已有就绪的I/O事件（例如可读，可写，网络连接完成等），这样程序就可以很简单地使用一个线程高效地**管理多个Channel**

#### 2.4.5 ChannelHandler

1.  ChannelHandler是一个接口，处理I/O事件或者拦截I/O操作，并将其转发到其ChannelPipeline（业务处理链）中的下一个处理程序。
2. ChannelPipeline提供了ChannelHandler链的容器。以客户端应用程序为例，如果事件的运动方向是从客户端到服务器端的，那么我们称这些事件为出站的，即客户端发送给服务器端的数据会通过pipeline中的一系列ChannelOutboundHandler,并被这些Handler处理，反之称为入站。
3.  我们经常需要自定义一个Handler类去继承ChannelInboundHandlerAdapter,然后通过重写相应方法实现业务逻辑

```
public class ChannelInboundHandlerAdapter extends ChannelHandlerAdapter implements ChannelInboundHandler {
		//通道就绪事件
		public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ctx.fireChannelActive();
    }
    //通道读取数据事件
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ctx.fireChannelRead(msg);
    }
    //数据读取完毕事件
     public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.fireChannelReadComplete();
    }
}
```



#### 2.4.6 Pipeline和ChannelPipeline

1. ChannelPipeline是一个Handler的集合，它负责处理和拦截inbound或者outbound的事件和操作，相当于一个贯穿Netty的链（也可以这样理解：ChannlePipeline是保存ChannelHandler的List,用于处理或拦截Channel的入站事件和出站操作）
2. ChannelPipeline实现了一种高级形式的拦截过滤器模式，使用户可以完全控制事件的处理方式，以及Channel中各个的ChannelHandler如何相互交互。
3. 一个Channel包含了一个ChannelPipeline,而ChannelPipeline中又维护了一个由ChannelHandlerContext组成的双向链表，并且每个ChannelHandlerContext中又关联着一个ChannelHandler
4. 常用方法

```
public interface ChannelPipeline extends ChannelInboundInvoker, ChannelOutboundInvoker, Iterable<Entry<String, ChannelHandler>> {

	//把一个业务处理类(handler)添加到链中的第一个位置
	ChannelPipeline addFirst(ChannelHandler... var1);
	//一个业务处理类(handler)添加到链中的最后一个位置
	ChannelPipeline addLast(ChannelHandler... var1);
}
```

#### 2.4.7 ChannelHandlerContext

1. 保存Channel相关的所有上下文，同时关联一个ChannelHandler对象
2. 即ChannelHandlerContext中包含了一个具体的事件处理器ChannelHandler,同时ChannelHandlerContext中也绑定了对应的pipeline和Channel的信息，方便对ChannelHandler进行调用
3. 常用方法

```

```

#### 2.4.8 ChannelOption

1. Netty在创建Channel实例后，一般都需要设置ChannelOption参数
2. ChannelOption参数如下：
	- ChannelOption.SO_BACKLOG:对应TCP/IP协议listen函数中的backlog参数，用来初始化服务器可连接队列大小。服务器处理客户端连接请求是顺序处理的，所以同一时间只能处理一个客户端连接请求。多个客户端来的时候，服务端将不能处理的客户端连接请求放在队列中等待处理，backlog参数指定队列的大小
	- ChannelOption.SO_KEEPALIVE:一直保持连接活动状态

#### 2.4.9 EventLoopGroup及其实现类NioEventLoopGroup

1. EventLoopGroup是一组EventLoop的抽象，Netty为了更好的利用多核CPU资源，一般会有多个EventLoop同时工作，每个EventLoop维护着一个Selector实例
2. EventLoopGroup提供next接口，可以从组里面按照一定规则获取其中一个EventLoop来处理任务。在Netty服务器端编程中，我们一般提供两个EventLoopGroup,例如:BossEventLoopGroup和WorkerEventLoopGroup
3. 通常一个服务端口即一个ServerSocketChannel对应一个Selector和一个EventLoop线程。BossEventLoop负责接收客户端的连接并将SocketChannel交给WorkerEventLoopGroup来进行IO处理。
4. 常用方法

```
//断开连接，关闭线程
Future<?> shutdownGracefully();
```



