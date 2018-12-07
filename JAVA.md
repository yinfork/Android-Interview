## Java
<span id = "home"></span>
#### 目录
1. [Java基础](#java_base)
2. [Java集合框架](#java_collection)
3. [Java内存](#java_memory)


----


<span id = "java_base"></span>
#### Java基础 [(TOP)](#home)
1. 关于finally<br>
	参考：https://www.cnblogs.com/fery/p/4709841.html
	1. 不管有没有出现异常，finally块中代码都会执行
	2. 当try和catch中有return时，finally仍然会执行，finally比return先执行
	3. **finally是在return后面的表达式运算后执行的**（此时并没有返回运算后的值，而是先把要返回的值保存起来，管finally中的代码怎么样，返回的值都不会改变，仍然是之前保存的值），所以函数返回值是在finally执行前确定的
	4. finally中最好不要包含return，否则程序会提前退出，返回值不是try或catch中保存的返回值
	5. finally不执行的几种情况：程序提前终止如调用了System.exit, 病毒，断电

2. final关键字<br>
	1. final修饰的变量是常量，必须进行初始化，可以显示初始化，也可以通过构造函数进行初始化，如果不初始化编译会报错。
	2. final修饰类，表示是最终类。最终类不能被继承, 最终方法不能被重写(可以重载)

3. 接口与抽象类的区别
	1. 一个子类只能继承一个抽象类，但能实现多个接口
	2. 抽象类可以有构造方法，接口没有构造方法
	3. 抽象类可以有普通成员变量，接口没有普通成员变量
	4. 抽象类和接口都可有静态成员变量，抽象类中静态成员变量访问类型任意，接口只能public static final(默认)
	5. 抽象类可以没有抽象方法，抽象类可以有普通方法,接口中都是抽象方法
	6. 抽象类可以有静态方法，接口不能有静态方法
	7. 抽象类中的方法可以是public、protected;接口方法只有public

4. 异常关键字<br>
	throw、throws、try...catch、finally
	1. throws 用在方法签名上, 以便抛出的异常可以被调用者处理
	2. throw 方法内部通过throw抛出异常
	3. try 用于检测包住的语句块, 若有异常, catch子句捕获并执行catch块

5. 异常的类型<br> 	所有的异常都是继承于Throwable接口
	1. Error: 属于严重错误，如系统崩溃、虚拟机错误、动态链接失败等，这些错误无法恢复或者不可能捕捉，将导致应用程序中断，Error不需要捕获
	2. Checked Exception: 继承于java.lang.Exception。其必须被try...catch语句块所捕获, 或者在方法签名里通过throws子句声明。受检查的异常必须在编译时被捕捉处理,因为Java编译器要进行检查, Java虚拟机也要进行检查, 以确保这个规则得到遵守。比如：IOException、ClassNotFoundException等
	3. Unchecked Exception: 继承于Runtime Exception。需要程序员自己分析代码决定是否捕获和处理,比如空指针,被0除

6. super出现在父类的子类中。有三种存在方式
	1. super.xxx(xxx为变量名或对象名)意思是获取父类中xxx的变量或引用
	2. super.xxx(); (xxx为方法名)意思是直接访问并调用父类中的方法
	3. super() 调用父类构造
	4. 注：super只能指代其直接父类

7. this() & super()在构造方法中的区别
	1. 调用super()必须写在子类构造方法的第一行, 否则编译不通过
	2. super()是从子类调用父类构造, this()是在同一类中调用其他构造。但均需要放在第一行
	3. 尽管可以用this调用一个构造器, 却不能调用2个
	4. this和super不能出现在同一个构造器中, 否则编译不通过
	5. this()、super()都指的对象,不可以在static环境中使用
	6. **本质this指向本对象的指针。super是一个关键字**


----


<span id = "java_collection"></span>
#### Java的集合框架 [(TOP)](#home)
1. 容器介绍<br>
	1. List
		1. Arraylist：数组（查询快,增删慢 线程不安全,效率高 ）
		2. Vector：数组（查询快,增删慢 线程安全,效率低 ）
		3. LinkedList：链表（查询慢,增删快 线程不安全,效率高 ）
		
	2. Set
		1. HashSet（无序，唯一）:哈希表或者叫散列集(hash table)
		2. LinkedHashSet：链表和哈希表组成 。 由链表保证元素的排序 ， 由哈希表证元素的唯一性
		3. TreeSet（有序，唯一）：红黑树(自平衡的排序二叉树。)
	
	3. Map
		1. HashMap：基于哈希表的Map接口实现（哈希表对键进行散列，Map结构即映射表存放键值对）
		2. LinkedHashMap:HashMap 的基础上加上了链表数据结构
		3. HashTable:哈希表
		4. TreeMap:红黑树（自平衡的排序二叉树）

2. HashMap和Hashtable的区别
	1. Hashtable是基于陈旧的Dictionary的Map接口的实现，而HashMap是基于哈希表的Map接口的实现
	2. 从方法上看，HashMap去掉了Hashtable的contains方法
	3. HashTable是同步的(线程安全)，而HashMap线程不安全，效率上HashMap更快
	4. HashMap允许空键值，而Hashtable不允许
	5. HashMap的iterator迭代器执行快速失败机制，也就是说在迭代过程中修改集合结构，除非调用迭代器自身的remove方法，否则以其他任何方式的修改都将抛出并发修改异常。而Hashtable返回的Enumeration不是快速失败的。

3. List，Set,Map三者的区别及总结
	1. List：对付顺序的好帮手<br>
	List接口存储一组不唯一（可以有多个元素引用相同的对象），有序的对象
	
	2. Set:注重独一无二的性质<br>
	不允许重复的集合。不会有多个元素引用相同的对象。
	
	3. Map:用Key来搜索的专家<br>
	使用键值对存储。Map会维护与Key有关联的值。两个Key可以引用相同的对象，但Key不能重复，典型的Key是String类型，但也可以是任何对象。

4. TODO: 了解 CopyOnWrite 的具体实现逻辑

5. Set介绍<br>
	Set集合元素存取无序，且元素不可重复。常用的Set介绍:
	1. HashSet不保证迭代顺序，线程不安全；
	2. LinkedHashSet是Set接口的哈希表和链接列表的实现，保证迭代顺序，线程不安全。
	3. TreeSet：可以对Set集合中的元素排序，元素以二叉树形式存放，线程不安全。

6. List介绍<br>
	1. 常用的的List容器：ArrayList、LinkedList、Vector
		1. ArrayList是基于可变大小的数组实现
		2. LinkedList是链接列表的实现
		3. Vector功能和实现与ArrayList类似，但是具备线程安全，不过性能较差
	2. 性能特点
		1. 存取方面（get和set）：ArrayList要优于LinkedList，因为LinkedList要移动指针。
		2. 插入和删除：LinkedList要好一些，因为ArrayList要移动数据，更新索引
		3. 内存消耗：LinkedList需要更多的内存，因为需要维护指向后继结点的指针

7. Map集合
	1. Hashtable:基于Dictionary类，线程安全，速度快。底层是哈希表数据结构。是同步的。 不允许null作为键，null作为值。
	2. Properties:Hashtable的子类。用于配置文件的定义和操作，使用频率非常高，同时键和值都是字符串。
	3. HashMap：线程不安全，底层是数组加链表实现的哈希表。允许null作为键，null作为值。HashMap去掉了contains方法。 注意：HashMap不保证元素的迭代顺序。如果需要元素存取有序，请使用LinkedHashMap
	4. TreeMap：可以用来对Map集合中的键进行排序。
	5. ConcurrentHashMap:是JUC包下的一个并发集合。ConcurrentHashMap既不允许key为null 也不允许value为null
	
8. HashMap的工作原理<br>
HashMap维护了一个Entry数组，Entry内部类有key,value，hash和next四个字段，其中next也是一个Entry类型。可以将Entry数组理解为一个个的散列桶。每一个桶实际上是一个单链表。当执行put操作时，会根据key的hashcode定位到相应的桶。遍历单链表检查该key是否已经存在，如果存在，覆盖该value，反之，新建一个新的Entry，并放在单链表的头部。当通过传递key调用get方法时，它再次使用key.hashCode()来找到相应的散列桶，然后使用key.equals()方法找出单链表中正确的Entry，然后返回它的值。	
		
9. HashSet和HashMap如何检查重复<br>
	当你把对象加入HashSet时，HashSet会先计算对象的hashcode值来判断对象加入的位置，同时也会与其他加入的对象的hashcode值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同hashcode值的对象，这时会调用equals（）方法来检查hashcode相等的对象是否真的相同。如果两者相同，HashSet就不会让加入操作成功。（摘自我的Java启蒙书《Head fist java》第二版）
<br>
	1. hashCode（）与equals（）的相关规定：<br>
		1.	如果两个对象相等，则hashcode一定也是相同的
		2. 两个对象相等,对两个equals方法返回true
		3. 两个对象有相同的hashcode值，它们也不一定是相等的
		4. 综上，equals方法被覆盖过，则hashCode方法也必须被覆盖
		5. hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写hashCode()，则该class的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）。	
	2. ==与equals的区别<br>
		1. ==是判断两个变量或实例是不是指向同一个内存空间 
		2. equals是判断两个变量或实例所指向的内存空间的值是不是相同
		
9. Map并发
	1. 为什么使用ConcurrentHashMap而不是HashMap或Hashtable？<br>
HashMap的缺点：主要是多线程同时put时，如果同时触发了rehash操作，会导致HashMap中的链表中出现循环节点，进而使得后面get的时候，会死循环，CPU达到100%，所以在并发情况下不能使用HashMap。让HashMap同步：Map m = Collections.synchronizeMap(hashMap);而Hashtable虽然是同步的，使用synchronized来保证线程安全，但在线程竞争激烈的情况下HashTable的效率非常低下。因为当一个线程访问HashTable的同步方法时，其他线程访问HashTable的同步方法时，可能会进入阻塞或轮询状态。如线程1使用put进行添加元素，线程2不但不能使用put方法添加元素，并且也不能使用get方法来获取元素，所以竞争越激烈效率越低。

	2. ConcurrentHashMap的原理：<br>
Hashtable容器在竞争激烈的并发环境下表现出效率低下的原因在于所有访问Hashtable的线程都必须竞争同一把锁，那假如容器里有多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效的提高并发访问效率，这就是ConcurrentHashMap所使用的锁分段技术，首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。

10. Collection 和 Collections的区别
	1. Collection是集合类的上级接口，子接口主要有Set 和List、Queue 
	2. Collections是针对集合类的一个辅助类，提供了操作集合的工具方法：一系列静态方法实现对各种集合的搜索、排序、线程安全化等操作。

11. Map的实现类的介绍
	1. HashMap基于散列表来的实现，即使用hashCode()进行快速查询元素的位置，显著提高性能。插入和查询“键值对”的开销是固定的。可以通过设置容量和装载因子，以调整容器的性能。
	2. LinkedHashMap, 类似于HashMap,但是迭代遍历它时，保证迭代的顺序是其插入的次序，因为它使用链表维护内部次序。此外可以在构造器中设定LinkedHashMap，使之采用LRU算法。使没有被访问过的元素或较少访问的元素出现在前面，访问过的或访问多的出现在后面。这对于需要定期清理元素以节省空间的程序员来说，此功能使得程序员很容易得以实现。
	3. TreeMap, 是基于红黑树的实现。同时TreeMap实现了SortedMap接口，该接口可以确保键处于排序状态。所以查看“键”和“键值对”时，所有得到的结果都是经过排序的，次序由自然排序或提供的Comparator决定。SortedMap接口拥有其他额外的功能，如：返回当前Map使用的Comparator比较器，firstKey()，lastKey(),headMap(toKey),tailMap(fromKey)以及可以返回一个子树的subMap()方法等。
	4. WeakHashMap，表示弱键映射，WeakHashMap 的工作与正常的 HashMap 类似，但是使用弱引用作为 key，意思就是当 key 对象没有任何引用时，key/value 将会被回收。
	5. ConcurrentHashMap， 在HashMap基础上分段锁机制实现的线程安全的HashMap。
	6. IdentityHashMap 使用==代替equals() 对“键”进行比较的散列映射。专为解决特殊问题而设计。
	7. HashTable：基于Dictionary类的Map接口的实现，它是线程安全的。

12. 队列
	1. LinkedList 支持双向列表操作
	2.  PriorityQueue 按优先级组织的队列，元素的出队次序由元素的自然排序或者由Comparator比较器指定。
	3. BlockingQueue:阻塞队列。在进行获取元素时，它会等待队列变为非空；当在添加一个元素时，它会等待队列中的可用空间。<br>BlockingQueue接口是Java集合框架的一部分，主要用于实现生产者-消费者模式。我们不需要担心等待生产者有可用的空间，或消费者有可用的对象，因为它都在BlockingQueue的实现类中被处理了。

13. 对一组数据排序<br>
	1. 对于数组:我们可以使用Arrays.sort()方法。假如对象是数组，要求实现接口 Comparable , 否则会抛出强制转型异常
	2. 对于对象列表:对象实现Comparable接口，然后使用Collections.sort()方法
	3. 性能比较:Collections内部使用数组排序方法，所有它们两者都有相同的性能，只是Collections需要花时间将列表转换为数组。底层实现都是归并排序

----

<span id = "java_memory"></span>
#### Java内存 [(TOP)](#home)
1. GC
2. Java虚拟机
3. 垃圾回收
