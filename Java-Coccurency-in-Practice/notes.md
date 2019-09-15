* 不良并发(poor concurrency)
* 临界区(critical section)
* 内存可见性(memery visibility)
* 线程封闭(thread confinement)
* java的一些同步注解

```
类annotation
@ThreadSafe:表示这个类是线程安全的，具体是否线程安全，就看实现者怎么实现了

域annotation:
@GuardedBy:表明这个状态变量，被哪个锁保护者。
```

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
synchronized是可重入的。

那为什么可重入锁在实际的应用会比较频繁呢？
因为可重入锁避免了自己锁死了自己的情况，也就是避免了死锁，否则会出现自己等待自己当前已经获取的锁的情况，这不是就是死锁了吗?死锁的情况无非就是
（1）a等待b已经获取到的锁，然后b等待a已经获取到的锁。
（2）a等待其自己已经持有的锁。

面试官可能还会问可重入锁是怎么实现的？
每个可重入锁会将其被持有的次数和持有的线程记录起来，也就是可重入锁每次还是都只能被一个线程持有，但是可以被已经持有的线程多次持有。所以，当计数值为0表示该锁未被任何线程持有，当计数值非0，意味着该锁被有且一个线程持有，而持有该锁的线程每退出一次锁的区域，计数值就减1。
```

* 何时使用同步

```
如果用同步来协调对某个变量的访问，那么在访问这个变量的所有位置上都需要使用同步。
```

```
很多人以为只有对共享变量进行写操作的时候才需要使用同步，其实并不是。
```

```
首先，对共享变量的操作无非就是写和读操作。为什么读操作也不要同步呢？因为还有这样一种情况，t0线程对a进行写操作时，t1线程再对a进行读操作，那么如果不对读操作进行同步，t1线程就有可能读到一个脏数据。所以同步机制并不一定是在写操作才需要，读操作同样需要。
因为我们的认知中t0线程开始写操作就是代表已经写了，但是写操作是复合操作，包括了读-修改-写三个原子操作，在读的这个指令时，t1也对a进行了读操作，但是读到的是脏数据。

```

* 内置锁的设计目的

```
每个对象都有一个内置锁，只是为了免去显示地创建锁对象。
```

* 定理

```
每个共享的和可变的变量都应该只由一个锁来保护，从而使维护人员知道是哪一个锁
```

```
所以，无论代码如何变化都好，本质上都有一个锁在那里。即使你的synchronized关键字是加在方法上的，当是对象方法时，本质上是对方法加了一个当前的对象锁；当时静态方法时，本质上是加了一个当前class对象锁。因此，在使用synchronized关键词的时候，我们都应该有意识的去想到底加的是哪一个锁。
```

* 避免混合使用同步机制

```
原子类的操作是线程安全的，但是原子类内部的同步机制使用的是无锁结构，当我们在synchronized块中可以避免使用原子类，因为使用两种不同的同步机制不仅可能会带来混乱，而且对性能和安全性上也不会带来任何好处。
```

* 在程序简单性和并发性取得平衡

```
意味着锁的粒度需要合理
```

* 定理

```
当执行时间较长的计算或者可能无法快速完成的操作时(例如网络io),一定不要持有锁。
```

* 无同步策略情况下并发可能出现的问题

```
1.没有同步情况下，编译器，处理器以及运行时都可能对操作的执行顺序进行一些意想不到的调整.
2.失效值问题.get和set方法得加锁，否则a线程调用set方法，b线程调用get方法，get会读到一个失效值，所以都得加上锁。
但是这种读到的set值也是某个线程之前set进去的，所以称为最低安全性(out-of-thin-airsafety)
3.非原子的64位操作，jvm会将64位的读操作或者写操作分解为两个32位的操作。所以对于共享和可变的long和double类型必须加锁或者使用volatile变量。
```

* 定理

```
加锁的含义不仅仅局限于互斥行为，还包括内存可见性。为了确保所有线程都能看到共享变量的最新值，所有执行读操作和写操作的线程都必须在同一个锁上同步.
```

```
其实个人认为互斥和可见性是一种因果关系。并发编程中的问题是原子性，可见性，有序性
加锁保证了临界区的原子性，同时保证了可见性，有序性。
```

* volatile 变量

```
可见性的本质原因在于cpu缓存和jvm指令重排

那么使用volatile的变量jvm禁止缓存和指令重排(结合jvm的happened-before原则)
```

```
平时可以理解为被volatile修饰的变量相当于get和set加了锁（虽然实际上并没有加锁，但是和加锁的效果是一样的，所以性能更好）。但是，并不建议过度依赖volatile变量提供的可见性，因为其比加锁的代码更脆弱和难以理解。
```

```
volatile的典型用法：
检查某个状态标记以判断是否退出循环。
```

```java
volatile boolean asleep;
...
    while(!asleep)
        countSomeSheep();
