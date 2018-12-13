## Android
<span id = "home"></span>
#### 目录
1. [Activity介绍](#android_activity)

----


<span id = "android_activity"></span>
#### Activity介绍 [(TOP)](#home)
1. 四种启动模式
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

2. 生命周期
	1. Activity生命周期的7个方法
		2. onCreate()：当Activity第一次被实例化的时候系统会调用，整个生命周期只调用1次这个方法。。
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
		
			
3. 被系统回收时的Activity	
	1. 生命周期<br>
		不会走onNewIntent()，走onCreate()------>onStart()------>onRestoreInstanceState()----->onResume()<br>
		此时onCreate(Bundle savedInstanceState) 中的 savedInstanceState是 savedInstanceState()时保存的Bundle。而第一次启动Activity的onCreate(), savedInstanceState会为null。
	2. activity恢复原则<br>
	如果多个activity在栈中，多个activity会被一起恢复吗？
不会的，只会恢复栈顶的activity，但是栈是恢复了的。在按backspace之后，会创建下一个activity实例,
