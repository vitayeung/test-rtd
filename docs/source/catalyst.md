# α137 Catalyst FX User Manual
**用户手册**

<p align ="center">
    <img src="./catalyst/main.jpg" alt="drawing" width="1280"/>
</p>

# 目录

 1. [功能特性介绍](#功能概述)
    - [基于GPU Sparse Voxel 的实时流体模拟](#id-section1)

---
## 功能概述

Catalys FX 是一款基于GPU Sparse Voxel 稀疏网格结构的实时流体模拟解解决方案，提供了对烟雾，火焰燃烧，爆炸，复杂湍流等多种气体流体力学效果的GPU实时仿真处理，以及基于节点视图(NodeGraph)的可视化编程接口

Catalyst FX 可以运行在多种Runtime API 环境的GPU设备上，包括DirectX 11，DirectX 12，CUDA，Vulkan , OpenGL等 Runtime API运行环境，并提供了内置的GPU体积渲染模块，以及该渲染器的第三方三维软件的Viweport 插件，可以在没有数据回读（Loadback）的情况下快速预览仿真结果

基于AlphaCore 强大的 AxGraph 编译器，我们对算子进行了深度优化，并提供了Houdini，Unreal，Unity，Maya的使用插件，用户可以在自己的工作流程环境中直接使用Catalyst FX


---

<div id='id-section1'/>

### GPU Base SparseVoxel 实时流体仿真

基于SparseVoxel数据结构的 Catalyst FX，相比传统稠密网格的数据，可以实现更合理的显存/内存开销，在不涉及可视化的流场区域，不分配任何存储，也不进行任何计算，仅在关键区域分配内存空间用于计算·
<p align ="center">
    <img src="./catalyst/sparse2.jpg" alt="drawing" width="1280"/>
</p>

**1. 性能对比 Performance Benchmark**

 以下提供了该文档中出现的Example相比Houdini PyroFX 2.0，PyroFX 3.0的计算用时开销，此处出现的场景将均以Catalyst Sample File的形式提供下载

|场景名称|Resolution|Frame|Pyro FX |Catalyst FX 1.0| Catalyst RT（MT GPU Only）
|--|--|--|--|--|--|
|硝烟|128 * 198 * 800|500|2 h 15 min | MT QY1: <br> RTX 3060: **8 min**|
|Gournd Explosion| 404 * 577 * 430|200|35min| MT QY1: <br> RTX 3060: **3 min**|
|喷火器 (substep 2)| 600 * 128 * 128 | 200 | 50 min |MT QY 1: <br>RTX 3060 **2min**
|火山| 256 * 512 * 256 | 200 | 40min | MT QY1 : <br>RTX 3060 **7 min** <br> 

相比之下，Catalyst FX的全GPU计算流程，优化了 CPU-GPU IO Overhead ，实现了更好的Substep 性能无关的特性，在不开启硬件加速的情况下，可以实现比Houdini 默认PyroFX解算器提速10~15倍，在开启GPU硬件电路加速的情况下，可以有进一步的性能增益

**2. DCC Support List**

* Houdini SOP 插件支持
  * SOP Source
  * 自定义节点操作支持 ( SOP Vector & Scalar Field Interface )
  * DCC Viewport hook 
* Unreal Unity Ready

**软硬件需求配置**
  
## 模块介绍
* 燃料和燃烧反应
<p align ="center">
    <img src="./catalyst/cb1.jpg" alt="drawing" width="1280"/>
    <img src="./catalyst/cb2.jpg" alt="drawing" width="1280"/>
</p>

* 体积绘制
  
* 可视化风场系统

<div style="page-break-after: always;"></div>

## Chapter2  Samples Example 

### 2.1 Smoke & Combustion
* 燃烧是什么【燃料反应过程】
  

---
### 2.2 爆炸 Explosion 
<p align ="center">
    <img src="./catalyst/exp2.jpg" alt="drawing" width="1024"/>
</p>

&emsp;&emsp; 爆炸在流体计算领域是一种常见的仿真需求，它的整个计算过程和燃烧反应差别不大，Catalyst FX 也提供了相应的计算features，用于模拟燃料燃烧后产生的爆炸效果

#### 物理意义

关于爆炸，最容易联想到的是世界大战时期，战场上最常用的TNT炸药，也就是聚能装药爆炸，在学术上可以把它归类为化学爆炸。新闻中出现的煤气罐爆炸，也属于化学爆炸的一种。但是在详细说化学爆炸之前，我们还是总体了解一下爆炸的种类。爆炸可以分为三种：

* 物理爆炸
* 核爆炸
* 化学爆炸

**物理爆炸：** 大致可分为三类：(i)压力容器的爆炸；(ii)火山的喷发；(iii)两种温差极大的液体快速混合。举例来说，家里高压锅爆炸就属于物理爆炸，在此过程中没有化学反应的发生，只有压力的快速释放。

**核爆炸：** 包括核裂变、核聚变，其能量来自原子核的链式反应。简单来说，核爆炸释放的能量可以分为三类：(i)粒子的动能；(ii)气体的内能；(iii)热能。热能主要以热辐射形式表现，动能和内能大约占核爆炸能量的一半，并最终形成空气中的冲击波。

**化学爆炸：** 主要表现为化学反应的进行和化学能的释放，在计算机图形学中进行的爆炸效果模拟，会用到化学爆炸中的一些概念，如：着火点（ Ignition Temperature），燃料 （Fuel）等，但是和真正的基于化学反应过程实现的燃烧爆炸模拟相比，不能说不太类似，只能说是毫不相干。


#### 核心参数

爆炸最核心的控制，就是对散度目标函数，或者说是对散度场（Divergence Field）的设置，通过设置 不同的散度（Divergence）数值，来实现各种爆破效果的调节，以下，我们先不考虑复杂的湍流细节，只考虑最重要的几个爆炸相关的参数
|名称|功能|效果注解|
|--|--|--|
|**燃料密度发射率**</br>Emit Fuel  |燃料会通过点燃，转换成燃烧（Burn），火焰（Heat），和温度（Temperature），并且这三者均会对散度有所贡献|发射器提供的燃料（Fuel）密度越高，爆炸越剧烈，想象一下30吨的TNT炸药爆炸，和30吨的大伊万（5000万吨TNT）爆炸有啥区别|
|**燃烧冷却时间** </br>Combustion CoolingDown Time|该参数控制火焰（Heat）的冷却时间，单位是秒，即：N 秒之后，火焰将完全消失|默认参数下，Heat场（或 brun场）是一个潜在的烟雾发射源，当Heat场的数值降低接近0 的时候，则被认为是燃烧不充分区域，这部分区域会产生浓烟。<br>同时，Heat场对会温度场进行贡献，Heat存在的时间越久，则温度持续的时间越久，烟雾受到的浮力则越久|
|GasRelease|控制燃料燃烧的反应过程，对散度场（Divergence Field）的贡献系数|整个燃烧反应，对散场（Divergence Field）度贡献的一个全局系数|


&emsp;&emsp; 所以，如果要实现一个爆炸，最直接的方法，就是调整发射器的燃料（Fuel）发射强度和 GasRelease参数随时间的变换,以下我们step by step的演示以下

#### 爆炸 101 Basic Explosion With Houdini

**创建发射器 Source**

<p align ="center">
    <img src="./catalyst/exp101_houdini_source.jpg" alt="drawing" width="1280"/>
</p>

&emsp;&emsp; 如图（上）所示：在SOP Network中创建一套标准Pyro发射器，Catalyst FX 节点可以读取Houdini Pyro Source 节点已经创建好的Field，作为自己的发射源使用。这里我们直接将一个Houdini自带的甜甜圈转换成体积，对其设置合理的Noise密度分布，并将其作为爆炸发射器使用
&emsp;&emsp; 接下来，我们可以创建 SOP 下的 CatalystSolver节点，将Solver节点第一个Input 连接至当前编辑好的发射器作为输入

**燃料发射强度设置**

<p align ="center">
    <img src="./catalyst/exp101_solver_fuel.jpg" alt="drawing" width="1280"/>
</p>

&emsp;&emsp; 我们在创建好的Catalyst Solver节点的 Sourcing 面板中依次勾选 Fuel，Temperature 和Velocity 的 Enable 开关。启动发射器的燃料，温度和速度的发射，在这里速度可以设置为正Y（0，1，0） 方向上一个比较大的数值，使得爆炸有较高的向上的初始速度
&emsp;&emsp; 比较关键的是燃料参数的整体缩放系数，如图所示，我们对燃料参数进行一个key 帧处理，将燃料的初始发射密度设置成5，在经过24帧左右的时间之后，燃料发射器的整体发射强度，降低至0.2，创造出解算第一秒流场中产生爆炸和膨胀的条件


<p align ="center">
    <img src="./catalyst/exp3.jpg" alt="drawing" width="1280"/>
</p>

* 增加适当的湍流效果，最终得到 如图所示的爆炸模拟结果（**精度：** 220 * 480 * 220 **时长：**120 frames ，**解算用时：** 1 min with RTX 3060 ）

<p align ="center"><img src="./catalyst/exp4.jpg" alt="drawing" width="1280"/></p>

* 为了得到更明显的爆炸效果，我们修改fuel的发射曲线，发射强度由最开的 5 ~ 0.2，修改成 10 ~ 0.2，得到图（右）的效果 

小节：通过控制发射器的燃料发射密度，是最直接的制作爆炸的方法，更详细的有关于 Catalyst FX 制作爆炸的调参细节，可以参考手册后续的 Parameter Map章节，该章节会对每一种效果使用不同的参数有详细的对比和注解

---
#### 爆炸 102 Ground Explosion With Houdini

**Divergence Control**

除了控制燃料发射密度的方式制作爆炸，我们也可以直接控制发射器直接对散度场进行填充，配合 Houdini 的粒子系统作为Catalyst Solver的发射器输入源，用于制作更复杂的爆炸效果

相比直接控制燃料发射密度的工作流程相比，通过直接控制散度 Divergence的方式，不会产生额外的燃烧反应温度，换句话说，散度控制（Divergence Control）和先前章节提到的爆炸分类中的**物理爆炸**，这分类有些类似，即整个爆炸的过程中没有任何化学能的转换

<p align ="center">
    <img src="./catalyst/exp1.jpg" alt="drawing" width="1280"/>
</p>

**创建发射器 Source**

<p align ="center">
    <img src="./catalyst/sample_102_scatter.jpg" alt="drawing" width="1280"/>
</p>

* 我们使用Circle节点创建一块椭圆形区域，并在该区域撒点填充，作为接下来爆炸模拟将会产生的爆炸点的位置

<p align ="center">
    <img src="./catalyst/sample_102_particle_emit.jpg" alt="drawing" width="1280"/>
</p>

 * 配合 Houdini SOP自带的 Pyro Burst Source 制作粒子爆炸发射器，在Burst Animation面板中调节每一个点产生爆炸的时间，通过设置Start Frame参数后下拉菜单为：UseAttribute。使用我们我们已经编辑好的 StartFrame属性。这样可以让每一个点在Frame = 自身属性 StartFrame属性的情况下产生爆炸

<p align ="center">
    <img src="./catalyst/sample_102_convert.jpg" alt="drawing" width="1280"/>
</p>

* 通过Volume Rasterize节点，将制作好的粒子发射器，转换成体积发射器，通过鼠标中键的节点Info面板，可以查看到，当前粒子发射器提供了 fuel（燃料），density（密度），divergence（散度），temperature（温度），v（速度）信息的发射，接下来我们链接 Catalyst Solver节点，将这部分的数据和 Catalyst 解算器对接

<p align ="center">
    <img src="./catalyst/sample_102_rst.jpg" alt="drawing" width="1280"/>
</p>

* 设置Catalyst Sovler的 Sourcing 面板，发射器各个数据场的填充强度，其中发射器的速度发射强度为10倍，divergence 散度发射强度为5倍，并设置合适的湍流效果，进行解算后，得到如图viewport呈现的爆炸效果

#### Particle Emitter【火山】
* particle

#### Wind


## Parameter Map 详解

<p align ="center">
    <img src="./catalyst/combustion_cooling.jpg" alt="drawing" width="1024"/>
</p>