```

* 使用volatile的实机

```
(1)对变量的写入操作不依赖变量的当前值,或者你能确保只有当个线程更新变量的值
(2)该变量不会与其他状态变量一起纳入不变性条件中。
(3)在访问变量时不需要加锁。
```

```
所以，实际上使用volatile的机会相对来说还是比较少的。
第一点，当有类似a+=1这种并发操作时，并且需要a+=1是原子操作时，并不适合将a使用volatile来声明。因为volatile的效果只是类似a的set和get方法进行了加锁来保证可见性。
第二点，其实不变性条件意思是该变量不需要和其他变量一起作为一个原子操作。
第三点，之前说过应该避免多个同步机制进行混合使用，会有性能或者安全问题。
```

* 发布(publish)

```
发布一个对象指的是使对象能够在当前作用域之外的代码中使用。
```
* 逸出(escaped)
```
当某个不应该发布的对象被发布时，这种情况就被称为逸出(escaped).
```
```java
public static Set<Scret> knownSecrets;

public void initialize(){
    knownSecrets = new HashSet<Secret>();
}
```

```
首先，对象的内存空间都是在堆空间开辟，但是可以可以通过对象的引用来控制它的作用域。上面的例子中，我们在方法作用域里new了一个对象，并把它赋予了一个静态引用，该引用是public，由当前作用域(方法作用域)提升到了全局作用域，由小到大，所以发布了。
```

```
所以判断是否发布，就判断是否从当前作用域提升到了一个更大的一个作用域，而且得注意发布的传递性。
```

```
判断是否逸出，则第一步判断是否发布了，第二部，判断该发布是否是应该的.
```

* 为什么要注意发布和逸出

```
当某个对象逸出后，你必须假设有某个类或线程可能会误用该对象。注意非静态内部类对外部类的隐含引用。
```

* 注意

```
当发布某个对象时，可能会间接地发布其他对象。因为当发布一个对象时，在该对象的非私有域中引用的所有对象同样会被发布。
```

* 定理

```
不要在构造过程中使this引用逸出
```

```
原因就是当且仅当对象的构造函数返回时,对象才处于可预测的和一致的状态。此时，如果对象的this引用在构造函数中逸出，此时若其它线程调用了该对象(因为逸出了，所以必须假设某个类或线程可能会误用该对象)，就会出现问题，因为此时该对象尚未完全构造完成。
```

```
具体来说，只有当构造函数返回时，this才应该从线程中逸出.构造函数可以将this引用保存到某个地方，只要其它线程不会在构造函数完成之前使用它，
```

```
所以，this可以在构造函数逸出，但是得保证其它线程不会在构造函数完成之前使用它。
```

* 线程封闭

```
需要使用同步的情况：多个线程访问共享的可变的数据。
```

```
所以，避免使用同步的方式就是不共享数据，如果仅在单线程内访问数据，就不需要同步，即使被封闭的对象并不是线程安全的。这种技术就是线程封闭，实现线程安全最简单的方式之一。常见应用是JDBC，虽然connectiond对象并不是线程安全的，但是但是connection pool的实现是线程安全的，被分配给其它线程的connection对象不会再分配给其它线程，所以connection是线程封闭的，它不会同时被多个线程持有。
```

* 线程封闭技术的种类

```
ad-hoc线程封闭:  
维护线程封闭性的职责完全由程序实现来承担。这种是非常脆弱的，不推荐使用
```

```
栈封闭:  
    局部变量的固有属性之一就是封闭在执行过程中。而在jvm的内存模型中，我们都知道栈空间是线程私有的空间，所以其他线程无法访问这个栈。它比ad-hoc线程封闭技术更易于维护和健壮。有一点有注意的是，在维护对象引用的栈封闭性时，程序员需要多做一些工作来防止被引用的对象逸出。
```

```
threadlocal类：
    这是最规范的线程封闭技术，它保证了线程中的某个值与保存值的对象关联起来。它本质是为每个使用该变量的线程都保存了副本，并提供了get和set方法，所以get总是返回由当前线程在调用set时设置的最新值。
    threadlocal对象通常用于防止对可变的单实例变量或全局变量进行共享。
    常见应用:
    1.维护一个全局的数据库连接，程序启动时初始化这个连接对象，由于这个连接并不是线程安全的，所以将连接对象保存在threadlocal中，这样每个线程获取连接时都会有该连接的副本，当线程结束时，该连接也会跟着被回收。但是这样会有个问题，如果有n个线程同时获取该连接，那么就会有n个connection的副本，如果一个连接占m内存大小，那么就占了虚拟机n*m的空间大小。可以理解为TreadLocal进行了全局对象的拷贝操作。
```

```java
private static ThreadLocal<Connection> connectionHolder = new ThreadLocal<connection>(){
    public Connection initialValue(){
        return DriverManager.getConnection(DB_URL);
    }
}

