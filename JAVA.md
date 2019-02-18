## Java
<span id = "home"></span>
#### 目录
1. [Java基础](#java_base)
2. [Java集合框架](#java_collection)
3. [Java虚拟机&内存](#java_memory)
4. [Java多线程&并发处理](#java_concurrent)

----


<span id = "java_base"></span>
#### Java基础 [(TOP)](#home)
1. 关于finally<br>
	1. 定义
		1. 不管有没有出现异常，finally块中代码都会执行
		2. 当try和catch中有return时，finally仍然会执行，finally比return先执行
		3. **finally是在return后面的表达式运算后执行的**（此时并没有返回运算后的值，而是先把要返回的值保存起来，管finally中的代码怎么样，返回的值都不会改变，仍然是之前保存的值），所以函数返回值是在finally执行前确定的
		4. finally中最好不要包含return，否则程序会提前退出，返回值不是try或catch中保存的返回值
		5. finally不执行的几种情况：程序提前终止如调用了System.exit, 病毒，断电

	2. 例子
		1. 情况1：<br>
			
			```
			try{} 
			catch(){}
			finally{} 
			return;
			```
			
			显然程序按顺序执行。
		
		2. 情况2:<br>
			
			```
			try{ return; }
			catch(){} 
			finally{} 
			return;
			```
			
			1. 程序执行try块中return之前（包括return语句中的表达式运算）代码；
			2. 再执行finally块，最后执行try中return;
			3. finally块之后的语句return，因为程序在try中已经return所以不再执行。

		3. 情况3:<br>

			```
			try{ } 
			catch(){return;} 
			finally{} 
			return;
         	```
         	
			1. 程序先执行try，如果遇到异常执行catch块，
			2. 有异常：则执行catch中return之前（包括return语句中的表达式运算）代码，再执行finally语句中全部代码，最后执行catch块中return. finally之后也就是4处的代码不再执行。
			3. 无异常：执行完try再finally再return.
			
		4. 情况4:<br>
			
			```
			try{ return; }
			catch(){} 
			finally{return;}
			```  
			
			1. 程序执行try块中return之前（包括return语句中的表达式运算）代码；
			2. 再执行finally块，因为finally块中有return所以提前退出。
			
		5. 情况5:<br>
		
			```
			try{} 
			catch(){return;}
			finally{return;}
          ```
          
          1. 程序执行catch块中return之前（包括return语句中的表达式运算）代码；
          2. 再执行finally块，因为finally块中有return所以提前退出.
          
		6. 情况6:<br>
		
			```
			try{ return;}
			catch(){return;} 
			finally{return;}
			```
			
          1. 程序执行try块中return之前（包括return语句中的表达式运算）代码；
          2. 有异常：执行catch块中return之前（包括return语句中的表达式运算）代码；则再执行finally块，因为finally块中有return所以提前退出。
          3. 无异常：则再执行finally块，因为finally块中有return所以提前退出。
	
	3. 参考
		1. https://www.cnblogs.com/fery/p/4709841.html

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
	当你把对象加入HashSet时，HashSet会先计算对象的hashcode值来判断对象加入的位置，同时也会与其他加入的对象的hashcode值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同hashcode值的对象，这时会调用equals（）方法来检查hashcode相等的对象是否真的相同。如果两者相同，HashSet就不会让加入操作成功。（摘自我的Java启蒙书《Head first java》第二版）
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
#### Java虚拟机&内存 [(TOP)](#home)
1. Java的内存区域
	1. 程序计数器<br>
		程序计数器是一块较小的内存空间，可以看作是当前线程所执行的字节码的行号指示器。字节码解释器工作时通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等功能都需要依赖这个计数器来完。<br>
		另外，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。
	
	2. Java虚拟机栈<br>
		与程序计数器一样，Java虚拟机栈也是线程私有的，它的生命周期和线程相同，描述的是Java方法执行的内存模型。<br>
		Java内存可以粗糙的区分为堆内存（Heap）和栈内存(Stack),其中栈就是现在说的虚拟机栈，或者说是虚拟机栈中局部变量表部分。<br>
		局部变量表主要存放了编译器可知的各种数据类型、对象引用。
	
	3. 本地方法栈<br>
		和虚拟机栈所发挥的作用非常相似，区别是： 虚拟机栈为虚拟机执行Java方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的Native方法服务。
	
	4. 堆<br>
		Java虚拟机所管理的内存中最大的一块，Java堆是所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。Java堆是垃圾收集器管理的主要区域，因此也被称作GC堆（Garbage Collected Heap）.从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以Java堆还可以细分为：新生代和老年代：在细致一点有：Eden空间、From Survivor、To Survivor空间等。进一步划分的目的是更好地回收内存，或者更快地分配内存。<br>
	
	5. 方法区<br>
	方法区与Java堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即使编译器编译后的代码等数据。
<br><br>
	HotSpot虚拟机中方法区也常被称为 “永久代”，本质上两者并不等价。仅仅是因为HotSpot虚拟机设计团队用永久代来实现方法区而已，这样HotSpot虚拟机的垃圾收集器就可以像管理Java堆一样管理这部分内存了。但是这并不是一个好主意，因为这样更容易遇到内存溢出问题。
相对而言，垃圾收集行为在这个区域是比较出现的，但并非数据进入方法区后就“永久存在”了。

	6. 运行时常量池<br>
	运行时常量池是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有常量池信息（用于存放编译期生成的各种字面量和符号引用）
	
	7. 直接内存<br>
	直接内存并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致OutOfMemoryError异常出现。
<br><br>
	JDK1.4中新加入的NIO(New Input/Output)类，引入了一种基于通道（Channel） 与缓存区（Buffer） 的I/O方式，它可以直接使用Native函数库直接分配堆外内存，然后通过一个存储在java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为避免了在Java堆和Native堆之间来回复制数据。
<br><br>
	本机直接内存的分配不会收到Java堆的限制，但是，既然是内存就会受到本机总内存大小以及处理器寻址空间的限制。

2. HotSpot虚拟机对象处理<br>
	HotSpot虚拟机在Java堆中对象分配、布局和访问的全过程。
	1. 对象的创建<br>
虚拟机遇到一条new指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执行相应的类加载过程。
<br><br>
	在类加载检查通过后，接下来虚拟机将为新生对象分配内存。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从Java堆中划分出来。分配方式有 “指针碰撞” 和 “空闲列表” 两种，选择那种分配方式由Java堆是否规整决定，而Java堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定。
<br><br>
虚拟机采用CAS配上失败重试的方式保证更新操作的原子性。
<br><br>
接下来，虚拟机要对对象进行必要的设置，例如这个对象是那个类的实例、如何才能找到类的元数据信息、对象的哈希吗、对象的GC分代年龄等信息。这些信息存放在对象头中，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会与不同的设置方式。
new指令执行完后，再按照程序员的意愿执行init方法后一个真正可用的对象才诞生。

	2. 对象的内存布局<br>
在Hotspot虚拟机中，对象在内存中的布局可以分为3快区域：对象头、实例数据和对齐填充。
<br><br>
	Hotspot虚拟机的对象头包括两部分信息，第一部分用于存储对象自身的自身运行时数据（哈希吗、GC分代年龄、锁状态标志等等），另一部分是类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是那个类的实例。
<br><br>
	实例数据部分是对象真正存储的有效信息，也是在程序中所定义的各种类型的字段内容。
<br><br>
	对齐填充部分不是必然存在的，也没有什么特别的含义，仅仅起占位作用。 因为Hotspot虚拟机的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说就是对象的大小必须是8字节的整数倍。而对象头部分正好是8字节的倍数（1倍或2倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。

	3. 对象的访问定位<br>
		建立对象就是为了使用对象，我们的Java程序通过Java虚拟机栈上的reference数据来操作堆上的具体对象。对象的访问方式有虚拟机实现而定，目前主流的访问方式有①使用句柄和②直接指针两种：
<br><br>
	如果使用句柄的话，那么Java堆中将会划分出一块内存来作为句柄池，reference中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息； 
<br><br>
	如果使用直接指针访问，那么Java堆对像的布局中就必须考虑如何防止访问类型数据的相关信息，reference中存储的直接就是对象的地址。
<br><br>
这两种对象访问方式各有优势。使用句柄来访问的最大好处是reference中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而reference本身不需要修改。使用直接指针访问方式最大的好处就是速度快，它节省了一次指针定位的时间开销。
	
3. 判断对象是否有被引用<br>
	堆中几乎放着所有的对象实例，对堆垃圾回收前的第一步就是要判断那些对象已经死亡（即不能再被任何途径使用的对象）
	1. 引用计数法<br>
		给对象中添加一个引用计数器，每当有一个地方引用它，计数器就加1；当引用失效，计数器就减1；任何时候计数器为0的对象就是不可能再被使用的。
<br><br>
这个方法实现简单，效率高，但是目前主流的虚拟机中并没有选择这个算法来管理内存，其最主要的原因是它很难解决对象之间相互循环引用的问题。
	2. 可达性分析法<br>
	这个算法的基本思想就是通过一系列的称为 “GC Roots” 的对象作为起点，从这些节点开始向下搜索，节点所走过的路径称为引用链，当一个对象到GC Roots没有任何引用链相连的话，则证明此对象是不可用的。
	
4. 四种引用类型
	1. 强引用<br>
		以前我们使用的大部分引用实际上都是强引用，这是使用最普遍的引用。如果一个对象具有强引用，那就类似于必不可少的生活用品，垃圾回收器绝不会回收它。当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足问题。

	2. 软引用（SoftReference）<br>
		如果一个对象只具有软引用，那就类似于可有可物的生活用品。如果内存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。
<br><br>
		软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收，JAVA虚拟机就会把这个软引用加入到与之关联的引用队列中。

	3. 弱引用（WeakReference）<br>
		如果一个对象只具有弱引用，那就类似于可有可物的生活用品。弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它 所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程， 因此不一定会很快发现那些只具有弱引用的对象。
<br><br>
		弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。

	4. 虚引用（PhantomReference）<br>
		"虚引用"顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收。
	<br><br>
		虚引用主要用来跟踪对象被垃圾回收的活动。
		<br>
		虚引用必须和引用队列（ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。程序如果发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。
	
	5. 垃圾回收的细节
		1. 即使在可达性分析法中不可达的对象，也并非是“非死不可”。<br>
		这时候它们暂时处于“缓刑阶段”，要真正宣告一个对象死亡，至少要经历两次标记过程；可达性分析法中不可达的对象被第一次标记并且进行一次筛选，筛选的条件是此对象是否有必要执行finalize方法。当对象没有覆盖finalize方法，或finalize方法已经被虚拟机调用过时，虚拟机将这两种情况视为没有必要执行。被判定为需要执行的对象将会被放在一个队列中进行第二次标记，除非这个对象与引用链上的任何一个对象建立关联，否则就会被真的回收。

		2. 回收方法区<br>
			方法区（或Hotspot虚拟中的永久代）的垃圾收集主要回收两部分内容：废弃常量和无用的类。
<br><br>
			判定一个常量是否是“废弃常量”比较简单，而要判定一个类是否是“无用的类”的条件则相对苛刻许多。类需要同时满足下面3个条件才能算是 “无用的类” ：
			1. 该类所有的实例都已经被回收，也就是Java堆中不存在该类的任何实例。
			2. 加载该类的ClassLoader已经被回收。
			3. 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。
	
5. 垃圾收集算法
	1. 标记-清除算法<br>
		算法分为“标记”和“清除”阶段：首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象。它是最基础的收集算法，会带来两个明显的问题；1：效率问题和2：空间问题（标记清除后会产生大量不连续的碎片）
	![](https://github.com/yinfork/Android-Interview/blob/master/res/java/jvm/mark_clear.png?raw=true)

	2. 复制算法<br>
		为了解决效率问题，“复制”收集算法出现了。它可以将内存分为大小相同的两块，每次使用其中的一块。当这一块的内存使用完后，就将还存活的对象复制到另一块去，然后再把使用的空间一次清理掉。这样就使每次的内存回收都是对内存区间的一半进行回收。
	![](https://github.com/yinfork/Android-Interview/blob/master/res/java/jvm/copy.png?raw=true)

	3. 标记-整理算法<br>
		根据老年代的特点特出的一种标记算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象回收，而是让所有存活的对象向一段移动，然后直接清理掉端边界以外的内存。
	![](https://github.com/yinfork/Android-Interview/blob/master/res/java/jvm/resize.png?raw=true)

	4. 分代收集算法<br>
	当前虚拟机的垃圾收集都采用分代收集算法，这种算法没有什么新的思想，只是根据对象存活周期的不同将内存分为几块。一般将java堆分为新生代和老年代，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。
	<br><br>
	比如在新生代中，每次收集都会有大量对象死去，所以可以选择复制算法，只需要付出少量对象的复制成本就可以完成每次垃圾收集。而老年代的对象存活几率是比较高的所以我们可以选择“标记-清理”或“标记-整理”算法进行垃圾收集。	
	
6. 堆内存分配与回收策略
	1. 堆内存分布介绍<br>
		堆内存分为新生代、老年代和永久代。新生代又被进一步分为：Eden区＋Survivor1区＋Survivor2区。值得注意的是，在 JDK 1.8中移除整个永久代，取而代之的是一个叫元空间（Metaspace）的区域（永久代使用的是JVM的堆内存空间，而元空间使用的是物理内存，直接受到本机的物理内存限制）。
		![](https://github.com/yinfork/Android-Interview/blob/master/res/java/jvm/memory.jpeg?raw=true)
	 
	2. 对象优先在Eden区分配<br>
大多数情况下，对象在新生代中Eden区分配。当Eden区没有足够空间进行分配时，虚拟机将发起一次Minor GC.
<br><br>
Minor Gc和Full GC 有什么不同呢？
		1. 新生代GC（Minor GC）:指发生新生代的的垃圾收集动作，Minor GC非常频繁，回收速度一般也比较快。

		2. 老年代GC（Major GC/Full GC）:指发生在老年代的GC，出现了Major GC经常会伴随至少一次的Minor GC（并非绝对），Major GC的速度一般会比Minor GC的慢10倍以上。

	3. 大对象直接进入老年代<br>
大对象就是需要大量连续内存空间的对象（比如：字符串、数组）。

	4. 长期存活的对象将进入老年代<br>
	既然虚拟机采用了分代收集的思想来管理内存，那么内存回收时就必须能识别那些对象应放在新生代，那些对象应放在老年代中。为了做到这一点，虚拟机给每个对象一个对象年龄（Age）计数器。

	5. 动态对象年龄判定<br>
	为了更好的适应不同程序的内存情况，虚拟机不是永远要求对象年龄必须达到了某个值才能进入老年代，如果Survivor 空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无需达到要求的年龄。

7. 类加载器
	1. 类与类加载器<br>
	**对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立其在Java虚拟机中的唯一性。如果两个类来源于同一个Class文件，只要加载它们的类加载器不同，那么这两个类就必定不相等。**

	2. 类加载器介绍<br>
	从Java虚拟机的角度分为两种不同的类加载器：启动类加载器（Bootstrap ClassLoader） 和其他类加载器。其中启动类加载器，使用C++语言实现，是虚拟机自身的一部分；其余的类加载器都由Java语言实现，独立于虚拟机之外，并且全都继承自java.lang.ClassLoader类。（这里只限于HotSpot虚拟机）。
	<br><br>
	从Java开发人员的角度来看，绝大部分Java程序都会使用到以下3种系统提供的类加载器。<br>
		1. 启动类加载器（Bootstrap ClassLoader）：<br>
		这个类加载器负责将存放在\lib目录中的，或者被-Xbootclasspath参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如rt.jar，名字不符合的类库即使放在lib目录中也不会被加载）类库加载到虚拟机内存中。

		2. 扩展类加载器（Extension ClassLoader）：<br>
		这个加载器由sun.misc.Launcher$ExtClassLoader实现，它负责加载\lib\ext目录中的，或者被java.ext.dirs系统变量所指定的路径中的所有类库，开发者可以直接使用扩展类加载器。

		3. 应用程序类加载器（Application ClassLoader）：<br>
		这个类加载器由sun.misc.Launcher$AppClassLoader实现。由于这个类加载器是ClassLoader中的getSystemClassLoader()方法的返回值，所以一般也称它为系统类加载器。它负责加载用户类路径（ClassPath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。
<br><br>
我们的应用程序都是由这3种类加载器互相配合进行加载的，如果有必要，还可以加入自己定义的类加载器。

	3. 双亲委派模型<br>
	双亲委派模型（Pattern Delegation Model）,要求除了顶层的启动类加载器外，其余的类加载器都应该有自己的父类加载器。这里父子关系通常是子类通过组合关系而不是继承关系来复用父加载器的代码。
	![](https://github.com/yinfork/Android-Interview/blob/master/res/java/jvm/classloader.png?raw=true) <br>
双亲委派模型的工作过程： 如果一个类加载器收到了类加载的请求，先把这个请求委派给父类加载器去完成（所以所有的加载请求最终都应该传送到顶层的启动类加载器中），只有当父加载器反馈自己无法完成加载请求时，子加载器才会尝试自己去加载。
<br><br>
使用双亲委派模型来组织类加载器之间的关系，有一个显而易见的好处就是java类随着它的类加载器一起具备了一种带有优先级的层次关系。
<br><br>
注意：双亲委派模型是Java设计者们推荐给开发者们的一种类加载器实现方式，并不是一个强制性 的约束模型。在java的世界中大部分的类加载器都遵循这个模型，但也有例外。

8. Java内存模型规定相关<br>
	Java内存模型规定，对于多个线程共享的变量，存储在主内存当中，每个线程都有自己独立的工作内存（比如CPU的寄存器），线程只能访问自己的工作内存，不可以访问其它线程的工作内存。<br>
工作内存中保存了主内存共享变量的副本，线程要操作这些共享变量，只能通过操作工作内存中的副本来实现，操作完毕之后再同步回到主内存当中。

9. 类的加载时机和加载过程<br>
	TODO
	
10. 参考
	1. https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483910&idx=1&sn=246f39051a85fc312577499691fba89f&chksm=fd985467caefdd71f9a7c275952be34484b14f9e092723c19bd4ef557c324169ed084f868bdb#rd	
	2. https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483914&idx=1&sn=9aa157d4a1570962c39783cdeec7e539&chksm=fd98546bcaefdd7d9f61cd356e5584e56b64e234c3a403ed93cb6d4dde07a505e3000fd0c427#rd
	
	3. https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483934&idx=1&sn=f247f9bee4e240f5e7fac25659da3bff&chksm=fd98547fcaefdd6996e1a7046e03f29df9308bdf82ceeffd111112766ffd3187892700f64b40#rd

----	
	
<span id = "java_concurrent"></span>
#### Java多线程&并发处理 [(TOP)](#home)	
1. 基础概念
	1. 进程<br>
		进程是程序的一次执行过程，是系统运行程序的基本单位，因此进程是动态的。系统运行一个程序即是一个进程从创建，运行到消亡的过程。
	2. 线程<br>
		线程与进程相似，但线程是一个比进程更小的执行单位,是CPU调度和分派的基本单位。<br>
		一个进程在其执行的过程中可以产生多个线程。与进程不同的是同类的多个线程共享同一块内存空间和一组系统资源，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多
	3. 临界区<br>
临界区用来表示一种公共资源或者说是共享数据，可以被多个线程使用。但是每一次，只能有一个线程使用它，一旦临界区资源被占用，其他线程要想使用这个资源，就必须等待。在并行程序中，临界区资源是保护的对象。

2. 使用多线程的三种方法
	1. 	继承Thread类，实现run()方法，然后启动这个线程
	2. 实现Runnable接口，然后作为构造函数的参数传人到Thread，并且启动这个线程
	3. 使用线程池

3. 停止线程的方法<br>
	1. 不能使用废弃的方法停止：stop(),suspend()
	2. interrupt()方法虽然没有废弃，但是直接调用，线程时不用停止的。
	3. 假如线程正在阻塞，调用 interrupt() 会让线程内部抛出InterruptedException，从而提早地终结被阻塞状态
	4. 正确做法是使用 interrupted() 方法判断线程是否停止，如果是停止状态则return
	
		```
		public class MyThread extends Thread {
	
			@Override
			public void run() {
				while (true) {
					if (this.isInterrupted()) {
						System.out.println("ֹͣ停止了!");
						return;
					}
					System.out.println("timer=" + System.currentTimeMillis());
				}
			}
			public static void main(String[] args) throws InterruptedException {
				MyThread t=new MyThread();
				t.start();
				Thread.sleep(2000);
				t.interrupt();
			}
		}
	
		```
		
4. 线程的优先级<br>
	1. 每个线程都具有各自的优先级，线程的优先级可以在程序中表明该线程的重要性，如果有很多线程处于就绪状态，系统会根据优先级来决定首先使哪个线程进入运行状态。但这个并不意味着低
优先级的线程得不到运行，而只是它运行的几率比较小，如垃圾回收机制线程的优先级就比较低。所以很多垃圾得不到及时的回收处理。
	2. 线程优先级具有继承特性比如A线程启动B线程，则B线程的优先级和A是一样的。
	3. 线程优先级具有随机性也就是说线程优先级高的不一定每一次都先执行完。
	4. 用1～10表示优先级。优先级越高，数值越大
	
5. 线程的分类
	1. 用户线程：运行在前台，执行具体的任务，如程序的主线程、连接网络的子线程等都是用户线程
	2. 守护线程：运行在后台，为其他前台线程服务
		1. 特点：一旦所有用户线程都结束运行，守护线程会随JVM一起结束工作
		2. 应用：数据库连接池中的检测线程、垃圾回收线程
		3. 注意：
			1. 在守护线程中产生的新线程也是守护线程
			2. 不是所有的任务都可以分配给守护线程来执行，比如读写操作或者计算逻辑
	
6. Synchronized原理
	1. Java中的每一个对象都可以作为锁。
		1. 对于同步方法，锁是当前实例对象。
		2. 对于静态同步方法，锁是当前对象的Class对象。
		3. 对于同步方法块，锁是Synchonized括号里配置的对象。
	2. 锁存放的位置<br>
		锁存在Java对象头里。如果对象是数组类型，则虚拟机用3个Word（字宽）存储对象头，如果对象是非数组类型，则用2字宽存储对象头。在32位虚拟机中，一字宽等于四字节，即32bit。
	3. 锁的升级<br>
		Java SE1.6为了减少获得锁和释放锁所带来的性能消耗，引入了“偏向锁”和“轻量级锁”，所以在Java SE1.6里锁一共有四种状态，无锁状态，偏向锁状态，轻量级状态和重量级锁状态，它会随着竞争情况逐渐升级。锁可以升级但不能降级，意味着偏向锁升级成轻量级锁后不能降级成偏向锁。这种锁升级却不能降级的策略，目的是为了提高获得锁和释放锁的效率。
		
7. Threadlocal原理<br>
ThreadLocal 为解决多线程程序的并发问题提供了一种新的思路。使用这个工具类可以很简洁地编写出优美的多线程程序。当使用 ThreadLocal 维护变量时，ThreadLocal 为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。
<br><br>
每个线程中都保有一个ThreadLocalMap的成员变量，ThreadLocalMap 内部采用WeakReference数组保存，数组的key即为ThreadLocal 内部的Hash值。

8. 指令的重排序
	1. 问题背景<br>
		现代CPU的主频越来越高，与cache的交互次数也越来越多。当CPU的计算速度远远超过访问cache时，会产生cache wait，过多的cache wait就会造成性能瓶颈 
	2. 解决方法<br>
		1. 原理<br>
			针对这种情况，多数架构（包括X86）采用了一种将cache分片的解决方案，即将一块cache划分成互不关联地多个 slots (逻辑存储单元，又名 Memory Bank 或 Cache Bank)，CPU可以自行选择在多个 idle bank 中进行存取。这种 SMP 的设计，显著提高了CPU的并行处理能力，也回避了cache访问瓶颈。
		2. Memory Bank的划分<br>
			一般 Memory bank 是按cache address来划分的。比如 偶数adress 0×12345000分到 bank 0, 奇数address 0×12345100分到 bank1。
		3. 架构图<br>
			![](https://github.com/yinfork/Android-Interview/blob/master/res/java/concurrent/java_cpu.jpg?raw=true)
		3. 实例讲解<br>
			一台配备双CPU的计算机，cache 按地址被分成了两块 cache banks，分别是cache bank0 和 cache bank1。
			1. 理想的内存访问指令顺序：
				1. CPU0往cache address 0×12345000 写入一个数字 1。因为address 0×12345000是偶数，所以值被写入 bank0.
				2. CPU1读取 bank0 address 0×12345000 的值，即数字1。
				3. CPU0往 cache 地址 0×12345100 ?写入一个数字 2。因为address 0×12345100是奇数，所以值被写入 bank1.
				4. CPU1读取 bank1 address ?0×12345100 的值，即数字2。
			2. 重排序后的内存访问指令顺序：
				1. CPU0 准备往 bank0 address 0×12345000 写入数字 1。
				2. CPU0检查 bank0 的可用性。发现 bank0 处于 busy 状态。
				3. **CPU0 为了防止 cache等待，发挥最大效能，将内存访问指令重排序。即先执行后面的 bank1 address 0×12345100 数字2的写入请求。**
				4. CPU0检查 bank1 可用性，发现bank1处于 idle 状态。
				5. CPU0 将数字2写入 bank1 address 0×12345100。
				6. **CPU1来读取 bank0 0×12345000，未读到 数字1，出错。**
				7. CPU0 继续检查 bank0 的可用性，发现这次bank0 可用了，然后将数字1写入 0×12345000。
				8. CPU1 读取 bank1 0×12345100，读到数字2，正确。
				9. **从上述触发步骤中，可以看到第 3 步发生了指令重排序，并导致第6步读到错误的数据。**
		
		4. 小结<br>
			通过对指令重排，CPU可以获得更快地响应速度，但也给编写并发程序的程序员带来了诸多挑战。<br>
内存屏障是用来防止CPU出现指令重排序的利器之一。
			
	3. 重排序的类型
		1. 编译时重排序<br>
			编译源代码时，编译器依据对上下文的分析，对指令进行重排序，以之更适合于CPU的并行执行。
		2. 运行时重排序<br>
			CPU在执行过程中，动态分析依赖部件的效能，对指令做重排序优化	
	
	4. 指令重排序的规则:as-if-serial<br>
		as-if-serial 语义的意思指:不管怎么重排序，单线程下的执行结果不能被改变	
		
	5. 指令重排序带来的问题
		1. A线程指令重排导致B线程出错
对于在同一个线程内，这样的改变是不会对逻辑产生影响的，但是在多线程的情况下指令重排序会带来问题。看下面这个情景:<br>
			在线程A中:
			
			```
			context = loadContext();
			inited = true;
			```

			在线程B中:
			
			```
			while(!inited ){ //根据线程A中对inited变量的修改决定是否使用context变量
			   sleep(100);
			}
			doSomethingwithconfig(context);
			```
			
			假设线程A中发生了指令重排序:
			
			```
			inited = true;
			context = loadContext();
			```
			
			那么B中很可能就会拿到一个尚未初始化或尚未初始化完成的context,从而引发程序错误。
			解决方法：对 inited 加上volatile修饰，防止指令重排序。所以以后这些 isInited, isLoad 的变量都可以考虑加上 volatile 修饰

		2. 指令重排导致双重判断单例模式失效
			
			```
			public class Singleton {
			private static Singleton instance = null;
			private Singleton() { }
			public static Singleton getInstance() {
				if(instance == null) {
					synchronzied(Singleton.class) {
					   if(instance == null) {
					   		instance = new Singleton();  //非原子操作
					   }
					}
				}
				return instance;
			}
			```
			
			看似简单的一段赋值语句：instance= new Singleton()，但是很不幸它并不是一个原子操作，其实际上可以抽象为下面几条JVM指令
			
			```
			memory =allocate();    //1：分配对象的内存空间 
			ctorInstance(memory);  //2：初始化对象 
			instance =memory;     //3：设置instance指向刚分配的内存地址
			```
			
			上面操作2依赖于操作1，但是操作3并不依赖于操作2，所以JVM是可以针对它们进行指令的优化重排序的，经过重排序后如下：
			
			```
			memory =allocate();    //1：分配对象的内存空间 
			instance =memory;     //3：instance指向刚分配的内存地址，此时对象还未初始化
			ctorInstance(memory);  //2：初始化对象
			```

			可以看到指令重排之后，instance指向分配好的内存放在了前面，而这段内存的初始化被排在了后面。
			<br>
			在线程A执行这段赋值语句，在初始化分配对象之前就已经将其赋值给instance引用，恰好另一个线程进入方法判断instance引用不为null，然后就将其返回使用，导致出错
			<br>
			解决方法：
			
			```
			private static Singleton instance = null;
			// 改成 volatile修饰 从而防止指令重排
			private volatile static Singleton instance = null;
			```
			
	6. 防止指令重排<br>
		在JDK1.5之后，可以使用volatile变量禁止指令重排序。<br>
		volatile关键字通过提供“内存屏障”的方式来防止指令被重排序，为了实现volatile的内存语义，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。
	
9. 内存屏障(Memory Barrier)
	1. 定义<br>
		内存屏障是一种CPU指令，用于控制特定条件下的重排序和内存可见性问题。Java编译器也会根据内存屏障的规则禁止重排序。
	2. 屏障类型<br>
		对于不同的处理器/指令集，JVM有不同的实现方式。Java的规范抽象出以下4中内存屏障类型
		1. LoadLoad屏障：对于这样的语句
			
			```
			Load1; 
			LoadLoad; 
			Load2
			```
			
			在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕。
			
		2. StoreStore屏障：对于这样的语句
			
			```
			Store1; 
			StoreStore;
			Store2
			```
			
			在Store2及后续写入操作执行前，保证Store1的写入操作对其它处理器可见。
		
		3. LoadStore屏障：对于这样的语句
			
			```
			Load1; 
			LoadStore; 
			Store2
			```
			
			在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕。
			
		4. StoreLoad屏障：对于这样的语句
			
			```
			Store1; 
			StoreLoad; 
			Load2
			```
			
			在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。它的开销是四种屏障中最大的。在大多数处理器的实现中，这个屏障是个万能屏障，兼具其它三种内存屏障的功能。
	
	3. 内存屏障指令并不是禁止重排序指令<br>
		其实不存在任何一种指令能够禁止乱序执行，我们能做到的只是把这一堆指令根据"分段"，比如在指令中插入一条full barrier指令，然后所有指令被分成了两段，barrier前面的我们称之为程序段A，后面的称之为程序段B，通过barrier我们能够保证A所有指令的执行都在B之前，也就是说，程序段A和B分别都是乱序执行的。<br><br>
		举例
		
		```
		full barrier;
		instance = new Singleton(); //C
		full barrier;	
		```
		
		在一个变量的赋值前后各加一个barrier，在外界看起来就好像是禁止了C处指令重排一样，其实C处又可以拆成一堆指令，这一堆指令在两个barrier之间其实又是乱序的

	4. 内存屏障的应用
		1. volatile禁止重排序的实现
			1. 在每个 volatile 写操作的前面插入一个 StoreStore 屏障
			2. 在每个 volatile 写操作的后面插入一个 StoreLoad 屏障
			3. 在每个 volatile 读操作的后面插入一个 LoadLoad 屏障
			4. 在每个 volatile 读操作的后面插入一个 LoadStore 屏障
			5. 示例图<br>
				![](https://github.com/yinfork/Android-Interview/blob/master/res/java/concurrent/volatile_read.png?raw=true)
				<br>
				![](https://github.com/yinfork/Android-Interview/blob/master/res/java/concurrent/volatile_write.png?raw=true)
	
		2. synchronized<br>
			synchronized也是可以保证线程可见性的，我们知道信号量只能实现锁的功能，它是没有我们之前说过的内存屏障的功能的，那其实synchronized在代码块最后也是会加入一个barrier的(应该是store barrier)
			
		3. final<br>
			final除了我们平时所理解的语义之外，其实还蕴含着**禁止把构造器final变量的赋值重排序到构造器外面**，实现方式就是在final变量的写之后插入一个store-store barrier
	
9. happen-before原则<br> 	Java内存模型规定，对于多个线程共享的变量，存储在主内存当中，每个线程都有自己独立的工作内存（比如CPU的寄存器），线程只能访问自己的工作内存，不可以访问其它线程的工作内存。<br>
	工作内存中保存了主内存共享变量的副本，线程要操作这些共享变量，只能通过操作工作内存中的副本来实现，操作完毕之后再同步回到主内存当中。<br>
	如何保证多个线程操作主内存的数据完整性是一个难题，Java内存模型也规定了工作内存与主内存之间交互的协议，也就是happen-before原则
	1. 程序次序法则，在同一个线程中，书写在前面的操作happen-before后面的操作；
	2. 监视器法则,对一个监视器的解锁一定发生在后续对同一监视器加锁之前；
	3. Volatie变量法则：写volatile变量一定发生在后续对它的读之前；
	4. 线程启动法则：Thread.start一定发生在线程中的动作；
	5. 线程终结法则：线程中的任何动作一定发生在括号中的动作之前（其他线程检测到这个线程已经终止，从Thread.join调用成功返回，Thread.isAlive()返回false）;
	6. 中断法则：一个线程调用另一个线程的interrupt一定发生在另一线程发现中断；
	7. 终结法则：一个对象的构造函数结束一定发生在对象的finalizer之前；
	8. 传递性：A发生在B之前，B发生在C之前，A一定发生在C之前。
	
	
10. Volatile<br>
	1. 原理<br>
		Java内存模型规定所有的变量都是存在主存当中（类似于前面说的物理内存），每个线程都有自己的工作内存（类似于前面的高速缓存）。线程对变量的所有操作都必须在工作内存中进行，而不能直接对主存进行操作。并且每个线程不能访问其他线程的工作内存。<br>
		在Java中，执行下面这个语句：<br>
		```
		i  = 10;
		```
		<br>
		执行线程必须先在自己的工作线程中对变量i所在的缓存行进行赋值操作，然后再写入主存当中。而不是直接将数值10写入主存当中

	2. volatile的特点
		1. Volatile保证了可见性
		2. Volatile不保证原子性
		3. Volatile可以避免指令重排序
	3. volatile和synchronized区别
		1. volatile仅能保证内存可见性，不能保证原子性；而synchronized则可以保证变量的修改可见性和原子性
		2. volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞。
		3. volatile本质是在告诉jvm当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取； synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住


11. 原子性
	1. 原子性的定义<br>
		即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。
	2. 	在Java中，对基本数据类型的变量的读取和赋值操作是原子性操作，即这些操作是不可被中断的，要么执行，要么不执行。如果要实现更大范围操作的原子性，可以通过synchronized和Lock来实现。
	3. 在32位平台下，对64位数据的读取和赋值是需要通过两个操作来完成的，不能保证其原子性。但是好像在最新的JDK中，JVM已经保证对64位数据的读取和赋值也是原子性操作了。
	4. 举例<br>
	
		```
		x = 10;        //语句1
		y = x;         //语句2
		x++;           //语句3
		x = x + 1;     //语句4
		```
		<br>
		以上其实只有语句1是原子性操作，其他三个语句都不是原子性操作。
	
12. 可见性<br>	
	可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。
	<br><br>
对于可见性，Java提供了volatile关键字来保证可见性。当一个共享变量被volatile修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。而普通的共享变量不能保证可见性，因为普通共享变量被修改之后，什么时候被写入主存是不确定的，当其他线程去读取时，此时内存中可能还是原来的旧值，因此无法保证可见性。
<br><br>
另外，通过synchronized和Lock也能够保证可见性，synchronized和Lock能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存当中。因此可以保证可见性。

13. 各种锁的介绍<br>
TODO

14. 死锁<br>
	TODO
	1. 银行家算法

20. 参考
	1. https://github.com/hadyang/interview/blob/master/java/volatile.md
	2. https://github.com/hadyang/interview/blob/master/java/threadlocal.md
	3. https://github.com/hadyang/interview/blob/master/java/synchronized.md
	4. https://github.com/Snailclimb/JavaGuide/blob/master/Java%E7%9B%B8%E5%85%B3/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E7%B3%BB%E5%88%97.md 
	5. http://ifeve.com/jvm-memory-reordering/
	6. http://www.importnew.com/23535.html
	7. https://www.jianshu.com/p/ec7200a26d8b


