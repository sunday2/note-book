* 定理

```
无状态对象一定是线程安全的。
```
```
无状态，说明即使该对象被多个线程共享，也没有状态可共享。
```

* 正确编程方法

```
首先使代码正确运行，然后再提高代码的速度。
```
```
即先实现功能，再优化。
```

* 线程安全类怎么实现的

```
在线程安全类中封装了必要的同步机制，因为客户端无须进一步采取同步措施。
```
```
面试经常问xxx类是线程安全的吗?比如经常用的HashMap是线程安全的吗？你会回答说不是，那么其对应的线程安全类是谁呢？其对应的线程安全类是HashTable，CoccurentHashMap。而HashTable如果你看过其源码就会发现，其实现线程安全的机制是在方法级别上加了synchronized关键字，也就是加了this互斥锁。当然，实际中我们会使用更好的CoccurrentHashMap来作为HashMap的并发类，其性能更好。
```

* 竞争条件(结果依赖于多个线程的执行顺序)类型

```
最常见的就是“先检查后执行”，即Check-Then-Act(a,b约时间去星巴克见面的例子)。
```
```
还有一个常见的Check-Then-Act例子就是延迟初始化，比如单例模式我们经常会说有懒汉式和饿汉式。首先提倡的是延迟加载的，因为很多时候初始化对象是个高开销的操作。其次，我们实现懒汉式经常举的例子是DCL，也就是double check lock的模式，其实这并不是最优的example，所以我一般都使用静态内部类来实现懒汉式。

```

* 复合操作

```
很多时候代码会给我们一个假象，比如a+=1这行代码，其实一开始我们以为它就是原子操作，但是其实在操作系统层次，其包括了三条指令，读取-修改-写入，所以在操作系统层次上它就是符合操作。意味着当t0，t1线程共享了该数据，它可以在任何一个指令上被其它线程中断，所以平时写代码一定要有意识的告诉自己这是一个复合操作。
```

* 如何将复合操作原子化

```
其实通过复合操作还是复合操作，所谓的原子化只是jvm层上的，我们并没有改变底层操作系统的逻辑。
```
```
(1)使用现有的线程安全对象
(2)使用锁机制
```

* 有哪些锁

```
(1)内置锁(intrinsic lock)或者监视器锁(monitor lock)或者叫同步代码块。使用java对象作为锁，配合syncronized使用。
```
```
其实编译器会在同步代码块前后加上lock和unlock。
为什么加锁能保证被加锁的区域线程安全呢？
其实本质上就是将加锁区域变成了只能串行执行，无论你有多少个线程对加锁区域进行并发操作。
```
```
(2)可重入锁，jdk已经有实现ReentrantLock。“重入”意味着获取锁的操作的粒度是“线程”，而不是“调用“。
```

```
之前面试官问java中的锁有哪些，什么是可重入锁?
因为可重入锁在实际应用中还是比较频繁的，我会按自己的理解解释一番，但是其实我自己觉得并没有扎到底。书中从锁的粒度上给了一个解释，我觉得真的给了一个思路，所以说每次读都会有不一样的收获。

那为什么可重入锁在实际的应用会比较频繁呢？
因为可重入锁避免了自己锁死了自己的情况，也就是避免了死锁，否则会出现自己等待自己当前已经获取的锁的情况，这不是就是死锁了吗?死锁的情况无非就是
（1）a等待b已经获取到的锁，然后b等待a已经获取到的锁。
（2）a等待其自己已经持有的锁。

面试官可能还会问可重入锁是怎么实现的？
每个可重入锁会将其被持有的次数和持有的线程记录起来，也就是可重入锁每次还是都只能被一个线程持有，但是可以被已经持有的线程多次持有。所以，当计数值为0表示该锁未被任何线程持有，当计数值非0，意味着该锁被有且一个线程持有，而持有该锁的线程每退出一次锁的区域，计数值就减1。
```

* 何时使用同步

```
很多人以为只有对共享变量进行写操作的时候才需要使用同步，其实并不是。
```

```
首先，对共享变量的操作无非就是写和读操作。为什么读操作也不要同步呢？因为还有这样一种情况，t0线程对a进行写操作时，t1线程再对a进行读操作，那么如果不对读操作进行同步，t1线程就有可能读到一个脏数据。所以同步机制并不一定是在写操作才需要，读操作同样需要。
因为我们的认知中t0线程开始写操作就是代表已经写了，但是写操作是复合操作，包括了读-修改-写三个原子操作，在读的这个指令时，t1也对a进行了读操作，但是读到的是脏数据。
```
