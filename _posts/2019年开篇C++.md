---
layout:     post
title:      开篇
subtitle:   C++学习路线和注意要点
date:       2019/02/04
author:     HY
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - C++
---


# 开头

刚开始参加工作，工作内容没有涉及到太多的专业内容，担心自己的业务能力会退化，于是在这里梳理一下学习和练习使用C++的计划。之后的部分博客就围绕C++进行展开。另外，出于个人兴趣，关于机器学习、深度学习、网络安全、操作系统、数据库、HTML等内容也会涉及。

## 工具

因为会同时涉及在Windows & Linux系统进行code，所以Visual studio/vim & gcc都会用到。基础学习通过菜鸟教程和CSDN博客，练习通过leetcode和Newcoder。平时会在github上闲逛，看一看一些有趣的开源项目。

## 基础概念和专业术语

这一部分内容结合[菜鸟教程]（http://www.runoob.com/cplusplus/cpp-tutorial.html)进行梳理。梳理顺序：数据类型 -> 变量类型 -> 修饰符类型 -> 



## 转化公式

获取的的测量值为 `-160 ~ 0dB` ，如何转化为我们所要的噪音值呢？在网上找了很多资料都没有结果，于是就自己摸索转化公式。

刚开始想到的是利用分贝计算公式`SPL = 20lg[p(e)/p(ref)]`进行计算，后来直接放弃这个方案，因为这是一个对数运算，获取到的值非常稳定，几乎不会波动，与其他的测噪软件所得的分贝值出入太大。

然后发现有个App在麦克风没有输入时显示-55dB

![](http://ww2.sinaimg.cn/large/7853084cgw1f9u0nu3xv3j205n0a0glq.jpg)

于是思路就有了。

其他测噪音软件的量程均为`0~110dB`,而我们获取的的测量值为 `-160 ~ 0dB`，两者之间差了`50dB`，也就是说以麦克风的测量值的`-160dB+50dB = -110dB`作为起点，`0dB`作为Max值,恰好量程为`0~110dB`.

问题看似结束，但是直接以`50dB`作为补偿测量结果会偏大。最后选择了分段进行处理，代码如下

```

-(void)audioPowerChange{
    
    [self.audioRecorder updateMeters];//更新测量值
    float power = [self.audioRecorder averagePowerForChannel:0];// 均值
    float powerMax = [self.audioRecorder peakPowerForChannel:0];// 峰值
    NSLog(@"power = %f, powerMax = %f",power, powerMax);
    
    CGFloat progress = (1.0 / 160.0) * (power + 160.0);
    
    // 关键代码
    power = power + 160  - 50;
    
    int dB = 0;
    if (power < 0.f) {
        dB = 0;
    } else if (power < 40.f) {
        dB = (int)(power * 0.875);
    } else if (power < 100.f) {
        dB = (int)(power - 15);
    } else if (power < 110.f) {
        dB = (int)(power * 2.5 - 165);
    } else {
        dB = 110;
    }
    
    NSLog(@"progress = %f, dB = %d", progress, dB);
    self.powerLabel.text = [NSString stringWithFormat:@"%ddB", dB];
    [self.audioPowerProgress setProgress:progress];

}

```

# 效果

效果如下：

![](http://ww4.sinaimg.cn/large/7853084cgw1f9u1gqgqieg20k00zk7d8.gif)

# 下载地址

Demo下载地址：[Noise-meter-Demo](https://github.com/qiubaiying/Noise-meter-Demo)
