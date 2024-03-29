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

* 闭锁

```
闭锁是一次性对象，一旦进入终止状态，就不能被重置。（CountdownLatch,Future,Semaphore）
```

* 栅栏(Barrier)

```
栅栏类似于闭锁，它能阻塞一组线程直到某个事件(所有线程到达阻塞位置)发生。

CyclicBarrier为什么说是类似闭锁?
类似是因为都有等待这个action,但是等待的对象不一样，实际使用中可以仔细体验一下。
另外一个不一样的地方在于闭锁是一次性对象，而CyclicBarrier是循环栅栏，也就是可以循环使用的。

为什么叫作栅栏？
因为它会阻塞一组线程，直到所有线程到达阻塞点，所有阻塞点相当于一个栅栏。所有线程到达阻塞点(栅栏处),再继续执行回调方法(相当于打开栅栏，此时栅栏会重置，供下次使用)

有哪几种情况会打开栅栏?
(1)正常情况下所有线程到达栅栏处。那么await将为每个线程返回一个一个唯一的到达索引号。
(2)其中一个await方法调用超时。抛出BrokenBarrierException
(3)await阻塞点线程被中断了。抛出BrokenBarrierException



```

* 构建高效且可伸缩的结果缓存

```
这个demo可以学习到很多的知识。
P85
为什么使用缓存?
(1)若原本的计算结果需要损耗很长的时间，则重用之前的计算结果，降低延迟，提高响应能力，提高吞吐量->只有原本需要耗费很长的时间操作才考虑使用缓存，不要为了缓存而缓存；放在map中说明缓存放在内存中，速度快
(2)消耗更多的内存->本质上无非就是空间换时间，时间换空间的问题。使用缓存是空间换时间

考虑问题要看两面，利弊两面。具体问题具体分析，当利大于弊则可以使用。其实就是简单的二分法分析问题，初中政治教过的知识。就是这么简单，别想复杂。

缓存的设计思路及演进:
(1)使用HashMap存放计算的结果，即存放缓存。->缓存放在内存中,读写快
(2)在原本的方法上加上方法锁，synchronized。->同步策略,map的get和set必须是线程安全的。通过加方法锁，但是粒度太大，相当该方法是串型操作，导致性能上还不如不使用缓存。(伸缩性差)
(3)同步策略调整为使用Coccurent包下线程安全的ConcurrentHashMap代替HashMap.->同步的粒度变小了，说明ConcurrentHashMap内部实现的锁机制粒度更小(分段锁,粒度更小。HashTable之所以会被淘汰就是HashTable也是在方法级别上加锁，粒度大，性能不好)，所以性能更好。
(4)同步策略有缺陷，存在重复计算的可能->缓存的作用是避免相同的数据被计算多次。由于计算需要耗费时间，并发情况下t0线程并不知道t1线程在计算它需要的结果(由于map存放的是计算结果，而非计算过程)，所以其也开始计算任务(重复计算了).所以此处调整为存放计算过程(FutureTask)
(5)调整后的同步策略其实仍然存在重复计算的问题，但是这个概率比之前的小了(因为并发下存在重复计算的那个时间间隔小了，若没有则添加问题)->再次调整，通过CocurrentHashMap内部方法putIfAbsent可以避免存放重复计算的问题。其实我们绕过了我们的外部代码。这个设计挺好
(6)缓存污染，由于map存放的是计算过程而非计算结果，当future被取消或者发生异常。->当发现future被取消或者抛出异常，那么从map中移除，避免缓存污染。
(7)内存持续损耗问题->因为这里缓存是内存级别的，得考虑内存使用持续使用的问题。缓存逾期可以通过FutureTask的子类解决，为每个计算结果指定逾期时间，并定期扫描缓存中逾期的元素，移除旧的计算结果，从而腾出内存空间。

同步策略的设计思路:
1.确定并发下的所有不变性条件
2.设计出性能最好的同步策略

tips:
1.并发下保护同一个不变性条件的所有变量需要使用同一个锁。
```





* 任务执行

```
多线程的目的本质上就是分解任务,所以线程->任务的概念一定程度上是一样的。、
任务边界：理想情况下，各个任务应该是独立的，任务不依赖于其它任务的状态，结果或者边界，此时能够实现最大的并发性。
平时我们使用的web服务器都提供了一种自然的任务边界选择方式:以独立的客户请求为边界，每个请求相互独立(相当于一个请求就启动一个线程处理该连接,这个是服务器程序帮我们实现的，所以程序在并发情况下有瓶颈，也有最大连接数，因为不可能无限启动线程处理)。
例子:
Web服务器，邮件服务器，文件服务器，数据库服务器
```

