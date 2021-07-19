# JUC

## 1. JMM

共享变量：存储在堆内存中（堆内存在线程之间共享）的实例域，静态域和数组元素

## 2. Volatile

### 2.1 volatile的三大特性

1. **保证可见性**

2. **不保证原子性**

	问题：如果不加lock和synchronized,怎样保证原子性？

	解决方案：使用**原子类**，解决原子性问题

3. **禁止指令重排**

### 2.2 内存屏障

CPU指令，作用：

1. 保证特定的操作的执行顺序
2. 可以保证变量的内存可见性

利用这些特性，volatile实现了可见性和禁止指令重排

## 3. 单例模式



## 4. CAS

java无法操作内存，java可以调用c++(native),c++可以操作内存

```

```



## 5. 原子引用

**解决ABA问题**，对应的思想是乐观锁

## 6. 锁

### 6.1 公平锁与非公平锁

公平锁：线程必须排队，

非公平锁：无论synchronized还是lock，默认都是非公平锁。

### 6.2 可重入锁

同步方法里调用了同步方法，获得外层同步方法的锁后，内层同步方法的锁也会获得。

### 6.3 自选锁

```
/**
 * 自旋锁,使用CAS实现
 */
public class SpinLock {

    //
    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    //加锁
    public void myLock() {
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName() + "==> mylock");

        //自旋锁
        while (!atomicReference.compareAndSet(null, thread)) {

        }
    }

    //解锁
    public void myUnLock() {
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName() + "==> myUnlock");

        atomicReference.compareAndSet(thread, null);
    }
}
```

### 6.4 读写锁

读可以被多个线程同时读，写的时候只能有一个线程写。

## 7. 阻塞队列

写入：如果队列满了，就必须阻塞等待

取：如果队列是空的，必须阻塞等待生产

`Queue`与`List`和`Set`都继承自`Collection`

`BlockingQueue`,`Deque（双端队列）`和`AbstractQueue（非阻塞队列）`都继承自`Queue`

`ArrayBlockingQueue`和`LinkedBlockingQueue`实现了`BlockingQueue`

**四组API**

|   **方式**   | 抛出异常 | 有返回值，不抛出异常 | 阻塞等待 | 超时等待 |
| :----------: | :------: | :------------------: | :------: | :------: |
|     添加     |   add    |        offer         |   put    |  offer   |
|     移除     |  remove  |         poll         |   take   |   poll   |
| 检测队首元素 | element  |         peek         |          |          |

**同步队列**`SynchronousQueue`

7个阻塞队列之一，不存储元素，每一个put操作必须等待一个take操作，否则不能继续添加元素

## 8. 线程池

### 8.1 三大方法

### 8.2 七大参数

### 8.3 四种拒绝策略

### 8.4 Executor框架

`Executor`

1. `ThreadPoolExecutor`:以下三个`ThreadPoolExecutor`可以通过工厂类`Executors`来创建

  - `FixedThreadPool`:

  	```
  	new ThreadPoolExecutor(nThreads, nThreads,0L, TimeUnit.MILLISECONDS,
  	                                      new LinkedBlockingQueue<Runnable>());
  	```

  - `SingleThreadExecutor`:

  	```
  	new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS,
  	                                    new LinkedBlockingQueue<Runnable>())
  	```

  - `CachedThreadPool`:

  	```
  	new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS,
  	                                      new SynchronousQueue<Runnable>());
  	```

  可见，底层都是创建`ThreadPoolExecutor`对象，所以创建线程池的时候，为了避免`OOM`，可以直接`new`一个`ThreadPoolExecutor`对象.
2. `ScheduledThreadPoolExecutor`:

## 9. 异步回调

## 10. 集合类不安全

`HashSet`的底层是什么？

```
private transient HashMap<E,Object> map;
private static final Object PRESENT = new Object();//不变的值
public HashSet() {
   map = new HashMap<>();
}
//add set 本质是map key,key是不能重复的。
public boolean add(E e) {
	return map.put(e, PRESENT)==null;
}
```

### 10.1 List

并发下List不安全，会抛出`java.util.ConcurrentModificationException`,**并发修改异常**

解决方案

```
List<String> list = Collections.synchronizedList(new ArrayList<>();
```

或

```
List<String> list = new CopyOnWriteArrayList<>();
```

JUC下List不安全的解决方案

```
List<String> list = new CopyOnWriteArrayList<>();
```

### 10.2 Set

### 10.3 Map

```
Map<String, String> map = new HashMap<>();
等价于
Map<String, String> map = new HashMap<>(16, 0.75f);//初始容量和加载因子
```

## 11. 常用辅助类

### 11.1 CountDownLatch

作用：

每次有线程调用`countDown()`,计数器数量-1，假设计数器变为0，`
countDownLatch.await()`就会被唤醒

```
CountDownLatch countDownLatch = new CountDownLatch(6);//
countDownLatch.countDown();//数量-1
countDownLatch.await();//等待计数器归零，然后再向下执行
```

### 11.2 CyclicBarrier

作用：同步屏障

```
CyclicBarrier cyclicBarrier = new CyclicBarrier(7, ()->{
    System.out.println("召唤神龙成功");
});
cyclicBarrier.await();
```

### 11.3 Semaphore

作用：信号量

```
Semaphore semaphore = new Semaphore(3);
semaphore.acquire();
semaphore.release();
```

## 12 ForkJoin



## 13 生产者和消费者

防止虚假唤醒：使用`while`而不是`if`

