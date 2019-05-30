## 单例模式（Singleton）
&emsp;单例模式的目标是确保一个类只有一个实例，并提供该实例的全局访问方式。

### 单例模式常见的实现方式
1.  懒汉式
2.  饿汉式
3.  内部类
4.  枚举
#### 懒汉式(线程不安全)
&emsp;懒汉式的方式可以理解为当我们用到的时候再去做，这样做的好处是，如果没用到该类，那么就不是实例化该类，解约了资源。具体实现如下:

```java
public class Singleton {
    private static Singleton singleton;

    private Singleton() {
    }

    public static Singleton getInstance(){
        if (null == singleton) {
            singleton = new Singleton();
        }
        return singleton;
    }
}
```
&emsp;以上这这种方式实现是会导致再并发的情况下出现Singleton实例不同的问题，假设多个线程同时调用Singleton.getInstance()并同时进入到(null == singleton) 的判断，此时singleton还没有被实例化，那么就会产生多个Singleton实例。
#### 懒汉式(线程安全)
&emsp;上面懒汉式的实现方式，存在着线程不安全的问题，为了保证线程安全，就需要保证getInstance()同一时间只能有一个线程进入该方法，实现方式如下:
```java
public class Singleton {
    private static Singleton singleton;
    private Singleton() {
    }

    public synchronized static Singleton getInstance(){
        if (null == singleton) {
            singleton = new Singleton();
        }
        return singleton;
    }
}
```
&emsp;如上，通过在getInstance()方法上加上锁synchronized，便可以保证其线程安全，不过这种方式存在的问题是，无论singleton有没有被实例化，每次调用都会有加锁、解锁的过程，这样对性能是有损耗的，所以一般也不会采用这种方式。

#### 懒汉式-双重校验锁(线程安全)
&emsp;singleton只需要被实例化一次，加锁的操作也应该只需要做一次，所以为了解决上述两种方式带来的问题，可以采用下面的方式解决：
```java
public class Singleton {
    private volatile static Singleton singleton;
    private Singleton() {
    }

    public static Singleton getInstance(){
        if (null == singleton) {
            synchronized (Singleton.class) {
                if (null == singleton) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```
&emsp;这种方式与上一种方式的区别在于，synchronized不是加在方法上而是代码块上，如果singleton已经被初始化，那么将不会进行加锁解锁的操作直接会返回singleton实例。第一次的校验是为了，那么它和上面的实现方式是没有差别的，第二次的校验是因为如果两个线程同时通过第一次校验，那么将都会执行singleton = new Singleton()的操作，所以需要加上第二次的校验。
<br>&emsp;还有一个注意点就是singleton需要使用volatile关键字修饰，因为singleton = new Singleton()这行代码实际执行时分为三步：
<br>&emsp;1.为singleton分配内存空间
<br>&emsp;2.初始化singleton
<br>&emsp;将singleton指向分配的内存地址
<br>&emsp;由于JVM具有指令重排的特性，执行顺序有可能变成1->3->2，那么在多线程环境下会导致一个线程获得还没有初始化的实例。例如线程1执行了1和3，此时线程2调用 getInstance() 后发现 singleton 不为空，因此返回singleton，但此时singleton还未被初始化，那么线程2就可能会产生空指针的异常。使用volatile可以禁止JVM的指令重排、并且其是变量具有内存可见性，保证在多线程环境下也能正常运行。


#### 饿汉式(线程安全)
&emsp;懒汉式是指需要的时候再去做，顾名思义，饿汉式是指无论需不需要都先做，其实现的方式很简单，如下：
```java
public class Singleton {
    private static final Singleton singleton = new Singleton();
    private Singleton() {
    }

    public static Singleton getInstance(){
        return singleton;
    }
}
```
&emsp;如上，通过直接实例化Singleton的方式，便避免了线程不安全的隐患，不过也失去了丢失延迟实例化带来的节约资源的好处。

#### 静态内部类方式
&emsp;当Singleton类加载时，静态内部类SingletonBuilder还没有被加载进内存，只有当调用过 getInstance()方法，执行SingletonBuilder.SINGLETON时SingletonBuilder才会被加载，此时会初始化INSTANCE实例。这种方式JVM能确保Singleton只被实例化一次，不仅可以延迟初始化，而且由JVM保证了对线程安全的支持。代码实现示例如下：
```java
public class Singleton {
    private Singleton() {
    }
    private static class SingletonBuilder{
        private static final Singleton SINGLETON = new Singleton();
    }

    public static Singleton getInstance(){
        return SingletonBuilder.SINGLETON;
    }
}

```

#### 枚举方式
```java
public enum Singleton {
    SINGLETON;
}
```
&emsp;枚举的方式在多次序列化再进行反序列化之后，不会得到多个实例。而其它实现需要使用 transient 修饰所有字段，并且实现序列化和反序列化的方法。这种方式还可以防止反射攻击，JVM保证只会实例化一次。其它方式通过setAccessible()方法可以将私有构造函数的访问级别设置为public，然后调用构造函数从而实例化对象，如果要防止这种攻击，需要在构造函数中添加防止多次实例化的代码。