```java
//bad demo,串行的服务器
class SingleThreadWebServer{
    pubilc static void main(String[] args) throws IOException{
        ServerSocket socket = new ServerSocket(80);
        while(true){
            Socket connection = socket.accept();
            handleRequest(connection);
        }
    }
}
```

```
思考:
上面显然启动了一个后台服务，一直监听80端口，当有请求进来的时候，会一个一个串行处理。上一个请求没处理完，那么下一个请求就会阻塞住了，阻塞超过限制时间显然就会timeout异常。所以处理速度取决于handleRequest的耗时。如果我们将每个请求放在单独的线程处理，这样子就提高了并发能力。
```

```java
//bad demo,并行的服务器，多线程，但是存在无限制创建线程的不足
class ThreadPerTaskWebServer{
    pubilc static void main(String[] args) throws IOException{
        ServerSocket socket = new ServerSocket(80);
        while(true){
            final Socket connection = socket.accept();
            Runnable task = new Runnable(){
                handleRequest(connection);
            };
            new Thread(task).start();
        }
    }
}
```

```
代码已经有改进了，串行处理通过每个请求单独启动一个线程处理来达到并行处理的目的，但是存在问题，无限制创建线程。
线程其实并无法无限制创建下去，肯定会有一个瓶颈，也就是最存在最大线程数，这个最大线程数的限制值是随着平台的不同而不同，并受很多因素制约，包括JVM的启动参数，底层系统对线程的限制等。如果不约束最终会导致oom错误。
```

* 无限制创建线程的不足

```
1.线程生命周期开销非常高。无限制创建线程无法复用之前的线程。时间开销
<<<<<<< HEAD
2.资源消耗。物理资源，比如内存。
=======
2.资源消耗。比如内存
>>>>>>> b06b65982dafb31d2e5c74da2130eb80edb93db1
3.稳定性。任何程序都有瓶颈，而无限制显然意味着无瓶颈。
```

* 任务和线程概念区别

```
1.任务是一组逻辑工作单元，线程是使任务异步执行的机制。
2.两种通过线程来执行任务的策略。
(1)把所有任务放在单个线程串行执行。糟糕的相应性和吞吐量。
(2)把任务放在各自的线程中执行。“为每一个任务创建一个线程”的方式存在资源管理的复杂性。

```

* 强大又简单的Excutor

```java
public interface Excutor{
    void excute(Runnable command);
}
```

```
<<<<<<< HEAD
就是这个简单的接口，却为是很多异步任务执行框架提供了基础。
=======
就是这个简单的接口，却为是很多异步任务执行框架提供了基础，它提供了一种标准的方法将任务的提交过程与执行过程
解耦。
```

* 隐藏在Excutor中的设计模式

```
Excutor是基于生产者-消费者模式的。
在Java类库中，任务执行的主要抽象不是Thread而是Excutor。

为什么说Excutor是基于生产者和消费者？
关键在于找到生产者和消费者角色。提交任务(Runnable)给Excutor的操作就是生产者，Excutor内部会有一个workqueue队列，存放任务，任务会源源不断的放到work queue队列中。消费者就是线程，比如在ThreadPoolExcutor中内部就有ThreadFactory，会new 线程来执行(消费)任务(Runnable)。所以在自定义线程池中有一个参数肯定是指定消费者(Thread)的最大数量。

```

* Excutor的解耦

```
任务的提交过程和执行过程解耦是如何解耦的呢？
我们把线程(Runnable)交给Excutor去统一调度，这个就是提交过程。
任务的执行过程交给了Excutor内部的调度策略去解决。
```

```java
class TaskExcutionWebServer{
    private static final int NTHREADS = 100;
    private static final Excutor exec = Excutors.newFixedThreadPool(NTHREADS);
    
   	public static void main(String[] args){
        ServerSocket socket = new ServerSocket(80);
        while(true){
            final Socket = new ServerSocket(80);
            Runnable task = new Runnable(){
                public void run(){
                    handleRequest(connection);
                }
            };
            exec.execute(task);
        }
    }
}
```

```
思考:
我们将之前的粗暴的执行方式调整为任务的具体执行策略交给了Excutor内部处理，所以，任务的提交和执行进行了解耦。如果我们需要调整任务的执行策略，那么我们只需要改变Excutor就可以了，像上面的代码中的Excutor使用的是concurrent下内置的已经帮我们实现的Excutor:FixedThreadPool。
从字面上看可以知道该Excutor使用了线程池来管理内部的线程，而且该线程池有个特点是固定大小的。国内的一线互联网可能会更提倡通过ThreadPoolExcutor自定义线程池，但是看了Excutors.newFixedThreadPool你会发现它其实也是通过new ThreadPoolExcutor来实现的，只不过keepAliveTime设置为了0，也就是new ThreadPoolExcutor中的keepAliveTime参数设置为0，那么就相当于一个固定大小的线程池了。这里使用的是jdk内置的Excutor，如果我们想要调整任务的执行策略，那么我们只需要实现Excutor接口就可以了.
```

