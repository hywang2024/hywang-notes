# JDK常用新特性
JDK8用了很多年，是不是也有冲动想升级看看。下面是总结了一些，升级之后常用到的新特性

## 类型推断
`JDK10新特性`

在Java8之前，定义的变量都必须有确定的类型，JDK10之后可以直接使用var定义，在使用变量的时候也可以直接调用变量的方法和属性。
```java
public class Test {
    /**
     * 老版本使用 必须指定类型 
     */
    public void testVarOld() throws IOException {
        URL url = new URL("www.baidu.com");
        URLConnection urlConnection = url.openConnection();
        urlConnection.setReadTimeout(1);
    }

    /**
     * 新版本直接使用var定义 
     */
    public void testVar() throws IOException {
        var url = new URL("www.baidu.com");
        var urlConnection = url.openConnection();
        urlConnection.setReadTimeout(1);
    }
}

```
## instanceof
`JDK14新特性`

老版本中使用instanceof之后还必须要要将对象强转为此类型，才可以调用方法,在JDK16之后，instanceof运算符用作返回已转换对象的表达式
```java
public class Test {
    public long testInstanceofOld() {
        Object obj = new Date();
        if (obj instanceof Date) {
            Date date = (Date) obj;
            return date.getTime();
        }
        return 0;
    }

    public long testInstanceofNew() {
        Object obj = new Date();
        if (obj instanceof Date) {
            return obj.getTime();
        }
        return 0;
    }
}
```
## switch 
`JDK14新特性`

```java
public class Test {
    public void testSwitchOld() {
        int value = 1;
        int res;
        switch (value) {
            case 1:
            case 3:
            case 5:
            case 7:
            case 9:
                res = 10;
                break;
            case 2:
            case 4:
            case 6:
            case 8:
                res = 11;
                break;
            default:
                res = 0;
                break;
        }
    }

    public void testSwitchNew() {
        int value = 1;
        int res;
        switch (value) {
            case 1, 3, 5, 7, 9 -> res = 10;
            case 2, 4, 6, 8 -> res = 11;
            default -> res = 0;
        }
    }
}
```

## 文本快
`JDK14新特性` 

""" 直接包裹文本框，不用在换行+
```text
    //老方式
    String  str = " select * from user\n" +
        "                where id > 10\n" +
        "                order by id desc \n" +
        "                limit 1";
    //新特性
    String sqlStr = """
                select * from user
                where id > 10
                order by id desc 
                limit 1
                """;
```
## 记录Records
`JDK14新特性`

创建类更加简介
````java
public class Test {
    class User{
        private String name;
        private int age;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public int getAge() {
            return age;
        }

        public void setAge(int age) {
            this.age = age;
        }

        public User(String name, int age) {
            this.name = name;
            this.age = age;
        }
    }

    /**
     * 记录模式
     * @param name
     * @param age
     */
    record UserInfo(String name,int age){}
    
    public void testRecords(){
        User user = new User("zhangsan",22);
        UserInfo userInfo = new UserInfo("zhangsan",22);
    }
}
````
## NullPointerException
在JDK14空指针打印的异常信息更多
```java
public class Test {
    /**
     * 制造一个空指针
     * @param args
     */
    public static void main(String[] args) {
        Date date = null;
        long time = date.getTime();
        System.out.println(time);
    }
}
```
异常的堆栈信息
```text
//老版本打印信息
Exception in thread "main" java.lang.NullPointerException
	at com.hywang.jdk21.notes.Test.main(Test.java:14)

//JDK14后打印信息
Exception in thread "main" java.lang.NullPointerException: Cannot invoke "java.util.Date.getTime()" because "date" is null
	at com.hywang.jdk21.notes.Test.main(Test.java:18)
```

## 密封类
`JDK17新特性` 

密封类允许将类或接口的继承限制为指定的子类,sealed关键字可以限制继承的类

密封类的子类可以声明为final或non-sealed。final 子类不能进一步扩展，而非密封子类可以进一步扩展。
```java
public class Test{
    
    public abstract sealed class User permits People,Live{}
    
    public non-sealed class User extends People{}
    public final class User extends People{}
}
```

## Stream API增强
`JDK17新特性`

### takeWhile和dropWhile
它们会对流中每个元素逐一校验，遇到第一个不符合条件的元素终止

takeWhile返回终止位置前面的所有元素，而dropWhile则返回包含终止位置后面的所有元素。

它们功能虽然与filter类似，区别是前者并非对整个流进行校验，可以提升过滤效率，但需要注意流内元素的顺序。

```java
public class Test{
    var list = List.of(1, 2, 3, 4, 5, 6);
    list.stream().takeWhile(t -> t < 3).forEach(System.out::println);
    //1,2
    list.stream().dropWhile(t -> t < 3).forEach(System.out::println);
    //3,4,5,6
}
```
### iterate
iterate可以生成一个无限的流，它在JDK9之前需要limit()等操作来配合终止，否则将无限递归下去。在JDK9中iterate新增了一个重载方法，现在支持使用条件来终止，它在语法上更简洁，也提供了更多的灵活性。

```java
public class Test{ 
    Stream.iterate(1, i -> i <= 1024, i -> i * 2).forEach(System.out::println);
    //1,2,4,8,...,512,1024
}
```

## ZGC
`JDK11新特性`

ZGC在JDK11作为实验性的GC算法被引入时，最初的设计目标是实现10毫秒以内的最大停顿时间。

在过去一段时间里，ZGC经过JDK版本的数次迭代，在JDK15中被宣布为可用于生产，目前据官方介绍已经可以实现亚毫秒级的最大停顿时间，且停顿时间不随堆内存、存活对象集合或GCRoot集合大小的增加而增加，它可以处理从8MB到16TB的大范围堆内存。

在官方介绍里，ZGC是并发的、基于区域的（Region-based）、压缩的（Compacting）、NUMA感知（NUMA-aware）的垃圾回收器。它主要使用了染色指针（Colored Pointor）和读屏障（Load Barriers）技术，并在新一代的JDK21版本中实现了分代回收，它的主要工作是在用户线程工作执行时完成的，这大大降低了GC对应用响应时间的影响。

```shell
-XX:+UseZGC
```
