## Legacy BIOS

> :fontawesome-regular-face-grin: everything411 and LoveCatC
>
> :material-clock-edit-outline: 2019年1月1日 0:00:00

## BIOS简介

BIOS是Basic Input Output System的缩写。顾名思义，BIOS是一个底端程序。大体来说，BIOS负责完成计算机硬件自检、CMOS设置、引导操作系统的启动、提供硬件I/O以及硬件中断。

最早的BIOS是IBM工程师由汇编语言写成的，并且只能用于特定的计算机。然而计算机发展到今天，BIOS早已不再是那么简单。



## 为什么要有BIOS

X86和ARM对于理解BIOS这一概念很重要。为什么要提ARM和X86呢？因为只有X86平台才有BIOS这一说法。这主要是因为对于X86平台用户来说，客制化程度很高。换言之，X86的整个产业链是相对开放的，OS厂商（OSV）会发布操作系统，芯片厂商提供CPU，主板厂商（OEM）提供主板，等等。而形成鲜明对比的是ARM平台下，基本是由品牌商整合产品，形成相对封闭的产业链。或者说，定制化程度极高。二者的差别就在于此。举个栗子，在PC上，如果Windows蓝屏了用户会吐槽巨硬，Ubuntu的Nvidia显卡驱动出了问题用户会问候F**K Nvidia，SSD出问题会大骂无良厂商……而使用MAC的时候，不论是系统崩溃，或是硬件问题，用户第一时间都会想到Apple背锅而不是去找上游厂商。

说了这么多，想要说明的，其实就是这样一个问题：X86平台上的操作系统往往要面对成千上万千奇百怪的硬件搭配，要想让操作系统等软件服务于如此多的硬件，就必须要一个抽象层来封装这种硬件上的差别，并且为软件层/操作系统提供这种接口。于是**BIOS（Basic Input Output System）**就应运而生。而BIOS的意义所就是初始化硬件和提供硬件的抽象软件接口。

> ARM体系没有BIOS，但其当然也要初始化主板的相关硬件。这一般是通过BSP（Board Support Package，板级支持包）完成的。但前文已经提过，ARM架构下的产品，硬件基本都是定制化的，在初始化时预先就知道了有哪些硬件需要初始化，相应的，BSP几乎就只需要“照方抓药”。而X86系统的硬件配置情况在开机时是未知的。需要通过*探测（Probe）、Training（内存和PCIe）和枚举（即插即用设备）*进行分析，最终才能得出硬件的配置信息。得出配置信息之后，BIOS还要对这些硬件进行软件抽象，并将抽象出的接口提供给操作系统（OS）。





## 开机时发生了什么

当用户按下计算机电源开关的那一刹，电源首先开始向主板和其他硬件供电，但这时的电压并不稳定。早期，主板上还存在北桥时，由北桥向CPU发出RESET信号使CPU初始化，电压稳定后即撤去RESET信号，CPU的初始化也就完成。现在主板只有一个南桥，CPU的初始化过程实际上是由CPU自身调节稳定电压实现的。总之，经历了这样一个电压稳定的过程后，CPU会在BIOS保留的内存地址处执行一个跳转到**系统BIOS**起始处的指令，POST自检即将开始。

> **TIPS**
>
> 1. 早期计算机上是有Reset物理按钮的，按下Reset会重启机器。松开Reset按钮时，RESET信号也就随之撤去。
> 2. 内存地址FFFF:0000H上存储了一条跳转指令。CPU初始化完成后读取这条指令，然后跳转到BIOS起始处，BIOS开始运行。



**系统BIOS**代码进行的第一步是POST自检（Power-On Self Test，加电后自检）。POST首先会进行**关键部件检测**。更确切的说，这个测试主要检测这些关键硬件是否存在、是否正常工作。一般关键部件检测会检查CPU、ROM、64KB RAM。

> **TIPS**
>
> 完整的关键硬件监测大致顺序是：CPU－ROM－BIOS－System Clock－DMA－64KB RAM－IRQ。 其中，System Clock、DMA、IRQ更多算是处理机制。有兴趣可以自行了解。

需要注意的是，在关键硬件的监测过程中，显卡事实上是没有初始化的，所以更不用提在屏幕上显示错误信息了。在这一步出问题能够作为反馈的，一般只有主板上的报警扬声器（**不是声卡！**），或是主板上带有的指示灯/LED显示板。



关键部件检测完成后就轮到了显卡。系统BIOS查找并读取、调用相应地址上的**显卡BIOS**代码，开始显卡的初始化。显卡初始化的过程中，大多数显卡都会在屏幕上打印一些初始化信息。但这个画面基本是**一闪而过**的。

> **TIPS**
>
> 显卡BIOS对应的起始地址一般在C0000H处。



接下来系统BIOS会像初始化显卡一样，读取其他硬件的BIOS代码并完成这些硬件的初始化。

初始化完成后，**系统BIOS**会显示有关自身的信息，包括类型、版本、序列号等。

接下来系统BIOS会继续进行硬件的检测和配置。比较重要的包括检测、显示CPU的类型、频率，测试RAM，配置即插即用设备等。

这些都完成后，大多数BIOS会清屏并在屏幕上方显示硬件自检后各硬件的参数和信息。

到这里，可以说POST就结束了。



接下来，系统BIOS**可能**需要更新ESCD（Extended System Configuration Data，扩展系统配置数据）。这是系统BIOS和操作系统OS交换硬件配置信息的一种手段。这些数据被存储在CMOS中。一般来说，ESCD只在硬件信息发生改动后进行更新。也存在一些特殊情况，操作系统使用了与系统BIOS不同的ESCD写入格式，这就使得每次系统BIOS更新后，OS启动时又会重写ESCD，下次再由系统BIOS重写…如此陷入到循环中。



BIOS的最后一项工作就是移交控制权给下一阶段的启动程序。但，移交的前提是，“知道移交给谁”。这时首先要确定下一阶段的启动程序在哪个设备中。这时就到了一个熟悉的选项——**Boot Sequence （启动顺序）**。BIOS会按照启动顺序，将控制权移交给第一个设备。



BIOS的使命，到此就告一段落。当然，即使是到OS启动之后，现代的BIOS依然有其用武之地。但那就牵扯到更多的历史和标准规范了。



以上介绍的加电-POST-引导在很长的时间内都是一种主流解决方案。然而，随着时代的发展，一种新的启动方式——UEFI启动出现了。UEFI启动比上述BIOS启动方案有很多优势。为了区分，上面这种BIOS启动方式也被称作Legacy BIOS。
