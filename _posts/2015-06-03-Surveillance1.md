---
layout: post
title: "前瞻性暴发探测"
date: 2015-06-3
comments: true
categories: 
- 统计
tags:
- surveillance
- 暴发探测
- R
---

Welcome to surveillance Project! 这是一篇针对surveillance程序包相关方法的总结，有疑问请联系[张兵: Spatial-R](zhangbing4502431@outlook.com)

[Surveillance](http://surveillance.r-forge.r-project.org/) 程序包提供了一系列前瞻性爆发探测和回顾性的时空模型，主要适用的数据类型是时间序列频数、时序百分比和时序分类数据。

-----------------------------------------------------

## 前瞻性暴发探测

Aberration Detection(Change-point detection, Outbreak Detection)，我将其都翻译为暴发探测，但仔细推敲，各词组间含义差别还是有点大。具体如下： 
  
   1. [Aberration](http://baike.baidu.com/view/25914.htm?fromtitle=Aberration&fromid=8102429&type=syn) 原指光行差，是指同一瞬间运动中观测者所观测的天体视方向与静止观察者所观测到天体的真方向之差，通俗理解就是偏差。
  
  2. [Change-Point](http://baike.baidu.com/view/4399435.htm) 指在某一位置或时刻点，前后观测值遵循两个不同的模型。[changepoint](http://CRAN.R-project.org/package=changepoint) 程序包也提供了一系列变点探测的方法，主要依托于变点前后均值或和方差的差异性。
  
  3. [Disease Outbreak](http://baike.baidu.com/link?url=xnLkGt0RfVwftGje17US3jJHmST1IKCoS9qEZnJ9A-5eGQ8ms4GAgTp5mCstsGjYpzkRHpaZiJ6F8Gt7nHqQj_) 指在局限的区域范围和短时间内突然出现许多同类病例的现象，对于区域范围、时间范围和案例数目都没有确切定义，因而不同空间尺度下，可能会存在截然不同的疾病分布状态。  

  综上所述，用Aberration或Change-point来归纳surveillance程序包中的方法可能更加确切。同时也需注意，上述两种都是正向改变，也就是说个案数目应增多。

**surveillance**程序包以**S4**方式，按照**STS**(surveillance time series)格式编排数据(该数据格式的说明可参考[此文](http://arxiv.org/pdf/1411.1292v1.pdf))。很多文章都对传染病监测方法进行了介绍，包括若干已经成功应用于疾病预防控制中心常规监测系统中的方法，如rki，已成功应用于德国的Robert Koch Institute。总的说来，[Salmon et.al 2014](http://arxiv.org/abs/1411.1292)一文系统详细地介绍了目前异常值监测方法，主要分为**单个异常点监测**、**异常时段监测**和 **分类数据异常监测**。

异常点监测较为常规的方式是基于历史数据(**in-control**)来估计某个时刻的病例数，并与阈值想比较，若高于阈值，则将其定义为 **out-control**。通过比较每个时间点的in-control和out-control状态，即可获得相应变点。

---------------------------------------------------------------

### 单个异常点

单个异常点监测是监测系统中较为常见的方式，通俗地理解就是一段时序数据中在某个时间点发生了较为重要的改变。

   1. Method **EARS**： CDC异常检测系统中的早期方法，surveillance程序包中的 **sarsC** 可实现这个过程，更为详细地说明可参考[Fricker，Hegler 和 Dunfee](http://onlinelibrary.wiley.com/doi/10.1002/sim.3197/abstract)。当疾病历史数据较少时且数据较平稳时，此方法较合适，同时该程序包提供了**C1**、**C2**和**C3**方式来实现变点探测。但该方法没有考虑到疾病时序数据如过度离散、周期性、长期趋势和过往疾病暴发对现状的影响等特性，同时也没有对将来数据进行区间预测。
   
   2.  Method **Farrington**： 该方法最初由Farrington在[Farrington et.al. 1996](http://www.jstor.org/pss/2983331) 提出，[Hohle和Mazick 2010](http://www.statistik.lmu.de/~hoehle/pubs/hoehle_mazick2009-preprint.pdf)一文中详细介绍了该方法，其主要基于广义线性或相加模型来探测异常值，而[Noufaily et.al 2012](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3692796/)在原方法的基础上，采用因子来校准季节性，同时当Anscombe残差大于2或者3时采用再加权进而有效改善了**"低加权"**现象(Down-weighting)。 FarringtonFlexible函数可实现该过程。
   
   3. Method **Boda**: 在FarringtonFlexible方法的基础上，boda (Bayesian Outbreak Detection Algorithm)采用成熟的贝叶斯广义线性或相加模型，通过惩罚样条函数(Penalized Spline)来校准长期和季节性趋势，并能即时调整以往暴发或以往疾病情况对当前个案数的影响。为了更好地获得参数的后验分布，boda采用[Integrated Nested Laplace Approximaxion](http://www.sciencedirect.com/science/article/pii/S0167947313001552)方法来逼近贝叶斯推断。

----------------------------------------------------------------------

### 异常时段

时序中的异常时段由单个异常点组成，但其具体实现过程则完全不同于单个异常点监测。控制图(Control Chart)是实现异常时段监测常用方式，其来源于[统计控制过程](http://baike.baidu.com/link?url=YzdmviFnqYGO1IDJ-bN0UlPA4KG1em8MixjvW5y45cpmsULww494C3pEFtwvP2mCNVOsjtbLif7h492JVb0II_)(Statistical process control)。然后需要注意的是，控制图并没有考虑到监测数据的过度离散和季节性特性，因而，[Hohle 和 Paul 2008](http://www.sciencedirect.com/science/article/pii/S0167947308000716)在传统监测方法上结合统计控制过程，有效地实现异常时段监测。surveillance中的 **glrnb**(负二项分布) 和 **glrPois**(Possion分布) 函数可实现这个过程。该程序包中其他函数如 **cusum** 和 **rogerson** 等也可实现针对异常时段的监测，其效果与glrnb相比可参考[Hohle 和 Paul 2008](http://www.sciencedirect.com/science/article/pii/S0167947308000716)。

在控制图中，以广义相加模型结合Possion 或 Negative binomial 分布来定义 in control 状态，而 out control 状态则由 in control 状态来反映。in control 状态的均值可由当前时刻点案例数和参数k获得（参数k可以事先设定或通过模型再估计获得），或是结合当前时刻点案例数和自相关成分综合反映。

-------------------------------------------------------------------

### 分类数据

在公共卫生监测中，目标变量可能会是分为若干亚组(Categories)的时序数据，如在疫苗接种情况下，不同年龄层传染病的发病情况异常值监测。[Chen 1978](http://www.tandfonline.com/doi/abs/10.1080/01621459.1978.10481577)、 [Reynods和Stoumbos 2000](http://link.springer.com/article/10.1023%2FA%3A1007671903559)和 [Rogerson和Yamada 2004](http://www.cdc.gov/mmwr//preview/mmwrhtml/su5301a18.htm) 等文章强调了针对二分类时序数据异常值探测方法，这些方法都基于常规CUSUM方法，并没有考虑到时序数据过度离散的特性。[Hohle 2010](http://www.statistik.lmu.de/~hoehle/pubs/hoehle2010-preprint.pdf)在上述方法的基础上，提出通过采用广义线性模型框架进而有效解决了基础SUSUM方法的缺陷(1,仅适用二分类时序数据；2，无法掌控过度离散特性)，该方法可由surveillance程序包中的**categoricalCUSUM** 函数实现。









