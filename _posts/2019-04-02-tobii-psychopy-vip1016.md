---
layout: post
title: 1016-VIP-LAB打怪指南
date: 2019-04-02
categories: blog
tags: [1016, 眼动仪, 实验]
---

# 眼动仪

# 实验显示器

# 实验 

###显示器

目前我们实验室常用的眼动仪为Tobii Pro Tx300，采样率为300Hz。关于眼动仪的更多信息可以点击[Tobii Pro Tx300官网](<https://www.tobiipro.com/product-listing/tobii-pro-tx300/>)。参考官网给出的数据，这款眼动仪的垂直同步率（Vertical Sync Frequencey）为**49~75**Hz，水平同步率（Horizontal Sync Frequency）为**54.2~83.8kHz**。显示器尺寸是23英寸，最大屏幕分辨率为1920✖1080，是一块TFT的LCD显示器。

一般我们说一个显示器的刷新率，就是指它的垂直同步率。影响刷新率的因素有两个，一是**显示器的带宽**。显示器的屏幕越大，带宽也越大，那么理论上能达到的刷新率也越高。由于我们没法改变既有屏幕尺寸，所以这个因素了解一下就行。另一个影响因素是**屏幕分辨率**。同样大小的显示器，分辨率越高，刷新率也就越低。通过屏幕的显示设置，我们可以很直接的调节屏幕分辩率，进而达到更改刷新率的目的。当然，这个条件也受到显示器自身的限制。脑电实验用的显示器为CRT显示器。CRT的工作方式是在屏幕上将图像从上至下，一行一行地打出来。假设分辨率为800✖600，则要打印800行。而如果分辩率更高，如1024✖768，则需要打印1024行。这样，我们就可以理解为什么分辨率越低，刷新率也就越低（因为打的行数更少，刷新需要的时间也就更短）。TFT显示的原理略有不同：

> 现代显示器的TFT是逐行扫描工作的，假设画面高1080像素，那每一帧时间就被切成1080份，每一份有一行像素的TFT被选通，读入source line上的灰阶数据被点亮。
>
> 但TFT妙的是，他是sample and hold类型的，TFT点亮的过程实际上是给像素上的几个小电容充电，电容是可以保持电压一段时间的。所以，假设有一行像素在一帧/1080时间内被选通，那在这一帧余下的1079/1080时间内，这行像素是没有外部电输入的，但因为电容保持电压的缘故，这一行像素会一直保持原有的亮度(电容会有很微小的漏电，忽略之)，不会熄灭。等到下一帧的选通时间来到，这一行就会根据source line上新的讯号来点亮。
>
> 作者：匿名用户
>
> 链接：https://www.zhihu.com/question/54890643/answer/517141145
>
> 来源：知乎
>
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

丁老师说，垂直同步率对于心理学实验，特别是呈现视觉刺激的实验来说特别重要。若一台显示器的刷新率为60Hz，也就是说1秒种刷新60次，平均刷新一次的时间16.6 ms。这也意味着在CRT显示器上，屏幕首画面出现的时间与底端出现时间有十几毫秒之间的差距。一个问题是，我们该在什么时候向被试呈现刺激？如果刺激刚绘制到一半，而设定的刺激呈现时间到了，那么刺激图像就会呈现一部分。

心理学的视觉刺激呈现时间较短，一般在几百毫秒和几秒之间。如果我们需要向被试呈现一个120ms的图片，显示器刷新频率为60Hz。在实际运行中，刺激的呈现时间则在116.7毫秒（7帧）和133.8毫秒（8帧）之间。

###帧数控制

#### Psychopy

对于目前的眼动实验来说，常用编程工具为Matlab中的Psychtoolbox以及基于Python的Psychopy。两者针对刷新率的问题都有一定的处理方法。

在Psychopy中，有两种控制刺激呈现时间的方法。一种是常见的时间控制，使用的函数是**core.Clock()**。该函数的时间精度在1毫秒左右。

> ```python
> from psychopy import visual, core
> 
> # setup stimulus
> win=visual.Window([400,400])
> gabor = visual.GratingStim(win, tex='sin', mask='gauss', sf=5, name='gabor')
> gabor.setAutoDraw(True)  # automatically draw every frame
> gabor.autoLog=False#or we'll get many messages about phase change
> 
> clock = core.Clock()
> # let's draw a stimulus for 2s, drifting for middle 0.5s
> while clock.getTime() < 2.0:  # clock times are in seconds
>     if 0.5 <= clock.getTime() < 1.0:
>         gabor.setPhase(0.1, '+')  # increment by 10th of cycle
>     win.flip()
> ```
>
> 代码链接：https://www.psychopy.org/coder/codeStimuli.html

上段代码在屏幕中央呈现一个2秒的光栅，光栅变化0.5秒。如果屏幕刷新率为60Hz时，当getTime()返回的时间时1.999秒是，因为满足while循环的判断，所以会再呈现一帧的时间，达到2.0167秒。如果getTime()返回的时间时2.001秒，则不会绘制。所以，用这种方法控制光栅变化，得到的只是一个近似值。我们呈现一个100毫秒，误差大于会10%。

因此，在Psychopy中控制呈现时间最精确的方法是按帧数呈现，当调用filp()函数绘制刺激时，可以与刷新率同步。如果绘制没完成，程序脚本不会继续执行。

>```python
>from psychopy import visual, core
>
>#setup stimulus
>win=visual.Window([400,400])
>gabor = visual.GratingStim(win, tex='sin', mask='gauss', sf=5,
>    name='gabor', autoLog=False)
>fixation = visual.GratingStim(win, tex=None, mask='gauss', sf=0, size=0.02,
>    name='fixation', autoLog=False)
>
>clock = core.Clock()
>#let's draw a stimulus for 200 frames, drifting for frames 50:100
>for frameN in range(200):#for exactly 200 frames
>    if 10 <= frameN < 150:  # present fixation for a subset of frames
>        fixation.draw()
>    if 50 <= frameN < 100:  # present stim for a different subset
>        gabor.setPhase(0.1, '+')  # increment by 10th of cycle
>        gabor.draw()
>    win.flip()
>```
>
>代码链接：https://www.psychopy.org/coder/codeStimuli.html

##### 补上Builder模式下，关于刺激呈现方式的区别

***

# 参考链接

1. <https://www.tobiipro.com/product-listing/tobii-pro-tx300/>
2. <https://jingyan.baidu.com/article/9f63fb91ae2ebac8410f0e44.html>
3. <https://www.psychopy.org/general/timing/timing.html#timing>
4. <https://www.psychopy.org/coder/codeStimuli.html>
5. <https://www.psychopy.org/general/timing/detectingFrameDrops.html#detectdroppedframes>



#  Update log

- 二〇一九年四月二日 22:35:50，首次更新
- 二〇一九年四月三日 15:30:10，更新