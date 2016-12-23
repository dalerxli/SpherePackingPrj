# SpherePacking Project

## 开发环境
> Windows 7

> VS2013 WinForm

> Emgucv

> Activiz.Net

> json.net

> log4net

## V1.0

### 简介
* 用DEM(Discrete Element Method)的方法对三维空间的小球进行随机堆积。在DEM每一步的迭代中，根据小球上一时刻的速度计算小球的位置，并对位置进行边界的限幅；利用上一时刻的加速度计算小球当前的速度，并计算小球计算小球受到的力，计算下一时刻需要用到的加速度。
* 项目支持两种不同的边界条件：立方体边界和圆柱体边界
* 半径是随机产生的，均匀半径是0.5，即边界的尺寸会随着需要堆积的小球的个数变大
### 功能
* 可以在不同的边界条件下模拟小球的运动情况
* 可以将实时运动的情况以图像的形式保存在文件夹中
* 可以以txt形式保存实时的加速度、速度、位置以及小球的半径等
* 对于界面来说，窗体中的控件可以随界面大小的改变而改变；当小于初始大小时，控件大小恢复为初始状态，并加上scrollbar。
* 可以实时对小球的位置和半径进行实时保存，并可以从文件中导入小球的位置和半径，即可以在上一次的仿真的基础上继续仿真
* 小球的属性值是通过json文件进行导入与导出
* 在迭代的过程中，可以生成日志文件，记录当前的状态信息。
* 可以生成基于三维图像的切片序列，即用于三维重建。

### 具体介绍
#### 立方体边界
* 设小球的边界为N_BASE，则在这里产生的小球的个数为`3*N_BASE*N_BASE*N_BASE`.
* 划分立方体网格，在网格点的周围随机产生球心的位置，半径按照初始方法随机产生。
* 小球和边界碰撞后，法向的速度会有一定程度的衰减，切线速度保持不变；和小球碰撞以后，则小球的速度会有一定的衰减。


#### 圆柱形边界
* 设圆柱的半径和高为R和H，小球的半径为r(这里是0.5)，则小球的个数为`3*R*R*h/(4*r*r*r)`,即圆柱和小球的体积之比
* 小球的初始分布也是呈网格分布，在圆柱的地面的内切正方形内产生网格，高度即逐渐向上叠加，知道小球的个数达到要求为止。
* 这里小球碰到边界后也是法向方向上速度衰减，切线方向速度不变，计算小球碰撞后速度时需要用到坐标变换的东西。

### 使用说明：
* 点击`求解问题`按钮，则开始迭代求解，此时点击`停止求解`，则停止迭代；在停止后，当前所有小球的状态会作为下次重新开始求解时小球的状态的初始值。

## V1.1
### 主界面改进的地方
* 将操作需要单击的按钮改完菜单栏，通过点击菜单栏的选项可以实现多种功能的操作，之前的功能全部保留
* 添加了显示系统状态的文本框，显示系统目前正在进行的操作等
* 完善了日志的输出和结果的保存

### 之前的问题修复
* 问题1 ： 显示体绘制的切片图像序列，会出现无法关闭该窗口，关闭主界面时，也会出现异常
	* 解决办法：新建一个界面，在其中添加RenderWindowControl控件，用于显示切片图像序列的体绘制结果，关闭它时不会出现异常。

### 功能完善
* 添加系统设置界面
	* 支持容器的类型以及对应尺寸参数的修改，即修改系统中小球的个数
	* 支持系统设置的存储，在初始化时，如果没有找到该配置文件，则采用程序中设置的默认参数
	* 在对系统参数进行修改之后，无需重启系统即可直接基于新的设置参数进行模型求解等
* 添加json数据文件的**回放**功能
	* 需求原因：小球个数比较多时，系统仿真较慢，在每次迭代时可以存储小球的位置等信息到数据文件，方便之后可以通过加载数据文件直接查看小球所在的位置并进行可视化
	* 考虑到回放过程中，可能需要改变界面的大小等，因此将回放功能放到新建的线程中运行。
* 添加对小球能量的计算，即将所有小球的动能和重力势能进行求和，可以看出系统能量在迭代的过程中是逐渐减小的


### 改进过程中遇到的问题以及解决的办法
* 问题1：在修改设置之后，即改变小球的个数，对应`RenderWindowControl`中`RemoveAllViewProps`时会出现`wglMakeCurrent failed in MakeCurrent()`的问题，查阅资料发现可能是驱动的问题。
	* 解决办法：在每次修改系统设置时，删除原来窗口中的`RenderWindowControl`控件，新建一个添加到其中，问题解决
* 问题2：添加新的线程之后，在关闭主界面时，线程有可能没有被关闭（CPU使用率高居不下。。。）
	* 解决办法：将系统中新开的线程的BackGround设置为true即可，在关闭窗口的时候，其中所有开启的线程都会被关闭
* 其他
