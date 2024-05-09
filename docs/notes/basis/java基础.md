------
[TOC]

## String，StringBuilder，StringBuffer详解

先来看一下他们底层的设计

* String源码

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0
}
```

* StringBuilder，StringBuffer都继承了AbstractStringBuilder

```java
abstract class AbstractStringBuilder implements Appendable, CharSequence {
    /**
     * The value is used for character storage.
     */
    char[] value;

    /**
     * The count is the number of characters used.
     */
    int count;
     AbstractStringBuilder(int capacity) {
        value = new char[capacity];
    }
    
    public int length() {
        return count;
    }
    public char charAt(int index) {
        if ((index < 0) || (index >= count))
            throw new StringIndexOutOfBoundsException(index);
        return value[index];
    }
    public AbstractStringBuilder append(String str) {
        if (str == null)
            return appendNull();
        int len = str.length();
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }
    public AbstractStringBuilder delete(int start, int end) {
        if (start < 0)
            throw new StringIndexOutOfBoundsException(start);
        if (end > count)
            end = count;
        if (start > end)
            throw new StringIndexOutOfBoundsException();
        int len = end - start;
        if (len > 0) {
            System.arraycopy(value, start+len, value, start, count-end);
            count -= len;
        }
        return this;
    }
}
```

String底层使用了用`final`修饰的`char[] value;`。使用final修饰可以保证对象不可变，也可以理解为常量，线程安全的。

> java9之后对String做了优化，改用byte 数组存储字符串`private final byte[] value;`

StringBuilder,StringBuffer都继承了`AbstractStringBuilder` 从源码中可以看到，它也是使用`char[] value;`来保存字符串，但是没有使用final修饰，所以是可以改变的。

在`AbstractStringBuilder` 中定义了一些字符串的基本操作,`length()`,`charAt()`,`append()`,`delete()`等等。

`StringBuffer` 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。`StringBuilder` 并没有对方法进行加同步锁，所以是非线程安全的。

------

## Synchronized详解

### Synchronized使用方式

1. 修饰静态方法：静态方法属于类，而不是属于对象。相当于给当前类加锁
2. 修饰代码块：同步大括号内的代码
3. 修饰实例方法：synchronized修饰方法和修饰一个代码块类似，只是作用范围不一样，修饰代码块是大括号括起来的范围，而修饰方法范围是整个函数。

> 不要使用Synchronized(String a)，在JVM中字符串常量池具有缓存功能。

### Synchronized底层原理

#### Synchronized同步代码块

```java
public class SynchronizedDemo {
    public void method() {
        synchronized (this) {
            System.out.println("synchronized 代码块");
        }
    }
}
```

通过javap 命令查看 Synchronized类的字节码信息

```
javac SynchronizedDemo.java -- 编译生成class文件
javap -c -s -v -l SynchronizedDemo.class -- 查看字节码信息
```


从字节码中可以看出

synchronized 同步语句块的实现使用的是 monitorenter 和 monitorexit 指令，其中 monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置。当执行 monitorenter 指令时，线程试图获取锁也就是获取 monitor(monitor对象存在于每个Java对象的对象头中，synchronized 锁便是通过这种方式获取锁的，也是为什么Java中任意对象可以作为锁的原因) 的持有权。当计数器为0则可以成功获取，获取后将锁计数器设为1也就是加1。相应的在执行 monitorexit 指令后，将锁计数器设为0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。

#### Synchronized修饰方法

```java
public class SynchronizedDemo2 {
    public synchronized void method() {
        System.out.println("synchronized 方法");
    }
}
```


Synchronized 修饰的方法并没有 monitorenter 指令和 monitorexit 指令，取得代之的确实是 ACC_SYNCHRONIZED 标识，该标识指明了该方法是一个同步方法，JVM 通过该 ACC_SYNCHRONIZED 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。

#### Synchronized锁升级过程

* 在Java1.5之后对锁的实现做了大量优化，如偏向锁、轻量级锁、自旋锁、适应性自旋锁、锁消除、锁粗化等技术来减少锁操作的开销。

  >  无锁-> 偏向锁 -> 轻量级锁 -> 重量级锁
  >
  >  它们会随着竞争的激烈而逐渐升级。注意锁可以升级不可降级，这种策略是为了提高获得锁和释放锁的效率。
* 网上抄一张锁升级过程图
  ![锁升级过程](../../image/basis/synchronized原理.jpg)

##### Java 对象头

以 Hotspot 虚拟机为例，Hotspot 的对象头主要包括两部分数据：Mark Word（标记字段）、Klass Pointer（类型指针）。

**Mark Word**：默认存储对象的 HashCode，分代年龄和锁标志位信息。这些信息都是与对象自身定义无关的数据，所以 Mark Word 被设计成一个非固定的数据结构以便在极小的空间内存存储尽量多的数据。它会根据对象的状态复用自己的存储空间，也就是说在运行期间 Mark Word 里存储的数据会随着锁标志位的变化而变化。

**Klass Point**：对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。

##### 无锁

当对象刚被创建,并且偏向锁还未开始,也无线程访问,则出于无锁状态MarkWord标志位01

##### 偏向锁

MarkWord标志位01（和无锁标志位一样）。偏向锁是通过在bitfields中通过CAS设置当前正在执行的ThreadID来实现的。假设线程A获取偏向锁执行代码块（即对象头设置了ThreadA_ID），线程A同步块未执行结束时，线程B通过CAS尝试设置ThreadB_ID会失败，因为存在锁竞争情况，这时候就需要升级为轻量级锁。**注：偏向锁是针对于不存在资源抢占情况时候使用的锁，如果被synchronized修饰的方法/代码块竞争线程多可以通过禁用偏向锁来减少一步锁升级过程。可以通过JVM参数-XX:-UseBiasedLocking = false来关闭偏向锁。**

##### 轻量级锁

MarkWord标志位00。**轻量级锁是采用自旋锁的方式来实现的，自旋锁分为固定次数自旋锁和自适应自旋锁。****轻量级锁是针对竞争锁对象线程不多且线程持有锁时间不长的场景,** 因为阻塞线程需要CPU从用户态转到内核态，代价很大，如果一个刚刚阻塞不久就被释放代价有大。**具体实现和升级为重量级锁过程：**线程A获取轻量级锁时会把对象头中的MarkWord复制一份到线程A的栈帧中创建用于存储锁记录的空间DisplacedMarkWord，然后使用CAS将对象头中的内容替换成线程A存储DisplacedMarkWord的地址。如果这时候出现线程B来获取锁，线程B也跟线程A同样复制对象头的MarkWord到自己的DisplacedMarkWord中，如果线程A锁还没释放，这时候那么线程B的CAS操作会失败，会继续自旋，当然不可能让线程B一直自旋下去，自旋到一定次数（固定次数/自适应）就会升级为重量级锁。

##### 重量级锁

通过对象内部监视器（monitor）实现，monitor本质前面也提到了是基于操作系统互斥（mutex）实现的，操作系统实现线程之间切换需要从用户态到内核态切换，成本非常高。