public static Connection getConnection(){
    return connectionHolder.get();
}
```

```
在实现应用程序框架是大量使用了ThreadLocal.
```

* 对象不变性

```
定理:不可变对象一定是线程安全的，使用关键词final修饰。
```

* 小总结

```
什么时候会出现线程安全的问题:在线程之间共享了可变的对象。

我们提取到了两个关键词，可变，共享

所以从两个关键词入手，怎样防止非线程安全?
1.不进行共享，即线程封闭技术，典型的就是栈封闭(局部变量)，ThreadLocal。
2.让对象不可变.典型就是使用finla修饰对象。
3.当可变的变量仍然需要进行共享，那么我们常见的就是通过锁的机制
```

```
满足以下条件对象才是不可变的:
1.对象创建以后状态不可修改
2.对象的所有域都是final类型
3.对象是正确创建的(在对象的创建期间，this引用没有逸出)
```

* 设计线程安全的类

```
三个基本要素:
1.找出构成对象状态的所有变量
2.找出约束状态变量的不变性条件
3.建立对象状态的并发访问策略
```

```
1.对象的状态可能是基本类型字段，也可能是对象类型。如果是对象类型，那么对象的状态包括被引用对象的域。
2.找出在多线程访问对象的情况下，对象状态的不变性条件。确保它的不变性条件不会在并发访问的情况下被破坏。
3.同步策略，定义了如何在不违背对象不变性条件或后验条件的情况下对其状态的访问操作进行协同。
```

```java
@ThreadSafe
public final class Counter{
    private long value = 0;
    public synchronized long getValue(){
        return value;
    }
    public synchronized long increment(){
        if(value == Long.MAX_VALUE)
            throw new IllgegalStateException("counter overflow");
        return ++value;

   }

}
```

```
1.上面的例子中，构成对象的状态的所有变量只有一个，就是基本变量value
2.状态空间:即所有可能取值。上面的例子中，不变性条件包括
（1）value的状态空间(所有可能取值)为Long.MIN_VALUE到Long.MAX_VALUE;
（2）value不能是负值
（3）后验条件:判断状态迁移是否有效的。上面的例子中，如果value当前值为17，那么下一个值只能是18.
3.基于第二点的分析制定同步策略，由于第二点的第1，2小点并不会因为多线程的并发访问而破坏该不变性条件，所以不需要制定同步策略。但是第3小点的不变性(后验条件)在并发访问情况下会被破坏，所以我们需要为此制定同步策略，可以看到我们在getValue和increment方法上都加上了对象互斥锁。
increment方法加上锁是因为第3小点value的值每次只能+1这个约束条件。getValue方法加锁是因为increment并不是原子操作。

一个基本准则是:
如果一个不变性条件中包含多个变量,那么在执行任何访问相关变量的操作时，都必须持有保护这些变量的锁。

所以按这个准则，例子两个对value加锁是正确的。
```

* 定理

```
如果不了解对象的不变性条件与后验条件，那么就不能确保线程安全性。要满足在状态变量的有效值或状态转换上的各种约束条件，就需要借助于原子性与封装性。
```

* 设计线程安全的类的补充	

```Ji
设计线程安全的类三步骤的第二步:
要考虑的状态有三类:
1.不变性条件：对象的状态是否是有效的
2.后验条件:状态转换是否有效.比如++i,进行同步操作的话，意味着i的状态变化每次都是+1，只能从1，2，3这样的一个状态变化
3.先验条件:有些操作要等到先验条件为真才执行,而先验条件可能会由于其他线程的操作而变成真。比如:不能从空队列中移除一个元素，在删除元素前，队列必须处于"非空的"状态。
```

```
设计线程安全的类三步骤中的第一步:
如果以某个对象为根结点构造一张对象图，那么该对象的状态将是对象图中所有对象包含的域的一个子集。
为什么是子集?
因为从对象可以达到的所有域中，有一部分不属于对象状态的一部分。
比如:
HashMap对象的逻辑状态包括所有的Map.Entry对象以及内部对象，即使这些对象都是一些独立的对象。
```

```
设计线程安全的类的三步骤中的第三步:
将数据封装在对象内部，可以将数据的访问限制在对象的方法上，从而更容易确保线程在访问数据时总能持有正确的锁。
```

```java
@ThreadSafe
public class PersonSet{
    @GuardedBy
    private final Set<Person> mySet = new HashSet<Person>();
    
    public synchronized void addPerson(Person p){
        mySet.add(p);
    }
    
