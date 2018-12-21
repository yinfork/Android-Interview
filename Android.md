## Android
<span id = "home"></span>
#### 目录
1. [Activity](#android_activity)
2. [Service](#android_service)
3. [View](#android_view)


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

5. 几种Layout的性能比较

