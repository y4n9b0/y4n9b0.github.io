---
layout: post
title:  "Supporting Multiple Screens"
date:   2017-12-28 12:30:00 +0800
categories: Android
tags: adaptation
published: true
---

* content
{:toc}


# Prologue
由于Android市场终端产品多元化以及系统碎片化，给开发者的适配工作带来不小烦恼。
<br>
严格意义上来说，一台到用户手里的Android终端需要经历好几个阶段的适配：
<br>
 * 硬件驱动适配<br>
   主要是芯片供应商进行驱动适配，在早期还不太流行提供整体打包方案的时候，相当一部分的驱动适配还是得手机厂商来做。还有一些其他三方硬件驱动适配，比如现在主流使用的指纹识别。
 * Android系统适配<br>
   主流的手机厂商大多都和google是partner关系，能第一时间内拿到最新版本的Android系统，在原生的基础上进行feature的增加和删除（为了保证修改之后还属于Android阵营，需要通过cts测试）。
 * Android应用适配<br>
   * 语言和文化习俗适配：<br>
     语言适配主要是翻译以及翻译后字符串长度变化可能带来的布局显示问题。<br>
     文化习俗适配指国家和地区本地化，比如阿拉伯语从右到左书写和阅读。
   * Android系统版本适配，主要是新API的增加和过期API的替换，比如Android 6.0新增了动态权限申请，7.0多窗口支持，8.0画中画。
   * 多屏适配

# Terminology
进行多屏适配之前，先介绍Android尺寸常用的几个术语：<br>
 * **px** (pixel)：屏幕分辨率的像素，比如 Google Pixel C 平板分辨率2560*1800。
 * **ppi** (pixel per inch)：每英寸像素点数，分辨率除以屏幕物理尺寸（单位为英寸）。数值越大，单位面积内的显示效果越清晰。
 <br><br>
 * **density**：基于160的密度因子。160这个值是Google定义的dpi标准量。
 * **dpi** (dots per inch)：每英寸点数。<br>
   这儿的dot是完全**虚拟**的，是Google为了搭建一套通用的显示尺寸而构造的一个量，值设定在各个手机的系统属性里。
 * **dp**/**dip** (Density-independent pixel)：设备独立像素。<br>
   px = dp \* density = dp \* (dpi / 160)

 ```java
 /**
  * Standard quantized DPI for medium-density screens.
  */
  public static final int DENSITY_MEDIUM = 160;

 /**
  * The reference density used throughout the system.
  */
  public static final int DENSITY_DEFAULT = DENSITY_MEDIUM;

  public static int DENSITY_DEVICE = getDeviceDensity();
  public float density =  DENSITY_DEVICE / (float) DENSITY_DEFAULT;
  public float densityDpi =  DENSITY_DEVICE;

  private static int getDeviceDensity() {
    // qemu.sf.lcd_density can be used to override ro.sf.lcd_density
    // when running in the emulator, allowing for dynamic configurations.
    // The reason for this is that ro.sf.lcd_density is write-once and is
    // set by the init process when it parses build.prop before anything else.
    return SystemProperties.getInt("qemu.sf.lcd_density",
            SystemProperties.getInt("ro.sf.lcd_density", DENSITY_DEFAULT));
}
 ```
 某种程度上我们可以说1 density = 160 dpi，就像数学里的1弧度等于180/π ≈ 57.3°一样。唯一不同的是数学里的57.3这个值是不能更改的，它是由弧度的定义推导出来的；而density里的这个160是可以更改的，如果当初Google把这个值定义为159或者161，或者250也是可行的，只是计算没那么方便（另一方面，160这个值估计也是Google统计和预测过Android终端分辨率的分布情况来确定的）。

 * **sp** (Scale-independent pixels)：比例的设备独立像素，正比于用户的字体设置。理论上只用于设置字体尺寸单位，实际上用作其他地方也没人拦。因为决定最终显示大小的还是像素值px，无论是dp还是sp 都只是转化为px的一种媒介单位；足够任性，只用px都可以:smile:。


未完 累了 回头 继续···
