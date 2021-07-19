# NIO

>​	NIO是一种同步非阻塞的I/O模型，NIO提供了与传统BIO模型中的Socket和ServerSocket相对应的SocketChannel和ServerSocketChannel两种不同的套接字通道实现，两种通道都支持阻塞和非阻塞两种模式。
>
>​	AIO是NIO2。在Java7中引入了NIO的改进版NIO2,它是异步非阻塞的IO模型。

**标准IO是阻塞式的,NIO是非阻塞式的，**阻塞与非阻塞是相较于网络通信而言的。

​			NIO的非阻塞模式，使用了Selector选择器,会把每一个通道都注册到选择器上，选择器的作用是监控这些通道的IO状况。

### 使用NIO完成网络通信的三个核心

1. **通道(Channel)**:负责连接

```
java.nio.channels.Channel接口
	|--SelectableChannel
		｜--SocketChannel
		｜--ServerSocketChannel
		｜--DatagramChannel
		
		｜--Pipe.SinkChannel
		｜--Pipe.SourceChannel
```

2. **缓冲器(Buffer)**:负责数据的存取
3. **选择器(Selector)**:是SelectableChannel的多路复用器，用于监控SelectableChannel的IO状况

### 通道



### 缓冲区

**缓冲区存取数据的两个核心方法**

```
put():存入数据到缓冲区中
get():获取缓冲区中的数据
```

**缓冲区的四个核心属性**

```
capacity: 容量，表示缓冲区中最大存储数据的容量.一旦声明不能改变（底层数组）
limit: 界限，表示缓冲区中可以操作数据的大小。(limit后面的数据不能进行读写)
position: 位置，表示缓冲区中正在操作数据的位置
mark: 标记，表示记录当前position的位置，可以通过reset()恢复到mark的位置

他们的关系是
0<=mark<=position<=limit<=capacity
```

**直接缓冲区与非直接缓冲区**

- **非直接缓冲区**:通过allocate()方法分配缓冲区，将缓冲区建立在JVM的内存中
- **直接缓冲区**：通过allocateDirect()方法分配直接缓冲区，将缓冲区建立在物理内存中。可以提高效率











