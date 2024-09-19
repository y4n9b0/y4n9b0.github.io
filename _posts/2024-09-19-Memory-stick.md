---
layout: post
title: 内存条真假辨别
date: 2024-09-19 20:00:00 +0800
categories: computer
tags: memory
published: true
---

* content
{:toc}

## 分类

从技术维度区分：

* **(SDR) SDRAM**<br>
    单倍数据率同步动态随机存储器，全称 Single Data Rate Synchronous Dynamic Random Access Memory，旧式内存，常用于 2002 年之前生产的计算机。

* **DDR SDRAM**<br>
    双倍数据率同步动态随机存储器，全称 Double Data Rate Synchronous Dynamic Random Access Memory。

    在 2002 年左右进入主流计算机市场，这种内存直接在 SDR SDRAM 的基础上发展而成。DDR 和 SDR 之间的显著区别是，DDR 会读取时钟信号的上升沿和下降沿上的数据，因此，DDR 内存模块的数据传输速度是 SDR 内存模块的两倍。

    采用 DDR 后续技术的系统被称为 DDR2，于 2004 年中推出。DDR2 达到的速度超过了 DDR，可提供高达 8.5 GB/秒的带宽。基于 DDR2 的系统通常可使用成对安装的内存，以便在"双通道模式"下运行，从而进一步增加内存吞吐量。

    DDR3、DDR4 和 DDR5 代表了内存技术的进一步改进，增加了带宽并降低了能量功耗，随着时间的推移和标准的演变，可以带来更好的性能和稳定性。

    一般来说，主板只能支持一种内存。在任何系统中，都不能在同一块主板上组合使用 SDRAM、DDR、DDR2、DDR3、DDR4 或DDR5 内存。否则，内存将无法运行，甚至不能装入到插槽中。

从物理外形区分：

* **DIMM**<br>
    双列直插内存模块，全称 dual in-line memory module，台式机内存条。

* **SODIMM**<br>
    全称 small outline dual in-line memory module，笔记本内存条，尺寸更小。

内存条是由 PCB 板和内存颗粒构成，PCB 板只有一面镶嵌有内存颗粒的叫单面内存，两面都嵌有颗粒的叫双面内存（多用于大内存）。除此之外，还有按容量、频率区分。

加装内存时最好和现有内存品牌、容量、频率保持一致，不然可能出现兼容性问题，轻则降频、重则无法使用。

## 三星

[内存官网](https://semiconductor.samsung.com/cn/dram/ddr/){:target="_blank"}

[DDR4 Product Guide](https://download.semiconductor.samsung.com/resources/product-guide/DDR4_Product_guide_May.18.pdf){:target="_blank"}

![secret-box]({{ '/styles/images/memory-stick/samsung-sdram-module-information.png' | prepend: site.baseurl }}){:width="750" height="auto"} 

![secret-box]({{ '/styles/images/memory-stick/samsung-label-example.png' | prepend: site.baseurl }}){:width="750" height="auto"} 

https://zhuanlan.zhihu.com/p/335016746

https://tieba.baidu.com/p/7820154100

颗粒信息

sec 425

sec 是 Samsung Electronics Co. 首字母的缩写
425 指的是生产日期 24 年第 25 周

二维码不同

颗粒白色字体偏黄，纯白色一定假

颗粒日期和贴纸日期相差不太远。以前大概是4个周以内，但ddr5滞销，相差好几个月的也有，所以基本不再作为判断方法，但是颗粒日期一定是小于或等于贴纸日期的。

## 海力士


## 镁光

英睿达

图中左边二维码下面的数字就是 物料号 （MATERIAL NUMBER），物料号下面的是 零件号 （PART NUMBER）

https://post.smzdm.com/p/a5kl398k/

https://www.crucial.com/usa/en/support-identify-parts

https://www.crucial.com/usa/en/returns

https://zhuanlan.zhihu.com/p/134715350

美光官网FBGA查询系统可知C9BJZ颗粒，实际编号是“CT40A1G8SA-62M:E”
https://post.smzdm.com/p/a5klk5p7/

https://ngabbs.com/read.php?tid=25256913&rand=499

https://www.bilibili.com/read/cv15043947/
美光官方解码颗粒
https://www.micron.com/support/tools-and-utilities/fbga
https://zhuanlan.zhihu.com/p/333783874
https://www.micron.com/numbering
https://www.micron.com/content/dam/micron/global/public/products/part-numbering-guide/numsdrammod.pdf
https://www.micron.com/content/dam/micron/global/public/products/customer-service-note/csn11.pdf

FBGA Code C9BPL 
Part Number SCM3G8Y4CAB8RW-56B
https://www.digikey.cn/zh/products/detail/micron-technology-inc/MT60B3G8RW-56B-B/22041490

https://www.micron.com/sales-support/downloads
https://www.micron.com/sales-support/design-tools
https://www.crucial.com/support/identify-parts
https://www.crucial.cn/support/identify-parts (山寨？)

https://www.toolify.ai/tw/hardwaretw/%E5%93%AA%E4%B8%80%E7%A8%AEcpu%E5%8C%85%E8%A3%9D%E9%81%A9%E5%90%88%E4%BD%A0trade-retail-tray%E8%A7%A3%E6%9E%90-3184600

https://www.compuram.biz/


<!-- https://www.crucial.cn/support/articles-faq-memory/difference-between-ddr4-ddr3-ddr2-ddr-sdram -->

<!-- https://semiconductor.samsung.com/cn/dram/ddr/ -->
<!-- https://download.semiconductor.samsung.com/resources/product-guide/DDR4_Product_guide_May.18.pdf -->
<!-- https://post.smzdm.com/p/ax0rqov2/ -->