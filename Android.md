## Android
<span id = "home"></span>
#### 目录
1. [Activity](#android_activity)
2. [Service](#android_service)
3. [View](#android_view)
4. [Android系统深入](#android_system)
5. [Android进程间通信和消息分发机制](#android_ipc)
6. [性能优化](#performance_optimization)
7. [Android存储](#android_storage)
8. [图片加载](#android_image)

----

<span id = "android_activity"></span>
#### Activity [(TOP)](#home)
1. 四种启动模式<br>
	Activity的管理采用任务栈的形式。对应以下4种启动模式(LaunchMode)
	1. Standard<br>
		标准模式。每次启动Activity都会创建新的实例。谁启动了这个Activity，那么这个Activity就运行在谁的Task中。不能使用非Activity类型的context启动这种模式的Activity，因为这种context并没有Task。
		<br>
		假如需要用非Activity的Context启动Activity，就可以加一个FLAG_ACTIVITY_NEW_TASK标记位，这个时候启动Activity实际上是以singleTask模式启动。
	
	2. SingleTop<br>
		栈顶复用模式。如果当前栈顶是要启动的Activity，那么直接引用，如果不是，则新建。在直接引用的时候会调用onNewIntent()方法。这种模式一样是运行在其他人的Task中。
		<br>
		使用情景:
		1. 避免正在A页面，然后通过任务栏的Push再进入A页面后，假如按返回键有两个A页面
		2. 避免由于点击速度过快，同时启动多个A页面
		
	3. SingleTask<br>
		栈内复用模式，也是一种单实例模式。
		1. 模式介绍<br>
		这种模式下，只要Activity只要在一个栈内存在，那么就不会创建新的实例，会调用onNewIntent()方法。
			1. 如果要调用的Activity在同一应用中：调用singleTask模式的Activity会清空在它之上的所有Activity。
			2. 若其他应用启动该Activity：如果不存在，则建立新的Task。如果已经存在后台，那么启动后，后台的Task会一起被切换到前台。	
		2. 使用情景：
			1. 适合作为程序入口点，例如浏览器的主界面。不管从多少个应用启动浏览器，只会启动主界面一次，其余情况都会走onNewIntent，并且会清空主界面上面的其他页面。
			2. 保证这个Activity是唯一实例，减少内存占用 
		3. 指明Task名<br>
			android:taskAffinity="xxxx"。如果Task没启动就新建一个任务栈，如果Task启动了就直接加入它的任务栈。
		4. 特殊例子<br>
			假设D是SingleTask，ABC是Standard。栈为ADBC，此时D进栈。栈变成为AD	

	4. SingleInstance<br>
		单一实例模式，整个手机操作系统里面只有一个实例存在。不同的应用去打开这个activity 共享公用的同一个activity。他会运行在自己单独，独立的任务栈里面，并且任务栈里面只有他一个实例存在。应用场景：呼叫来电界面。这种模式的使用情况比较罕见，在Launcher中可能使用。或者你确定你需要使Activity只有一个实例。 <br>
	可以得出以下结论： 
		1. 以singleInstance模式启动的Activity具有全局唯一性，即整个系统中只会存在一个这样的实例。 
		2. 以singleInstance模式启动的Activity在整个系统中是单例的，如果在启动这样的Activiyt时，已经存在了一个实例，那么会把它所在的任务调度到前台，重用这个实例。 
		3. 以singleInstance模式启动的Activity具有独占性，即它会独自占用一个任务，被他开启的任何activity都会运行在其他任务中。 
		4. 被singleInstance模式的Activity开启的其他activity，能够在新的任务中启动，但不一定开启新的任务，也可能在已有的一个任务中开启。
	
	5. 前台任务栈和后台任务栈交互<br>
		假如前台任务栈是AB、后台任务栈是CD。假如以SingleTask的方式启动Activity D，只会清掉Activity C，不会清掉另外一个任务栈里面的AB。

2. 生命周期
	1. Activity生命周期的7个方法
		1. onCreate()：当Activity第一次被实例化的时候系统会调用，整个生命周期只调用1次这个方法。。
		2. onStart()：当Activity可见未获得用户焦点不能交互时系统会调用。
		3. onRestart()：当Activity已经停止然后重新被启动时系统会调用。
		4. onResume()：当Activity可见且获得用户焦点能交互时系统会调用。
		5. onPause()：当Activity失去焦点，但是还是可见状态时被系统调用。
		6. onStop()：当Activity被新的Activity完全覆盖不可见时被系统调用。
		7. onDestroy()：当Activity被用户调用finish()或按下返回键时系统调用，整个生命周期只调用1次。  
		
	2. onNewIntent()相关
		<br>启动SingleTop、SingleTask、SingleInstance模式的Activity时，如果栈顶(对于SingleTop)或栈中（对于SingleTask、SingleInstance）已经存在该Activity时，
		1. 如果该Activity的实例存在并且在后台，走的生命周期: onNewIntent()---->onResart()------>onStart()----->onResume()。<br>
		当调用到onNewIntent(intent)的时候，需要在onNewIntent() 中使用setIntent(intent)赋值给Activity的Intent.否则，后续的getIntent()都是得到老的Intent。
		2. 如果该Activity的实例存在并且在前台，走的生命周期: onPause()---->onNewIntent()----->onResume()。<br>
		3. 如果该Activity被系统回收，走的生命周期:走onCreate()------>onStart()------>onRestoreInstanceState()----->onResume()。
		4. 处理Intent的例子：
		
			```
			public void onCreate(Bundle savedInstanceState) {
			    super.onCreate(savedInstanceState);
			    setContentView(R.layout.main);
			    processExtraData();
			}
			protected void onNewIntent(Intent intent) {
			    super.onNewIntent(intent);
			    setIntent(intent);//must store the new intent unless getIntent() will return the old one
			    processExtraData();
			}
			private void processExtraData(){
			    Intent intent = getIntent();
			    //use the data received here
			}
			
			```
	2. 在一个Activity启动另一个Activity
		1. 第一个Activity的启动顺序：onCreate()------>onStart()------>onResume()
		2. 当另一个Activity启动时：第一个Activity onPause()------>第二个Activity onCreate()------>onStart()------>onResume()------>第一个Activity onStop()
		3. 当返回到第一个Activity时：第二个Activity onPause()——> 第一个Activity onRestart()——>onStart()——>onResume()——>第二个Activity onStop()——>onDestroy()
			
	3. A B C 顺序的Activity，C Activity 后台被杀
		
		```
		// SecondActivity 启动 ThirdActivity
		ActivityStack: SecondActivity onPause
		ActivityStack: ThirdActivity onCreate
		ActivityStack: ThirdActivity onStart
		ActivityStack: ThirdActivity onResume
		ActivityStack: SecondActivity onSaveInstanceState
		ActivityStack: SecondActivity onStop
		
		// 应用退到后台
		ActivityStack: ThirdActivity onPause
		ActivityStack: ThirdActivity onSaveInstanceState
		ActivityStack: ThirdActivity onStop
		
		// 系统后台杀进程，然后手动重新启动APP
		ActivityStack: ThirdActivity onCreate
		ActivityStack: ThirdActivity onStart
		ActivityStack: ThirdActivity onRestoreInstanceState
		ActivityStack: ThirdActivity onResume
		
		// 按返回键。创建SecondActivity，并调用SecondActivity的onRestoreInstanceState
		ActivityStack: ThirdActivity onPause
		ActivityStack: SecondActivity onCreate
		ActivityStack: SecondActivity onStart
		ActivityStack: SecondActivity onRestoreInstanceState
		ActivityStack: SecondActivity onResume
		ActivityStack: ThirdActivity onStop
		ActivityStack: ThirdActivity onDestroy
		```		
	
		小结：
		1. ThirdActivity后台时，APP被杀。由于已经调用了ThirdActivity的onSaveInstanceState，所以重启APP，恢复的是ThirdActivity
		2. 多Activity时应用被杀，重启APP不会把多Activity都创建，只会创建栈顶的Activity，但是Activity栈记录还会保留，按Back键依然会返回并创建SecondActivity		
		
	4. A B C 顺序的Activity，C Activity 前台被杀

		```
		// 从SecondActivity跳转到ThirdActivity
		ActivityStack: SecondActivity onPause
		ActivityStack: ThirdActivity onCreate
		ActivityStack: ThirdActivity onStart
		ActivityStack: ThirdActivity onResume
		ActivityStack: SecondActivity onSaveInstanceState
		ActivityStack: SecondActivity onStop
		
		// ThirdActivity前台时，进程被杀
		ActivityStack: SecondActivity onCreate
		ActivityStack: SecondActivity onStart
		ActivityStack: SecondActivity onRestoreInstanceState
		ActivityStack: SecondActivity onResume
		```
		小结
		1. 由于ThirdActivity没有来得及调用onSaveInstanceState，所以恢复的是SecondActivity
		2. 出现情景：ThirdActivity 位于前台，不可能由于内存不足而被系统回收(内存不足，最多就抛OOM)。只可能出现Crash或被命令行强制kill。
	
	5. 不保留活动下
		1. Activity退到后台: onPause() ----> onSaveInstanceState() ----> onStop() ----> onDestroy()
		2. Activity回到前台: onCreate() ----> onStart() ----> onRestoreInstanceState() ----> onResume()
		
	6. 横竖屏切换<br>
		生命周期：onPause() -> onSaveInstanceState() -> onStop() -> onDestroy() -> onCreate() -> onStart() -> onRestoreInstanceState -> onResume()
		<br>
		可以通过在AndroidManifest文件的Activity中指定如下属性：
		```
		android:configChanges = "orientation| screenSize"
		```
		来避免横竖屏切换时Activity的销毁和重建。而是改为回调onConfigurationChanged()

			
3. 被系统回收时的Activity	
	1. 生命周期<br>
		不会走onNewIntent()，走onCreate()------>onStart()------>onRestoreInstanceState()----->onResume()<br>
		此时onCreate(Bundle savedInstanceState) 中的 savedInstanceState是 savedInstanceState()时保存的Bundle。而第一次启动Activity的onCreate(), savedInstanceState会为null。
	2. activity恢复原则<br>
	如果多个activity在栈中，多个activity会被一起恢复吗？
不会的，只会恢复栈顶的activity，但是栈是恢复了的。在按backspace之后，会创建下一个activity实例,

4. 几个Activity不常用的回调方法<br>
	onContentChanged，onPostCreate，onPostResume
	<br>
	程序启动运行并结束上述生命周期的方法执行顺序是这样的：<br>
onCreate –> **onContentChanged** –> onStart –> onRestoreInstanceState -> **onPostCreate** –> onResume –> **onPostResume** –> onPause –> onSaveInstanceState -> onStop –> onDestroy
	1. onContentChanged<br>
		onContentChanged()是Activity中的一个回调方法 当Activity的布局改动时，即setContentView()或者addContentView()方法执行完毕时就会调用该方法， 例如，Activity中各种View的findViewById()方法都可以放到该方法中。
		
	2. onPostCreate<br>
		onPostCreate方法是指onCreate方法彻底执行完毕的回调

	3. onPostResume<br>
		onPostResume方法是指onResume方法彻底执行完毕的回调
		
5. Activity的三种运行状态
	1. Resumed（活动状态）<br>
	又叫Running状态，这个Activity正在屏幕上显示，并且有用户焦点。这个很好理解，就是用户正在操作的那个界面。
	2. Paused（暂停状态）<br>
这是一个比较不常见的状态。这个Activity在屏幕上是可见的，但是并不是在屏幕最前端的那个Activity。比如有另一个非全屏或者透明的Activity是Resumed状态，没有完全遮盖这个Activity。
	3. Stopped（停止状态）<br>
当Activity完全不可见时，此时Activity还在后台运行，仍然在内存中保留Activity的状态，并不是完全销毁。这个也很好理解，当跳转的另外一个界面，之前的界面还在后台，按回退按钮还会恢复原来的状态，大部分软件在打开的时候，直接按Home键，并不会关闭它，此时的Activity就是Stopped状态。		

6. Activity的两种启动方式
	1. 显式启动:明确指定被启动对象的组件信息，包括包名和类名
	2. 隐式启动:不需要明确指定组件信息

7. Intent Filter<br>
android的3个核心组件——Activity、services、广播接收器——是通过intent传递消息的。intent消息用于在运行时绑定不同的组件。<br> 
在 Android 的 AndroidManifest.xml 配置文件中可以通过 intent-filter 节点为一个 Activity 指定其 Intent Filter，以便告诉系统该 Activity 可以响应什么类型的 Intent。<br>
	intent-filter 的三大属性
	1. Action<br>
		一个 Intent Filter 可以包含多个 Action，Action 列表用于标示 Activity 所能接受的“动作”，它是一个用户自定义的字符串。
		
		```
		<intent-filter > 
		<action android:name="android.intent.action.MAIN" /> 
		<action android:name="com.scu.amazing7Action" /> 
		……
		</intent-filter>
		```
		
		在代码中使用以下语句便可以隐式启动该Intent对象
		
		```
		Intent i=new Intent(); 
		i.setAction("com.scu.amazing7Action");
		```
		
		Action 列表中包含了“com.scu.amazing7Action”的 Activity 都将会匹配成功
	
	2. URL<br>
		在 intent-filter 节点中，通过 data节点匹配外部数据，也就是通过 URI 携带外部数据给目标组件。<br>
		注意：只有data的所有的属性都匹配成功时 URI 数据匹配才会成功
		
		```
		<data android:mimeType="mimeType" 
			android:scheme="scheme" 
		 	android:host="host"
		 	android:port="port" 
		 	android:path="path"/>
		```
		
	3. Category<br>
		为组件定义一个 类别列表，当 Intent 中包含这个类别列表的所有项目时才会匹配成功。
	
		```
		<intent-filter . . . >
	   <action android:name="code android.intent.action.MAIN" />
	   <category android:name="code　android.intent.category.LAUNCHER" />
		</intent-filter>
		```
	
	4. Activity 种 Intent Filter 的匹配过程
		1. 加载所有的Intent Filter列表
		2. 去掉action匹配失败的Intent Filter
		3. 去掉url匹配失败的Intent Filter
		4. 去掉Category匹配失败的Intent Filter
		5. 判断剩下的Intent Filter数目是否为0。如果为0查找失败返回异常；如果大于0，就按优先级排序，返回最高优先级的Intent Filter
	
8. 其他<br>
	1. 大部分不需要公开给外部调用的Activity，都应该设置成非公开：android:exported="false"

----

<span id = "android_service"></span>
#### Service [(TOP)](#home)
TODO

----

<span id = "android_view"></span>
#### View [(TOP)](#home)
1. View的坐标体系
	1. View的坐标图<br>
		注意x轴和y轴的方向<br>
		![](https://github.com/yinfork/Android-Interview/blob/master/res/android/view/view_x_y.png?raw=true)

	2. 4个基本属性<br>
		对应着View的四个顶点：top、left、right、bottom
	
	3. 宽度和高度的计算
		1. width = right - left
		2. height = bottom - top
	
	4. 坐标的偏移量
		1. translationX: View左上角的点相对于父容器的X方向的偏移量
		2. translationY: View左上角的点相对于父容器的Y方向的偏移量
	
	5. x、y的坐标<br>
		x、y是表示View的左上角的点的坐标。当View平移的时候，top和left的原始值不会改变，改变的是 x、y、translationX 和 translationY。
		1. x = left + translationX
		2. y = top + translationY
	
2. View的事件分发流程
	1. 事件<br>
		1. MotionEvent.ACTION_DOWN　：手指按下屏幕的瞬间（一切事件的开始）
		2. MotionEvent.ACTION_MOVE　：手指在屏幕上移动
		3. MotionEvent.ACTION_UP　：手指离开屏幕瞬间
		4. MotionEvent.ACTION_CANCEL 　：取消手势，一般由程序产生，不会由用户产生
		5. TODO 补充其他事件
	
	2. 点击事件传递的对象流程<br>
		Activity（Window） -> ViewGroup -> View
	
	3. 事件分发的三个重要事件<br>
		1. 事件分发: dispatchTouchEvent()。Activity、ViewGroup和View都有此方法。
			1. return true :表示该View内部消化掉了所有事件。
			2. return false :事件在本层不再继续进行分发，并交由上层控件的onTouchEvent方法进行消费（如果本层控件已经是Activity，那么事件将被系统消费或处理）。　
			3. return super.dispatchTouchEvent(ev): 事件将分发给本层的事件拦截onInterceptTouchEvent 方法进行处理
			
		2. 事件拦截: onInterceptTouchEvent()。只有ViewGroup才有，Activity和View都没有
			1. return true: 表示将事件进行拦截，并将拦截到的事件交由本层控件 的 onTouchEvent 进行处理；
			2. return false: 则表示不对事件进行拦截，事件得以成功分发到子View。并由子View的dispatchTouchEvent进行处理。　
			3. return super.onInterceptTouchEvent(ev): 默认不拦截该事件，和return true一样。
		
		3. 事件响应: onTouchEvent()。Activity、ViewGroup和View都有此方法。
			1. return true: 表示onTouchEvent处理完事件后消费了此次事件。此时事件终结；
			2. return fasle: 则表示不响应事件，那么该事件将会不断向上层View的onTouchEvent方法传递，直到某个View的onTouchEvent方法返回true，如果到了最顶层View还是返回false，那么认为该事件不消耗，则在同一个事件系列中，当前View无法再次接收到事件，该事件会交由Activity的onTouchEvent进行处理；　　
			3. return super.dispatchTouchEvent(ev): 则表示不响应事件，结果与return false一样。
	
	4. 事件分发的流程	
		1. 伪代码描述：
				
			```
			// Activity
			public boolean dispatchTouchEvent(MotionEvent ev) {
		        if (getWindow().superDispatchTouchEvent(ev)) {
		            return true;
		        }
		        return onTouchEvent(ev);
		    }
			
			// ViewGroup
			public boolean dispatchTouchEvent(MotionEvent ev) {
			    boolean consume = false;
			    if (onInterceptTouchEvent(ev)) {
			        consume = onTouchEvent(ev);
			    } else {
			        consume = child.dispatchTouchEvent(ev);
			    }
			
			    return consume;
			}
			
			// View
			public boolean dispatchTouchEvent(MotionEvent ev) {
				// 省略了一堆其他事件的处理
				return onTouchEvent(ev);
		    }
			```
		2. 流程图
		![](https://github.com/yinfork/Android-Interview/blob/master/res/android/view/view_event.png?raw=true)
		
	5. 事件序列
		1. 定义<br>
			同一个事件序列是从手指触摸屏幕的那一刻起，到手指离开屏幕那一刻结束，这个过程中所产生的一系列事件。这个事件序列以down事件开始，中间含有数量不定的move事件，最终以up事件结束
		
		2. 默认情况：不拦截、不消费<br>
			如果ViewGroup的onInterceptTouchEvent方法对DOWN事件返回了false，后续的事件（MOVE、UP）依然会传递给它的onInterceptTouchEvent()
		
		3. 处理事件：消费该事件
			1. DOWN事件被传递给C的onTouchEvent方法，该方法返回true，表示处理这个事件
			2. 因为C正在处理这个事件，那么DOWN事件将不再往上传递给B和A的onTouchEvent()；
			3. 该事件列的其他事件（Move、Up）也将传递给C的onTouchEvent()
		
		4. 拦截DOWN事件<br>
			假设ViewGroup希望处理这个点击事件，即覆写了onInterceptTouchEvent()返回true、onTouchEvent()返回true。事件传递情况：
			1. DOWN事件被传递给B的onInterceptTouchEvent()方法，该方法返回true，表示拦截这个事件，即自己处理这个事件（不再往下传递）
			2. 调用onTouchEvent()处理事件（DOWN事件将不再往上传递给A的onTouchEvent()）
			3. 该事件列的其他事件（Move、Up）将直接传递给B的onTouchEvent()
			4. 该事件列的其他事件（Move、Up）将不会再传递给B的onInterceptTouchEvent方法，该方法一旦返回一次true，就再也不会被调用了。
		
		5. 不拦截DOWN，但拦截其他后续事件<br>
			假设ViewGroup B没有拦截DOWN事件（还是子View C来处理DOWN事件），但ViewGroup B拦截了接下来的MOVE事件
			1. DOWN事件传递到C的onTouchEvent方法，返回了true。
			2. **在后续到来的MOVE事件，B的onInterceptTouchEvent方法返回true拦截该MOVE事件，但该事件并没有传递给B；这个MOVE事件将会被系统变成一个CANCEL事件传递给C的onTouchEvent方法**
			3. 直到后续又来了一个MOVE事件，该MOVE事件才会直接传递给B的onTouchEvent()，并且后续事件将直接传递给B的onTouchEvent()处理
			4. 后续事件将不会再传递给B的onInterceptTouchEvent方法，该方法一旦返回一次true，就再也不会被调用了
			5. C再也不会收到该事件列产生的后续事件。
		
	6. 其他注意点
		1. dispatchTouchEvent无论返回true还是false，事件都不再进行分发，只有当其返回super.dispatchTouchEvent(ev)，才表明其具有向下层分发的愿望
		2. 点击事件分发过程如下 dispatchTouchEvent—->OnTouchListener的onTouch方法—->onTouchEvent-->OnClickListener的onClick方法。也就是说，我们平时调用的setOnClickListener，优先级是最低的，所以，onTouchEvent或OnTouchListener的onTouch方法如果返回true，则不响应onClick方法
		3. 子View可以通过调用getParent().requestDisallowInterceptTouchEvent(true); 阻止ViewGroup对其MOVE或者UP事件进行拦截；
		4. onTouch()和onTouchEvent()的区别
			1. 这两个方法都是在View的dispatchTouchEvent中调用，但onTouch优先于onTouchEvent执行。
			2. 如果在onTouch方法中返回true将事件消费掉，onTouchEvent()将不会再执行。
			3. 如果你有一个控件是非enable的，那么给它注册onTouch事件将永远得不到执行。对于这一类控件，如果我们想要监听它的touch事件，就必须通过在该控件中重写onTouchEvent方法来实现

	7. 参考
		1. https://www.jianshu.com/p/e99b5e8bd67b 
		2. https://github.com/LRH1993/android_interview/blob/master/android/basis/Event-Dispatch.md
		3. https://github.com/Mr-YangCheng/ForAndroidInterview/blob/master/android/Android%20View%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91%E6%9C%BA%E5%88%B6%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.md

3. View的绘制流程<br>
	1. 整体流程
		1. 综述<br>
			整个View树的绘图流程是在ViewRootImpl类的performTraversals()方法开始的，该函数做的执行过程主要是根据之前设置的状态，判断是否重新计算视图大小(measure)、是否重新放置视图的位置(layout)、以及是否重绘 (draw)。
		2. 流程图<br>
			![](https://github.com/yinfork/Android-Interview/blob/master/res/android/view/view_draw.jpg?raw=true)
	
	2. Measure流程<br>
		测量每个控件的大小。
		1. 流程<br>
			1. 调用measure()方法，进行一些逻辑处理；
			2. 然后在measure()里面调用onMeasure()方法；
			3. 假如是ViewGroup，还会在onMeasure()里面测量它里面的子View，最后一般根据测量的子View的大小确定自己的测量大小
			4. 最终在onMeasure()里面调用**setMeasuredDimension()**设定View的宽高信息，完成View的测量操作。
			5. 代码：
			
				```
				public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
				
				}
				
				protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
					setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
			    }
				```
			
		2. MeasureSpec
			1. MeasureSpec的组成<br>
				MeasureSpec(32位的int值)由两部分组成：测量模式(高2位，31、32位) 和 测量的尺寸大小(底30位)。 
			2. 测量模式
				1. UNSPECIFIED ：不对View进行任何限制，要多大给多大，一般用于系统内部
				2. EXACTLY：对应LayoutParams中的match_parent和具体数值这两种模式。检测到View所需要的精确大小，这时候View的最终大小就是SpecSize所指定的值，
				3. AT_MOST ：对应LayoutParams中的wrap_content。View的大小不能大于父容器的大小。 	

			3. 确定MeasureSpec的数值
				1. 对于DecorView，其确定是通过屏幕的大小，和自身的布局参数LayoutParams。<br>
					根据LayoutParams的布局格式（match_parent，wrap_content或指定大小），将自身大小，和屏幕大小相比，设置一个不超过屏幕大小的宽高，以及对应模式。

				2. 对于其他View（包括ViewGroup），其确定是通过父布局的MeasureSpec和自身的布局参数LayoutParams。	
					![](https://github.com/yinfork/Android-Interview/blob/master/res/android/view/view_measurespec.png?raw=true)
			
	3. Layout流程<br>
		确定View的位置。
		1. 流程
			1. ViewGroup先在layout()中通过 **setFrame() 或者 setOpticalFrame()** 先确定自己的布局。
			2. 然后在layout()里面调用onLayout()方法
			3. 假如是ViewGroup，会在onLayout()中再调用子View的layout()方法，让子View布局。假如是View，则在onLayout()不会做什么
			4. layout流程和measure流程的区别<br>
				layout()中确定自己的布局，然后在onLayout()方法中再调用子View的layout()方法，让子View布局。而在Measure过程中，ViewGroup一般是先测量子View的大小，然后再确定自身的大小。

		2. 注意点
			1. layout()的入参是left、top、right、bottom，都是相对于父View而言
			2. getWidth() 和 getMeasuredWidth() 的区别
				1. getWidth() 里面的 width = right - left，表示这个view最终显示的大小。
					需要在onLayout()流程里面才能有有效的返回值。因为layout流程是先确定自己的值，然后再调用onLayout()
				2. getMeasuredWidth() 的返回值是 mMeasuredWidth 里面的低24位（注意不是和MeasureSpec一样，MeasureSpec的值是低30位）:mMeasuredWidth & MEASURED_SIZE_MASK，表示view原始的大小，也就是这个view在XML文件中配置或者是代码中设置的大小。<br>
					要在onMeasure流程结束才有值，因为onMeasure()里面调用setMeasuredDimension()设定View的宽高信息

	4. Draw流程
		1. 流程
			1. 绘制背景 background.draw(canvas)
			2. 绘制自己（onDraw）
			3. 绘制Children(dispatchDraw)
			4. 绘制装饰（onDrawScrollBars）
		2. 注意点
			1. 自定义View一般要重写onDraw()方法，在其中绘制不同的样式
			2. View默认不会绘制任何内容，真正的绘制都需要自己在子类中实现
			3. View的绘制是借助onDraw方法传入的Canvas类来进行的
	
	5. 参考
		1. https://github.com/LRH1993/android_interview/blob/master/android/basis/custom_view.md
		2. https://github.com/hadyang/interview/blob/master/android/draw.md 	
			
4. View、Activity 和 Window的关系
	1. Activity<br>
		Activity并不负责视图控制，它只是控制生命周期和处理事件。真正控制视图的是Window。一个Activity包含了一个Window，Window才是真正代表一个窗口。Activity就像一个控制器，统筹视图的添加与显示，以及通过其他回调方法，来与Window、以及View进行交互。
		
	2. Window<br>
		Window是视图的承载器，内部持有一个 DecorView，而这个DecorView才是 view 的根布局。Window是一个抽象类，实际在Activity中持有的是其子类PhoneWindow。PhoneWindow中有个内部类DecorView，通过创建DecorView来加载Activity中设置的布局R.layout.activity_main。Window通过WindowManager将DecorView加载其中，并将DecorView交给ViewRoot，进行视图绘制以及其他交互。

	3. View			
		1. DecorView<br>
			DecorView是FrameLayout的子类，它可以被认为是Android视图树的根节点视图。DecorView作为顶级View，一般情况下它内部包含一个竖直方向的LinearLayout，在这个LinearLayout里面有上下三个部分，上面是个ViewStub，延迟加载的视图（应该是设置ActionBar，根据Theme设置），中间的是标题栏(根据Theme设置，有的布局没有)，下面的是内容栏。 具体情况和Android版本及主体有关

		2. ViewRoot<br>
			ViewRoot其作用非常重大。所有View的绘制以及事件分发等交互都是通过它来执行或传递的。<br>
			ViewRoot对应ViewRootImpl类，它是连接WindowManagerService和DecorView的纽带，用于让DecorView通过WindowManager向WindowManagerService进行进程间通信。View的三大流程（测量（measure），布局（layout），绘制（draw））均通过ViewRoot来完成。<br>
			ViewRoot并不属于View树的一份子。从源码实现上来看，它既非View的子类，也非View的父类，但是，它实现了ViewParent接口，这让它可以作为View的名义上的父视图。RootView继承了Handler类，可以接收事件并分发，Android的所有触屏事件、按键事件、界面刷新等事件都是通过ViewRoot进行分发的。

	4. 三者的关系图<br>
		![](https://github.com/yinfork/Android-Interview/blob/master/res/android/view/view_activity_window.png?raw=true)

	5. 简要界面显示流程
		1. Activity#attach()<br>
			创建PhoneWindow
			
			```
			final void attach(...) {
				..........................................................
				mWindow = new PhoneWindow(this, window);//创建一个Window对象
				mWindow.setWindowControllerCallback(this);
				mWindow.setCallback(this);//设置回调，向Activity分发点击或状态改变等事件
				mWindow.setOnWindowDismissedCallback(this);
				.................................................................
				mWindow.setWindowManager(
					(WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
					mToken, mComponent.flattenToString(),
				        (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);//给Window设置WindowManager对象
				........................................................
			}
			```
			
		2. Activity#setContentView()<br>
			
			```
			// Activity
			public void setContentView(@LayoutRes int layoutResID) {
				getWindow().setContentView(layoutResID);
				initWindowDecorActionBar();
			}
			```
						
			可以看出，Activity#setContentView() 最终调用到 Window#setContentView()
			
			```
			// Window
			public void setContentView(int layoutResID) {
				if (mContentParent == null) {//mContentParent为空，创建一个DecroView
					installDecor();
				} else {
				    mContentParent.removeAllViews();//mContentParent不为空，删除其中的View
				}
				mLayoutInflater.inflate(layoutResID, mContentParent);//为mContentParent添加子View,即Activity中设置的布局文件
				final Callback cb = getCallback();
				if (cb != null && !isDestroyed()) {
				    cb.onContentChanged();//回调通知，内容改变
				}
			}
			```
			
			mContentParent是前面布局中@android:id/content所对应的FrameLayout。假如发现mContentParent为空，则会去调用installDecor()创建DecorView
			
		3. 生成DecorView<br>
			
			```
			private void installDecor() {
			    if (mDecor == null) {
			        mDecor = generateDecor(); //生成DecorView
			        mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
			        mDecor.setIsRootNamespace(true);
			        if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
			            mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
			        }
			    }
			    if (mContentParent == null) {
			        mContentParent = generateLayout(mDecor); // 为DecorView设置布局格式，并返回mContentParent
			        ...
			        } 
			    }
			}
			
			```			
			
			创建DecorView
			
			```
			protected DecorView generateDecor() {
			    return new DecorView(getContext(), -1);
			}
			```

			根据主题样式设置DocorView，然后再添加setContentView()传入的Layout
			
			```

			protected ViewGroup generateLayout(DecorView decor) {
			    // 从主题文件中获取样式信息
			    TypedArray a = getWindowStyle();
			
			    ...................
			
			    if (a.getBoolean(R.styleable.Window_windowNoTitle, false)) {
			        requestFeature(FEATURE_NO_TITLE);
			    } else if (a.getBoolean(R.styleable.Window_windowActionBar, false)) {
			        // Don't allow an action bar if there is no title.
			        requestFeature(FEATURE_ACTION_BAR);
			    }
			    ................
			    // 根据主题样式，加载窗口布局
			    int layoutResource;
			    int features = getLocalFeatures();
			    // System.out.println("Features: 0x" + Integer.toHexString(features));
			    if ((features & (1 << FEATURE_SWIPE_TO_DISMISS)) != 0) {
			        layoutResource = R.layout.screen_swipe_dismiss;
			    } else if(...){
			        ...
			    }
			
			    View in = mLayoutInflater.inflate(layoutResource, null);//加载layoutResource
			
			    //往DecorView中添加子View，即文章开头介绍DecorView时提到的布局格式，那只是一个例子，根据主题样式不同，加载不同的布局。
			    decor.addView(in, new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT)); 
			    mContentRoot = (ViewGroup) in;
			
			    ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);// 这里获取的就是mContentParent
			    ...
			
			    return contentParent;
			}			
	
			```
			
		4. 显示DecorView<br>
			ActivityThread#handleResumeActivity()里会把DecorView显示出来
			
			```
			final void handleResumeActivity(IBinder token, boolean clearHide, 
			                                boolean isForward, boolean reallyResume) {
			
			    //这个时候，Activity.onResume()已经调用了，但是现在界面还是不可见的
			    ActivityClientRecord r = performResumeActivity(token, clearHide);
			
			    if (r != null) {
			        final Activity a = r.activity;
			        if (r.window == null && !a.mFinished && willBeVisible) {
			            r.window = r.activity.getWindow();
			            View decor = r.window.getDecorView();
			            //decor对用户不可见
			            decor.setVisibility(View.INVISIBLE);
			            ViewManager wm = a.getWindowManager();
			            WindowManager.LayoutParams l = r.window.getAttributes();
			            a.mDecor = decor;
			
			            l.type = WindowManager.LayoutParams.TYPE_BASE_APPLICATION;
			
			            if (a.mVisibleFromClient) {
			                a.mWindowAdded = true;
			                //被添加进WindowManager了，但是这个时候，还是不可见的
			                wm.addView(decor, l);
			            }
			
			            if (!r.activity.mFinished && willBeVisible
			                    && r.activity.mDecor != null && !r.hideForNow) {
			                //在这里，执行了重要的操作,使得DecorView可见
			                if (r.activity.mVisibleFromClient) {
			                    r.activity.makeVisible();
			                }
			            }
			        }
			    }
			}
			
			```

			当执行了Activity#makeVisible()方法之后，界面才对我们是可见的。而这个函数关键的方法是调用了WindowManagerImpl#addView()
			
			```
			void makeVisible() {
			   if (!mWindowAdded) {
			        ViewManager wm = getWindowManager();
			        wm.addView(mDecor, getWindow().getAttributes());//将DecorView添加到WindowManager
			        mWindowAdded = true;
			    }
			    mDecor.setVisibility(View.VISIBLE);//DecorView可见
			}
			```
			
			WindowManagerGlobal#addView():其中实例化了ViewRootImpl对象，然后调用其setView()方法。其中setView()方法经过一些列折腾，最终调用了performTraversals()方法，然后依照流程层层调用measure、layout和draw，完成绘制，最终界面才显示出来。
			
			```
			public void addView(View view, ViewGroup.LayoutParams params,
			                    Display display, Window parentWindow) {
			
			    final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams) params;
			
			    ......
			
			    synchronized (mLock) {
			
			        ViewRootImpl root;
			        //实例化一个ViewRootImpl对象
			        root = new ViewRootImpl(view.getContext(), display);
			        view.setLayoutParams(wparams);
			
			        mViews.add(view);
			        mRoots.add(root);
			        mParams.add(wparams);
			    }
			
			    ......
			
			    try {
			        //将DecorView交给ViewRootImpl
			        root.setView(view, wparams, panelParentView);
			    } catch (RuntimeException e) {
			
			    }
			}
			```
			
			ViewRootImpl还有事件分发的作用，事件传递顺序：<br>
			硬件 -> ViewRootImpl -> DecorView -> PhoneWindow -> Activity

	5. 小结
		1. Activity就像个控制器，不负责视图部分。Window像个承载器，装着内部视图。DecorView就是个顶层视图，是所有View的最外层布局。ViewRoot像个连接器，负责沟通，通过硬件的感知来通知视图，进行用户之间的交互。
		2. 本质上讲，我们要显示一个窗口出来，的确可以不需要Activity。悬浮窗口中不就是没有使用Activity来显示一个悬浮窗。<br>
			Android中的应用中，里面对各个窗口的管理相当复杂（任务栈、状态等等），Android系统当然可以不用Activity，让用户自己直接操作Window来开发自己的应用。但是如果让用户自己去管理这些Window，开发量会很大。<br>
			为了让大家能简单、快速的开发应用，Android通过定义Activity，让Activity帮我们管理好，我们只需简单的去重写几个回调函数，无需直接与Window对象接触。
	
	6. 参考
		1. https://github.com/LRH1993/android_interview/blob/master/android/basis/decorview.md
		2. https://github.com/hadyang/interview/blob/master/android/activity-view-window.md

5. 几种Layout的性能比较

----

<span id = "android_system"></span>
#### Android系统深入 [(TOP)](#home)
1. Android应用启动流程<br>
	用两个图结合看APP的启动流程<br>
	![](https://github.com/yinfork/Android-Interview/blob/master/res/android/application/app_launch.png?raw=true)<br>
	
	![](https://github.com/yinfork/Android-Interview/blob/master/res/android/application/app_launch_step.png?raw=true)<br>
	
	具体流程：
	1. 点击桌面App图标，Launcher进程采用Binder IPC向system_server进程发起startActivity请求；
	2. system_server进程接收到请求后，向zygote进程发送创建进程的请求；
	3. Zygote进程fork出新的子进程，即App进程；
	4. App进程，通过Binder IPC向sytem_server进程发起attachApplication请求；
	5. system_server进程在收到请求后，进行一系列准备工作后，再通过binder IPC向App进程发送scheduleLaunchActivity请求；
	6. App进程的binder线程（ApplicationThread）在收到请求后，通过handler向主线程发送LAUNCH_ACTIVITY消息；
	7. 主线程在收到Message后，通过发射机制创建目标Activity，并回调Activity.onCreate()等方法。
	8. 到此，App便正式启动，开始进入Activity生命周期，执行完onCreate/onStart/onResume方法，UI渲染结束后便可以看到App的主界面。


2. 理论基础
	1. zygote<br>
		1. zygote意为“受精卵“。在Android系统里面，zygote是一个进程的名字。Android是基于Linux System的，当你的手机开机的时候，Linux的内核加载完成之后就会启动一个叫“init“的进程。在Linux System里面，所有的进程都是由init进程fork出来的，我们的zygote进程也不例外。<br>
		2. 我们都知道，每一个App其实都是:一个单独的dalvik虚拟机;一个单独的进程<br>	
			所以当系统里面的第一个zygote进程运行之后，在这之后再开启App，就相当于开启一个新的进程。而为了实现资源共用和更快的启动速度，Android系统开启新进程的方式，是通过fork第一个zygote进程实现的。所以说，除了第一个zygote进程，其他应用所在的进程都是zygote的子进程，这下你明白为什么这个进程叫“受精卵”了吧？因为就像是一个受精卵一样，它能快速的分裂，并且产生遗传物质一样的细胞！
	
	
	2. system_server
		1. SystemServer也是一个进程，而且是由zygote进程fork出来的。这个进程是Android Framework里面两大非常重要的进程之一——另外一个进程就是上面的zygote进程。
		2. 为什么说SystemServer非常重要呢？因为系统里面重要的服务都是在这个进程里面开启的，比如 ActivityManagerService、PackageManagerService、WindowManagerService等等。
	
	
	3. ActivityManagerService
		1. ActivityManagerService，简称AMS，服务端对象，负责系统中所有Activity的生命周期。
		2. ActivityManagerService进行初始化的时机很明确，就是在SystemServer进程开启的时候，就会初始化ActivityManagerService。

	4. Launcher<br>
		当我们点击手机桌面上的图标的时候，App就由Launcher开始启动了。
		1. Launcher本质上也是一个应用程序，和我们的App一样，也是继承自Activity
		2. Launcher实现了点击、长按等回调接口，来接收用户的输入。捕捉图标点击事件，然后startActivity()发送对应的Intent请求
		
	5. Instrumentation<br>
		每个Activity都持有Instrumentation对象的一个引用，但是整个进程只会存在一个Instrumentation对象。 Instrumentation这个类里面的方法大多数和Application和Activity有关，**这个类就是完成对Application和Activity初始化和生命周期的工具类。**Instrumentation这个类很重要，对Activity生命周期方法的调用根本就离不开他，他可以说是一个大管家。

	6. ActivityThread<br>
		ActivityThread，依赖于UI线程。App和AMS是通过Binder传递信息的，那么ActivityThread就是专门与AMS的外交工作的。

	7. Android系统里面的服务器和客户端的概念<br>
		在Android的框架设计中，使用的也是这一种模式。服务器端指的就是所有App共用的系统服务，比如我们这里提到的ActivityManagerService，和前面提到的PackageManagerService、WindowManagerService等等，这些基础的系统服务是被所有的App公用的，当某个App想实现某个操作的时候，要告诉这些系统服务。<br>
		App与AMS通过Binder进行IPC通信。


3. Android系统架构
	1. Android官方的系统组成图<br>
		![](https://github.com/yinfork/Android-Interview/blob/master/res/android/application/android-arch1.png?raw=true) <br>
		从下往上依次分为Linux内核、系统库和Android运行时环境、框架层以及应用层这4层架构，其中每一层都包含大量的子模块或子系统。

4. 系统启动流程<br>
	简要图：<br>
	![](https://github.com/yinfork/Android-Interview/blob/master/res/android/application/android-boot2.png?raw=true)<br>
	启动流程架构图:<br>
	![](https://github.com/yinfork/Android-Interview/blob/master/res/android/application/android-boot1.png?raw=true)
	1. Loader层
		1. Boot ROM: 当手机处于关机状态时，长按Power键开机，引导芯片开始从固化在ROM里的预设出代码开始执行，然后加载引导程序到RAM；
		2. Boot Loader：这是启动Android系统之前的引导程序，主要是检查RAM，初始化硬件参数等功能。
	
	2. Kernel层<br>
		Kernel层是指Android内核层，到这里才刚刚开始进入Android系统。
		1. 启动Kernel的swapper进程(pid=0)：该进程又称为idle进程, 系统初始化过程Kernel由无到有开创的第一个进程, 用于初始化进程管理、内存管理，加载Display,Camera Driver，Binder Driver等相关工作；
		2. 启动kthreadd进程（pid=2）：是Linux系统的内核进程，会创建内核工作线程kworkder，软中断线程ksoftirqd，thermal等内核守护进程。**kthreadd进程是所有内核进程的鼻祖**。

	3. Native层<br>
		这里的Native层主要包括init孵化来的用户空间的守护进程、HAL层以及开机动画等。启动**init进程(pid=1),是Linux系统的用户进程，init进程是所有用户进程的鼻祖**。	
		1. init进程会孵化出ueventd、logd、healthd、installd、adbd、lmkd等用户守护进程；
		2. init进程还启动servicemanager(binder服务管家)、bootanim(开机动画)等重要服务
		3. init进程孵化出Zygote进程，Zygote进程是Android系统的第一个Java进程(即虚拟机进程)，Zygote是所有Java进程的父进程，Zygote进程本身是由init进程孵化而来的。

	4. Framework层
		1. Zygote进程，是由init进程通过解析init.rc文件后fork生成的，Zygote进程主要包含：
			1. 加载ZygoteInit类，注册Zygote Socket服务端套接字；
			2. 加载虚拟机；
			3. preloadClasses；
			4. preloadResouces。
		2. System Server进程，是由Zygote进程fork而来，System Server是Zygote孵化的第一个进程，System Server负责启动和管理整个Java framework，包含ActivityManager，PowerManager等服务。
		3. Media Server进程，是由init进程fork而来，负责启动和管理整个C++ framework，包含AudioFlinger，Camera Service，等服务。
	
	5. App层
		1. Zygote进程孵化出的第一个App进程是Launcher，这是用户看到的桌面App；
		2. Zygote进程还会创建Browser，Phone，Email等App进程，每个App至少运行在一个进程上。
		3. 所有的App进程都是由Zygote进程fork生成的。
	
	6. Syscall && JNI
		1. Native与Kernel之间有一层系统调用(SysCall)层
		2. Java层与Native(C/C++)层之间的纽带JNI	
	7. 参考
		1. http://gityuan.com/android/#syscall--jni
		2. http://blog.jobbole.com/67931/
			
10. 参考
	1. https://github.com/LRH1993/android_interview/blob/master/android/advance/app-launch.md 
	2. https://www.jianshu.com/p/c129eea78d61

----

<span id = "android_ipc"></span>
#### Android进程间通信和消息分发机制 [(TOP)](#home)
1. Android进程间通信方式
	1. 使用 Intent<br>
		Activity，Service，Receiver 都支持在 Intent 中传递 Bundle 数据，而 Bundle 实现了 Parcelable 接口，可以在不同的进程间进行传输。
	
	2. 使用 文件共享
		
	3. 使用 Messenger<br>
		Messenger 是一种轻量级的 IPC 方案，它的底层实现是 AIDL ，可以在不同进程中传递 Message 对象，它一次只处理一个请求，在服务端不需要考虑线程同步的问题，服务端不存在并发执行的情形。
	
	4. 使用 AIDL<br>
		AIDL (Android Interface Definition Language) 是一种接口定义语言，用于生成可以在Android设备上两个进程之间进行进程间通信(Interprocess Communication, IPC)的代码。
		1. 支持的类型
			1. Java 的基本数据类型
			2. List 和 Map
				1. 元素必须是 AIDL 支持的数据类型
				2. Server 端具体的类里则必须是 ArrayList 或者 HashMap
			3. 其他 AIDL 生成的接口
			4. 实现 Parcelable 的实体（TODO 要研究支不支持Serializable，因为Intent传输的时候可以传Serializable）<br>
				Android的Parcelable的设计初衷是因为Serializable效率过慢，为了在程序内不同组件间以及不同Android程序间(AIDL)高效的传输数据而设计，这些数据仅在内存中存在，Parcelable是通过IBinder通信的消息的载体
		
		2. 客户端调用远程服务的方法，被调用的方法运行在服务端的 Binder 线程池中，同时客户端的线程会被挂起，如果服务端方法执行比较耗时，就会导致客户端线程长时间阻塞，导致 ANR 。<br>
			解决方法：使用 oneway修饰从而不会阻塞，但是这样函数不管有没有返回值也不回返回结果
		3. 客户端的 onServiceConnected 和 onServiceDisconnected 方法都在 UI 线程中。
		4. 与Messenger的区别<br>
			Messenger 是以串行的方式处理客户端发来的消息，如果大量消息同时发送到服务端，服务端只能一个一个处理，所以大量并发请求就不适合用 Messenger ，而且 Messenger 只适合传递消息，不能跨进程调用服务端的方法。AIDL 可以解决并发和跨进程调用方法的问题，要知道 Messenger 本质上也是 AIDL ，只不过系统做了封装方便上层的调用而已。
		
	5. 使用 ContentProvider<br>
		用于不同应用间数据共享，和 Messenger 底层实现同样是 Binder 和 AIDL，系统做了封装，使用简单。 系统预置了许多 ContentProvider ，如通讯录、日程表，需要跨进程访问。 使用方法：继承 ContentProvider 类实现 6 个抽象方法，这六个方法均运行在 ContentProvider 进程中，除 onCreate 运行在主线程里，其他五个方法均由外界回调运行在 Binder 线程池中。<br>
		ContentProvider 的底层数据，可以是 SQLite 数据库，可以是文件，也可以是内存中的数据。
	
	6. 使用 Socket
		1. Socket 本身可以传输任意字节流。
		2. Socket 是连接应用层与传输层之间接口（API）。 
	
	7. 参考
		1. https://github.com/LRH1993/android_interview/blob/master/android/basis/ipc.md#android-%E4%B8%AD%E7%9A%84-ipc-%E6%96%B9%E5%BC%8F

2. Binder介绍
	1. Binder框架原理图<br>
		![](https://github.com/yinfork/Android-Interview/blob/master/res/android/ipc/android_binder.png?raw=true)
	
	2. Binder框架的四个角色
		1. Client进程：使用服务的进程，运行在用户空间。
		2. Server进程：提供服务的进程，运行在用户空间。
		3. ServiceManager进程：运行在用户空间。ServiceManager的作用是将字符形式的Binder名字转化成Client中对该Binder的引用，使得Client能够通过Binder名字获得对Server中Binder实体的引用。
		4. Binder驱动：运行在内核空间。驱动负责进程之间Binder通信的建立，Binder在进程之间的传递，Binder引用计数管理，数据包在进程之间的传递和交互等一系列底层支持。
		5. 四者的关系<br>
			Binder驱动程序提供设备文件/dev/binder与用户空间交互，Client、Server和Service Manager通过open和ioctl文件操作函数与Binder驱动程序进行通信。
	
	3. Binder通信流程
		1. Server创建了Binder实体，为其取一个字符形式，可读易记的名字。
		2. 将这个Binder连同名字以数据包的形式通过Binder驱动发送给ServiceManager，通知ServiceManager注册一个名字为XX的Binder，它位于Server中。
		3. 驱动为这个穿过进程边界的Binder创建位于内核中的实体结点以及ServiceManager对实体的引用，将名字以及新建的引用打包给ServiceManager。
		4. ServiceManager收数据包后，从中取出名字和引用填入一张查找表中。但是一个Server若向ServiceManager注册自己Binder就必须通过这个引用和ServiceManager的Binder通信。
		5. Server向ServiceManager注册了Binder实体及其名字后，Client就可以通过名字获得该Binder的引用了。Clent也利用保留的引用向ServiceManager请求访问某个Binder：我申请名字叫XX的Binder的引用。
		6. ServiceManager收到这个连接请求，从请求数据包里获得Binder的名字，在查找表里找到该名字对应的条目，从条目中取出Binder引用，将该引用作为回复发送给发起请求的Client。
		7. 使用服务的具体执行过程<br>
			![](https://github.com/yinfork/Android-Interview/blob/master/res/android/ipc/binder_step.png?raw=true)
			1. Client通过获得一个Server的代理接口，对Server进行调用。
			2. 代理接口中定义的方法与Server中定义的方法是一一对应的。
			3. Client调用某个代理接口中的方法时，代理接口的方法会将Client传递的参数打包成Parcel对象。
			4. 代理接口将Parcel发送给内核中的Binder Driver。
			5. Server会读取Binder Driver中的请求数据，如果是发送给自己的，解包Parcel对象，处理并将结果返回。
			6. 整个的调用过程是一个同步过程，在Server处理的时候，Client会Block住。因此Client调用过程不应在主线程。
		8. 此外还有匿名Binder<br>
			不是所有的Binder都需要注册给ServiceManager广而告之的。Server端可以通过已经建立的Binder连接将创建的Binder实体传给Client，当然这条已经建立的Binder连接必须是通过实名Binder实现。由于这个Binder没有向ServiceManager注册名字，所以是 匿名Binder。Client将会收到这个匿名Binder的引用，通过这个引用向位于Server中的实体发送请求。匿名Binder为通信双方建立一条私密通道，只要Server没有把匿名Binder发给别的进程，别的进程就无法通过穷举或猜测等任何方式获得该Binder的引用，向该Binder发送请求。
	
	4. Binder的数据拷贝<br>
		1. Binder数据拷贝只需要一次<br>
			Linux内核实际上没有从一个用户空间到另一个用户空间直接拷贝的函数，需要先用 copy_from_user() 拷贝到内核空间，再用 copy_to_user() 拷贝到另一个用户空间。为了实现用户空间到用户空间的拷贝，mmap()分配的内存除了映射进了接收方进程里，还映射进了内核空间。所以调用 copy_from_user() 将数据拷贝进内核空间也相当于拷贝进了接收方的用户空间，这就是Binder只需一次拷贝的"秘密"。
	
		2. 对比其他进程间通信方式<br>
			在移动设备上（性能受限制的设备，比如要省电），广泛地使用跨进程通信对通信机制的性能有严格的要求，Binder相对于传统的Socket方式，更加高效。Binder数据拷贝只需要一次，而管道、消息队列、Socket都需要2次，共享内存方式一次内存拷贝都不需要，但实现方式又比较复杂。
	
		3. Linux IPC原理<br>
			![](https://github.com/yinfork/Android-Interview/blob/master/res/android/ipc/linux_ipc.png?raw=true)<br>
			每个Android的进程，只能运行在自己进程所拥有的虚拟地址空间。例如，对应一个4GB的虚拟地址空间，其中3GB是用户空间，1GB是内核空间。当然内核空间的大小是可以通过参数配置调整的。对于用户空间，不同进程之间是不能共享的，而内核空间却是可共享的。Client进程向Server进程通信，恰恰是利用进程间可共享的内核内存空间来完成底层通信工作的。Client端与Server端进程往往采用ioctl等方法与内核空间的驱动进行交互。
	
	5. Binder的优势
		1. 性能方面：Binder相对于传统的Socket方式，更加高效。Binder数据拷贝只需要一次
		2. 安全方面：传统的进程通信方式对于通信双方的身份并没有做出严格的验证，比如Socket通信的IP地址是客户端手动填入，很容易进行伪造。然而，Binder机制从协议本身就支持对通信双方做身份校检，从而大大提升了安全性。
		
	10. 参考
		1. https://github.com/LRH1993/android_interview/blob/master/android/advance/binder.md
		2. https://github.com/hadyang/interview/blob/master/android/binder.md 	
	
3. Android线程间消息分发机制
	1. 定义<br>
		一套 Android 消息传递机制
	2. 构成<br>
		Android中的异步消息处理主要由四个部分组成，Message、Handler、MessageQueue和Looper。
		1. Message<br>
			Message是在线程之间传递的消息，它可以在内部携带少量的信息，用于在不同线程之间交换数据。上一小节我们使用到了Message的what字段，初次之外还可以使用arg1和arg2字段来携带一些整型数据，使用obj字段携带一个Object对象。
		2. Handler<br>
			Handler顾名思义也就是处理者的意思，它主要是用于发送和处理消息的。发送消息一般是使用Handler的sendMessage()方法，而发出的消息经过一系列地辗转处理后，最终会传递到Handler的handleMessage()方法中。
		3. MessageQueue<br>
			MessageQueue是消息队列的意思，它主要是用于存放所有的Handler发送的消息。这部分消息会一直存在于消息队列中，等待被处理。每个线程中只有一个MessageQueue对象。
		4. Looper<br>
			Looper是每个线程中的MessageQueue的管家，调用Looper的loop()方法后，就会进入到一个无限循环当中，然后每当发现MessageQueue中存在一条消息，就会将它取出，并传递到Handler的handleMessage()方法中。每个线程中也只会有一个Looper对象。

	3. 工作过程	
		1. 首先需要在主线程当中创建一个Handler对象，并重写handleMessage()方法。
		2. 然后当子线程中需要进行UI操作时，就创建一个Message对象，并通过Handler将这条信息发送出去。
		3. 之后这条消息会被添加到MessageQueue的队列中等待被处理，
		4. 而Looper则会一直尝试从MessageQueue中取出待处理消息，最后分发回Handler的handleMessage()方法中。由于Handler是在主线程中创建的，所以此时handleMessage()方法中的代码也会在主线程运行，于是我们在这里就可以安心地进行UI操作了。
	
	4. 使用情景
		1. 在将来的某个时间点调度处理消息和runnable对象；
		2. 将需要执行的操作放到其他线程之中，而不是自己的。比如：将工作线程需操作UI的消息传递到主线程，使得主线程可根据工作线程的需求 更新UI，从而避免线程操作不安全的问题

	5. 参考
		1. https://www.jianshu.com/p/e5b89601562a
		
----

<span id = "performance_optimization"></span>
#### 性能优化 [(TOP)](#home)
1. 布局优化<br>
	关于布局优化的思想很简单，就是尽量减少布局文件的层级。这个道理很浅显，布局中的层级少了，就意味着Android绘制时的工作量少了，那么程序的性能自然就提高了。
	1. 删除布局中无用的控件和层次
	
	2. 有选择地使用性能比较低的ViewGroup<br>
		例如：如果布局中既可以使用LinearLayout也可以使用RelativeLayout，那么就采用LinearLayout，这是因为RelativeLayout的功能比较复杂，它的布局过程需要花费更多的CPU时间。FrameLayout和LinearLayout一样都是一种简单高效的ViewGroup，因此可以考虑使用它们，但是很多时候单纯通过一个LinearLayout或者FrameLayout无法实现产品效果，需要通过嵌套的方式来完成。这种情况下还是建议采用RelativeLayout,因为ViewGroup的嵌套就相当于增加了布局的层级，同样会降低程序的性能。
	
	3. 采用ViewStub<br>
		ViewStub提供了按需加载的功能，当需要时才会将ViewStub中的布局加载到内存，提高了程序初始化效率。
	
	4. 避免多度绘制<br>
		过度绘制（Overdraw）描述的是屏幕上的某个像素在同一帧的时间内被绘制了多次。在多层次重叠的 UI 结构里面，如果不可见的 UI 也在做绘制的操作，会导致某些像素区域被绘制了多次，同时也会浪费大量的 CPU 以及 GPU 资源。


2. 绘制优化<br>
	绘制优化是指**View的onDraw方法要避免执行大量的操作**
	1. onDraw中不要创建新的局部对象<br>
		因为onDraw方法可能会被频繁调用，这样就会在一瞬间产生大量的临时对象，这不仅占用了过多的内存而且还会导致系统更加频繁gc，降低了程序的执行效率。
	
	2. onDraw方法中不要做耗时的任务，也不能执行成千上万次的循环操作，尽管每次循环都很轻量级，但是大量的循环仍然十分抢占CPU的时间片，这会造成View的绘制过程不流畅。<br>
		按照Google官方给出的性能优化典范中的标准，View的绘制频率保证60fps是最佳的，这就要求每帧绘制时间不超过16ms(16ms = 1000/60)，虽然程序很难保证16ms这个时间，但是尽量降低onDraw方法中的复杂度总是切实有效的。

3. 内存泄漏优化
	1. 什么是内存泄漏<br>
		Java垃圾回收器会帮我们干掉大部分无用的内存空间，但是对于还保持着引用，但逻辑上已经不会再用到的对象，垃圾回收器不会回收它们。这些对象积累在内存中，直到程序结束，就是我们所说的“内存泄漏”。 
	
	2. 内存泄漏导致的问题<br>
		用户对单次的内存泄漏并没有什么感知，但当泄漏积累到内存都被消耗完，就会导致卡顿(由于频繁GC)，崩溃(OOM)。

	3. 优化方式
		1. 在开发过程中避免写出有内存泄漏的代码
		2. 通过一些分析工具比如MAT来找出潜在的内存泄露，然后解决。

	4. Android常见出现内存泄漏的情况
		1. 集合类泄漏 
		2. 单例/静态变量造成的内存泄漏 
		3. 匿名内部类/非静态内部类 
		4. 资源未关闭造成的内存泄漏

	5. Android分析内存泄漏的方法
		1. 假如已经发生内存泄漏，可以通过MAT(Memory Analyzer Tool)分析现场
		2. 日常可以给APK加上LeakCanary来检测Android中的内存泄漏

4. 响应速度优化<br>
	响应速度优化的核心思想就是避免在主线程中做耗时操作
	
	1. 解决方法
		1. 如果有耗时操作，可以开启子线程执行，即采用异步的方式来执行耗时操作。
		2. 非核心操作延后加载，保证核心功能优先处理
	
	2. 分析工具
		1. TraceView
		2. SysTrace
		3. 开发者模式下一个工具：Profile GPU Rendering，看滑动到哪里有红色的柱子高了
	
	3. ANR
		1. 出现ANR的情况<br>
			Android规定，Activity如果5秒钟之内无法响应屏幕触摸事件或者键盘输入事件就会出现ANR，而BroadcastReceiver如果10秒钟之内还未执行完操作也会出现ANR。
		2. ANR日志分析<br>
			当一个进程发生了ANR之后，系统会在/data/anr目录下创建一个文件traces.txt，通过分析这个文件就能定位出ANR的原因

	4. APP启动速度优化常见方法
		1. 优化UI布局，尽量简单，或者采用ViewStub
		2. 优化启动时的加载逻辑：分部加载、异步加载、延期加载等
		3. 数据准备：提前准备好数据或者开子线程准备数据

5. Bitmap优化<br>
	避免加载图片多大导致OOM出现
	1. 图片进行裁剪压缩
	2. 低端机加载565的图片，高端机才加载888的图片

6. 线程优化<br>
	线程优化的思想就是**采用线程池，避免程序中存在大量的Thread**。线程池可以重用内部的线程，从而避免了线程的创建和销毁锁带来的性能开销，同时线程池还能有效地控制线程池的最大并法术，避免大量的线程因互相抢占系统资源从而导致阻塞现象的发生。因此在实际开发中，尽量采用线程池，而不是每次都要创建一个Thread对象。


7. 稳定性优化<br>
	Android 应用的稳定性定义很宽泛，影响稳定性的原因很多，比如内存使用不合理、代码异常场景考虑不周全、代码逻辑不合理等，都会对应用的稳定性造成影响。其中最常见的两个场景是：Crash 和 ANR，这两个错误将会使得程序无法使用，比较常用的解决方式如下：
	1. 提高代码质量。比如开发期间的代码审核，看些代码设计逻辑，业务合理性等。
	2. 代码静态扫描工具。常见工具有Android Lint、Findbugs、Checkstyle、PMD等等。
	3. Crash监控。把一些崩溃的信息，异常信息及时地记录下来，以便后续分析解决。
	4. Crash上传机制。在Crash后，尽量先保存日志到本地，然后等下一次网络正常时再上传日志信息。

8. 安装包大小优化
	1. 代码混淆。使用proGuard 代码混淆器工具，它包括压缩、优化、混淆等功能。
	2. 资源优化。比如使用 Android Lint 删除冗余资源，资源文件最少化等。
	3. 图片优化。比如利用 AAPT 工具对 PNG 格式的图片做压缩处理，降低图片色彩位数等。
	4. 避免重复功能的库，使用 WebP图片格式等。
	5. 插件化。比如功能模块放在服务器上，按需下载，可以减少安装包大小。


9. 耗电优化 
	1. 分析工具
		Battery Historian 是一款由 Google 提供的 Android 系统电量分析工具，和Systrace 一样，是一款图形化数据分析工具，直观地展示出手机的电量消耗过程，通过输入电量分析文件，显示消耗情况，最后提供一些可供参考电量优化的方法。

	2. 常用优化方案
		1. 计算优化，避开浮点运算等。
		2. 避免 WaleLock 使用不当。
		3. 使用 Job Scheduler。

10. 参考
	1. https://github.com/LRH1993/android_interview/blob/master/android/advance/performance-optimization.md 
	2. https://www.cnblogs.com/cr330326/p/8011523.html

----		

<span id = "android_storage"></span>
#### Android存储 [(TOP)](#home)
1. SharedPreferences
	SharedPreferences读写是否线程安全

2. Parcelable和Serializable的作用、效率、区别及选择。
	1. 作用
		1. Serializable的作用是为了保存对象的属性到本地文件、数据库、网络流、rmi以方便数据传输，当然这种传输可以是程序内的也可以是两个程序间的。
		2. Parcelable的设计初衷是因为Serializable效率过慢，为了在程序内不同组件间以及不同Android程序间(AIDL)高效的传输数据而设计，这些数据仅在内存中存在，Parcelable是通过IBinder通信的消息的载体。
		
	2. 效率及选择
		1. Parcelable的性能比Serializable好，在内存开销方面较小，所以在内存间数据传输时推荐使用Parcelable，如activity间传输数据
		2. Serializable可将数据持久化方便保存，所以在需要保存或网络传输数据时选择Serializable，因为android不同版本Parcelable可能不同，所以不推荐使用Parcelable进行数据持久化。

10. 参考
	1. https://github.com/LRH1993/android_interview/blob/master/android/advance/serializable.md

----

<span id = "android_image"></span>
#### 图片加载 [(TOP)](#home)
1. Bitmap.Config 有四种枚举类型。
	1. ARGB_8888<br>
		ARGB 四个通道的值都是 8 位，加起来 32 位，也就是 4 个字节。每个像素点占用 4 个字节的大小。
	
	2. ARGB_4444<br>
		ARGB 四个通道的值都是 4 位，加起来 16 位，也就是 2 个字节。每个像素点占用 2 个字节的大小。
	
	3. RGB_565<br>
		RGB 三个通道分别是 5 位、6 位、5 位，没有 A 通道，加起来 16 位，也就是 2 个字节。每个像素点占用 2 个字节的大小。
	
	4. ALPHA_8<br>
		只有 A 通道，占 8 位，1 个字节。每个像素点占用 1 个字节的大小。
	
	5. 计算
		假设图片是ARGB_8888，512*384像素。需要占用0.75M

2. 高效加载大图片
	1. 背景：防止OOM或卡顿（频繁GC） 
	2. 加载步骤
		1. 先测量图片的宽高<br>
			使用BitmapFactory.Options参数，将这个参数的inJustDecodeBounds属性设置为true，就可以让解析方法禁止为bitmap分配内存，返回的BitmapFactory.Options的outWidth、outHeight和outMimeType属性都会被赋值。<br>
			这个技巧让我们可以在加载图片之前就获取到图片的长宽值和MIME类型，从而根据情况对图片进行压缩<br>
			假如直接用BitmapFactory.decodeXXX()加载图片，是直接加载原图，很容易OOM。
			
			```
			BitmapFactory.Options options = new BitmapFactory.Options();  
			options.inJustDecodeBounds = true;  
			BitmapFactory.decodeResource(getResources(), R.id.myimage, options);  
			int imageHeight = options.outHeight;  
			int imageWidth = options.outWidth;  
			String imageType = options.outMimeType;  
			```
			
		2. 根据实际展示的View的大小，计算出合适的采样率从而实现压缩<br>			通过设置BitmapFactory.Options中inSampleSize的值就可以实现。<br>
			比如我们有一张2048*1536像素的图片，将inSampleSize的值设置为4，就可以把这张图片压缩成512*384像素。原本加载这张图片需要占用13M的内存，压缩后就只需要占用0.75M了(假设图片是ARGB_8888类型，即每个像素点占用4个字节)。下面的方法可以根据传入的宽和高<br>
			下面的方法可以根据传入的宽和高，计算出合适的inSampleSize值：
			
			```
			public static int calculateInSampleSize(BitmapFactory.Options options,  
        int reqWidth, int reqHeight) {  
			    // 源图片的高度和宽度  
			    final int height = options.outHeight;  
			    final int width = options.outWidth;  
			    int inSampleSize = 1;  
			    if (height > reqHeight || width > reqWidth) {  
			        // 计算出实际宽高和目标宽高的比率  
			        final int heightRatio = Math.round((float) height / (float) reqHeight);  
			        final int widthRatio = Math.round((float) width / (float) reqWidth);  
			        // 选择宽和高中最小的比率作为inSampleSize的值，这样可以保证最终图片的宽和高  
			        // 一定都会大于等于目标的宽和高。  
			        inSampleSize = heightRatio < widthRatio ? heightRatio : widthRatio;  
			    }  
			    return inSampleSize;  
			} 
			```

3. 非压缩下加载大图
	1. 使用场景<br>
		假如我们现在在做一个图片显示工具，需要我们去加载一张长图超级大的图（将近10M），又不允许我们去压缩处理。

	2. 使用BitmapRegionDecoder<br>
		BitmapRegionDecoder主要用于显示图片的某一块矩形区域，如果你需要显示某个图片的指定区域，那么这个类非常合适。<br>
		所以针对这种超大图片，我们可以通过一点点显示图片的某一块区域，然后通过手机滑动图片一点点查看即可，那么至少只需要一个方法去设置图片；一个方法传入显示的区域即可；


4. 图片缓存技术			
	1. 简介<br>
		为了节省用户流量，提高图片加载效率，我们通常使用图片三级缓存策略，即通过网络、本地、内存三级缓存图片，来减少不必要的网络交互，避免浪费流量。 

	2. 原理<br>
		以url为key，先从内存中寻找，再从本地文件中寻找，最后再请求网络。

	3. 内存缓存
		1. 实现类<br>
			内存缓存最核心的类是LruCache (此类在android-support-v4的包中提供) 。这个类非常适合用来缓存图片，它的主要算法原理是把最近使用的对象用强引用存储在 LinkedHashMap 中，并且把最近最少使用的对象在缓存值达到预设定值之前从内存中移除。
		2. 弱饮用和软引用被放弃<br>
			在过去，我们经常会使用一种非常流行的内存缓存技术的实现，即软引用或弱引用 (SoftReference or WeakReference)。但是现在已经不再推荐使用这种方式了，因为从 Android 2.3 (API Level 9)开始，垃圾回收器会更倾向于回收持有软引用或弱引用的对象，这让软引用和弱引用变得不再可靠。
		
		3. 确定合适缓存大小的因素
			1. 你的设备可以为每个应用程序分配多大的内存？
			2. 设备屏幕上一次最多能显示多少张图片？有多少图片需要进行预加载，因为有可能很快也会显示在屏幕上？
			3. 你的设备的屏幕大小和分辨率分别是多少？一个超高分辨率的设备（例如 Galaxy Nexus) 比起一个较低分辨率的设备（例如 Nexus S），在持有相同数量图片的时候，需要更大的缓存空间。
			4. 图片的尺寸和大小，还有每张图片会占据多少内存空间。
			5. 图片被访问的频率有多高？会不会有一些图片的访问频率比其它图片要高？如果有的话，你也许应该让一些图片常驻在内存当中，或者使用多个LruCache 对象来区分不同组的图片。
			6. 你能维持好数量和质量之间的平衡吗？有些时候，存储多个低像素的图片，而在后台去开线程加载高像素的图片会更加的有效。

5. 图片的开源框架学习<br>
	TODO
	
10. 参考
	1. https://www.jianshu.com/p/da754f9fad51
	2. https://www.jianshu.com/p/80b2068a90a8 