    public synchronized boolean containsPerson(Person p){
        return mySet.contains(p);
    }
    
    
}
```

```
//通过
封闭+加锁等机制=使一个类成为线程安全的
1.PersonSet的状态是由HashSet来管理的，而HashSet并非线程安全的(这里并未考虑Person对象包括的状态)
2.通过使用private访问控制符来修饰Set达到将Set封闭在类的一个实例中。 封闭
3.唯一能访问Set的代码路径是addPerson和containsPerson，它们均加上了锁，因此PersonSet的状态完由它的内置锁来保护。

所以说PesonSet是一个线程安全的类，即使构成其状态的Set并不是线程安全的。所谓类是否线程安全，指的是它的构成类的状态是否是线程安全的。
```

```
封闭机制更易于构造线程安全的类，因为当封闭类的状态时，在分析类的线程安全性时就无须检查整个程序。
```

* 监视器模式

```
遵循java监视器模式的对象会把对象的所有可变状态都封装起来，并由对象自己的内置锁来保护。(线程封闭+内置锁,一种编写代码的约定，好处在于简单性)
在上面的例子中就是使用了监视器模式，在jdk中许多类都使用了Java监视器模式，例如Vector和HashTable。
```

```java
public class PrivateLock{
    private final Object myLock = new Object();
    @GuardedBy("myLock") Widget  widget;
    void someMethod(){
        synchronized(myLock){
            
        }
    }
}
```

```
上面的例子中，使用私有的对象锁的好处:
1.私有的锁对象可以将锁封装起来，使客户代码无法得到锁（但客户代码可以通过公有方法来获访问锁，以便(正确或不正确)地参与到它的同步策略中）
2.使用公有的内置锁的话，如果客户代码错误地获得了另一个对象锁，那么可能会产生活跃性问题。
3.使用公有访问的锁在程序中是否被正确地使用，需要检查整个程序，而不是单个的类。
```

* 监视器模式例子
```java

@Treadsafe
public class MonitorVehicleTracker{
    @GuardedBy(this)
    private final Map<String,MutablePoint> locations;
    
    public MonitorVehicleTracker(Map<String,MutablePoint> locations){
        this.locations = deepCopy(locations);
    }
    
    public synchronized Map<String，MutablePoint> getLocations(){
        return deepCopy(locations);
    }
    
    public synchronized MutablePoint getLocation(String id){
        MutablePoint loc = locations.get(id);
        retrun loc == null?null:new MutablePoint(loc);
    }
    
    public synchronized void setLocation(String id
                                        ,int x,int y){
        MutablePoint loc = locations.get(id);
        if(loc == null){
            throw new IllegalArgumentException("No such ID:"+id);
        }
        loc.x = x;
        loc.y = y;
    }
    
    public static Map<String,MutablePoint> deepCopy(Map<String,MutablePoint> m){
        Map<String,MutablePoint> result = new HashMap<String,MutablePoint>();
        for(String id : m.keySet){
            result.put(id,new MutablePoint(m.get(id)));
        }
        return Collections.unmodifiableMap(result);
    }
}
```

```
1.get map的时候通过拷贝Map返回了map的快照，之前一直不知道怎么理解快照这个概念，这里就知道快照的实现是通过复制目标对象生成一个新的对象来达到快照的目的的。
get 元素的时候也返回了元素的一个拷贝的新的对象
2.这种返回快照的方式，在容器或者元素比较大，就是拷贝耗时比较长的话会影响程序性能
3.返回快照只有一个地方就是当被拷贝的元素变化了，无法作到实时更新，所以需要根据实际情况决定，优点就是保持了内部一致性的需求。
4.这里还使用了Collections.unmodifiedableMap来修饰map，unmodifiedable指的是map不允许put和remove操作。
```

* 线程安全性的委托的例子

```
将多个非线程安全的类组合为一个类时，java监视器模式是非常有用的。当类中的各个组件已经是线程安全的，那么是否还需要加多一个线程安全层，这个得"视情况而定"。如果可以直接使用线程安全的类，那么我们称为"线程安全性的委托"，将线程安全性委托给了线程安全类。
```

```java
//单个状态的例子
@Threadsafe
public class DelegatingVehicleTracker{
    private final CoccurentMap<String,Point> locations;
    private final Map<String,Point> unmodifiedableMap;
    
    public DelegatingVehicleTracker(Map<String,Point> points){
        this.locations = new CoccurentHashMap<String,Point>(points);
        unmodifialbeMap = Collections.unmodifableMap(locations);
    }
    
    public Map<String,Point> getLocations(){
        return unmodifiableMap;
    }
    
    public Point getLocation(String id){
        return locations.get(id);
    }
    
    public void setLocation(String id,int x,int y){
        if(locations.replace(id,new Point(x,y)) == null){
            throw new IllegalArgumentException("invalid vehicle name:"+id);
        }
    }
}
```

```
上面的例子中，DelegatingVehicleTracker对象的状态为locations变量，该变量是CoccurentMap，它是线程安全的类。所以相当于将DelegatingVehicleTracker委托给了CoccurentMap,并且locations变量是声明为了final。
```

```java
//多个状态下线程安全的委托
@Threadsafe
public class VisualComponent{
    private final List<KeyListener> keyListeners = new CopyOnWriteArrayList<KeyListener>();
    private final List<MouseListener> mouseListeners = new CopyOnWriteArrayList<MouseListner>();
    
