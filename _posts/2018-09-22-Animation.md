---
layout: post
title: "Animation in R (R与动画)"
date: 2018-07-11
comments: true
categories: 
- 技能
tags:
- R
- 动画
- 中秋
---
---

#@写在这中秋前夜: 动起来

##动画制作

近期有不少朋友一直在咨询如何用R来做动画(Animation)。制作动画，对于传染病工作者去发现传染病时空传播规律具有较大的价值。对其他学科而言，当时点较多时，通过动画来展现其分布趋势也是另有一番价值。当酷炫的动画穿插在你精美的PPT中时，是不是也要加个几个分啊？ 

简单而言之，动画可以将其理解为就是将多张静态图堆在一起，当然，不是随意堆砌，一般是根据某个类别属性(可以是时间点，也可以是城市等)。总体起来，动画制作包括两个步骤: 静态图制作及图形组装。静态图制作，这个就比较容易理解，毕竟R软件众多显著优势之一就是图形制作。对于图形组装，则可以依托[谢益辉](https://yihui.name/en/)的[animation](https://cran.r-project.org/web/packages/animation/index.html)程序包程序包来实现。考虑到R软件中绘图系统的差异性, 我们可用基础绘图系统来绘制静态图，然后用animation程序包来封装。不过本人更加偏爱[ggplot2](https://cran.r-project.org/web/packages/ggplot2/index.html)绘图系统，它的伟大之处在于**系统**(妈妈再也不会担心改变图形某个属性是哪个函数控制了)和**美观**(比十八岁的美女还要美,你们信吗?). 正所谓条条道路通罗马, 而本文将要讲述的也是后者(好像我对美确实没什么抵抗力). 今晚的猪脚是gganimate程序包: extends the grammar of graphics as implemented by ggplot2 to include the description of animation, 一个可让你们体会图形制作和组装一步到位的酸爽.

###程序包安装
 [gganimate](https://github.com/thomasp85/gganimate)程序包即可通过官网上下载,也可通过github([thomasp85](https://github.com/thomasp85))安装, 需要注意的是github版本一般会较早更新. 安装的具体代码如下:  

- **官网途径**(因更新版本,官网途径已经失效):  
```
install.packages("gganimate")
```
- **Github途径**: 
需要先下载devtools程序包,然后用install_github函数进行安装:  
```
# install.packages('devtools')  
devtools::install_github('thomasp85/gganimate')
```
###主要函数

-  **`transition_*()`**:  定义动画是根据哪个变量进行"动".
-   **`view_*()`**: 定义标尺的位置.
-   **`shadow_*()`**: 影子?定义点相继出现的方式.
-   **`enter_*()`/`exit_*()`**: 定义新数据产生和旧数据删除的方式. 
-   **`ease_aes()`**:  美观定义(如何让整个动画看起来更舒适).

(注意,此处只列出部分函数,正如作者所说的: extends the grammar of graphics as implemented by ggplot2, 所以,这个包所包含的应该是一整套关于动画绘制体系,而不仅仅只是几个函数.)

###动画制作
首先,我们用程序包自带的例子给大家做一个演示:

```
library(ggplot2)  
library(gganimate)  
ggplot(mtcars, aes(factor(cyl), mpg)) + 
  geom_boxplot() + 
  # Here comes the gganimate code
  transition_states(
    gear,
    transition_length = 2,state_length = 1) 
```
![](https://raw.githubusercontent.com/Spatial-R/cn/gh-pages/images/Animation/test1.gif)

大家可以看到, 相比与常规的ggplot2语法, 此处多了一个 transition_states的设置: 第一个参数**states**定义根据哪个变量(此处是gear)进行动画,第二(`transition_length`)和第三(`state_length`)分别定义了转变和状态变量停留的时间. 


再来看一个例子:

```
library(gapminder)

ggplot(gapminder, aes(gdpPercap, lifeExp, size = pop,
                                      colour = country)) +
  geom_point(alpha = 0.7, show.legend = FALSE) +
  scale_colour_manual(values = country_colors) +
  scale_size(range = c(2, 12)) +
  scale_x_log10() +
  facet_wrap(~continent) +
  # Here comes the gganimate specific bits
  labs(title = 'Year: {frame_time}', 
          x = 'GDP per capita', y = 'life expectancy') +
  transition_time(year) +
  ease_aes('linear')
```
![](https://raw.githubusercontent.com/Spatial-R/cn/gh-pages/images/Animation/test2.gif)

如果玩过googleVis程序包的人应该对这个图形很熟,或者是听过[汉斯罗斯林](https://baike.baidu.com/item/%E6%B1%89%E6%96%AF%C2%B7%E7%BD%97%E6%96%AF%E6%9E%97/10735820)Ted演讲的人也会对这个似曾相识. 他曾担任世界卫生组织、联合国儿童基金会和其他援助机构的顾问, 当时用[Trendalyzer](https://www.gapminder.org/tag/trendalyzer/)来对全球发展的相关指标进行动态可视化, 相关视频见[此处](https://www.gapminder.org/tag/trendalyzer/). 
(PS: 回想当年,本科毕业设计就是用R来实现这种动态图,可惜功底不够,只做了个交互式的)   
回归正题, 上述代码与传统的ggplot2绘图系统语言相比,多了几个部分: **transition_time**, **ease_aes** 和**labs**,各自定义了随时间改变的变量名称, 动画美观的方式(由离散形变量转变成连续型的,总要进行相应设置以及标签显示的规则(显示年份).


### 传染病动画

哈哈,好戏往往放在最后.那我们接下来看看如何用gganimate来对传染病暴发疫情数据进行动画. 其实如果说前面两个动画制作的方式您已经知晓了,那接下来的传染病数据的动画制作,其实也是很简单. 当然, 我们需要首先注意一点, 传染病点暴发数据, 往往具有空间属性(经纬度信息), 因此,最好的动画方式自然是将相关病例点描绘到地图上进行动态展示,也就是说,我们首先需要进行**空间可视化**.   

**空间可视化** 难吗? 好像确实不难, 无非就是产生地图,然后把具有经纬度信息的点画上去,具体而言,我们可以看看如下代码:

```
library(gapminder)
library(rgdal)
GD<- readOGR("Data/guangdong.shp")
data10 <- read.csv("Data/animation.csv",header = T)

p = ggplot(GD)+
  geom_point(data= data10, aes(x = long, y = lat, colour=as.factor(type)), size=2) +
  geom_polygon( aes(x = long, y = lat, group =group), size=0.4,
                colour = "BLACK", fill = "transparent") +
  coord_map(project="conic", lat0 = 30) +
 theme_bw(base_size = 15) +
  # Here comes the gganimate specific bits
  lab(title = 'Disease in Guangdong Province in: {frame_time}', x = '', y = '') +
 transition_time(onset_date) +
  ease_aes('linear')
```
![](https://raw.githubusercontent.com/Spatial-R/cn/gh-pages/images/Animation/test3.gif)

 整个代码框架基本一致,相比之前多了geom_polygon和coord_map等函数去定义空间相关属性. 整个地图上描绘的点会随着**`onset_date`**这个变量进行改动, 因而你的原始数据中则必须含有onset_date这个变量. 其中data10这个数据框包含了三列: long, lat和onset_date, 分别表示病例发生的经度,纬度和发病时间. 当然, 你也可以用 **shadow_mark**函数将过去病例点重点标出. 


### 动画保存
gganimate程序包默认产生的gif,动画保存的函数为**anim\_save**:  

```
`anim_save(filename = "test.gif", animation = last_animation())`
```

GIF可很好嵌入到PPT里去. 但有些时候,可能你更希望产生的动画能够以视频(MP4)的方式保存下来, 这个时候你可以用如下语句进行保存:

```
`anim_save(filename = "test.mp4", animation = last_animation())`
```

### 注意事项
- 根据以往旧版本的经验, Window用户需下载imagemagick这个软件,同时"点点点"进行安装(即下一步下一步,全部可采用默认的方式), 最后在系统环境变量进行设置(某些电脑之前不设置好像也可以).

- 某些Window用户, 其计算机名字取得有点任性,比如全部汉字啊, 又比如汉字加上上二次元文字啊. 这些在电脑读取系统环境变量(Path)时会认为是乱码,然后你会发现不管你下载R程序包也好,还是进行动画保存也好,遭遇各种BUGS. 所以, 请各位珍惜自己计算机名,取个**高大上**的可好?

- Linux用户在安装gganimate程序包时候,可能会出现错误.我电脑(Ubantu16.04)遇到的问题是gifski包安装不了, 它也会有相关提示: sudo apt-get install cargo 即可, 不过这个貌似还蛮大(70M). 可参考如下两个网页: [网页一](https://linux-os.net/gifski-un-programa-para-crear-imagenes-gif-de-alta-calidad/)和[网页二](https://stackoverflow.com/questions/47565203/cargo-build-hangs-with-blocking-waiting-for-file-lock-on-the-registry-index-a). 另外,好像不安装cargo也可以,直接安装Rust,且必须保证版本大于1.28. 

- 新版本的gganimate程序包有很大的改动, 此次也是基于新版本进行测试, 虽然目前整个体系还不是很完善,但是勉强还是够用的. 若各位看管不急,可以坐等开发者将整个project做完后再去测试. 

