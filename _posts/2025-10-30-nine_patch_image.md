---
layout: post
title: .9 图
date: 2025-10-30 16:00:00 +0800
categories: tool
tags: image
published: true
---

* content
{:toc}

新来的设计妹子经验有点欠缺不太会制作 .9 图，只好要来原图自己搞了一把。<br>
年纪大了忘性也大，每次制作的时候都得重新搜下那几条黑边分别干嘛的，干脆记录一下。
<br><br>

**左边黑边代表纵向拉伸区域**

![/styles/images/nine-patch/nine_patch_area_scale_vertical.jpg]({{'/styles/images/nine-patch/nine_patch_area_scale_vertical.jpg'|prepend:site.baseurl}}){:height="300" width="600"}
<br><br>

**顶部黑边代表横向拉伸区域**

![/styles/images/nine-patch/nine_patch_area_scale_horizontal.jpg]({{'/styles/images/nine-patch/nine_patch_area_scale_horizontal.jpg'|prepend:site.baseurl}}){:height="300" width="600"}
<br><br>

**底部和右边的黑边交叉区域代表内容显示区域**

![/styles/images/nine-patch/nine_patch_area_display_content.jpg]({{'/styles/images/nine-patch/nine_patch_area_display_content.jpg'|prepend:site.baseurl}}){:height="300" width="600"}

使用 Android Studio 制作 .9 图右边预览区域会有对应 3 张效果图，<br>
分别是纵向拉伸效果、横向拉伸效果、内容区域显示效果。
<br><br>

**几点注意事项**

* 拉伸区域尽可能背景颜色单调，比如上面示例图里的纵向拉伸区域就是渐变的，拉伸后会导致纵向渐变曲线不平滑。
* 如果控件宽高不够，.9 图会被整体压缩，而不是我们希望的仅仅压缩黑边区域，这会导致非拉伸区域也被压缩变形。所以制作 .9 的原图就不能太宽太高，应保证控件的最小尺寸大于等于原图尺寸。
