## Java
<span id = "home"></span>
#### 目录
1. [Java基础](#java_base)
2. [Java集合框架](#java_collection)
3. [Java内存区域](#java_memory_pos)
4. [Java内存模型](#java_memory)
4. [Java类](#java_class)
5. [Java垃圾回收](#java_gc)
6. [Java多线程&并发处理](#java_concurrent)
7. [设计模式](#design_pattern)

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
	1. final修饰的变量是常量，必须进行初始化。可以显示初始化，也可以通过构造函数进行初始化。如果不初始化编译会报错。
	2. final修饰类，表示是最终类。最终类不能被继承, 最终方法不能被重写(可以重载)
	3. final修饰方法，代表这个方法不可以被子类的方法重写。final方法比非final方法要快，因为在编译的时候已经静态绑定了，不需要在运行时再动态绑定。
	4. 总结
		1. 接口中声明的所有变量本身是final的。
		2. final和abstract这两个关键字是反相关的，final类就不可能是abstract的。
		3. final方法在编译阶段绑定，称为静态绑定(static binding)。
	5. 好处
		1. 将类、方法、变量声明为final能够提高性能，这样JVM就有机会进行估计，然后优化 
		2. final变量可以安全的在多线程环境下进行共享，而不需要额外的同步开销。


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

8. null可以被强制转型为任意类型的对象

9. 代码执行次序
	1. 基本概念
		1. 构造函数
			1. 构造函数的作用是用于给对象进行初始化。
			2. 一个对象建立，构造函数只运行一次，而一般方法可以被该对象调用多次。
		2. 构造代码块
			1. 构造代码块的作用是给对象进行初始化。
			2. 对象一建立就运行构造代码块了，而且优先于构造函数执行。这里要强调一下，有对象建立，才会运行构造代码块，类不能调用构造代码块的，而且构造代码块与构造函数的执行顺序是前者先于后者执行。
		3. 静态代码块
			1. 它是随着类的加载而执行，只执行一次，并优先于主函数。具体说，静态代码块是由类调用的。类调用时，先执行静态代码块，然后才执行主函数的。
			2. **静态代码块其实就是给类初始化的，而构造代码块是给对象初始化的**。
			3. 静态代码块中的变量是局部变量，与普通函数中的局部变量性质没有区别。
			4. 一个类中可以有多个静态代码块
		4. 构造代码块与构造函数的区别是：<br>
			**构造代码块是给所有对象进行统一初始化，而构造函数是给对应的对象初始化，因为构造函数是可以多个的**，运行哪个构造函数就会建立什么样的对象，但无论建立哪个对象，都会先执行相同的构造代码块。也就是说，构造代码块中定义的是不同对象共性的初始化内容。
	
	2. 代码执行流程
		1. 多个静态成员变量, 静态代码块按顺序执行
		2. 单个类中: 静态代码 -> main方法 -> 构造块 -> 构造方法
		3. 构造块在每一次创建对象时执行
		4. 涉及父类和子类的初始化过程 
			1. 初始化父类中的静态成员变量和静态代码块
			2. 初始化子类中的静态成员变量和静态代码块
			3. 初始化父类的普通成员变量和构造代码块(按次序)，再执行父类的构造方法(注意父类构造方法中的子类方法覆盖)
			4. 初始化子类的普通成员变量和构造代码块(按次序)，再执行子类的构造方法

	3. 例子
		1. 单个类
			
			```
			public class HelloA {
			    public HelloA(){//构造函数
			        System.out.println("A的构造函数");   
			    }
			    {//构造代码块
			        System.out.println("A的构造代码块");  
			    }
			    static {//静态代码块
			        System.out.println("A的静态代码块");      
			    }
			    public static void main(String[] args) {
			        HelloA a=new HelloA();
			        HelloA b=new HelloA();
			    }
			}
			运行结果：
			A的静态代码块
			A的构造代码块
			A的构造函数
			A的构造代码块
			A的构造函数
			```
		
		
		2. 继承关系下
			
			```
			public class HelloA {
			    public HelloA(){//构造函数
			        System.out.println("A的构造函数");   
			    }
			    {//构造代码块
			        System.out.println("A的构造代码块");  
			    }
			    static {//静态代码块
			        System.out.println("A的静态代码块");      
			    }
			}
			public class HelloB extends HelloA{
			    public HelloB(){//构造函数
			        System.out.println("B的构造函数");   
			    }
			    {//构造代码块
			        System.out.println("B的构造代码块");  
			    }
			    static {//静态代码块
			        System.out.println("B的静态代码块");      
			    }
			    public static void main(String[] args) {
			        HelloB b=new HelloB();      
			    }
			}
			运行结果：
			A的静态代码块
			B的静态代码块
			A的构造代码块
			A的构造函数
			B的构造代码块
			B的构造函数
			```
	
	4. 参考
		1. https://www.jianshu.com/p/8a3d0699a923
		2. https://github.com/it-interview/EasyJob/blob/master/Questions/java_summary.md
	

10. 内部类和静态内部类
	1. 静态内部类不需要有指向外部类的引用。但非静态内部类需要持有对外部类的引用。
	2. 非静态内部类能够访问外部类的静态和非静态成员。静态内部类不能访问外部类的非静态成员，只能访问外部类的静态成员。

11. 数组复制方法
	1. for逐一复制
	2. System.arraycopy() -> 效率最高native方法
	3. Arrays.copyOf() -> 本质调用arraycopy
	4. clone方法 -> 返回Object[],需要强制类型转换

12. Java移位运算符<br>
	java中有三种移位运算符
	1. **<<** :左移运算符,x << 1,相当于x乘以2(不溢出的情况下),低位补0<br>
		注意：左移位数大于等于32位操作时，会先求余（%）后再进行左移操作。<br>
		示意图：<br>
		![](https://github.com/yinfork/Android-Interview/blob/master/res/java/base/left_shift.png?raw=true)
		
	2. **>>** :带符号右移,x >> 1,相当于x除以2,正数高位补0,负数高位补1<br>
		![](https://github.com/yinfork/Android-Interview/blob/master/res/java/base/right_shift.png?raw=true)<br>
		![](https://github.com/yinfork/Android-Interview/blob/master/res/java/base/right_shift_2.png?raw=true)
		
	3. **>>>** :无符号右移,忽略符号位,空位都以0补齐<br>
		![](https://github.com/yinfork/Android-Interview/blob/master/res/java/base/no_sign_right_shift.png?raw=true)

	4. 参考：https://zhuanlan.zhihu.com/p/30108890
	
	
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

3. ArrayList和LinkedList的区别如下：
	1. ArrayList的实现是基于数组，LinkedList的实现是基于双向链表。
	2. 对于随机访问，ArrayList优于LinkedList，ArrayList可以根据下标以O(1)时间复杂度对元素进行随机访问。而LinkedList的每一个元素都依靠地址指针和它后一个元素连接在一起，在这种情况下，查找某个元素的时间复杂度是O(n)
	3. 对于插入和删除操作，LinkedList优于ArrayList，因为当元素被添加到LinkedList任意位置的时候，不需要像ArrayList那样重新计算大小或者是更新索引。
	4. LinkedList比ArrayList更占内存，因为LinkedList的节点除了存储数据，还存储了两个引用，一个指向前一个元素，一个指向后一个元素。

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
	1. 哈希函数<br>
		哈希函数也叫散列函数，它对不同的输出值得到一个固定长度的消息摘要。理想的哈希函数对于不同的输入应该产生不同的结构，同时散列结果应当具有同一性（输出值尽量均匀）和雪崩效应（微小的输入值变化使得输出值发生巨大的变化）。

	2. 冲突解决
		1. 开放地址法：以发生冲突的哈希地址为输入，通过某种哈希冲突函数得到一个新的空闲的哈希地址的方法。有以下几种方式：<br>
			![](https://github.com/yinfork/Android-Interview/blob/master/res/java/collection/hash_conflict.png?raw=true)
			1. 线性探查法：从发生冲突的地址开始，依次探查下一个地址，直到找到一个空闲单元。
			2. 平方探查法(二次方探测)：设冲突地址为d0，则探查序列为：d0+1^2,d0-1^2,d0+2^2...
		
		2. 拉链法：把所有的同义词用单链表链接起来。在这种方法下，哈希表每个单元中存放的不再是元素本身，而是相应同义词单链表的头指针。HashMap就是使用这种方法解决冲突的。	
		
	3. 参考
		1. https://github.com/hadyang/interview/blob/master/basic/algo/hash.md
		2. https://www.oschina.net/question/2540936_2186006
		
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
		
9. ConcurrentHashMap
	1. 为什么使用ConcurrentHashMap而不是HashMap或Hashtable？<br>
HashMap的缺点：主要是多线程同时put时，如果同时触发了rehash操作，会导致HashMap中的链表中出现循环节点，进而使得后面get的时候，会死循环，CPU达到100%，所以在并发情况下不能使用HashMap。让HashMap同步：Map m = Collections.synchronizeMap(hashMap);而Hashtable虽然是同步的，使用synchronized来保证线程安全，但在线程竞争激烈的情况下HashTable的效率非常低下。因为当一个线程访问HashTable的同步方法时，其他线程访问HashTable的同步方法时，可能会进入阻塞或轮询状态。如线程1使用put进行添加元素，线程2不但不能使用put方法添加元素，并且也不能使用get方法来获取元素，所以竞争越激烈效率越低。

	2. ConcurrentHashMap的原理：<br>
		首先将数据分成一段一段地存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。
		
	3. ConcurrentHashMap的结构<br>
		ConcurrentHashMap由Segment数组结构和HashEntry数组结构组成；
Segment是一种可重入锁（ReentrantLock），HashEntry用于存储键值对数据；
一个ConcurrentHashMap包含一个由若干个Segment对象组成的数组，每个Segment对象守护整个散列映射表的若干个桶，每个桶是由若干个HashEntry对象链接起来的链表，table是一个由HashEntry对象组成的数组，table数组的每一个数组成员就是散列映射表的一个桶。<br>
			![](https://github.com/yinfork/Android-Interview/blob/master/res/java/collection/ConcurrentHashMap_struct.jpg?raw=true)
		1. HashEntry类
			
			```
			static final class HashEntry<K,V> { 
			       final K key;                       // 声明 key 为 final 型
			       final int hash;                   // 声明 hash 值为 final 型 
			       volatile V value;                 // 声明 value 为 volatile 型
			       final HashEntry<K,V> next;      // 声明 next 为 final 型 
			 
			       HashEntry(K key, int hash, HashEntry<K,V> next, V value) { 
			           this.key = key; 
			           this.hash = hash; 
			           this.next = next; 
			           this.value = value; 
			       } 
			}
			
			```
			
			1. 冲突处理<br>
				在ConcurrentHashMap中，在散列时如果产生“碰撞”，将采用“分离链接法”来处理“碰撞”：把“碰撞”的HashEntry对象链接成一个链表。由于HashEntry的next域为final型，所以新节点只能在链表的表头处插入。	
			2. HashEntry对象的不变性(使得get不用加锁)<br>
				HashEntry对象的key、hash、next都声明为final类型，这意味着不能把节点添加到链表的中间和尾部，也不能再链表的中间和尾部删除节点；这个特性可以保证：在访问某个节点时，这个节点之后的链接不改变。<br>
				同时，HashEntry的value被声明为volatile类型，Java的内存模型可以保证：某个写线程对value的写入马上可以被后续的读线程看到。ConcurrentHashMap不允许用null为键和值，当读线程读到某个HashEntry的value为null时，便知道产生了冲突——发生了重排序现象，需要加锁后重新读这个value值。这些特性保证读线程不用加锁也能正确访问ConcurrentHashMap。<br>
		2. Segment类

			```
			static final class Segment<K,V> extends ReentrantLock implements Serializable { 
			       /** 
			        * 在本 segment 范围内，包含的 HashEntry 元素的个数
			        * 该变量被声明为 volatile 型
			        */ 
			       transient volatile int count; 
			 
			       /** 
			        * table 被更新的次数
			        */ 
			       transient int modCount; 
			 
			       /** 
			        * 当 table 中包含的 HashEntry 元素的个数超过本变量值时，触发 table 的再散列
			        */ 
			       transient int threshold; 
			 
			       /** 
			        * table 是由 HashEntry 对象组成的数组
			        * 如果散列时发生碰撞，碰撞的 HashEntry 对象就以链表的形式链接成一个链表
			        * table 数组的数组成员代表散列映射表的一个桶
			        * 每个 table 守护整个 ConcurrentHashMap 包含桶总数的一部分
			        * 如果并发级别为 16，table 则守护 ConcurrentHashMap 包含的桶总数的 1/16 
			        */ 
			       transient volatile HashEntry<K,V>[] table; 
			 
			       /** 
			        * 装载因子
			        */ 
			       final float loadFactor; 
			 
			       Segment(int initialCapacity, float lf) { 
			           loadFactor = lf; 
			           setTable(HashEntry.<K,V>newArray(initialCapacity)); 
			       } 
			 
			       /** 
			        * 设置 table 引用到这个新生成的 HashEntry 数组
			        * 只能在持有锁或构造函数中调用本方法
			        */ 
			       void setTable(HashEntry<K,V>[] newTable) { 
			           // 计算临界阀值为新数组的长度与装载因子的乘积
			           threshold = (int)(newTable.length * loadFactor); 
			           table = newTable; 
			       } 
			 
			       /** 
			        * 根据 key 的散列值，找到 table 中对应的那个桶（table 数组的某个数组成员）
			        */ 
			       HashEntry<K,V> getFirst(int hash) { 
			           HashEntry<K,V>[] tab = table; 
			           // 把散列值与 table 数组长度减 1 的值相“与”，
			           // 得到散列值对应的 table 数组的下标
			           // 然后返回 table 数组中此下标对应的 HashEntry 元素
			           return tab[hash & (tab.length - 1)]; 
			       } 
			}
			
			```
		3. ConcurrentHashMap类	
			1. ConcurrentHashMap在默认并发级别会创建包含16个Segment对象的数组。
			2. 每个Segment的成员对象table包含若干个散列表的桶。
			3. 每个桶是由HashEntry链接起来的一个链表。
			4. 如果键能均匀散列，每个Segment大约守护整个散列表中桶总数的 1/16。
	4. ConcurrentHashMap的操作
		1. put操作
			1. 根据key算出对应的hash值

				```
				public V put(K key, V value) { 
				        if (value == null)          //ConcurrentHashMap 中不允许用 null 作为映射值
				           throw new NullPointerException();
				        // 计算键对应的散列码
				        int hash = hash(key.hashCode());        
				        // 根据散列码找到对应的 Segment 
				        return segmentFor(hash).put(key, hash, value, false); 
				}
				
				```
			2. 根据hash值找到对应的Segment对象

				```
				/** 
				 * 使用 key 的散列码来得到 segments 数组中对应的 Segment 
				 */ 
				final Segment<K,V> segmentFor(int hash) { 
				    // 将散列值无符号右移 segmentShift 个位，并在高位填充 0 
				    // 然后把得到的值与 segmentMask 相“与”
				    // 从而得到 hash 值对应的 segments 数组的下标值
				    // 最后根据下标值返回散列码对应的 Segment 对象
				    return segments[(hash >>> segmentShift) & segmentMask]; 
				}
				
				```	
			3. 在Segment中执行具体的put操作

			```
			V put(K key, int hash, V value, boolean onlyIfAbsent) { 
	        lock();  // 加锁，这里是锁定某个 Segment 对象而非整个 ConcurrentHashMap 
	        try { 
	            int c = count; 
	            if (c++ > threshold)     // 如果超过再散列的阈值
	                rehash();            // 执行再散列，table 数组的长度将扩充一倍
	 
	            HashEntry<K,V>[] tab = table; 
	            // 把散列码值与 table 数组的长度减 1 的值相“与”
	            // 得到该散列码对应的 table 数组的下标值
	            int index = hash & (tab.length - 1); 
	            // 找到散列码对应的具体的那个桶
	            HashEntry<K,V> first = tab[index]; 
	 
	            HashEntry<K,V> e = first; 
	            while (e != null && (e.hash != hash || !key.equals(e.key))) 
	                   e = e.next; 
	 
	            V oldValue; 
	            if (e != null) {            // 如果键值对以经存在
	                oldValue = e.value; 
	                if (!onlyIfAbsent) 
	                    e.value = value;    // 设置 value 值
	                } 
	                else {                        // 键值对不存在 
	                    oldValue = null; 
	                    ++modCount;         // 要添加新节点到链表中，所以 modCont 要加 1  
	                    // 创建新节点，并添加到链表的头部 
	                    tab[index] = new HashEntry<K,V>(key, hash, first, value); 
	                    count = c;               // 写 count 变量
	                } 
	                return oldValue; 
	            } finally { 
	                unlock();                     // 解锁
	            } 
	       }
			
			```	
			
			4. 插入操作需要两个步骤
				1. 是否需要扩容
				在插入元素前会先判断Segment里面的HashEntry数组是否超过容量（threshold)，如果超过则进行扩容。Segment的扩容比HashMap更恰当，HashMap是在插入元素后再判断元素是否已经到达容量。
				2. 如何扩容
				首先会创建一个容量是原来容量两倍的数组，然后将原数组里面的元素进行再散列后插入到新数组里；**ConcurrentHashMap不会对整个容器进行扩容，只对某个Segment进行扩容。**
		2. get操作			
			1. 先经过一次再散列，然后使用这个散列值通过散列运算定位到Segment，再通过散列算法定位到元素。<br>
			**get操作的高效之处在于get过程不需要加锁，除非读到的值是null才会加锁重读。原因是它的get方法里面将要使用的共享变量都定义成volatile类型，在多线程之间保持可见性，原理是根据Java内存模型的happen-before原则，对volatile字段的写入操作先于读操作。**<br>
			代码如下：
			
				```
				public V get(Object key){
				    int hash = hash(key.hashCode());
				    return segmentFor(hash).get(key, hash);
				}
				```
			2. 定位Segment和HashEntry的不同： 定位Segment使用的是元素的hashcode通过再散列后得到值的高位；定位HashEntry直接使用的是再散列后的值。

				```
				//定位Segment的算法
				(hash >>> segmentShift) & segmentMask;
				
				//定位HashEntry的算法
				int index = hash & (tab.length-1);
				```
		3. size操作<br>
			做法是累加每个Segment里面的全局变量count，它是volatile类型，用来统计Segment中HashEntry的个数，但是不能直接进行累加，因为累加的时候count可能会发生变化。所以ConcurrentHashMap的做法是**先尝试2次通过不锁住Segment的方式来统计各个Segment大小，如果统计过程中，容器的count发生了变化，则采用再加锁的方式来统计所有Segment的大小。**<br>
			那是如何判断在统计的时候容器是否发生了变化呢？ 使用modCount变量，在put、remove、clean方法里操作元素前都会将该变量进行加一，在统计size前后比较modCount是否发生变化，从而得知容器的大小是否发生了变化。
		4. remove操作<br>	
			加锁
	5. ConcurrentHashMap实现高并发的总结
		1. 读操作的高效率
			在实际的应用中，散列表一般的应用场景是：除了少数插入操作和删除操作外，绝大多数都是读取操作，而且读操作在大多数时候都是成功的。正是基于这个前提，ConcurrentHashMap针对读操作做了大量的优化。**通过HashEntry对象的不变性和用volatile型变量协调线程间的内存可见性，使得大多数时候，读操作不需要加锁就可以正确获得值。**
		2. ConcurrentHashMap的高并发性主要来自于三个方面
			1. 用分离锁实现多个线程间的更深层次的共享访问。
			2. 用HashEntery对象的不变性来降低执行读操作的线程在遍历链表期间对加锁的需求。
			3. 通过对同一个volatile变量的写/读访问，协调不同线程间读/写操作的内存可见性。
	6. 参考<br>
		https://juejin.cn/post/6844904057128091655#comment		

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
	2. PriorityQueue 按优先级组织的队列，元素的出队次序由元素的自然排序或者由Comparator比较器指定。
	3. BlockingQueue:阻塞队列。在进行获取元素时，它会等待队列变为非空；当在添加一个元素时，它会等待队列中的可用空间。<br>BlockingQueue接口是Java集合框架的一部分，主要用于实现生产者-消费者模式。我们不需要担心等待生产者有可用的空间，或消费者有可用的对象，因为它都在BlockingQueue的实现类中被处理了。

13. 对一组数据排序<br>
	1. 对于数组:我们可以使用Arrays.sort()方法。假如对象是数组，要求实现接口 Comparable , 否则会抛出强制转型异常
	2. 对于对象列表:对象实现Comparable接口，然后使用Collections.sort()方法
	3. 性能比较:Collections内部使用数组排序方法，所有它们两者都有相同的性能，只是Collections需要花时间将列表转换为数组。底层实现都是归并排序

----
<span id = "java_memory_pos"></span>
#### Java内存区域 [(TOP)](#home)

如果没有特殊说明，都是针对的是 HotSpot 虚拟机。

## 写在前面 (常见面试题)

### 基本问题

- **介绍下 Java 内存区域（运行时数据区）**
- **Java 对象的创建过程（五步，建议能默写出来并且要知道每一步虚拟机做了什么）**
- **对象的访问定位的两种方式（句柄和直接指针两种方式）**

### 拓展问题

- **String 类和常量池**
- **8 种基本类型的包装类和常量池**

## 一 概述

对于 Java 程序员来说，在虚拟机自动内存管理机制下，不再需要像 C/C++程序开发程序员这样为每一个 new 操作去写对应的 delete/free 操作，不容易出现内存泄漏和内存溢出问题。正是因为 Java 程序员把内存控制权利交给 Java 虚拟机，一旦出现内存泄漏和溢出方面的问题，如果不了解虚拟机是怎样使用内存的，那么排查错误将会是一个非常艰巨的任务。

## 二 运行时数据区域

Java 虚拟机在执行 Java 程序的过程中会把它管理的内存划分成若干个不同的数据区域。JDK 1.8 和之前的版本略有不同，下面会介绍到。

**JDK 1.8 之前：**

![](./res/java/memory/java内存区域/JVM运行时数据区域.png)

**JDK 1.8 ：**

![](./res/java/memory/java内存区域/Java运行时数据区域JDK1.8.png)

**线程私有的：**

- 程序计数器
- 虚拟机栈
- 本地方法栈

**线程共享的：**

- 堆
- 方法区
- 直接内存 (非运行时数据区的一部分)

### 2.1 程序计数器

程序计数器是一块较小的内存空间，可以看作是当前线程所执行的字节码的行号指示器。**字节码解释器工作时通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等功能都需要依赖这个计数器来完成。**

另外，**为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。**

**从上面的介绍中我们知道程序计数器主要有两个作用：**

1. 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。
2. 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

**注意：程序计数器是唯一一个不会出现 `OutOfMemoryError` 的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。**

### 2.2 Java 虚拟机栈

**与程序计数器一样，Java 虚拟机栈也是线程私有的，它的生命周期和线程相同，描述的是 Java 方法执行的内存模型，每次方法调用的数据都是通过栈传递的。**

**Java 内存可以粗糙的区分为堆内存（Heap）和栈内存 (Stack)，其中栈就是现在说的虚拟机栈，或者说是虚拟机栈中局部变量表部分。** （实际上，Java 虚拟机栈是由一个个栈帧组成，而每个栈帧中都拥有：局部变量表、操作数栈、动态链接、方法出口信息。）

**局部变量表主要存放了编译期可知的各种数据类型**（boolean、byte、char、short、int、float、long、double）、**对象引用**（reference 类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）。

**Java 虚拟机栈会出现两种错误：`StackOverFlowError` 和 `OutOfMemoryError`。**

- **`StackOverFlowError`：** 若 Java 虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 StackOverFlowError 错误。
- **`OutOfMemoryError`：** Java 虚拟机栈的内存大小可以动态扩展， 如果虚拟机在动态扩展栈时无法申请到足够的内存空间，则抛出`OutOfMemoryError`异常。

![](./res/java/memory/java内存区域/《深入理解虚拟机》第三版的第2章-虚拟机栈.png)

Java 虚拟机栈也是线程私有的，每个线程都有各自的 Java 虚拟机栈，而且随着线程的创建而创建，随着线程的死亡而死亡。

**扩展：那么方法/函数如何调用？**

Java 栈可以类比数据结构中栈，Java 栈中保存的主要内容是栈帧，每一次函数调用都会有一个对应的栈帧被压入 Java 栈，每一个函数调用结束后，都会有一个栈帧被弹出。

Java 方法有两种返回方式：

1. return 语句。
2. 抛出异常。

不管哪种返回方式都会导致栈帧被弹出。

### 2.3 本地方法栈

和虚拟机栈所发挥的作用非常相似，区别是： **虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。** 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。

本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态链接、出口信息。

方法执行完毕后相应的栈帧也会出栈并释放内存空间，也会出现 `StackOverFlowError` 和 `OutOfMemoryError` 两种错误。

### 2.4 堆

Java 虚拟机所管理的内存中最大的一块，Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。**此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。**

Java 世界中“几乎”所有的对象都在堆中分配，但是，随着 JIT 编译器的发展与逃逸分析技术逐渐成熟，栈上分配、标量替换优化技术将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么“绝对”了。从 JDK 1.7 开始已经默认开启逃逸分析，如果某些方法中的对象引用没有被返回或者未被外面使用（也就是未逃逸出去），那么对象可以直接在栈上分配内存。

Java 堆是垃圾收集器管理的主要区域，因此也被称作**GC 堆（Garbage Collected Heap）**。从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以 Java 堆还可以细分为：新生代和老年代；再细致一点有：Eden 空间、From Survivor、To Survivor 空间等。**进一步划分的目的是更好地回收内存，或者更快地分配内存。**

在 JDK 7 版本及 JDK 7 版本之前，堆内存被通常分为下面三部分：

1. 新生代内存(Young Generation)
2. 老生代(Old Generation)
3. 永生代(Permanent Generation)

![JVM堆内存结构-JDK7](./res/java/memory/java内存区域/JVM堆内存结构-JDK7.png)

JDK 8 版本之后方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是直接内存。

![JVM堆内存结构-JDK8](./res/java/memory/java内存区域/JVM堆内存结构-jdk8.png)

**上图所示的 Eden 区、两个 Survivor 区都属于新生代（为了区分，这两个 Survivor 区域按照顺序被命名为 from 和 to），中间一层属于老年代。**

大部分情况，对象都会首先在 Eden 区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入 s0 或者 s1，并且对象的年龄还会加 1(Eden 区->Survivor 区后对象的初始年龄变为 1)，当它的年龄增加到一定程度（默认为 15 岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。

> **🐛 修正（参见：[issue552](https://github.com/Snailclimb/JavaGuide/issues/552)）** ：“Hotspot 遍历所有对象时，按照年龄从小到大对其所占用的大小进行累积，当累积的某个年龄大小超过了 survivor 区的一半时，取这个年龄和 MaxTenuringThreshold 中更小的一个值，作为新的晋升年龄阈值”。
>
> **动态年龄计算的代码如下**
>
> ```c++
> uint ageTable::compute_tenuring_threshold(size_t survivor_capacity) {
> 	//survivor_capacity是survivor空间的大小
> size_t desired_survivor_size = (size_t)((((double) survivor_capacity)*TargetSurvivorRatio)/100);
> size_t total = 0;
> uint age = 1;
> while (age < table_size) {
> total += sizes[age];//sizes数组是每个年龄段对象大小
> if (total > desired_survivor_size) break;
> age++;
> }
> uint result = age < MaxTenuringThreshold ? age : MaxTenuringThreshold;
> 	...
> }
> ```

堆这里最容易出现的就是 OutOfMemoryError 错误，并且出现这种错误之后的表现形式还会有几种，比如：

1. **`java.lang.OutOfMemoryError: GC Overhead Limit Exceeded`** ： 当 JVM 花太多时间执行垃圾回收并且只能回收很少的堆空间时，就会发生此错误。
2. **`java.lang.OutOfMemoryError: Java heap space`** :假如在创建新的对象时, 堆内存中的空间不足以存放新创建的对象, 就会引发此错误。(和配置的最大堆内存有关，且受制于物理内存大小。最大堆内存可通过`-Xmx`参数配置，若没有特别配置，将会使用默认值，详见：[Default Java 8 max heap size](https://stackoverflow.com/questions/28272923/default-xmxsize-in-java-8-max-heap-size))
3. ......

### 2.5 方法区

方法区与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。虽然 **Java 虚拟机规范把方法区描述为堆的一个逻辑部分**，但是它却有一个别名叫做 **Non-Heap（非堆）**，目的应该是与 Java 堆区分开来。

方法区也被称为永久代。很多人都会分不清方法区和永久代的关系，为此我也查阅了文献。

#### 2.5.1 方法区和永久代的关系

> 《Java 虚拟机规范》只是规定了有方法区这么个概念和它的作用，并没有规定如何去实现它。那么，在不同的 JVM 上方法区的实现肯定是不同的了。 **方法区和永久代的关系很像 Java 中接口和类的关系，类实现了接口，而永久代就是 HotSpot 虚拟机对虚拟机规范中方法区的一种实现方式。** 也就是说，永久代是 HotSpot 的概念，方法区是 Java 虚拟机规范中的定义，是一种规范，而永久代是一种实现，一个是标准一个是实现，其他的虚拟机实现并没有永久代这一说法。

#### 2.5.2 常用参数

JDK 1.8 之前永久代还没被彻底移除的时候通常通过下面这些参数来调节方法区大小

```java
-XX:PermSize=N //方法区 (永久代) 初始大小
-XX:MaxPermSize=N //方法区 (永久代) 最大大小,超过这个值将会抛出 OutOfMemoryError 异常:java.lang.OutOfMemoryError: PermGen
```

相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。

JDK 1.8 的时候，方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是直接内存。

下面是一些常用参数：

```java
-XX:MetaspaceSize=N //设置 Metaspace 的初始（和最小大小）
-XX:MaxMetaspaceSize=N //设置 Metaspace 的最大大小
```

与永久代很大的不同就是，如果不指定大小的话，随着更多类的创建，虚拟机会耗尽所有可用的系统内存。

#### 2.5.3 为什么要将永久代 (PermGen) 替换为元空间 (MetaSpace) 呢?

下图来自《深入理解 Java 虚拟机》第 3 版 2.2.5

![](https://img-blog.csdnimg.cn/20210425134508117.png)

1. 整个永久代有一个 JVM 本身设置的固定大小上限，无法进行调整，而元空间使用的是直接内存，受本机可用内存的限制，虽然元空间仍旧可能溢出，但是比原来出现的几率会更小。
   
   > 当元空间溢出时会得到如下错误： `java.lang.OutOfMemoryError: MetaSpace`

你可以使用 `-XX：MaxMetaspaceSize` 标志设置最大元空间大小，默认值为 unlimited，这意味着它只受系统内存的限制。`-XX：MetaspaceSize` 调整标志定义元空间的初始大小如果未指定此标志，则 Metaspace 将根据运行时的应用程序需求动态地重新调整大小。

2. 元空间里面存放的是类的元数据，这样加载多少类的元数据就不由 `MaxPermSize` 控制了, 而由系统的实际可用空间来控制，这样能加载的类就更多了。

3. 在 JDK8，合并 HotSpot 和 JRockit 的代码时, JRockit 从来没有一个叫永久代的东西, 合并之后就没有必要额外的设置这么一个永久代的地方了。

### 2.6 运行时常量池

运行时常量池是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有常量池表（用于存放编译期生成的各种字面量和符号引用）

既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 OutOfMemoryError 错误。

~~**JDK1.7 及之后版本的 JVM 已经将运行时常量池从方法区中移了出来，在 Java 堆（Heap）中开辟了一块区域存放运行时常量池。**~~

> **🐛 修正（参见：[issue747](https://github.com/Snailclimb/JavaGuide/issues/747)，[reference](https://blog.csdn.net/q5706503/article/details/84640762)）** ：
>
> 1. **JDK1.7 之前运行时常量池逻辑包含字符串常量池存放在方法区, 此时 hotspot 虚拟机对方法区的实现为永久代**
> 2. **JDK1.7 字符串常量池被从方法区拿到了堆中, 这里没有提到运行时常量池,也就是说字符串常量池被单独拿到堆,运行时常量池剩下的东西还在方法区, 也就是 hotspot 中的永久代** 。
> 3. **JDK1.8 hotspot 移除了永久代用元空间(Metaspace)取而代之, 这时候字符串常量池还在堆, 运行时常量池还在方法区, 只不过方法区的实现从永久代变成了元空间(Metaspace)**

相关问题：JVM 常量池中存储的是对象还是引用呢？： https://www.zhihu.com/question/57109429/answer/151717241 by RednaxelaFX

### 2.7 直接内存

**直接内存并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致 OutOfMemoryError 错误出现。**

JDK1.4 中新加入的 **NIO(New Input/Output) 类**，引入了一种基于**通道（Channel）**与**缓存区（Buffer）**的 I/O 方式，它可以直接使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为**避免了在 Java 堆和 Native 堆之间来回复制数据**。

本机直接内存的分配不会受到 Java 堆的限制，但是，既然是内存就会受到本机总内存大小以及处理器寻址空间的限制。

## 三 HotSpot 虚拟机对象探秘

通过上面的介绍我们大概知道了虚拟机的内存情况，下面我们来详细的了解一下 HotSpot 虚拟机在 Java 堆中对象分配、布局和访问的全过程。

### 3.1 对象的创建

下图便是 Java 对象的创建过程，我建议最好是能默写出来，并且要掌握每一步在做什么。
![Java创建对象的过程](./res/java/memory/java内存区域/Java创建对象的过程.png)

#### Step1:类加载检查

虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执行相应的类加载过程。

#### Step2:分配内存

在**类加载检查**通过后，接下来虚拟机将为新生对象**分配内存**。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从 Java 堆中划分出来。**分配方式**有 **“指针碰撞”** 和 **“空闲列表”** 两种，**选择哪种分配方式由 Java 堆是否规整决定，而 Java 堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定**。

**内存分配的两种方式：（补充内容，需要掌握）**

选择以上两种方式中的哪一种，取决于 Java 堆内存是否规整。而 Java 堆内存是否规整，取决于 GC 收集器的算法是"标记-清除"，还是"标记-整理"（也称作"标记-压缩"），值得注意的是，复制算法内存也是规整的

![内存分配的两种方式](./res/java/memory/java内存区域/内存分配的两种方式.png)

**内存分配并发问题（补充内容，需要掌握）**

在创建对象的时候有一个很重要的问题，就是线程安全，因为在实际开发过程中，创建对象是很频繁的事情，作为虚拟机来说，必须要保证线程是安全的，通常来讲，虚拟机采用两种方式来保证线程安全：

- **CAS+失败重试：** CAS 是乐观锁的一种实现方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。**虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性。**
- **TLAB：** 为每一个线程预先在 Eden 区分配一块儿内存，JVM 在给线程中的对象分配内存时，首先在 TLAB 分配，当对象大于 TLAB 中的剩余内存或 TLAB 的内存已用尽时，再采用上述的 CAS 进行内存分配

#### Step3:初始化零值

内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。

#### Step4:设置对象头

初始化零值完成之后，**虚拟机要对对象进行必要的设置**，例如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。 **这些信息存放在对象头中。** 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。

#### Step5:执行 init 方法

在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，`<init>` 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 `<init>` 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。

### 3.2 对象的内存布局

在 Hotspot 虚拟机中，对象在内存中的布局可以分为 3 块区域：**对象头**、**实例数据**和**对齐填充**。

**Hotspot 虚拟机的对象头包括两部分信息**，**第一部分用于存储对象自身的运行时数据**（哈希码、GC 分代年龄、锁状态标志等等），**另一部分是类型指针**，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是那个类的实例。

**实例数据部分是对象真正存储的有效信息**，也是在程序中所定义的各种类型的字段内容。

**对齐填充部分不是必然存在的，也没有什么特别的含义，仅仅起占位作用。** 因为 Hotspot 虚拟机的自动内存管理系统要求对象起始地址必须是 8 字节的整数倍，换句话说就是对象的大小必须是 8 字节的整数倍。而对象头部分正好是 8 字节的倍数（1 倍或 2 倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。

### 3.3 对象的访问定位

建立对象就是为了使用对象，我们的 Java 程序通过栈上的 reference 数据来操作堆上的具体对象。对象的访问方式由虚拟机实现而定，目前主流的访问方式有**① 使用句柄**和**② 直接指针**两种：

1. **句柄：** 如果使用句柄的话，那么 Java 堆中将会划分出一块内存来作为句柄池，reference 中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息；

   ![对象的访问定位-使用句柄](./res/java/memory/java内存区域/对象的访问定位-使用句柄.png)

2. **直接指针：** 如果使用直接指针访问，那么 Java 堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，而 reference 中存储的直接就是对象的地址。

![对象的访问定位-直接指针](./res/java/memory/java内存区域/对象的访问定位-直接指针.png)

**这两种对象访问方式各有优势。使用句柄来访问的最大好处是 reference 中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而 reference 本身不需要修改。使用直接指针访问方式最大的好处就是速度快，它节省了一次指针定位的时间开销。**

## 四 重点补充内容

### 4.1 String 类和常量池

**String 对象的两种创建方式：**

```java
String str1 = "abcd";//先检查字符串常量池中有没有"abcd"，如果字符串常量池中没有，则创建一个，然后 str1 指向字符串常量池中的对象，如果有，则直接将 str1 指向"abcd""；
String str2 = new String("abcd");//堆中创建一个新的对象
String str3 = new String("abcd");//堆中创建一个新的对象
System.out.println(str1==str2);//false
System.out.println(str2==str3);//false
```

这两种不同的创建方法是有差别的。

- 第一种方式是在常量池中拿对象；
- 第二种方式是直接在堆内存空间创建一个新的对象。

记住一点：**只要使用 new 方法，便需要创建新的对象。**

再给大家一个图应该更容易理解，图片来源：<https://www.journaldev.com/797/what-is-java-string-pool>：

![String-Pool-Java](./res/java/memory/java内存区域/2019-3String-Pool-Java1-450x249.png)

**String 类型的常量池比较特殊。它的主要使用方法有两种：**

1. 直接使用双引号声明出来的 String 对象会直接存储在常量池中。
2. 如果不是用双引号声明的 String 对象，可以使用 String 提供的 `intern()` 方法。`String.intern()` 是一个 Native 方法，它的作用是：如果运行时常量池中已经包含一个等于此 String 对象内容的字符串，则返回常量池中该字符串的引用；如果没有，JDK1.7 之前（不包含 1.7）的处理方式是在常量池中创建与此 String 内容相同的字符串，并返回常量池中创建的字符串的引用，JDK1.7 以及之后的处理方式是在常量池中记录此字符串的引用，并返回该引用。

JDK8 :

```java
String s1 = "计算机";
String s2 = s1.intern();
String s3 = "计算机";
System.out.println(s2);//计算机
System.out.println(s1 == s2);//true
System.out.println(s3 == s2);//true，因为两个都是常量池中的 String 对象
```

**字符串拼接:**

```java
String str1 = "str";
String str2 = "ing";

String str3 = "str" + "ing";//常量池中的对象
String str4 = str1 + str2; //在堆上创建的新的对象
String str5 = "string";//常量池中的对象
System.out.println(str3 == str4);//false
System.out.println(str3 == str5);//true
System.out.println(str4 == str5);//false
```

![字符串拼接](./res/java/memory/java内存区域/字符串拼接-常量池.png)

尽量避免多个字符串拼接，因为这样会重新创建对象。如果需要改变字符串的话，可以使用 StringBuilder 或者 StringBuffer。

### 4.2 String s1 = new String("abc");这句话创建了几个字符串对象？

**将创建 1 或 2 个字符串。如果池中已存在字符串常量“abc”，则只会在堆空间创建一个字符串常量“abc”。如果池中没有字符串常量“abc”，那么它将首先在池中创建，然后在堆空间中创建，因此将创建总共 2 个字符串对象。**

**验证：**

```java
String s1 = new String("abc");// 堆内存的地址值
String s2 = "abc";
System.out.println(s1 == s2);// 输出 false,因为一个是堆内存，一个是常量池的内存，故两者是不同的。
System.out.println(s1.equals(s2));// 输出 true
```

**结果：**

```
false
true
```

### 4.3 8 种基本类型的包装类和常量池

**Java 基本类型的包装类的大部分都实现了常量池技术，即 Byte,Short,Integer,Long,Character,Boolean；前面 4 种包装类默认创建了数值[-128，127] 的相应类型的缓存数据，Character 创建了数值在[0,127]范围的缓存数据，Boolean 直接返回 True Or False。如果超出对应范围仍然会去创建新的对象。** 为啥把缓存设置为[-128，127]区间？（[参见 issue/461](https://github.com/Snailclimb/JavaGuide/issues/461)）性能和资源之间的权衡。

```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```

```java
private static class CharacterCache {
    private CharacterCache(){}

    static final Character cache[] = new Character[127 + 1];
    static {
        for (int i = 0; i < cache.length; i++)
            cache[i] = new Character((char)i);
    }
}
```

两种浮点数类型的包装类 Float,Double 并没有实现常量池技术。

```java
Integer i1 = 33;
Integer i2 = 33;
System.out.println(i1 == i2);// 输出 true
Integer i11 = 333;
Integer i22 = 333;
System.out.println(i11 == i22);// 输出 false
Double i3 = 1.2;
Double i4 = 1.2;
System.out.println(i3 == i4);// 输出 false
```

**Integer 缓存源代码：**

```java
/**
*此方法将始终缓存-128 到 127（包括端点）范围内的值，并可以缓存此范围之外的其他值。
*/
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

**应用场景：**

1. Integer i1=40；Java 在编译的时候会直接将代码封装成 Integer i1=Integer.valueOf(40);，从而使用常量池中的对象。
2. Integer i1 = new Integer(40);这种情况下会创建新的对象。

```java
Integer i1 = 40;
Integer i2 = new Integer(40);
System.out.println(i1==i2);//输出 false
```

**Integer 比较更丰富的一个例子:**

```java
Integer i1 = 40;
Integer i2 = 40;
Integer i3 = 0;
Integer i4 = new Integer(40);
Integer i5 = new Integer(40);
Integer i6 = new Integer(0);

System.out.println("i1=i2   " + (i1 == i2));
System.out.println("i1=i2+i3   " + (i1 == i2 + i3));
System.out.println("i1=i4   " + (i1 == i4));
System.out.println("i4=i5   " + (i4 == i5));
System.out.println("i4=i5+i6   " + (i4 == i5 + i6));
System.out.println("40=i5+i6   " + (40 == i5 + i6));
```

结果：

```
i1=i2   	true
i1=i2+i3   	true
i1=i4   	false
i4=i5  		false
i4=i5+i6   	true
40=i5+i6  	true
```

解释：

语句 i4 == i5 + i6，因为+这个操作符不适用于 Integer 对象，首先 i5 和 i6 进行自动拆箱操作，进行数值相加，即 i4 == 40。然后 Integer 对象无法与数值进行直接比较，所以 i4 自动拆箱转为 int 值 40，最终这条语句转为 40 == 40 进行数值比较。

## 参考

- 《深入理解 Java 虚拟机：JVM 高级特性与最佳实践（第二版》
- 《实战 java 虚拟机》
- <https://docs.oracle.com/javase/specs/index.html>
- <http://www.pointsoftware.ch/en/under-the-hood-runtime-data-areas-javas-memory-model/>
- <https://dzone.com/articles/jvm-permgen-%E2%80%93-where-art-thou>
- <https://stackoverflow.com/questions/9095748/method-area-and-permgen>
- 深入解析 String#intern<https://tech.meituan.com/2014/03/06/in-depth-understanding-string-intern.html>

----

<span id = "java_memory"></span>
#### Java内存模型 [(TOP)](#home)
1. Java内存模型
	1. 概述<br>
		Java内存模型(即Java Memory Model，简称JMM)本身是一种抽象的概念，并不真实存在，它描述的是一组规则或规范。<br>
		JVM定义了一组规则，通过这组规则来决定一个线程对主内存的共享变量的写入何时对另一个线程可见，这组规则也称为Java内存模型（即JMM），JMM是围绕着程序执行的原子性、有序性、可见性展开的
		
	2. 主内存与工作内存<br>
		由于JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个工作内存(有些地方称为栈空间)，用于存储线程私有的数据，而Java内存模型中规定所有变量都存储在主内存，主内存是共享内存区域，所有线程都可以访问，但线程对变量的操作(读取赋值等)必须在工作内存中进行，首先要将变量从主内存拷贝的自己的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存，不能直接操作主内存中的变量，工作内存中存储着主内存中的变量副本拷贝，前面说过，工作内存是每个线程的私有数据区域，因此不同的线程间无法访问对方的工作内存，线程间的通信(传值)必须通过主内存来完成
		
		1. 主内存<br>
			主要存储的是Java实例对象，所有线程创建的实例对象都存放在主内存中，不管该实例对象是成员变量还是方法中的本地变量(也称局部变量)，当然也包括了共享的类信息、常量、静态变量。由于是共享数据区域，多条线程对同一个变量进行访问可能会发现线程安全问题。

		2. 工作内存<br>
			主要存储当前方法的所有本地变量信息(工作内存中存储着主内存中的变量副本拷贝)，每个线程只能访问自己的工作内存，即线程中的本地变量对其它线程是不可见的，就算是两个线程执行的是同一段代码，它们也会各自在自己的工作内存中创建属于当前线程的本地变量，当然也包括了字节码行号指示器、相关Native方法的信息。注意由于工作内存是每个线程的私有数据，线程间无法相互访问工作内存，因此存储在工作内存的数据不存在线程安全问题。

	3. Java内存模型与Java内存区域的区别<br>
		两者划分是不同的概念层次，更恰当说JMM描述的是一组规则，通过这组规则控制程序中各个变量在共享数据区域和私有数据区域的访问方式，JMM是围绕原子性，有序性、可见性展开的(稍后会分析)。JMM与Java内存区域唯一相似点，都存在共享数据区域和私有数据区域，在JMM中主内存属于共享数据区域，从某个程度上讲应该包括了堆和方法区，而工作内存数据线程私有数据区域，从某个程度上讲则应该包括程序计数器、虚拟机栈以及本地方法栈。或许在某些地方，我们可能会看见主内存被描述为堆内存，工作内存被称为线程栈，实际上他们表达的都是同一个含义。
	
	4. 结合JMM看线程安全问题<br>
		由于JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个工作内存(有些地方称为栈空间)，用于存储线程私有的数据，线程与主内存中的变量操作必须通过工作内存间接完成，主要过程是将变量从主内存拷贝的每个线程各自的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存，如果存在两个线程同时对一个主内存中的实例对象的变量进行操作就有可能诱发线程安全问题。
		1. 例子<br>
			主内存中存在一个共享变量x，现在有A和B两条线程分别对该变量x=1进行操作，A/B线程各自的工作内存中存在共享变量副本x。假设现在A线程想要修改x的值为2，而B线程却想要读取x的值，那么B线程读取到的值是A线程更新后的值2还是更新前的值1呢？答案是，不确定，即B线程有可能读取到A线程更新前的值1，也有可能读取到A线程更新后的值2，这是因为工作内存是每个线程私有的数据区域，而线程A变量x时，首先是将变量从主内存拷贝到A线程的工作内存中，然后对变量进行操作，操作完成后再将变量x写回主内，而对于B线程的也是类似的，这样就有可能造成主内存与工作内存间数据存在一致性问题，假如A线程修改完后正在将数据写回主内存，而B线程此时正在读取主内存，即将x=1拷贝到自己的工作内存中，这样B线程读取到的值就是x=1，但如果A线程已将x=2写回主内存后，B线程才开始读取的话，那么此时B线程读取到的就是x=2，但到底是哪种情况先发生呢？这是不确定的，这也就是所谓的线程安全问题。 

3. 程序并发时需要考虑的三个特性
	1. 原子性<br>
		1. 原子性的定义<br>
			即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。
		2. 	在Java中，对基本数据类型的变量的读取和赋值操作是原子性操作，即这些操作是不可被中断的，要么执行，要么不执行。如果要实现更大范围操作的原子性，可以通过synchronized和Lock来实现。
		3. 在32位平台下，对64位数据的读取和赋值是需要通过两个操作来完成的，不能保证其原子性。但是在最新的JDK中，大部分商用JVM已经保证对64位数据的读取和赋值也是原子性操作了。
		4. 举例<br>
		
			```
			x = 10;        //语句1
			y = x;         //语句2
			x++;           //语句3
			x = x + 1;     //语句4
			```
			
			以上其实只有语句1是原子性操作，其他三个语句都不是原子性操作。
	
	2. 可见性<br>
		可见性指的是当一个线程修改了某个共享变量的值，其他线程是否能够马上得知这个修改的值。对于串行程序来说，可见性是不存在的，因为我们在任何一个操作中修改了某个变量的值，后续的操作中都能读取这个变量值，并且是修改过的新值。但在多线程环境中可就不一定了，前面我们分析过，由于线程对共享变量的操作都是线程拷贝到各自的工作内存进行操作后才写回到主内存中的，这就可能存在一个线程A修改了共享变量x的值，还未写回主内存时，另外一个线程B又对主内存中同一个共享变量x进行操作，但此时A线程工作内存中共享变量x对线程B来说并不可见，这种工作内存与主内存同步延迟现象就造成了可见性问题，另外指令重排以及编译器优化也可能导致可见性问题，通过前面的分析，我们知道无论是编译器优化还是处理器优化的重排现象，在多线程环境下，确实会导致程序轮序执行的问题，从而也就导致可见性问题。
	
	3. 有序性<br>
		有序性是指对于单线程的执行代码，我们总是认为代码的执行是按顺序依次执行的，这样的理解并没有毛病，毕竟对于单线程而言确实如此，但对于多线程环境，则可能出现乱序现象，因为程序编译成机器码指令后可能会出现指令重排现象，重排后的指令与原指令的顺序未必一致，要明白的是，在Java程序中，倘若在本线程内，所有操作都视为有序行为，如果是多线程环境下，一个线程中观察另外一个线程，所有操作都是无序的，前半句指的是单线程内保证串行语义执行的一致性，后半句则指指令重排现象和工作内存与主内存同步延迟现象。

4. Java内存模型提供的解决方案<br>
	在理解了原子性，可见性以及有序性问题后，看看JMM是如何保证的，在Java内存模型中都提供一套解决方案供Java工程师在开发过程使用。<br>
	如原子性问题，除了JVM自身提供的对基本数据类型读写操作的原子性外，对于方法级别或者代码块级别的原子性操作，可以使用synchronized关键字或者重入锁(ReentrantLock)保证程序执行的原子性。<br>
	而工作内存与主内存同步延迟现象导致的可见性问题，可以使用synchronized关键字或者volatile关键字解决，它们都可以使一个线程修改后的变量立即对其他线程可见。<br>
	对于指令重排导致的可见性问题和有序性问题，则可以利用volatile关键字解决，因为volatile的另外一个作用就是禁止重排序优化，关于volatile稍后会进一步分析。<br>
	除了靠sychronized和volatile关键字来保证原子性、可见性以及有序性外，JMM内部还定义一套happens-before 原则来保证多线程环境下两个操作间的原子性、可见性以及有序性。

5. 指令的重排序
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
	
6. 内存屏障(Memory Barrier)
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
	
7. happen-before原则<br> 	Java内存模型规定，对于多个线程共享的变量，存储在主内存当中，每个线程都有自己独立的工作内存（比如CPU的寄存器），线程只能访问自己的工作内存，不可以访问其它线程的工作内存。<br>
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
	
	
8. Volatile<br>
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

10. 参考
	1. https://blog.csdn.net/javazejian/article/details/72772461
	2. https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483910&idx=1&sn=246f39051a85fc312577499691fba89f&chksm=fd985467caefdd71f9a7c275952be34484b14f9e092723c19bd4ef557c324169ed084f868bdb#rd

----

<span id = "java_class"></span>
#### Java类 [(TOP)](#home)
1. 类加载器
	1. 类与类加载器<br>
		**对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立其在Java虚拟机中的唯一性。如果两个类来源于同一个Class文件，只要加载它们的类加载器不同，那么这两个类就必定不相等。**

	2. 类加载器介绍<br>
		从Java虚拟机的角度分为两种不同的类加载器：启动类加载器（Bootstrap ClassLoader） 和其他类加载器。其中启动类加载器，使用C++语言实现，是虚拟机自身的一部分；其余的类加载器都由Java语言实现，独立于虚拟机之外，并且全都继承自java.lang.ClassLoader类。<br>
		从Java开发人员的角度来看，绝大部分Java程序都会使用到以下3种系统提供的类加载器。<br>
		1. 启动类加载器（Bootstrap ClassLoader）：<br>
			这个类加载器负责将存放在\lib目录中的，或者被-Xbootclasspath参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如rt.jar，名字不符合的类库即使放在lib目录中也不会被加载）类库加载到虚拟机内存中。

		2. 扩展类加载器（Extension ClassLoader）：<br>
			这个加载器由sun.misc.Launcher$ExtClassLoader实现，它负责加载\lib\ext目录中的，或者被java.ext.dirs系统变量所指定的路径中的所有类库，开发者可以直接使用扩展类加载器。

		3. 应用程序类加载器（Application ClassLoader）：<br>
			这个类加载器由sun.misc.Launcher$AppClassLoader实现。由于这个类加载器是ClassLoader中的getSystemClassLoader()方法的返回值，所以一般也称它为系统类加载器。它负责加载用户类路径（ClassPath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。<br>
			我们的应用程序都是由这3种类加载器互相配合进行加载的，如果有必要，还可以加入自己定义的类加载器。
		
		4. 自定义类加载器，父类加载器肯定为AppClassLoader。

	3. 双亲委派模型<br>
		1. 定义<br>
			双亲委派模型（Pattern Delegation Model）,要求除了顶层的启动类加载器外，其余的类加载器都应该有自己的父类加载器。这里父子关系通常是子类通过组合关系而不是继承关系来复用父加载器的代码。<br>
			![](https://github.com/yinfork/Android-Interview/blob/master/res/java/jvm/classloader.png?raw=true) <br>

		2. 双亲委派模型的工作过程<br>
			如果一个类加载器收到了类加载的请求，先把这个请求委派给父类加载器去完成（所以所有的加载请求最终都应该传送到顶层的启动类加载器中），只有当父加载器反馈自己无法完成加载请求时，子加载器才会尝试自己去加载。

		3. 优点<br>
			采用双亲委派模式的是好处是Java类随着它的类加载器一起具备了一种带有优先级的层次关系，通过这种层级关可以避免类的重复加载，当父亲已经加载了该类时，就没有必要子ClassLoader再加载一次。其次是考虑到安全因素，java核心api中定义类型不会被随意替换，假设通过网络传递一个名为java.lang.Integer的类，通过双亲委托模式传递到启动类加载器，而启动类加载器在核心Java API发现这个名字的类，发现该类已被加载，并不会重新加载网络传递的过来的java.lang.Integer，而直接返回已加载过的Integer.class，这样便可以防止核心API库被随意篡改。
			
	4. 打破双亲委派模型<br>
		双亲委派模型是Java设计者们推荐给开发者们的一种类加载器实现方式，并不是一个强制性 的约束模型。在java的世界中大部分的类加载器都遵循这个模型，但也有例外。

		1. 第一次破坏<br>
			是因为类加载器和抽象类java.lang.ClassLoader在JDK1.0就存在的，而双亲委派模型在JDK1.2之后才被引入，为了兼容已经存在的用户自定义类加载器，引入双亲委派模型时做了一定的妥协：在java.lang.ClassLoader中引入了一个findClass()方法，在此之前，用户去继承java.lang.Classloader的唯一目的就是重写loadClass()方法。JDK1.2之后不提倡用户去覆盖loadClass()方法，而是把自己的类加载逻辑写到findClass()方法中，如果loadClass()方法中如果父类加载失败，则会调用自己的findClass()方法来完成加载，这样就可以保证新写出来的类加载器是符合双亲委派模型规则的。

		2. 第二次破坏<br>
			是因为模型自身的缺陷，现实中存在这样的场景：基础的类加载器需要求调用用户的代码，而基础的类加载器可能不认识用户的代码。为此，Java设计团队引入的设计时“线程上下文类加载器（Thread Context ClassLoader）”。这样可以通过父类加载器请求子类加载器去完成类加载动作。已经违背了双亲委派模型的一般性原则。
			
			线程上下文类加载器（contextClassLoader）是从 JDK 1.2 开始引入的，我们可以通过java.lang.Thread类中的getContextClassLoader()和 setContextClassLoader(ClassLoader cl)方法来获取和设置线程的上下文类加载器。如果没有手动设置上下文类加载器，线程将继承其父线程的上下文类加载器，初始线程的上下文类加载器是系统类加载器（AppClassLoader）,在线程中运行的代码可以通过此类加载器来加载类和资源

		3. 第三次破坏<br>
			是由于用户对程序动态性的追求导致的。这里所说的动态性是指：“代码热替换”、“模块热部署”等等比较热门的词。说白了就是希望应用程序能够像我们的计算机外设一样，接上鼠标、U盘不用重启机器就能立即使用。OSGi是当前业界“事实上”的Java模块化标准，OSGi实现模块化热部署的关键是它自定义的类加载器机制的实现。每一个程序模块（OSGi中称为Bundle）都有一个自己的类加载器，当需要更换一个Bundle时，就把Bundle连同类加载器一起换掉以实现代码的热替换。在OSGi环境下，类加载器不再是双亲委派模型中的树状结构，而是进一步发展为更加复杂的网状结构。

9. 类的加载时机和加载过程<br>
	TODO
	
10. 参考
	1. https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483934&idx=1&sn=f247f9bee4e240f5e7fac25659da3bff&chksm=fd98547fcaefdd6996e1a7046e03f29df9308bdf82ceeffd111112766ffd3187892700f64b40#rd

----

<span id = "java_gc"></span>
#### Java垃圾回收 [(TOP)](#home)
1. 判断对象是否有被引用<br>
	堆中几乎放着所有的对象实例，对堆垃圾回收前的第一步就是要判断那些对象已经死亡（即不能再被任何途径使用的对象）
	1. 引用计数法<br>
		给对象中添加一个引用计数器，每当有一个地方引用它，计数器就加1；当引用失效，计数器就减1；任何时候计数器为0的对象就是不可能再被使用的。
<br><br>
这个方法实现简单，效率高，但是目前主流的虚拟机中并没有选择这个算法来管理内存，其最主要的原因是它很难解决对象之间相互循环引用的问题。
	2. 可达性分析法<br>
	这个算法的基本思想就是通过一系列的称为 “GC Roots” 的对象作为起点，从这些节点开始向下搜索，节点所走过的路径称为引用链，当一个对象到GC Roots没有任何引用链相连的话，则证明此对象是不可用的。
	
2. 四种引用类型
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
	
3. 垃圾回收的细节
	1. 即使在可达性分析法中不可达的对象，也并非是“非死不可”。<br>
	这时候它们暂时处于“缓刑阶段”，要真正宣告一个对象死亡，至少要经历两次标记过程；可达性分析法中不可达的对象被第一次标记并且进行一次筛选，筛选的条件是此对象是否有必要执行finalize方法。当对象没有覆盖finalize方法，或finalize方法已经被虚拟机调用过时，虚拟机将这两种情况视为没有必要执行。被判定为需要执行的对象将会被放在一个队列中进行第二次标记，除非这个对象与引用链上的任何一个对象建立关联，否则就会被真的回收。

	2. 回收方法区<br>
		方法区（或Hotspot虚拟中的永久代）的垃圾收集主要回收两部分内容：废弃常量和无用的类。
<br><br>
		判定一个常量是否是“废弃常量”比较简单，而要判定一个类是否是“无用的类”的条件则相对苛刻许多。类需要同时满足下面3个条件才能算是 “无用的类” ：
		1. 该类所有的实例都已经被回收，也就是Java堆中不存在该类的任何实例。
		2. 加载该类的ClassLoader已经被回收。
		3. 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。
	
4. 垃圾收集算法
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
	
5. 堆内存分配与回收策略
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
	
10. 参考	
	1. https://mp.weixin.qq.com/s?__biz=MzU4NDQ4MzU5OA==&mid=2247483914&idx=1&sn=9aa157d4a1570962c39783cdeec7e539&chksm=fd98546bcaefdd7d9f61cd356e5584e56b64e234c3a403ed93cb6d4dde07a505e3000fd0c427#rd
	

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

8. 各种锁的介绍<br>
TODO

9. 死锁<br>
	TODO
	1. 银行家算法

10. 线程池<br>
	TODO

20. 参考
	1. https://github.com/hadyang/interview/blob/master/java/volatile.md
	2. https://github.com/hadyang/interview/blob/master/java/threadlocal.md
	3. https://github.com/hadyang/interview/blob/master/java/synchronized.md
	4. https://github.com/Snailclimb/JavaGuide/blob/master/Java%E7%9B%B8%E5%85%B3/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E7%B3%BB%E5%88%97.md 
	5. http://ifeve.com/jvm-memory-reordering/
	6. http://www.importnew.com/23535.html
	7. https://www.jianshu.com/p/ec7200a26d8b

----	
	
<span id = "design_pattern"></span>
#### 设计模式 [(TOP)](#home)	
1. 六种方式写单例模式
	1. 懒汉式，线程不安全<br>
		这段代码简单明了，而且使用了懒加载模式，但是却存在致命的问题。当有多个线程并行调用 getInstance() 的时候，就会创建多个实例。也就是说在多线程下不能正常工作。

		```
		public class Singleton {
		    private static Singleton instance;
		    private Singleton (){}
		
		    public static Singleton getInstance() {
		     if (instance == null) {
		         instance = new Singleton();
		     }
		     return instance;
		    }
		}
		```

	2. 懒汉式，线程安全
		为了解决上面懒汉式线程不安全的问题，最简单的方法是将整个 getInstance() 方法设为同步（synchronized）。
		虽然做到了线程安全，并且解决了多实例的问题，但是它并不高效。因为在任何时候只能有一个线程调用 getInstance() 方法。但是同步操作只需要在第一次调用时才被需要，即第一次创建单例实例对象时。这就引出了双重检验锁。

		```
		public static synchronized Singleton getInstance() {
		    if (instance == null) {
		        instance = new Singleton();
		    }
		    return instance;
		}
		```

	3. 双重检验锁
		1. 两个注意点
			1. 两次判空
				这种写法在getSingleton方法中对singleton进行了两次判空，第一次是为了不必要的同步，第二次是在singleton等于null的情况下才创建实例
		
			2. 使用 volatile 的主要原因是：禁止指令重排序优化导致报错。
				1. 一个赋值操作的过程<br>
					instance = new Singleton()这句，这并非是一个原子操作，事实上在 JVM 中这句话大概做了下面 3 件事情。

					1. 给 instance 分配内存
					2. 调用 Singleton 的构造函数来初始化成员变量
					3. 将instance对象指向分配的内存空间（执行完这步 instance 就为非 null 了）
				
				2. 指令重排序的问题<br>
					但是在 JVM 的即时编译器中存在指令重排序的优化。也就是说上面的第二步和第三步的顺序是不能保证的，最终的执行顺序可能是 1-2-3 也可能是 1-3-2。如果是后者，则在 3 执行完毕、2 未执行之前，被线程二抢占了，这时 instance 已经是非 null 了（但却没有初始化），所以线程二会直接返回 instance，然后使用，然后顺理成章地报错。
				3. 使用 volatile 避免指令重排序的问题<br>
					在 volatile 变量的赋值操作后面会有一个内存屏障（生成的汇编代码上），读操作不会被重排序到内存屏障之前。比如上面的例子，取操作必须在执行完 1-2-3 之后或者 1-3-2 之后，不存在执行到 1-3 然后取到值的情况。从「先行发生原则」的角度理解的话，就是对于一个 volatile 变量的写操作都先行发生于后面对这个变量的读操作（这里的“后面”是时间上的先后顺序）。
		
				4. 在 Java 5 以前的版本使用了 volatile 的双检锁还是有问题的<br>
					其原因是 Java 5 以前的 JMM （Java 内存模型）是存在缺陷的，即时将变量声明成 volatile 也不能完全避免重排序，主要是 volatile 变量前后的代码仍然存在重排序问题。这个 volatile 屏蔽重排序的问题在 Java 5 中才得以修复。
				
		2. 例子<br>
	
			```
			public class Singleton {
			    private volatile static Singleton instance; //声明成 volatile
			    private Singleton (){}
			
			    public static Singleton getSingleton() {
			        if (instance == null) {                         
			            synchronized (Singleton.class) {
			                if (instance == null) {       
			                    instance = new Singleton();
			                }
			            }
			        }
			        return instance;
			    }
			   
			}
			```
	4. 饿汉式 static final field

		1. 优点<br>
			这种方法非常简单，因为单例的实例被声明成 static 和 final 变量了，在第一次加载类到内存中时就会初始化，所以创建实例本身是线程安全的。

		2. 缺点<br>
			缺点是它不是一种懒加载模式（lazy initialization），单例会在加载类后一开始就被初始化，即使客户端没有调用 getInstance()方法。<br>
			饿汉式的创建方式在一些场景中将无法使用：譬如 Singleton 实例的创建是依赖参数或者配置文件的，在 getInstance() 之前必须调用某个方法设置参数给它，那样这种单例写法就无法使用了。

		3. 例子<br>
		
			```
			public class Singleton{
			    //类加载时就初始化
			    private static final Singleton instance = new Singleton();
			    
			    private Singleton(){}
			
			    public static Singleton getInstance(){
			        return instance;
			    }
			}
			```

	5. 静态内部类 static nested class

		1. 优点<br>
			《Effective Java》上所推荐的。这种写法仍然使用JVM本身机制保证了线程安全问题；由于 SingletonHolder 是私有的，除了 getInstance() 之外没有办法访问它，因此它是懒汉式的；同时读取实例的时候不会进行同步，没有性能缺陷；也不依赖 JDK 版本。
		
		2. 缺点<br>
			无
		
		3. 例子<br>
			
			```
			public class Singleton {  
			    private static class SingletonHolder {  
			        private static final Singleton INSTANCE = new Singleton();  
			    }  
			    private Singleton (){}  
			    public static final Singleton getInstance() {  
			        return SingletonHolder.INSTANCE; 
			    }  
			}
			```
	6. 枚举 Enum<br>
		用枚举写单例实在太简单了！这也是它最大的优点。<br>
		创建枚举默认就是线程安全的，所以不需要担心double checked locking，而且还能防止反序列化导致重新创建新的对象。但是还是很少看到有人这样写，可能是因为不太熟悉吧。<br>

		```
		public enum Singleton {  
			INSTANCE;  
				public void doSomeThing() {  
			}  
		}  
		```
		
		<br>
		我们可以通过EasySingleton.INSTANCE来访问实例，这比调用getInstance()方法简单多了。

	10. 参考
		1. http://wuchong.me/blog/2014/08/28/how-to-correctly-write-singleton-pattern/
		2. https://blog.csdn.net/itachi85/article/details/50510124

		
		
#### 面向对象的特性
TODO