    public void addKeyListener(KeyListener listener){
        keyListeners.add(listener);
    }
    
    public void addMouseListener(MouseListener listener){
        mouseListeners.add(listener);
    }
    
    public void removeKeyListener(KeyListener listener){
        keyListener.remove(listener);
    }
    
    public void removeMouseListener(MouseListener listener){
        mouseListener.remove(listener);
    }
    
}
```

```
    上面的例子中VisualComponent是由两个状态变量的，分别是keyListeners，mouseListeners.由于这两个状态变量彼此是独立的，互不干扰的，所以可以直接委托给它们的线程安全类，而我们在实际中更多遇到的是多个状态变量是有关联的(或者说存在不变性条件)，这时只能通过加锁机制了，委托不了。
    而且我们可以看到ArrayList对应的线程安全类是coccurent包下的CopyOnWriteArrayList,就好像HashMap对应的线程安全类是CoccurentHashMap.
```

* 重用基础模块类(java类库中的类,特别是cocurrent包下的线程安全类)

```
重用能降低开发工作量,开发风险(因为现有的类都已经通过测试)以及维护成本。
```

```
虽然ArrayList经过Collections.synchronizedList(new ArrayList<E>())后会返回一个线程安全的list。
但是该list的操作有限，比如无法做到"若没有则添加"的功能。这时候我们如何实现重用的思想呢?
（1）要添加一个新的原子操作，最安全的方法是修改原始的类，但通常无法做到，因为大多数情况下无法访问或修改类的源代码。而且要修改原始的类，需要理解代码中的同步策略，与原有设计保持一致。
(2)扩镇(extends)原始类,然后添加新的方法。不推荐，因为这个比直接在原始类添加代码更加脆弱。
(3)客户端加锁机制。与扩展类机制类似，二者都是将派生类的行为与基类的实现耦合在一起，所以比扩展类的机制更加脆弱，它将类C的加锁代码放到鱼C完全无关的其他类中。
(4)组合机制。
```

```java
//组合机制的demo
@ThreadSafe
public class ImprovedList<T> implements List<T> {
    private final List<T> list;
    
    public ImprovedList<List<T> list>{ this.list = list;}
    
    public synchronized boolean putIfAbsent(T S){
        boolean contains = list.contains(x);
        if(contains){
            list.add(x);
        }
        return !contains;
    }
    
    public synchronized void clear(){
        list.clear();
    }
    
    // ...按照类似的方式委托List.其它方法。
    
}
```

* 将同步策略文档化

```
在维护线程安全性时,文档是最强大的(也是最未被充分利用的)工具之一。
```

```
(1)用户可以通过查阅文档来判断某个类是否是线程安全的。
(2)维护人员也可以通过查阅文档来理解其中的实现策略。
```

* jdk中的并发基础构建模块
* 同步容器类

```

早期的Vector,HashTable;JDK1.2后添加的一些功能类似的类,这些同步的封装器类是Collections.synchronizedXxx等工厂方法创建的。这些类实现的线程安全的方式是:
将它们的状态封装起来，并对每个公有方法都进行同步,使得每次只有一个线程能访问容器的状态。
```

```java
1.如果需要在同步类的基础额外扩展新的原子操作，那么可以通过客户端加锁的方式，只需要确认原生的类使用的是哪一个锁，就可以通过客户端加锁的方式达到和容器的其它操作同样的同步机制。
```

```java
//在使用客户端加锁的Vector上的复合操作
public static Object getLast(Vector list){
    synchronized(list){
        int lastIndex = list.size()-1;
        return last.get(lastIndex);
    }
}

public static void deleteLast(Vector list){
    synchronized(list){
        int lastIndex = list.size()-1;
        list.remove(lastIndex);
    }
}
```

```
我们知道Vector是线程安全的，然后分析其内部的同步机制使用的是vector的自身的锁，所以在客户端代码我们可以通过该锁来设计需要的原子操作。
```

```
2.同步类的同步机制是在类的公有方法上加上synchronized锁，锁的粒度比较大，所以实际使用中，比较影响性能。
```

```
3.同步容器类中并没有对迭代过程进行对应的同步机制，所以迭代器在迭代过程中是非线程安全的，它们表现出的行为是"即时失败"(fail-fast),即会抛出ConcurrentModificationException异常.它并不是完善的处理机制。
```

```java
//1.在迭代期间加锁来避免线程安全问题.长时间对容器加锁会降低程序的可伸缩性，极大的降低吞吐率和cpu的利用率;且得在所有迭代处加锁,但是得注意隐藏迭代器的问题.
synchronized(vector){
    for(int i=0;i<vector.size();i++){
        doSomething(vector.get(i));
    }
}
```

```java
//2.通过克隆容器来避免迭代期间线程安全问题.在副本上进行迭代，那么克隆期间还是需要加锁的.那么性能瓶颈在于克隆的耗时.
```

```java
//3.需要对共享容器的所有迭代期间都需要加锁,但是有时候迭代器会隐藏起来.
//字符串拼接编译器会调用String.append(Object)，并对set进行迭代添加
System.out.println("DEBUG:added ten elements to"+set);


