# JVM

## 1. 上篇：



## 2. 中篇：

### 2.1 类的加载器

`ClassLoader`类的主要方法：

1. `getParent`,返回该类加载器的超类加载器

```
public final ClassLoader getParent();
```

2. `loadClass`,加载名称为`name`的类，返回结果为`java.lang.Class`类的实例，如果找不到类，则抛出`ClassNotFoundException`异常,**该方法中的逻辑就是双亲委派机制的实现**

```
public Class<?> loadClass(String name) throws ClassNotFoundException;

//重载方法，假设通过ClassLoader.getSystemClassLoader().loadClass("com.hhp.User")调用
//参数：resolve:true-加载class的同时进行解析操作。
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException
    {
    		//同步操作，保证只能加载一次
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            //首先，在缓存中判断是否已经加载同名的类。
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                		//获取当前类加载器的父类加载器
                    if (parent != null) {
                    		//如果存在父类加载器，则调用父类加载器进行类的加载，递归。
                        c = parent.loadClass(name, false);
                    } else {
                    		//parent为null:父类加载器是引导类加载器
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }
								//当前类的加载器的父类加载器未加载此类 or 当前类的加载器未加载
                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    //如果仍未找到，调用当前ClassLoader的findClass()
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            //是否进行解析操作
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

3. `findClass`方法体在子类`URLClassLoader`中被重写。查找二进制名称为`name`的类，返回结果为`java.lang.Class`类的实例，这是一个受保护的方法，JVM鼓励我们重写此方法，需要自定义加载器遵循双亲委派机制，该方法会在检查完父类加载器

```
protected Class<?> findClass(String name) throws ClassNotFoundException {
        throw new ClassNotFoundException(name);
    }
```

4. `defineClass`根据给定的字节数组b转换为Class的实例，off和len参数表示实际Class信息在byte数组中的位置和长度，其中byte数组b是ClassLoader从外部获取的。这是受保护的方法，只有在自定义ClassLoader子类中可以使用。**一般情况下，在自定义类加载器时，会直接覆盖ClassLoader的findClass()方法并编写加载规则，取得要加载类的字节码后转换成流，然后调用defineClass()方法生成类的Class对象**

```
protected final Class<?> defineClass(String name, byte[] b, int off, int len);
```

`SecureClassLoader与URLClassLoader`:这两个类是与`ClassLoader`有继承关系的类，`SecureClassLoader`扩展了`ClassLoader`,新增了几个与使用相关的代码源（对代码源的位置及其证书的验证）和权限定义类验证（主要指对class源码的访问权限）的方法，一般我们不会直接跟这个类打交道，更多是与它的子类`URLClassLoader`有所关联

`ClassLoader`是一个抽象类，很多方法是空的没有实现，比如`findClass()`等。而`URLClassLoader`这个实现类为这些方法提供了具体的实现。**在编写自定义类的加载器时，如果没有太过于复杂的需求，可以直接继承URLClassLoader类**，这样就可以避免自己去编写`findClass()`方法及其获取字节码流的方式，使自定义类加载器编写更加简洁。

`Class.forName()`与`ClassLoader.loadClass()`的区别？:

- `Class.forName()`:是一个静态方法，最常用的是`Class.forName(String className)`;根据传入的类的全限定名返回一个Class对象。**该方法在将Class文件加载到内存的同时，会执行类的初始化。**
- `ClassLoader.loadClass()`:这是一个实例方法，需要一个ClassLoader对象来调用该方法。**该方法将Class文件加载到内存中，并不会执行类的初始化，直到这个类第一次使用时才进行初始化**

### 2.2 用户自定义类加载器

**实现方式**：Java提供了抽象类`java.lang.ClassLoader`,所有用户自定义的类的加载器都应该继承自`ClassLoader`类。

在自定义ClassLoader的子类时，我们通常会有两种做法

- 方式一：重写`loadClass()`方法
- 方式二：重写`findClass()`方法-->推荐

**对比**：这两种方法本质上差不多，毕竟`loadClass()`也会调用`findClass()`。但是从逻辑上讲我们最好不要直接修改`loadClass()`的内部逻辑。建议的做法是只在`findClass()`里重写自定义类的加载方法，根据参数指定类的名字，返回对应的`Class`对象的引用。

- 当编写好自定义类加载器后，便可以在程序中调用`loadClass()`来实现类加载操作。

为什么要自定义类加载器？

- 隔离加载类

	在某些框架内进行中间件与应用的模块隔离，把类加载到不同的环境。（类的仲裁->类冲突）

- 修改类加载的方式

	类的加载模型并非强制，除Bootstrap外，其他的加载并非一定要引入，或者根据实际情况在某个时间点按需进行动态加载

- 扩展加载源

	比如从数据库，网络，甚至电视机机顶盒进行加载

- 防止源码泄漏

常见的场景

- 实现类似进程内隔离，类加载器实际上用作不同的命名空间，以提供类似容器，模块化的效果。
- 应用需要从不同的数据源获取类定义信息。

```
package com.hhp.classLoader;

