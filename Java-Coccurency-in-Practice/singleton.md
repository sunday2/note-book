###  单例模式应该这样玩

> ​        单例模式普遍是我们入门设计模式的第一个模式(创建型模式的一种)，而且在面试题中有70%的概率会被面试到，如果这个你都写不出来，这次面试90%会挂了，这个模式能够写出来，对面试结果也不会有加分，因为这是基础。但是，也许连面试官都不知道，传统的double-checked-lock单例模式竟然有隐藏bug，当你指出隐藏的bug并写出更perfect的懒加载模式时，无疑对面试是有加分的。



网上普遍存在的单例懒加载模式是这样:

```java
public class LazyDoubleCheckedLockingSingleton {

    private static volatile LazyDoubleCheckedLockingSingleton instance;

    /** private constructor to prevent others from instantiating this class */
    private LazyDoubleCheckedLockingSingleton() {}

    /** Lazily initialize the singleton in a synchronized block */
    public static LazyDoubleCheckedLockingSingleton getInstance() {
        if(instance == null) {
            synchronized (LazyDoubleCheckedLockingSingleton.class) {
                // double-check
                if(instance == null) {
                    instance = new LazyDoubleCheckedLockingSingleton();
                }
            }
        }
        return instance;
    }
}
```

```
    懒加载中instance在并发中是竞争资源，但是因为是单例，所以只有一次写操作(new)。但是，new并不是原子操作，所以在new 的过程中，另外一个线程来读取instance时，此时instance并不为null，所以会返回正处于new中间态的instance。
```

懒加载的一种正确写法:

```java
public class LazyInnerClassSingleton {

    /** private constructor to prevent others from instantiating this class */
    private LazyInnerClassSingleton() {}

    /** This inner class is loaded only after getInstance() is called for the first time. */
    private static class SingletonHelper {
        private static final LazyInnerClassSingleton INSTANCE = new LazyInnerClassSingleton();
    }

    public static LazyInnerClassSingleton getInstance() {
        return SingletonHelper.INSTANCE;
    }
}
```

```java
上面采用静态内部类来实现懒加载的功能，因为静态内部类只有调用的时候才会去加载，所以达到懒的目的。
原先的double-checked-lock方式其实改造一下也是可以达到线程安全的目的的，你知道怎么改吗(在JCIP一书中并不提倡double-checked-lock的方式)？
```



面试tips:

```
当你抛掉传统的double-checked-lock的方式，告诉面试官你使用的是静态内部类的方式无疑让面试官耳目一新，大大加分喔(JCIP一书中并不提倡double-checked-lock方式).
```



参考文献:

```
1.《Java Concurrency in Practice》。
```