(1)容器的hashCode和equals等方法也会间接地执行迭代操作,当容器作为另一个容器的元素或键值时,就会出现这种情况;
(2)containsAll,removeAll和retainAll等方法，以及把容器作为参数的构造函数，都会对容器进行迭代。

这些间接的迭代操作都可能抛出ConcurrentModificationException.


```

* tips

```
如果状态与保护它的同步代码之间相隔越远,那么开发人员就越容易忘记在访问状态时使用正确的同步.
```



* 并发容器类

```
java5.0提供了多种并发容器类来改进同步容器的性能.
同步容器将所有容器对状态的访问都串行化,来保证线程安全性，这种方法的代价是严重降低并发性,多个线程竞争容器的锁时,吞吐量将严重降低。
比如:
(1)使用ConcurrentHashMap替代原线程安全的HashTable
(2)CopyOnWriteArrayList来实现list的线程安全
(3)在ConcurrentMap接口中增加了对一些复合常见复合操作的支持，例如"没有则添加"，替换，以及条件删除等。
(4)Java5.0增加了两种新的容器类型:Queue和BlockingQueue.
(5)java6引入了ConcurrentSkipListMap和ConcurrentSkipListSet分别作为同步的SortedMap和SortedSet的并发替代品。
```

* 并发容器类-ConcurrentHashMap

```
1.
   HashTable在每个操作方法上都加了同一个锁来实现线性操作达到同步的目的,这样一些遍历操作，比如HashMap.get,有可能需要遍历很长的链表,会花费很长的时间,这个期间,其它线程都不能访问这段容器.
   ConcurrentHashMap则使用了一种完全不同的加锁策略来提供更高的并发性和伸缩性.它并没有在每个方法上都使用同一个锁使得每次只有只能一个线程访问容器,其使用一种粒度更小的锁机制来实现最大程度的共享,这种锁叫作分段锁（Lock Striping）.
   
2.
   ConcurrentHashMap提供的迭代器并不会抛出ConcurrentModificationException,因此不需要在迭代时对容器进行加锁.
	原因就是ConcurrentHashMap使用的"弱一致性"策略(Wealy Consistent),而并非HashMap的"及时失败".相反HashMap是强一致性的.弱一致性的好处就是可以容忍并发的修改,但是不保证在迭代器构造后将修改操作返回给容器，就是如果你已经获得迭代器，后面如果有容器的修改操作,比如新增元素,已获得迭代器无法反应出来(相当于迭代器是个容器的快照或者说副本).比如size或者isEmpty这些方法的语义被弱化了以反映容器的并发特性,所谓弱化就是这些方法的结果在计算时可能已经过期了(并发操作的原因),实际上可能是个估计值.不过这些方法在并发环境的用处很小,因此被牺牲掉了来换取更重要操作(get,put,containsKey,remove)的性能优化.
	
3.
	ConcurrentHashMap与synchronizedMap和HashTable相比,有着更多优势和更少的劣势,大多数情况下并发推荐ConcurrentHashMap除非需要在某些操作上强一致性，进行"独占访问",才放弃.
	
	
适用场景:
并发时能够容忍容器的弱一致性.
```

```java
//ConcurrentMap在一些复合操作上已经是线程安全操作或者说是原子操作,"若没有则添加","若相等则移除","若相等则替换"
public interface ConcurrentMap<k,v> extends Map<K,V>{
    V putIfAbsent(K key,V value);
    boolean remove(K key,V value);
    boolean replace(K key,V oldValue,V newValue);
    V replace(K key,V newValue);
}
```

* 并发容器-CopyOnWriteArrayList／CopyOnWriteArraySet

```
1.CopyOnWriteArrayList用于并发时替代同步List,在迭代期间同样不需要对容器进行加锁或复制.(CopyOnWriteArraySet的作用是替代同步set)
2.写入时复制(Copy-On-Write)容器的线程安全在于只要正确发布一个事实不可变的对象,那么在访问该对象时就不再进一步的同步.

