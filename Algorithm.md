## 算法

1. 题目类型

2. 常见数据结构
	1. 链表
		1. 单链表的表示
			
			```
			class ListNode{
		        int value;
		        ListNode next;
		        public ListNode(int value,ListNode next){
		        }
		    }
			```
		
		2. 链表的反转
			1. 循环法
				
				```
				public ListNode reverseListNodeWithIterative(ListNode head){
			        if(head ==null || head.next==null){
			            return head;
			        }
			        ListNode pre = null;
			        ListNode cur = head;
			        while(cur!=null){
			            ListNode next = cur.next;
			            cur.next = pre;
			            pre = cur;
			            cur = next;
			        }
			        return pre;
			    }
				```
		
			2. 递归法

				```
				public ListNode reverseListNodeWithRecursive(ListNode head){
			        if(head==null || head.next==null){
			            return head;
			        }
			        ListNode nextNode = head.next;
			        ListNode newHead = reverseListNodeWithRecursive(head);
			        nextNode.next = head;
			        head.next = null;
			        return newHead;
			    }
				
				```
		10. 参考
			1. https://blog.csdn.net/xuexichiru/article/details/79118041		
	3. Map			
		1. HashMap<br>
			HashMap 大家都清楚，底层是 数组 + 红黑树 + 链表 ，同时其是无序的
		2. LinkHashMap<br>
			LinkedHashMap 刚好就比 HashMap 多这一个功能，就是其提供 有序，并且，LinkedHashMap的有序可以按两种顺序排列，一种是按照插入的顺序，一种是按照读取的顺序。<br>
			其内部是靠 建立一个双向链表 来维护这个顺序的，在每次插入、删除后，都会调用一个函数来进行 双向链表的维护。
		3. LRUCache<br>
			有两种方法：1）哈希表 + 双向链表；2）同样有类似的数据结构 LinkedHashMap
			1）哈希表 + 双向链表<br>
				双向链表按照被使用的顺序存储了这些键值对，靠近头部的键值对是最近使用的，而靠近尾部的键值对是最久未使用的。哈希表即为普通的哈希映射（HashMap），通过缓存数据的键映射到其在双向链表中的位置。<br>			
				这样一来，我们首先使用哈希表进行定位，找出缓存项在双向链表中的位置，随后将其移动到双向链表的头部。<br>
				对于 get 操作，首先判断 key 是否存在：<br>
					1. 如果 key 不存在，则返回 -1−1；<br>
					2. 如果 key 存在，则 key 对应的节点是最近被使用的节点。通过哈希表定位到该节点在双向链表中的位置，并将其移动到双向链表的头部，最后返回该节点的值。<br>
				对于 put 操作，首先判断 key 是否存在：<br>
					1. 如果 key 不存在，使用 key 和 value 创建一个新的节点，在双向链表的头部添加该节点，并将 key 和该节点添加进哈希表中。然后判断双向链表的节点数是否超出容量，如果超出容量，则删除双向链表的尾部节点，并删除哈希表中对应的项；<br>
					2. 如果 key 存在，则与 get 操作类似，先通过哈希表定位，再将对应的节点的值更新为 value，并将该节点移到双向链表的头部。
			
			```
			public class LRUCache {
			    class DLinkedNode {
			        int key;
			        int value;
			        DLinkedNode prev;
			        DLinkedNode next;
			        public DLinkedNode() {}
			        public DLinkedNode(int _key, int _value) {key = _key; value = _value;}
			    }
			
			    private Map<Integer, DLinkedNode> cache = new HashMap<Integer, DLinkedNode>();
			    private int size;
			    private int capacity;
			    private DLinkedNode head, tail;
			
			    public LRUCache(int capacity) {
			        this.size = 0;
			        this.capacity = capacity;
			        // 使用伪头部和伪尾部节点
			        head = new DLinkedNode();
			        tail = new DLinkedNode();
			        head.next = tail;
			        tail.prev = head;
			    }
			
			    public int get(int key) {
			        DLinkedNode node = cache.get(key);
			        if (node == null) {
			            return -1;
			        }
			        // 如果 key 存在，先通过哈希表定位，再移到头部
			        moveToHead(node);
			        return node.value;
			    }
			
			    public void put(int key, int value) {
			        DLinkedNode node = cache.get(key);
			        if (node == null) {
			            // 如果 key 不存在，创建一个新的节点
			            DLinkedNode newNode = new DLinkedNode(key, value);
			            // 添加进哈希表
			            cache.put(key, newNode);
			            // 添加至双向链表的头部
			            addToHead(newNode);
			            ++size;
			            if (size > capacity) {
			                // 如果超出容量，删除双向链表的尾部节点
			                DLinkedNode tail = removeTail();
			                // 删除哈希表中对应的项
			                cache.remove(tail.key);
			                --size;
			            }
			        }
			        else {
			            // 如果 key 存在，先通过哈希表定位，再修改 value，并移到头部
			            node.value = value;
			            moveToHead(node);
			        }
			    }
			
			    private void addToHead(DLinkedNode node) {
			        node.prev = head;
			        node.next = head.next;
			        head.next.prev = node;
			        head.next = node;
			    }
			
			    private void removeNode(DLinkedNode node) {
			        node.prev.next = node.next;
			        node.next.prev = node.prev;
			    }
			
			    private void moveToHead(DLinkedNode node) {
			        removeNode(node);
			        addToHead(node);
			    }
			
			    private DLinkedNode removeTail() {
			        DLinkedNode res = tail.prev;
			        removeNode(res);
			        return res;
			    }
			}
			```
			
			参考：https://leetcode-cn.com/problems/lru-cache/solution/lruhuan-cun-ji-zhi-by-leetcode-solution/
			
		

