# 一个线程OOM后，其他线程还能运行吗

首先解释一下OOM

堆溢出（`java.lang.OutOfMemoryError: Java heap space`）

永久带溢出（`java.lang.OutOfMemoryError:Permgen space`）

不能创建线程（`java.lang.OutOfMemoryError:Unable to create new native thread`）

...

再来个堆溢出的demo

```java
public class JvmOOMDemo {

    public static void main(String[] args) {
        new Thread(() -> {
            List<byte[]> list = new ArrayList<byte[]>();
            while (true) {
                System.out.println(new Date().toString() + Thread.currentThread() + "==");
                byte[] b = new byte[1024 * 1024 * 1];
                list.add(b);
                try {
                    Thread.sleep(1000);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();


        new Thread(() -> {
            while (true) {
                System.out.println(new Date().toString() + Thread.currentThread() + "==");
                try {
                    Thread.sleep(1000);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}

```

```java
//运行结果
Tue Apr 06 11:48:49 CST 2021Thread[Thread-1,5,main]==
Tue Apr 06 11:48:50 CST 2021Thread[Thread-0,5,main]==
Tue Apr 06 11:48:50 CST 2021Thread[Thread-1,5,main]==
Tue Apr 06 11:48:51 CST 2021Thread[Thread-1,5,main]==
Tue Apr 06 11:48:51 CST 2021Thread[Thread-0,5,main]==
Tue Apr 06 11:48:52 CST 2021Thread[Thread-1,5,main]==
Tue Apr 06 11:48:52 CST 2021Thread[Thread-0,5,main]==
Tue Apr 06 11:48:53 CST 2021Thread[Thread-1,5,main]==
Tue Apr 06 11:48:53 CST 2021Thread[Thread-0,5,main]==
Tue Apr 06 11:48:54 CST 2021Thread[Thread-0,5,main]==
Tue Apr 06 11:48:54 CST 2021Thread[Thread-1,5,main]==
Exception in thread "Thread-0" java.lang.OutOfMemoryError: Java heap space
	at com.hywang.concurrent.oom.JvmOOMDemo.lambda$main$0(JvmOOMDemo.java:21)
	at com.hywang.concurrent.oom.JvmOOMDemo$$Lambda$1/1324119927.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)
Tue Apr 06 11:48:55 CST 2021Thread[Thread-1,5,main]==
Tue Apr 06 11:48:56 CST 2021Thread[Thread-1,5,main]==
Tue Apr 06 11:48:57 CST 2021Thread[Thread-1,5,main]==
Tue Apr 06 11:48:58 CST 2021Thread[Thread-1,5,main]==
Tue Apr 06 11:48:59 CST 2021Thread[Thread-1,5,main]==
Tue Apr 06 11:49:00 CST 2021Thread[Thread-1,5,main]==
Tue Apr 06 11:49:01 CST 2021Thread[Thread-1,5,main]==
Tue Apr 06 11:49:02 CST 2021Thread[Thread-1,5,main]==

```



上面的demo中，线程0出现OOM后线程1还是正常的在运行。

然而所有的OOM都是这样吗？

在生产项目中如果有一段OOM的代码，会只有一个线程出现OOM吗? 肯定不会，这个段代码会导致调用的业务线瘫痪。

OOM如果是内存泄漏引起的，会导致整个JVM退出运行，所有的线程都不会运行

OOM可能会造成把整个JVM都kill掉



线上出现JVM的时候应该是解决OOM问题，而不是去看其他线程还在不在运行。