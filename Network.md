## 计算机网络
<span id = "home"></span>
#### 目录
1. [协议体系模型](#network_arch)
2. [物理层](#network_physical_layer)
3. [数据链路层](#network_data_link_layer)
4. [网络层](#network_network_layer)
5. [运输层](#network_transport_layer)
6. [应用层](#network_application_layer)

----

<span id = "network_arch"></span>
#### 协议体系模型 [(TOP)](#home)
![](https://github.com/yinfork/Android-Interview/blob/master/res/network/network_arch.png?raw=true)

----

<span id = "network_physical_layer"></span>
#### 物理层 [(TOP)](#home)
1. 定义<br>
物理层(physical layer)的作用是实现相邻计算机节点之间比特流的透明传送，尽可能屏蔽掉具体传输介质和物理设备的差异。<br>
	使其上面的数据链路层不必考虑网络的具体传输介质是什么。“透明传送比特流”表示经实际电路传送后的比特流没有发生变化，对传送的比特流来说，这个电路好像是看不见的。<br>
	在物理层上所传送的数据单位是比特。

2. 传输方向的工作方式
	1. 单工：只能有一个方向的通信而没有反方向的交互。
	2. 半双工：通信的双方都可以发送信息，但不能双方同时发送(当然也就不能同时接收)。
	3. 全双工：通信的双方可以同时发送和接收信息。

3. 信号
	1. 基带信号<br>
		来自信源的信号。指没有经过调制的数字信号或模拟信号。
		1. 曼彻斯特编码信号<br>
			每个原始信号码元固定用两个连续极性相反的脉冲来表示,1码用正、负电平表示(也就是先正后负,对应的编码为10)，0码用负、正电平表示(也就是先负后正,对应的编码为01)。其关键特点就是在每个码元的1/2码元位置必须进行极性跳变
			1. 优点: **每一位的中间有一跳变，位中间的跳变既作时钟信号，又作数据信号，所以具有自同步能力和良好的抗干扰性能。**
			2. 缺点: 每一个码元都被调成两个电平，所以编码效率50%
			
		2. 差分曼彻斯特编码<br>
			编码规则：对于二进制信号中的第一个码元与曼彻斯特的转换方式一样，都是在1/2码元时进行电平极性跳变,即把0码转换成01码,把1码转换成10码；后面的信号码就根据以下两条规则跳变：如果本码为1,开始处的电平不跳变；如果本码为0,开始处的电平必须跳变。<br>
			**相比曼彻斯特编码，可以解决当极性反转时引起的译码错误。**
			1. 优点: 收发双方可以根据编码自带的时钟信号来保持同步，无需专门传递同步信号的线路
			2. 缺点: 每一个码元都被调成两个电平，所以编码效率50%

		3. 不归零码<br>
			信号电平由0、1表示，并且在表示完一个码元后，电压不需回到0	。常见有CMI码
			
		4. 例子
			![](https://github.com/yinfork/Android-Interview/blob/master/res/network/manchester_encoding.jpg?raw=true)		
			
	2. 宽带信号<br>
		指将基带信号调制后形成频分复用模拟信号。

4. 物理层之下的传输媒体
	1. 双绞线
	2. 同轴电缆
	3. 光缆		

5. 信道复用技术<br>
	通信线路的架设费用较高，需要尽可能地充分使用每个信道的容量，尽可能不重复建设通信线路
	1. 频分复用FDM<br>
		ADSL技术采用频分复用技术把普通的电话线分成了电话、上行和下行三个相对独立的信道，从而避免了相互之间的干扰。用户可以边打电话边上网，不用担心上网速率和通话质量下降的情况。
	2. 时分复用TDM
	3. 码分多址CDMA：每个用户可以在同样的时间使用同样的频带进行通信。

6. 调制器和解调器
	1. 调制器：基带数字信号波形转为模拟信号的波形
	2. 解调器：将经过调制器变换的模拟信号恢复成原来的数字信号

7. 常用单位
	1. 带宽（bandwidth）<br>
		在计算机网络中，表示在单位时间内从网络中的某一点到另一点所能通过的“最高数据率”。常用来表示网络的通信线路所能传送数据的能力。单位是“比特每秒”，记为b/s。
	2. 码元（code）<br>
		在使用时间域（或简称为时域）的波形来表示数字信号时，代表不同离散数值的基本波形。
	3. 信噪比（signal-to-noise ratio ）<br>
		指信号的平均功率和噪声的平均功率之比，记为S/N。信噪比（dB）=10*log10（S/N）
	4. 比特率（bit rate ）
		单位时间（每秒）内传送的比特数。

8. 定理
	1. 奈氏准则<br>
		理想低通信道最高码元传输速率 = 2W Baud（即每赫兹理想低通信道最高码元传输速率每秒2个码元）		
	2. 香农定理<br>
		信道的极限传输速率C = Wlog2(1+S/N)，W为带宽，S为平均功率，N为高斯噪声功率，S/N也称为信噪比
	3. 采样定理<br>	
		只要采样频率不低于电话信号最高频率的2倍，就可以从采样脉冲信号无失真地恢复出原来的电话信号。

9. 参考
	1. 	https://zhuanlan.zhihu.com/p/52248268
	2. https://github.com/Snailclimb/JavaGuide/blob/master/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C%E4%B8%8E%E6%95%B0%E6%8D%AE%E9%80%9A%E4%BF%A1/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C.md#%E5%85%AB-http%E9%95%BF%E8%BF%9E%E6%8E%A5%E3%80%81%E7%9F%AD%E8%BF%9E%E6%8E%A5
	3. https://github.com/it-interview/EasyJob/blob/master/Network/network.md
	4. https://github.com/Snailclimb/JavaGuide/blob/master/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C%E4%B8%8E%E6%95%B0%E6%8D%AE%E9%80%9A%E4%BF%A1/%E5%B9%B2%E8%B4%A7%EF%BC%9A%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C%E7%9F%A5%E8%AF%86%E6%80%BB%E7%BB%93.md

----

<span id = "network_data_link_layer"></span>
#### 数据链路层 [(TOP)](#home)
TODO

----

<span id = "network_network_layer"></span>
#### 网络层 [(TOP)](#home)
TODO

----

<span id = "network_transport_layer"></span>
#### 运输层 [(TOP)](#home)
TODO

----

<span id = "network_application_layer"></span>
#### 应用层 [(TOP)](#home)
1. 定义<br>
	通过应用进程间的交互来完成特定网络应用。应用层协议定义的是应用进程（进程：主机中正在运行的程序）间的通信和交互的规则。

2. 常见协议
	1. Http
	2. DNS 

3. Http协议详解