3. 常见算法	
	1. 排序算法
		1. 排序的稳定性<br>
			假定在待排序的记录序列中，存在多个具有相同的关键字的记录，若经过排序，这些记录的相对次序保持不变，即在原序列中，r[i]=r[j]，且r[i]在r[j]之前，而在排序后的序列中，r[i]仍在r[j]之前，则称这种排序算法是稳定的；否则称为不稳定的
			1. 稳定排序：
				1. 冒泡排序 — O(n²)
				2. 插入排序 — O(n²)
				3. 桶排序 — O(n); 需要 O(k) 额外空间
				4. 归并排序 — O(nlogn); 需要 O(n) 额外空间
				5. 二叉排序树排序 — O(n log n) 期望时间; O(n²)最坏时间; 需要 O(n) 额外空间
				6. 基数排序 — O(n·k); 需要 O(n) 额外空间

			2. 不稳定排序
				1. 选择排序 — O(n²)
				2. 希尔排序 — O(nlogn)
				3. 堆排序 — O(nlogn)
				4. 快速排序 — O(nlogn) 期望时间, O(n²) 最坏情况; 对于大的、乱数串行一般相信是最快的已知排序
		
		2. 冒泡排序<br>
			它重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误就把他们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。冒泡排序总的平均时间复杂度为O(n^2)。冒泡排序是一种稳定排序算法。
			1. 比较相邻的元素。如果第一个比第二个大，就交换他们两个。
			2. 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。在这一点，最后的元素应该会是最大的数。
			3. 针对所有的元素重复以上的步骤，除了最后一个。
			4. 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。
			5. 例子
				
				```
				void bubble_sort(int a[], int n)
				{
				    int i, j, temp;
				    for (j = 0; j < n - 1; j++)
				        for (i = 0; i < n - 1 - j; i++)
				        {
				            if(a[i] > a[i + 1])
				            {
				                temp = a[i];
				                a[i] = a[i + 1];
				                a[i + 1] = temp;
				            }
				        }
				}
				```
		
		3. 直接选择排序<br>
			首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。实际适用的场合非常罕见。
			
			```
			void selection_sort(int arr[], int len) {
				int i, j, min, temp;
				for (i = 0; i < len - 1; i++) {
					min = i;
					for (j = i + 1; j < len; j++)
						if (arr[min] > arr[j])
							min = j;
				   	temp = arr[min];
					arr[min] = arr[i];
					arr[i] = temp;
				}
			}
			```
			
		4. 直接插入排序<br>
			插入排序的基本操作就是将一个数据插入到已经排好序的有序数据中，从而得到一个新的、个数加一的有序数据，算法适用于少量数据的排序，时间复杂度为O(n^2)。是稳定的排序方法。 插入算法把要排序的数组分成两部分：第一部分包含了这个数组的所有元素，但将最后一个元素除外（让数组多一个空间才有插入的位置），而第二部分就只包含这一个元素（即待插入元素）。在第一部分排序完成后，再将这个最后元素插入到已排好序的第一部分中。
			
			```
			void insert_sort(int* a, int len) {
			    for (int i = 1; i < len; ++i) {
			        int j = i - 1;
			        int temp = a[i];
			        while (j >= 0 && temp < a[j]) {
			            a[j + 1] = a[j];
			            j--;
			        }
			        a[j + 1] = temp;
			    }
			}
			```
		
		5. 快速排序<br>
			快速排序是一种 不稳定 的排序算法，平均时间复杂度为 O(nlogn)。快速排序使用分治法（Divide and conquer）策略来把一个序列（list）分为两个子序列（sub-lists）。 
			1. 步骤为：
				1. 从数列中挑出一个元素，称为"基准"（pivot），
				2. 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区结束之后，该基准就处于数列的中间位置。这个称为分区（partition）操作。
				3. 递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序。
			
			2. 时间复杂度
				1. 最坏情况：时间复杂度为O(n^2)。因为最坏情况发生在每次划分过程产生的两个区间分别包含n-1个元素和1个元素的时候。这样递归树就是一棵斜树。时间复杂度是 O(n^2)
				
				2. 最好情况：每次划分选取的基准都是当前无序区的中值。如果每次划分过程产生的区间大小都为n/2，则递归树就是一棵平衡二叉树？，时间复杂度是 O(nlogn)。
				
				3. 平均的情况，通过数学归纳法算出是 O(nlogn)

			3. 例子
				
				```
				void Qsort(int a[], int low, int high)
				{
				  if(low >= high)
				  {
				      return;
				  }
				  int first = low;
				  int last = high;
				  int key = a[first];
					
				  while(first < last)
				  {
				      while(first < last && a[last] >= key)
				      {
				          --last;
				      }
				      a[first] = a[last];
				      while(first < last && a[first] <= key)
				      {
				          ++first;
				      }
				      a[last] = a[first];    
				  }
				  a[first] = key;
				  Qsort(a, low, first-1);
				  Qsort(a, first+1, high);
				}
				```
			
			4. 快排的优化
				1. 当待排序序列的长度分割到一定大小后，使用插入排序。
				2. 快排函数在函数尾部有两次递归操作，我们可以对其使用尾递归优化。优化后，可以缩减堆栈深度，由原来的O(n)缩减为O(logn)，将会提高性能。
				3. 从左、中、右三个数中取中间值。
		
		6. 堆排序<br>
			堆排序利用了大根堆（或小根堆）堆顶记录的关键字最大（或最小）这一特征，使得在当前无序区中选取最大（或最小）关键字的记录变得简单。

			1. 先将初始文件R[1..n]建成一个大根堆，此堆为初始的无序区
			2. 再将关键字最大的记录R[1]（即堆顶）和无序区的最后一个记录R[n]交换，由此得到新的无序区R[1..n-1]和有序区R[n]，且满足R[1..n-1].keys≤R[n].key
			3. 由于交换后新的根R[1]可能违反堆性质，故应将当前无序区R[1..n-1]调整为堆。然后再次将R[1..n-1]中关键字最大的记录R[1]和该区间的最后一个记录R[n-1]交换，由此得到新的无序区R[1..n-2]和有序区R[n-1..n]，且仍满足关系R[1..n-2].keys≤R[n-1..n].keys，同样要将R[1..n-2]调整为堆。
			4. 直到无序区只有一个元素为止。
			5. 例子
				
				```
				static void max_heap(int[] num, int start, int end) {
				    int dad = start;
				    int son = dad * 2 + 1;
				
				    while (son < end) {
				        if (son + 1 < end && num[son] < num[son + 1])
				            son++;
				
				        if (num[dad] > num[son])
				            return;
				        else {
				            num[dad] ^= num[son];
				            num[son] ^= num[dad];
				            num[dad] ^= num[son];
				
				            dad = son;
				            son = dad * 2 + 1;
				        }
				    }
				}
				
				static void heap_sort(int[] num) {
				    for (int i = num.length / 2 - 1; i >= 0; --i) {
				        max_heap(num, i, num.length);
				    }
				
				    for (int i = num.length - 1; i >= 0; --i) {
				        num[i] ^= num[0];
				        num[0] ^= num[i];
				        num[i] ^= num[0];
				
				        max_heap(num, 0, i);
				    }
				}
				```
		
		10. 参考	
			1. https://github.com/hadyang/interview/blob/master/basic/algo/sort.md
	
	2. 查找算法
		1. 二分查找 <br>
			折半查找要求线性表是有序表。搜索过程从数组的中间元素开始，如果中间元素正好是要查找的元素，则搜索过程结束；如果某一特定元素大于或者小于中间元素，则在数组大于或小于中间元素的那一半中查找，而且跟开始一样从中间元素开始比较。如果在某一步骤数组为空，则代表找不到。这种搜索算法每一次比较都使搜索范围缩小一半。折半搜索每次把搜索区域减少一半，时间复杂度为O(log n)。
			1. 可以借助二叉判定树求得折半查找的平均查找长度：log2(n+1)-1。
			2. 折半查找在失败时所需比较的关键字个数不超过判定树的深度，n个元素的判定树的深度和n个元素的完全二叉树的深度相同log2(n)+1。
			3. 例子				
				
				```
				public int binarySearchStandard(int[] num, int target){
				    int start = 0;
				    int end = num.length - 1;
				    while(start <= end){ //注意1
				        int mid = start + ((end - start) >> 1);
				        if(num[mid] == target)
				            return mid;
				        else if(num[mid] > target){
				            end = mid - 1; //注意2
				        }
				        else{
				            start = mid + 1; //注意3
				        }
				    }
				    return -1;
				}
				```
			

3. 解题心得
	1. 先从万能的蛮力穷举法
		穷举法的时候，要点是先判断起点和终点，再穷举
	
	2. 再优化穷举法
		1. 用空间换时间：散列表
		2. 分治法：分而治之，然后归并
	
	3. 选择合适的数据结构
		比如用树：比如在寻找最小的第N个数、或者是海量的数据处理
	
	4. 选择先排序或者局部排序，期间考虑是否能用二分法
	
	5. 动态规划、保存变量：每次存最大/最小值