import java.io.BufferedInputStream;
import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.io.IOException;

/**
 * 自定义ClassLoader
 */
public class MyClassLoader extends ClassLoader{

    private String byteCodePath;

    public MyClassLoader(String byteCodePath) {
        this.byteCodePath = byteCodePath;
    }

    public MyClassLoader(ClassLoader parent, String byteCodePath) {
        super(parent);
        this.byteCodePath = byteCodePath;
    }

    @Override
    protected Class<?> findClass(String ClassName) throws ClassNotFoundException {
        BufferedInputStream bufferedInputStream = null;
        ByteArrayOutputStream byteArrayOutputStream = null;
        try {
            //获取字节码文件的完整路径
            String fileName = byteCodePath + ClassName + ".class";
            //获取输入流
            bufferedInputStream = new BufferedInputStream(new FileInputStream(fileName));
            //获取输出流
            byteArrayOutputStream = new ByteArrayOutputStream();
            //具体读入数据并写出的过程
            int len;
            byte[] data = new byte[1024];
            while ((len = bufferedInputStream.read(data)) != -1) {
                byteArrayOutputStream.write(data, 0, len);
            }
            //获取内存中完整的字节数据
            byte[] byteCodes = byteArrayOutputStream.toByteArray();
            //调用defineClass(),将字节数组的数据转换为Class的实例
            Class clazz = defineClass(null, byteCodes, 0, byteCodes.length);
            return clazz;
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (bufferedInputStream != null) bufferedInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (byteArrayOutputStream != null) byteArrayOutputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        return null;
    }
}

```

```
package com.hhp.classLoader;

public class MyClassLoaderTest {
    public static void main(String[] args) {
        MyClassLoader loader = new MyClassLoader("/Users/haipenghou/IdeaProjects/JVM/src/com/hhp/");
        try {
            Class clazz = loader.loadClass("JavapTest");
            System.out.println("加载此类的类的加载器为："+ clazz.getClassLoader().getClass().getName());
            System.out.println("加载当前JavapTest类的类的加载器的父类加载器为：" +
                    clazz.getClassLoader().getParent().getClass().getName());
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```



### 2.3 双亲委派机制

类加载器用来把类加载到Java虚拟机中，从JDK1.2开始，类的加载过程采用双亲委派机制。

**定义**：如果一个类加载器在接到加载类的请求时，它不会自己尝试去加载这个类，而是把这个请求任务委派给父类加载器去完成，依次递归，如果父类加载器可以完成类加载任务，就成功返回。只有父类加载器无法完成此加载无法完成加载任务时，才自己去加载。

1. 引导类加载器`Bootstrap ClassLoader`
2. 扩展类加载器`Extension ClassLoader`
3. 应用程序类加载器`Application ClassLoader`

4. 用户自定义类加载器`Custom ClassLoader`

**双亲委派机制的优势**

- 避免了类的重复加载，确保一个类的全局唯一性
- 保护程序安全，防止核心API被随意篡改

**双亲委派机制的弊端**

检查类是否加载的委托过程是单向的，这个方式虽然从结构上说比较清晰，使各个ClassLoader的职责非常明确，但是同时会带来一个问题，即**顶层的ClassLoader无法访问底层的ClassLoader所加载的类**。

**破坏双亲委派机制**

1. 第一次破坏：双亲委派机制的第一次"被破环"其实发生在双亲委派机制模型出现之前-即JDK1.2之前。

2. 第二次破坏：线程上下文类加载器

	双亲委派机制的第二次"被破坏"是有这个模型自身的缺陷导致的，双亲委派很好地解决了各个类加载器协作时基础类型的一致性问题，基础类型之所以被称为"基础"，是因为它们总是作为被用户代码继承，调用的API存在，但程序设计往往没有绝对不变的完美规则，**如果有基础类型又要调用用户的代码，该怎么办呢？**

	为了解决这个困境，Java的设计团队只好引入了一个不太优雅的设计：线程上下文类加载器（Thread Context ClassLoader）。这个类加载器可以通过`java.lang.Thread`类的`setContextClassLoader()`进行设置，如果创建线程时还未设置，它将会从父线程中继承一个，如果在应用程序的全局范围内都没有设置过的话，那这个类加载器默认就是应用程序类加载器。

3. 第三次破坏：双亲委派机制的第三次"被破环"是由于用户对程序动态性的追求而导致的。如：**代码热替换（Hot Swap）,模块热部署(Hot Deployment)等**

## 3. 下篇：

