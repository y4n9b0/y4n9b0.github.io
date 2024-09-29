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

[三星内存官网](https://semiconductor.samsung.com/cn/dram/ddr/){:target="_blank"}

[DDR4 Product Guide](https://download.semiconductor.samsung.com/resources/product-guide/DDR4_Product_guide_May.18.pdf){:target="_blank"}

![secret-box]({{ '/styles/images/memory-stick/samsung-memory-stick-marked.jpg' | prepend: site.baseurl }}){:width="585" height="auto"} 

![secret-box]({{ '/styles/images/memory-stick/samsung-memory-stick-back.jpg' | prepend: site.baseurl }}){:width="585" height="auto"} 

如上两图是 32G ddr5 笔记本内存的正反面，几乎所有的内存条都包含两个主要的信息区：颗粒信息（橙色框）和标签信息(红色框)。
* 颗粒信息是内存颗粒厂商（主要有三星、海力士、镁光、长江存储等）标识颗粒生产日期、批次等信息。
* 标签是内存条品牌厂商标识内存相关信息，内存条品牌厂商自己并不一定生产内存颗粒，而是使用内存颗粒厂商的产品搭配 PCB 板、控制芯片进行组装，比如金士顿、海盗船、联想、芝奇等。

一般地，标签中的编号也会包含部分颗粒信息，如果这部分交叉的信息对不上，那么基本可以确定内存条为假条子。当然假条子的说法可能并不准确，毕竟内存颗粒全世界就那么几家厂商能生产，
这种东西可不是深圳山寨小作坊就能雕刻出来的。信息对不上只是表明这肯定不是原厂出厂状态，存在人为二次加工，这类假条子大多有虚假指标、品质不稳定、没有售后保障等问题。<br>
价格相差不太大的情况下，建议优先购买内存颗粒厂商自有的品牌内存条，近水楼台先得月的道理大家都懂，毕竟颗粒生产过程中有良品率、同批次不同体质等问题。

由于三星官方目前没有放出 ddr5 相关产品文档，咱们结合目前的 
[DDR4 Product Guide](https://download.semiconductor.samsung.com/resources/product-guide/DDR4_Product_guide_May.18.pdf){:target="_blank"}
来解读 ddr5 内存条上相关编号的意义。

### 标签

![secret-box]({{ '/styles/images/memory-stick/samsung-label-example.png' | prepend: site.baseurl }}){:width="750" height="auto"} 

`DDR5 SODIMM      80CE0123080469059F`
* DDR5 SODIMM - 第 5 代笔记本内存
* 80CE01**2308**0469059F
    * 80CE01 - 生产批号
    * 2308 - 生产日期 23 年 08 周
    * 0469059F - 序列号，这几位在 CPUZ 的 SPD 选项中应该是一致的

`32GB 2Rx8 PC5-5600B-SB0-1010-XT`
* 32GB - 内存容量
* 2Rx8 - Rank
    * 2 - 2 个 Rank（R 指 Rank），https://blog.csdn.net/haolianglh/article/details/51935654
    * 8 - 内存颗粒位宽
* PC5，5 - 第 5 代 ddr，内存电压 1.1V（4 代为 1.2V）
* 5600B
    * 5600 - 内存频率 5600Mb/s
    * B - 内存频率等级。2400Mb/s 对应字母 T，5600Mb/s 对应 B，后续还会有 6400Mb/s 和 7200Mb/s，对应字母暂时未知。
* SB0
    * S - 内存模组类型，SODIMM，no ECC
    * B0 - [JEDEC reference design for Raw Card B](https://www.jedec.org/standards-documents/focus/memory-module-designs-dimms/DDR5/262-pin%20Unbuffered%20SODIMMs){:width="750" height="auto"}
* 1010 - 暂时未知
* XT - JEDEC SPD(Serial Presence Detect) Revision

`M425R4GA3BB0-CWM0D    NB0X`
* M - Memory Module
* 4 - 笔记本内存条 (3 为台式机内存条)
* 25 - 针脚：262pin Unbuffered SODIMM
* R - DRAM Type：DDR5 SDRAM（1.1V VDD），如果是字母 A 则对应 DDR4 SDRAM（1.2V VDD）
* 4G - memory depth 4Gb
* A - 8 Banks & POD-1.1V（4：16 Banks & POD-v1.2V）
* 3 - Bit Organization 内存颗粒位宽，0：x4，3：x8，4：x16
* B - Component Revision，这里的 B 就是经常说的 B-die，其实就是第 3 次修正版本的意思。理论上修正版本越高越完善，不知道为啥很多人迷信 B-die？？
* B - FBGA 封装 (Halogen-free & Lead-free, Flip Chip)
* 0 - PCB Revision，0 - None
* C - Commercial Temperature 商用级工作温度(0°C~85°C)
* WM - 内存频率 5600MHz（QK 为 4800MHz）
* 0D - PMIC（电源管理 IC），0D 是 [IDT P8911](https://www.renesas.cn/zh/products/memory-logic/memory-interface-products/ddr5-solutions/p8911-ddr5-client-pmic-udimms-and-sodimms){:width="750" height="auto"}，0L 是[三星 S2FPC01](https://semiconductor.samsung.com/cn/power-ic/accessory-power-ic/part-number/s2fpc01/){:width="750" height="auto"}
* 1010 - 暂时未知，组双通道时最好选择相同的 PMIC
* NB0X - 暂时未知，结合众多实物来看 0D-NB0X 和 0L-SX2Y 是成对出现的。

这一行信息都可以在三星官网上查询 https://semiconductor.samsung.com/cn/dram/module/sodimm/，
比如 [M425R4GA3BB0-CWM](https://semiconductor.samsung.com/cn/dram/module/sodimm/m425r4ga3bb0-cwm/){:target="_blank"}

![secret-box]({{ '/styles/images/memory-stick/samsung-sdram-module-information.png' | prepend: site.baseurl }}){:width="750" height="auto"} 

`UKCA` 是英国佬儿退出欧盟后，针对英国市场的一个认证标志。<br>
标签最右边的二维码可以使用支付宝扫一扫，扫出来的信息应该和标签一致，比如上面的二维码扫描信息：
```
(L)32GB 2Rx8 PC5-5600B-SB0-1010-XT(S)80CE0123080469550F(P)M425R4GA3BB0-CWMOD(M)301GL10
```

### 颗粒

![secret-box]({{ '/styles/images/memory-stick/samsung-sdram-memory-information.png' | prepend: site.baseurl }}){:width="750" height="auto"} 

```txt
 SEC 307
 K4RAH08
6VB BCWM
```

`SEC 307`
* sec - Samsung Electronics Co. 首字母的缩写
* 307 - 颗粒生产日期为 23 年 07 周

`K4RAH08`
* K - Samsung Memory
* 4 - DRAM
* R - DRAM Type：DDR5 SDRAM（1.1V VDD），如果是字母 A 则对应 DDR4 SDRAM（1.2V VDD），与标签第三行的第 5 位相同
* AH - Density 内存颗粒容量，4G：4Gb，8G：8Gb，AG：16Gb，BG：32Gb，暂时不知 AH 代表多少 Gb，猜测应该是 2Gb
* 08 - Bit Organization 内存颗粒位宽，与标签第三行第 9 位的 Bit Organization 相对应

`6VB BCWM`
* 6 - 8 Banks，5：16 Banks，与标签第三行第 8 位相对应
* V - VDD、VDDQ 电压接口：POD（1.1V，1.1V），W - POD（1.2V，1.2V）
* B - Revision，B-die，和标签第三行第 10 位相同
* B - FBGA 封装 (Halogen-free & Lead-free, Flip Chip)，和标签第三行第 11 位相同
* C - 商用级工作温度(0°C~85°C)，和标签第三行第 13 位相同
* WM - 内存频率 5600MHz（QK 为 4800MHz），和标签第三行第 14&15 位相同

颗粒信息的第二行和第三行合起来称为物料号（MATERIAL NUMBER），可在官网查询 https://semiconductor.samsung.com/cn/dram/ddr/ddr5/，比如 [K4RAH086VB-BCWM](https://semiconductor.samsung.com/cn/dram/ddr/ddr5/k4rah086vb-bcwm/){:target="_blank"}

颗粒信息第四行（底部）目前不知道有何定义。

辨别三星内存真假的几个关键点：

1. 标签信息和颗粒信息的容量、频率一致，这一点对所有品牌的内存条都适用。
2. 颗粒日期必定小于或等于标签中的生产日期，内存条品牌厂商一定是先有内存颗粒再进行组装，颗粒日期大于标签日期不合常理。
   以前颗粒日期和标签日期大多相差一个月以内，随着 ddr5 滞销，相差好几个月的也有。但是如果日期相差一年以上，几乎也可以确定为假。
   内存颗粒厂商如果滞销，它们往往会减产，而不是一昧生产堆积。所以几乎不太可能出现相差一年以上。同样地，这一点对所有品牌的内存条都适用。
3. 所有的内存颗粒二维码都是独一无二，如果颗粒二维码图案一致必定为假。
4. 三星内存颗粒字体白色偏黄，纯白色字体一定为假。
5. 三星内存条 PCB 板绿色偏黄，绿油油的 PCB 板一定为假（海力士的 PCB 板倒是绿得比较纯正）。三星内存条也没有黑色 PCB 板，黑色 PCB 板目前多见于英睿达、金士顿、海盗船这几个品牌。
6. 三星内存条 PCB 板上的电阻白色带黄，纯白电阻基本为假。

<!-- https://memory.net/product/m425r4ga3bb0-cwm-samsung-1x-32gb-ddr5-5600-sodimm-pc5-44800s-dual-rank-x8-module/ -->
<!-- https://zhuanlan.zhihu.com/p/335016746 -->
<!-- https://tieba.baidu.com/p/7820154100 -->
<!-- https://tieba.baidu.com/p/7763152927 -->

## 海力士

[海力士内存官网文档](https://product.skhynix.com/support/downloads/techs.go){:target="_blank"}

[DDR5 Module](https://mis-prod-koce-producthomepage-cdn-01-blob-ep.azureedge.net/web/TR-20230620152301091.pdf){:target="_blank"}

[DDR5 Component](https://mis-prod-koce-producthomepage-cdn-01-blob-ep.azureedge.net/web/TR-20230620152301412.pdf){:target="_blank"}

![secret-box]({{ '/styles/images/memory-stick/skhynix-memory-stick.jpg' | prepend: site.baseurl }}){:width="585" height="auto"} 

### 标签

海力士 ddr5 标签除了关键的 PN（Part Number）有着自己的识别规则，其他的都跟三星一样。<br>
标签最右边的二维码也可以使用支付宝扫一扫，信息应该和标签上一致，比如上面的二维码扫描信息：
```
(L)32GB 2Rx8 PC5-5600B-SB0-1010-XT(S)80AD0123237681804D(P)HMCG88AGBSA095N
```

Part Number 含义如下：

![secret-box]({{ '/styles/images/memory-stick/skhynix-ddr5-module.jpg' | prepend: site.baseurl }}){:width="750" height="auto"} 

### 颗粒

```txt
SKhynix
H5CG48AGBD
X018  322A
```

第三行的 `322` 表示颗粒生产日期 23 年 22 周，Material Number `H5CG48AGBDX018` 含义如下：

![secret-box]({{ '/styles/images/memory-stick/skhynix-ddr5-component.jpg' | prepend: site.baseurl }}){:width="750" height="auto"} 

## 镁光

镁光旗下包含镁光和英睿达两个内存品牌，其中镁光主要供应 OEM，英睿达面向个人消费市场。

https://www.micron.com/products/memory/dram-components/ddr5-sdram

https://www.micron.com/products/memory/dram-modules/sodimm

https://www.crucial.com/catalog/memory

[Micron Module Part Numbering Systems](https://www.micron.com/content/dam/micron/global/public/products/part-numbering-guide/numsdrammod.pdf){:target="_blank"}

![secret-box]({{ '/styles/images/memory-stick/micron-memory-stick.jpg' | prepend: site.baseurl }}){:width="585" height="auto"} 

![secret-box]({{ '/styles/images/memory-stick/crucial-memory-stick.jpg' | prepend: site.baseurl }}){:width="585" height="auto"} 

![secret-box]({{ '/styles/images/memory-stick/crucial-memory-stick1.jpg' | prepend: site.baseurl }}){:width="585" height="auto"} 

![secret-box]({{ '/styles/images/memory-stick/micron-module-part-number.jpg' | prepend: site.baseurl }}){:width="750" height="auto"} 

英睿达

图中左边二维码下面的数字就是 物料号 （MATERIAL NUMBER），物料号下面的是 零件号 （PART NUMBER）

https://www.micron.com/sales-support/design-tools/fbga-parts-decoder

https://blog.csdn.net/typ2004/article/details/130907586

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


最后，购买内存条避开发货地为深圳的商家，遇上假货的几率能骤减 90%。

<!-- https://www.crucial.cn/support/articles-faq-memory/difference-between-ddr4-ddr3-ddr2-ddr-sdram -->

<!-- https://semiconductor.samsung.com/cn/dram/ddr/ -->
<!-- https://download.semiconductor.samsung.com/resources/product-guide/DDR4_Product_guide_May.18.pdf -->
<!-- https://post.smzdm.com/p/ax0rqov2/ -->
<!-- https://post.smzdm.com/p/a5x8xw7k/ -->