```java
//自定义执行策略,为每个任务启动一个新线程处理
public class ThreadPerTaskExecutor implements Excutor{
    public void excute(){
        new Thread(r).start();
    }
}
```

```java
//自定义执行策略，每个任务不启动线程执行，而只是调用run方法，这样子其实相当于普通方法了，串行执行
public class WithinThreadExecutor implements Excutor{
    public void execute(Runnable r){
        r.run();
    }
}
```
* 隐藏在Excutors中的设计模式

```
Excutors.newFixedThreadPool是静态工厂方法。
```



* 执行策略和最佳执行策略

```
执行策略定义了任务执行的“WHAT,WHERE,WHEN,HOW”等方面。
```

```
最佳策略取决于可用的计算资源(内存，cpu等)以及对服务质量(响应速度)的需求。
将任务的提交与任务的执行策略分离开来，有助于在部署阶段选择与可用硬件资源最匹配的策略(将执行策略相关的参数比如corePoolSize,maxPoolSize,keepAliveTime等抽出来，可以方便调整).
```

* Excutors静态工厂方法中的那些内置线程池

```
newFixedThreadPool:
该线程池的特点就是固定大小的线程池，设置为10个那么将会固定为10个，需要注意的是它并不是一下子创建10个线程，而是根据每提交一个任务再创建，所以是懒创建的模式，直到线程数量达到设置的限定值，而且如果某个线程在执行过程中发生异常而销毁了，它会再补充一个进来。相当于将keepAliveTime设置为0L，且corePoolSize和MaxPoolSize值一样。使用的队列是LinkedBlockingQueue。
```

```
newCachedThreadPool:
将创建一个可缓存的线程池。线程池无限大。
```

```
newSingleThreadExcutor:
固定只有一个线程的线程池。相当于将keepAliveTime设置为0L，corePoolSize设置为0，MaxPoolSize设置为0L。
```

```
newScheduledThreadPool:
创建一个固定长度的线程池，而且以延迟或定时的方式来执行，类似Timer。注意延迟执行和定时执行的区别，对应方法分别为schedule和scheduleAtFixedRate方法。使用的队列是DelayedWorkQueue。

延时执行是基于上次任务执行完毕后再执行，任务执行时间不定；定时执行是不管上次任务是否执行完毕时间到了就执行，执行时间是确定的。每个几秒执行一次一般描述的是定时任务。定时任务要注意如果是单线程的话这个时间间隔和任务执行时间的关系。
```

* 小结


```
通过看源码可以发现这些静态工厂方法最终都是new ThreadPoolExcutor来实现的。只不过修改了一些参数，比如将keepAliveTime设置为0L，corePoolSize和maxPoolSize设置为一样，那么就是一个FixedThreadPool了。需要小心存放任务的队列大小，LinkedBlockingQueue如果创建的时候没有传递大小作为参数，那么意味着这是无限大小的队列，极端情况下会耗尽内存(blocking指的是取和放的操作是阻塞的)
```

* Excutor的生命周期管理

```
生命周期的管理是通过ExcutorService接口实现的，运行，关闭（粗暴关闭:直接关闭机房类似，平缓关闭：完成所有已经启动任务：是指已经在任务队列中的任务？，并且不再接受任何新的任务），已终止。
```

```java
public interface ExcutorService extends Executor{
    void shutdown();
    List<Runnable> shutdown();
    boolean isShutdown();
    boolean isTermimated();
    boolean awaitTermination()throws InterrutedException;
    //......其它用于任务提交的便利方法
}
```

```java
//支持关闭操作的web服务器
class LifecycleWebServer{
    private final ExcutorService exec= ...;
    
    public void start() throws IOException{
       ServerSocket socket = new ServerSocket(80);
        while(!exec.isShutdown()){
            try{
                final Socket conn = socket.accept();
                exec.execute(new Runnable(){
                    public void run(){
                        handleRequest(conn);
                    }
                });
            }catch(RejectedExecutionException e){
                if(!exec.isShutdown())
                    log("task submission rejected",e);
            }
        }
    }
    
    pubilc void stop(){
        exec.shutdown();
    }
    
    void handleRequest(Socket connection){
        Request req = readRequest(connection);
        if(isShutdownRequest(req)){
            stop();
        }else{
            dispatchRequest(req);
        }
    }
}
```

```
思考：
这段代码值得借鉴，因为这是一个简单的web服务器思路
```