实现原理(无锁实现的典型,读写分离,每次修改容器都会重新发布一个新的副本,在副本上进行写操作,结束后原容器的引用指向新的副本):
故名思义,每次修改时,都会创建并重新发布一个新的容器副本,从而实现可变性."写入时复制"容器的迭代器保留一个指向底层基础数组的引用，这个数组当前位于迭代器的起始位置,由于它不会被修改,因此在对其进行同步时只需确保数组内容的可见性.由于是无锁的实现方式,所以比有锁实现方式性能更好.


适用场景:
并发时对容器读多写少的情况.

缺点:
每一次修改容器都会触发复制底层数组操作,这需要一定开销，特别是容器的规模较大时,所以仅当迭代操作(读)远远多于修改(写)操作时,才应该使用"写入时复制"容器.
```

* 队列和生产者消费者模式(对象所有权的转移)

```
1.队列需要考虑的问题
阻塞和非阻塞,有界和无界(阻塞和有界队列能让程序更简单和健壮性,但是得考虑性能问题)


2.生产者和消费者模式
通过线程+队列实现生产者和消费者模式.一个线程作为生产者，一个线程作为消费者，队列作为线程间共享的资源.


3.java.util.concurrent包下实现的各种阻塞队列都包含了足够的内部同步机制,从而安全的将对象从生产者线程发布到消费者线程.

4.线程封闭对象只能由单个线程拥有,但可以通过安全地发布该对象来“转移”所有权.在转移所有权后，发布对象的线程不会再访问它,只有另一个线程能够获得这个对象的访问权限,所以对象永远在任意时刻都封闭在一个线程中,且当前线程具有独占权。

5.例子:
对象池:利用了串行线程封闭,将对象"借给"一个请求线程,只要线程池包含足够的内部同步机制来安全地发布池中的对象,并且客户代码本身不会发布池中对象，那么就可以安全地在线程之间传递所有权.比如线程池.

阻塞队列:

通过ConcurrentMap的原子方法remove来实现.


原则:
保证只有一个线程能接受被转移的对象.
```

* 阻塞方法

```
阻塞方法:
1.等待I/O结束
2.等待获取某个锁
3.等待从Thread.sleep()醒来

tips:
其实,线程是有生命周期的.生命周期由很多状态组成,从jvm角度可以划分为这几个阶段:new,runnable,running,blocked(包括了blocked,waiting,time_waiting),termianted.
在操作系统层面线程只有阻塞状态,在jvm层面细分出了三种(blocked,waiting,time_wating).

所谓的阻塞方法,就是在线程中调用该方法会造成该线程处于阻塞状态.调用Thread.sleep方法会使线程处于由running状态切换为time_waiting状态;调用wait()方法会是线程由running状态转换为waiting状态;线程等待某个锁则会由running状态切换为blocked状态.这些能造成线程处于阻塞状态的称为阻塞方法.
```

* 线程中断

```
中断是一种协作机制.一个线程不能强其它线程立即停止正在执行的操作.当线程a中断线程b,a仅仅是要求b在执行在某个可以暂停的地方停止正在执行的操作----前提是b线程愿意停止下来.

```

* Interrupted-Exception

```
抛出该异常需要两个条件:
1.线程处于阻塞状态(正好阻塞方法执行中)
2.线程背调用了中断方法,interrupt方法.
```

```
Interrupted-Exception异常处理方式:
1.传递InterruptedException.这是最明智的策略-将异常抛給调用者，即调用了中断的方法.
2.恢复中断.有时候不能抛出InterruptedException,例如当代码的是Runnable的一部分时,这种情况下必须捕获异常,并且调用当前线程上的interrupt方法恢复中断状态.
```

```java
public class TaskRunnable implements Runnable{
    BlockingQueue<Task> queue;
    ...
        
        
    public void run(){
        try{
            processTask(queue.take());
        }catch(InterruptedException){
            //恢复被中断的状态
            Thread.currentThread().interrupt().
        }
        
        
    }
}
```

* 同步工具类

```
定义:
同步工具类可以是任何一个对象，只要它根据其自身的状态来协调线程的控制流.它们都包含一些特定的结构化属性,它们封装了一些状态,这些状态将决定执行同步工具类的线程是继续执行还是等待(可以想下阻塞队列在空队列和满队列的状态下线程阻塞).

