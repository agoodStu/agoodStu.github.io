---
layout: post
title: 1016-VIP-LAB打怪指南
date: 2019-04-02
categories: blog
tags: [1016, 眼动仪]
---

# 眼动仪

目前我们实验室常用的眼动仪为Tobii Pro Tx300，采样率为300Hz。关于眼动仪的更多信息可以点击[Tobii Pro Tx300官网](<https://www.tobiipro.com/product-listing/tobii-pro-tx300/>)。参考官网给出的数据，这款眼动仪的垂直同步率（Vertical Sync Frequencey）在**49~75**Hz，水平同步率（Horizontal Sync Frequency）。显示器尺寸是23英寸，最大屏幕分辨率为1920*1080。

一般我们说一个显示器的刷新率，就是指它的垂直同步率。丁老师说，垂直同步率对于眼动实验，特别是呈现视觉刺激的实验来说特别重要。假设一台显示器的刷新率为60Hz，也就是说1秒种刷新60次，平均刷新一次的时间16 ms。影响刷新率的因素有两个，一是**显示器的带宽**。显示器的屏幕越大，带宽也越大，那么理论上能达到的刷新率也越高。由于我们没法改变既有屏幕尺寸，所以这个因素了解一下就行。另一个影响因素是**屏幕分辨率**。同样大小的显示器，分辨率越高，刷新率也就越低。通过屏幕的显示设置，我们可以很直接的调节屏幕分辩率，进而达到更改刷新率的目的。当然，这个条件也受到显示器自身的限制。

对于目前的眼动实验来说，我们常用编程工具为Matlab中的Psychtoolbox以及基于Python的Psychopy。





# 参考链接

1. <https://www.tobiipro.com/product-listing/tobii-pro-tx300/>
2. <https://jingyan.baidu.com/article/9f63fb91ae2ebac8410f0e44.html>



#  Update log

- 二〇一九年四月二日 22:35:50，首次更新