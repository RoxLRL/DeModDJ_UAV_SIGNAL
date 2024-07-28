# 无人机图传信号解调
 这是描述无人机信号破解技术的一个仓库，包含无人机信号的介绍、解调无人机信号的全流程，因为某些原因代码和数据不能公开。
 
 ![ff](https://github.com/user-attachments/assets/72a72982-8961-495d-bf1b-25cc4375f186)

## 功能描述：
- 读取无人机图传信号数据。
- 进行 OFDM 信号检测。
- 估计有效符号时间、带宽、载频等参数。
- 下变频处理。
- 估计循环前缀长度。
- 重采样。
- 搜索导频序列。
- 进行频偏估计和信道估计。
- 显示 OFDM 信号的 I/Q 图。

## 输入/输出
输入： 无人机图传信号的二进制文件。\
输出： 信号处理结果，包括符号时间、带宽、载频、循环前缀长度、频偏估计、信道估计等。

## 详细流程
- 初始化参数
- 清除环境并设置采样频率 fs。
- 设置是否绘制图形的标识 Is_plot。
- 读取信号数据

- 从指定路径读取二进制文件，获取一通道数据。
- OFDM 信号检测

- 归一化信号。
- 基于延迟相关的有效符号时间估计

![f1](https://github.com/user-attachments/assets/dac96b9d-3be6-4dd8-8f78-cc80091c72f9)
  
- 基于功率谱的带宽粗估计、载频粗估计

![f2](https://github.com/user-attachments/assets/1e67dc16-849c-40ff-bd5c-62d98344763b)
![f3](https://github.com/user-attachments/assets/9ad22fcd-d078-40d3-a0e6-9e6a8b894f7c)

- FFT长度和发送端采样率估计

![f4](https://github.com/user-attachments/assets/66ddb245-37c9-4cba-b6b3-9964f4cdb22a)
![f5](https://github.com/user-attachments/assets/d483a830-d141-415a-9bc5-1977381c3630)

- 下变频处理

- 循环前缀列表估计
- 估计短 CP 长度 low_cp 和长 CP 长度 long_cp。
 
 ![f6](https://github.com/user-attachments/assets/0838d3b2-59d9-4e58-9e03-017c69153bd7)

- 开始位置估计

 ![f7](https://github.com/user-attachments/assets/d9f717ed-1b9b-4436-8339-ecbdb48a071b)

- 重采样
 
-  搜索导频序列。

 ![f8](https://github.com/user-attachments/assets/dfd97f8e-37bf-4b95-9740-53defdd4bc31)

- 导频位置搜索

![f9](https://github.com/user-attachments/assets/2c231057-a2a7-47b6-85ac-94a7be25122d)

- 基于循环前缀的小数倍频偏修正、去循环前缀
- 整数倍频偏估计

![f10](https://github.com/user-attachments/assets/071b4793-aa0a-4eeb-8029-5b0fab23c3fe)

- 进行信道估计，计算信道矩阵 H_est。

![f11](https://github.com/user-attachments/assets/471ee8ae-db0a-4090-9def-6caddffad9e5)

- 绘制 OFDM 信号的 I/Q 图

![f22](https://github.com/user-attachments/assets/86c7b51b-f538-461a-810a-c909cf72b062)

# 无人机及其通信链路
## 无人机
无人驾驶飞机简称“无人机”，英文缩写为“UAV”，是利用无线电遥控设备和自备的程序控制装置操纵的不载人飞机，或者由车载计算机完全地或间歇地自主地操作。
近年来无人机市场井喷式发展，各类无人机走进了人们的生活，但又缺乏监管，因此针对非合作无人机信号的研究越发重要。
## 无人机通信链路
无人机外部通信链路，由遥控器端、无人机端、手机、卫星、无线/有线信道等构成。

![图片1](https://github.com/user-attachments/assets/99f7245e-2f35-4e92-8241-769c0a2de044)

目前已知有以下几种通信模式：
1）	无人机端和遥控器端通过自定义协议进行通信，有遥控信号和图传信号。遥控器通过遥控信号单方面向无人机发送飞行控制信息，为上行链路，无人机通过图传信号单方面向遥控器发送视频信息、飞行数据等，为下行链路。因为是自定义协议、且单向传输不需要进行握手，所以重连更快、时延更小、控制距离更远。
2）	遥控器连接无人机WIFI热点来通信，即通过使用WIFI协议通信。通信距离更短，且双方通信需要建立连接、数据包确认，时延较长、重连速度也慢。
3）	手机通过连接无人机WIFI热点进行通信。
4）	手机通过有线连接遥控器进行通信，遥控器再通过模式一或二中的通信方式和无人机通信。
5）	GPS卫星、北斗卫星等单方面向无人机、手机或遥控器发送GPS信号；无人机、手机或遥控器通过GPS信号计算出自己的位置信息，再通过前面的几种模式进行通信、或根据本地记录自动返航。
6）	无人机与遥控器通过中继卫星通信（GPS，北斗，地空一体数据链），实现超远距离通信、自动返航。
7）	无人机接入移动网络（4G、5G）通信。

# 图传信号及其盲解调设计
## 图传信号简述
   无人机信号主要有遥控信号和图传信号，其中无人机图传信号是用于无人机与遥控器进行视频传输的通信信号。对于图传信号，通常采用正交频分复用（Orthogonal Frequency Division Multiplexing,OFDM）技术来实现，其带宽为 10MHz、20MHz左右，频段范围与飞控信号一致，发送频率一般在2.4GHz、5.8GHz左右，可自适应变换中心频。还有一些无人机采用WIFI协议，进行通信。
   
![图2](https://github.com/user-attachments/assets/bc53d363-a476-4110-94ac-2085d036bbe7)

##  图传信号通信协议
经实验发现图传信号很多特性与LTE类似，其通信协议或许参考了LTE的部分内容。
###  LTE物理层协议
LTE采用OFDM技术，不仅可以灵活的分配时频资源还可以方便的支持多种物理带宽。LTE标准规定了六种带宽：从1.4MHz到20MHz，下表给出了LTE各种模式的标准规定。在LTE系统中带宽的分配是以RB为单位的，1个RB占用的带宽是180KHz，每个RB共有12个子载波，每个子载波宽度为15KHz。五种带宽模式的RB数从6到100不等，或许你会疑惑180K*6=1.08MHz不是1.4MHz，这是因为每一种带宽还需要和旁边的频带之间有保护带宽，从而避免干扰。从下表可以看到，3M-20M带宽系统的频谱利用率大约90%左右，而不是100%。1.4MHz带宽的频谱利用率仅仅有77%左右。

LTE标准\
信道带宽（MHz）	1.4	3	5	10	15	20\
帧长（ms）	10	10	10	10	10	10\
子载波间距（KHz）	15	15	15	15	15	15\
采样频率（MHz）	1.92	3.84	7.68	15.36	23.04	30.72\
FFT长度	128	256	512	1024	1536	2048\
占用子载波（包括DC）	73	181	301	601	901	1201\
保护子载波	55	75	211	423	635	847\
资源块（12个子载波）数	6	15	25	50	75	100\
占用信道带宽（MHz）	1.095	2.715	4.515	9.015	13.515	18.015\
副帧OFDM符号（短CP）	7	7	7	7	7	7\
CP长度（us）	第一个符号5.2，其他符号4.7

LTE的下行方向有一个未使用的子载波，即DC子载波（DC-subcarrier）。对于BS发射机，由于中频（如采用一次变频方案）或射频（如采用零中频方案）本振的泄漏，会在最终发射的信号中间（载频处）产生一个较大的噪声。如果发射时在DC调制了数据符号，则该数据符号的发射EVM会很差，信噪比通常是负的若干dB，因此LTE协议规定这个DC上是不发射任何数据符号。这或许也是图传信号DC子载波固定的原因。\
在LTE系统中，有两种上行参考信号：1）DMRS，2）Sounding Reference Signal(SRS)。这两种信号都基于ZC序列。ZC序列同样也用于生成下行链路的主同步信号（PSS）和上行前导符号（Preamble）（用于随机接入）。不同UE的参考信号通过对基序列进行循环移动生成。基站使用上行的DMRS信号解调PUCCH和PUSCH。在PUSCH信号中，当使用正常的循环前缀时，DMRS信号位于第四个OFDM符号中，每隔0.5ms重复一次，并延伸到整个RB。在PUCCH中，DMRS信号的位置依赖于控制信道式。基站使用SRS信号估计不同频率的上行信道响应。SRS信号也用于估计它。\
LTE标准帧结构：

![图片3](https://github.com/user-attachments/assets/5cafa9af-e930-437b-bebe-6a344e5f0fe3)

###   无人机图传信号通信协议
最新的大疆无人机支持不同的无线协议，如蓝牙和WiFi。例如，使用DJI Fly 应用程序将无人机相机拍摄的照片传输到智能手机。WiFi连接也可用于命令与控制(C2)链接以通过智能手机控制无人机。目前较新的大疆无人机上，此通信链路已被专有的OcuSync无线电协议所取代，该协议的性能更好且不易受到干扰。
DJI OcuSync2.0无人机使用的是一种基于OFDM的“自定义”通信方案，且是“SDR”。软件定义无线电(SoftwareDefined Radio，SDR)是一种无线电通信技术，它基于软件定义的无线通信协议而非通过硬连线实现。频带、空中接口协议和功能可通过软件下载和更新来升级，而不用完全更换硬件。其通信数据包可以是下行链路（无人机到RC即图传信号）、上行链路（RC到无人机）和广播（远程无人机 ID，又名航班信息）。它们都使用OFDM调制，但使用不同的带宽，并且上行链路使用跳频。每个符号要么是参考符号，要么是数据符号。参考符号，一般是ZC序列，在频域中生成，序列长度为1201（20MHz）或601（10MHz）。数据符号，一般为QPSK或更高阶的QAM。直流子载波是固定的，不使用以获取数据。 它不是基于WiFi或其他现成的通信系统，但是与LTE有一些类似的属性，如带宽、载波间隔等一些参数。如无人机信号的采样率为15.36e6的倍数，子载波间隔为15KHz，FFT长度特性 ，这些都与LTE一样。

## 实采图传信号分析
使用CoolEdit观察0号无人机信号，时域波形如下：

![图片4](https://github.com/user-attachments/assets/fed0fd7d-db3b-498d-a326-9d470c6943a9)

时频图：

 ![图片5](https://github.com/user-attachments/assets/2786c030-3500-4c07-9bdf-20f105bdec41)

突发信号形式有下面的种类：
 
 ![图片6](https://github.com/user-attachments/assets/2d412921-92f2-459b-ba65-51b60958b2a9)
![图片8](https://github.com/user-attachments/assets/99377085-9a7a-4957-ab4a-dabb8a9a02da)
![图片9](https://github.com/user-attachments/assets/b9134b7f-705e-47e4-96fc-a477b77f6b8d)

OFDM符号，结构类似下面插入导频的结构：

 ![图片7](https://github.com/user-attachments/assets/404e0046-96c2-400b-a550-746b85daa2e6)

或者是ZC序列（训练序列、参考符号），可以用来同步、信道估计等。

条纹符号加上它前面3个符号和后面3个一段，时间长度共为0.5ms，为一个时隙，一共是七个OFDM符号的长度。
观察条纹段数据的时域波形，看起来很像是ZC序列，如下：

 ![图片10](https://github.com/user-attachments/assets/2fd08d9f-f8f9-4128-bd4d-c6ddea2ee5a7)

在LTE系统中，PSS、SSS、cellRS、DMRS、SRS、PRACH、PUCCH等物理层信号，基本上都涉及到了ZC（Zadoff –Chu）序列信号。它有着鲜明的特点，振幅固定，良好的相关特性，并且FFT之后仍然是ZC序列，其星座图如下：

 ![图片11](https://github.com/user-attachments/assets/2064503a-8048-4037-bf7f-c8fc17129478)

##  图传信号盲解调设计

 ![图12](https://github.com/user-attachments/assets/3745a0e8-a212-41d0-b92e-b19bfd62cc1a)