例子:
阻塞队列,信号量(semaphore),栅栏(Barrier),闭锁(Latch)
```

* 闭锁

```
定义:
闭锁时一种同步工具类,可以延迟线程的进度直到其到达终止状态.
使用场景:
可以用来确保某些活动直到其它活动都完成后才继续执行.
比如:
(1)确保某个计算在其需要的所有资源都被初始化之后才继续执行.
(2)确保某个服务在其它依赖的服务都已经启动之后才启动.
(3)等待直到某个操作的所有参与者(例如,在多玩家游戏中的所有玩家)都就绪再继续执行.
```

* CountDownLatch(同步工具类-闭锁)

```
CountDownLatch属于并发包中对同步工具类中的闭锁的一种实现.
(1)它包括一个计数器,该计数器初始化为一个正数,表示需要等待的事件数量(new的时候作为构造函数参数传入);
(2)该类的countDown方法调用后会递减计数器,表示有一个事件已经发生.
(3)该类的await方法调用后会阻塞当前线程,直到计数器变为0.
```

```java
public class TestHarness{
    public long timeTasks(int nThreads,final Runnable task) throws InterruptedException{
        final CountDownLatch startGate = new CountDownLatch(1);
        final CountDownLatch endGate = new CountDownLatch(nThreads);
        
        for (int i = 0; i < nThreads; i++) {
            Thread t = new Thread() {
                public void run() {
                    try {
                        startGate.await();
                        try {
                            task.run();
                        } finally {
                            endGate.countDown();
                        }
                    } catch (InterruptedException ignored) {
                    }
                }
            };
            t.start();
        }

        long start = System.nanoTime();
        startGate.countDown();
        endGate.await();
        long end = System.nanoTime();
        return end - start;
    }
}
```

```
tips:
(1)例子中使用了两个闭锁,为什么需要startGate是为了提高统计耗时的准确性,endGate才是较重要的闭锁.
(2)实际使用中如例子中一般一个线程对应一个事件,所以endGate计数器的数量设置为需要等待的线程数.
(3)由于一个线程对应一个事件,所以在线程的方法体中需要调用endGate的countDown方法,表示当前事件执行完.

总结一句就是某个线程在等待其它线程执行完毕才能继续执行下去(例如上面的例子)
```

* FutureTask(同步工具类-闭锁)

```
FutureTask属于并发包中对同步工具类中的闭锁的一种实现.
(1)线程的实现有三种方式,extends,implements(Runnable),implements(Callable).FutureTask需要和Callable配合使用,是一种可生成结果的Callable.
(2)Future.get方法被调用会阻塞当前线程,直到FutureTask关联的线程返回结果或者返回异常.
使用场景:
通过提前启动计算,可以减少在等待结果时需要的时间.
```

```java
public class Preloader {
    ProductInfo loadProductInfo() throws DataLoadException {
        return null;
    }

    private final FutureTask<ProductInfo> future =
        new FutureTask<ProductInfo>(new Callable<ProductInfo>() {
            public ProductInfo call() throws DataLoadException {
                return loadProductInfo();
            }
        });
    private final Thread thread = new Thread(future);

    public void start() { thread.start(); }

    public ProductInfo get()
            throws DataLoadException, InterruptedException {
        try {
            return future.get();
        } catch (ExecutionException e) {
            Throwable cause = e.getCause();
            if (cause instanceof DataLoadException)
                throw (DataLoadException) cause;
            else
                throw LaunderThrowable.launderThrowable(cause);
        }
    }

    interface ProductInfo {
    }
}

class DataLoadException extends Exception { }

```

```
Callable表示的任务可以抛出受检查的或未受检查的异常,并且任何代码都可能抛出一个Error.不管抛出什么异常,都会被封装到一个ExcutionException并在Future.get中被重新抛出.这使得get的代码变得复杂,因为ExcutionException是作为一个Throwable类返回的，处理起来并不容易.
```

* Semaphore(同步工具类-闭锁)

```
Semaphore属于并发包下对同步工具类中闭锁的一种实现。
(1)控制同时访问某个特定资源的操作数量。所以可以实现某种资源池(数据库连接池)。
(2)将任何一种容器变成一种有界容器。
（2）同时执行某个指定操作的数量。
（3）最特殊的就是二值信号量，即初始值为1的Semaphore。此时信号量充当互斥体，并具备不可重入的加锁语义。
```

```
使用:
(1)Semaphore管理着一组虚拟的许可，许可的初始数量通过构造函数来指定。类似new Semaphore(1)。
(2)执行操作之前先通过acquire方法获取许可。
(3)执行完操作之后通过release方法获取许可。
```

```java
public class BoundedHashSet<T>{
    private final Set<T> set;
    private final Semaphore sem;
    
    public BoundedHashSet(int i){
        this.set = Collections.synchronizedSet(new HashSet<T>());
        sem = new Semaphore(bound);
    }
    
    public boolean add(T o) throws InterruptedException{
        sem.acquire();
        boolean wasAdded  = false;
        try{
            wasAdded = set.add(o);
        	return wasAdded；
        }finally{
            //校验是否add成功，否则释放掉许可
            if(!wasAdded){
                sem.release();
            }
        }
    }
    
    
    pulibc boolean remove(){
        boolean wasRemoved = set.remove(o);
        if(wasRemoved)
            sem.release();
        return wasRemoved;
    }
}
```

```
上面是使用Semaphore将一个容器封装变成一个有界容器。
这里要注意在调用acquire方法后要再次校验，如果没有add成功，要release掉该次许可。
```

