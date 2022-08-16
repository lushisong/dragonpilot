**Licensing**

dragonpilot/openpilot is released under the MIT license. Some parts of
the software are released under other licenses as specified.

Any user of this software shall indemnify and hold harmless Comma.ai,
Inc. and its directors, officers, employees, agents, stockholders,
affiliates, subcontractors, contributors and customers from and against
all allegations, claims, actions, suits, demands, damages, liabilities,
obligations, losses, settlements, judgments, costs and expenses
(including without limitation attorneys' fees and costs) which arise out
of, relate to or result from any use of this software by user.

**THIS IS ALPHA QUALITY SOFTWARE FOR RESEARCH PURPOSES ONLY. THIS IS NOT
A PRODUCT. YOU ARE RESPONSIBLE FOR COMPLYING WITH LOCAL LAWS AND
REGULATIONS. NO WARRANTY EXPRESSED OR IMPLIED.**

本软件（含代码、文档等）由版权持有人及其他责任者"按原样"提供，包括但不限于软件的内在保证和特殊目的适用，将不作任何承诺，不做任何明示或暗示的保证。
在任何情况下，不管原因和责任依据，也不追究是合同责任、后果责任或侵权行为(包括疏忽或其它)，即使被告知发生损坏的可能性，在使用本软件/代码或部分使用的任何环节造成的任何直接、间接、偶然、特殊、典型或重大的损失(包括但不限于按原样使用或修改后使用的后果：人身、财产、数据或利益的损失)，版权持有人、其他责任者或作者或所有者或参与者概不承担任何责任。

**这是阿尔法质量的软件（含代码、文档等），仅用于研究目的。
这不是一个可供使用的产品，对软件的质量、用途没有任何明示或暗示的保证。对任何环节造成的任何损失（包括但不限于人身损害、财产损失）不承担任何责任。
您对遵守当地法律法规负有全部责任。**

**如果您不具备完全民事行为能力，或不明白上述声明的含义，请立即停止！**

**代码移植 Porting Guide**

**本文档内容均从公开互联网上搜集、整理所得，不保证其内容的合法性、正确性、完整性或可靠性，不构成任何建议。**

dragonpilot/openpilot
通过移植可适配模拟器中的不同品牌、不同型号的汽车模型。

1.  **移植概述**

```{=html}
<!-- -->
```
1.  [[How to write a car port for openpilot - comma.ai
    > blog]{.underline}](https://blog.comma.ai/how-to-write-a-car-port-for-openpilot/)

2.  [[openpilot port guide for Toyota models - comma.ai
    > blog]{.underline}](https://blog.comma.ai/openpilot-port-guide-for-toyota-models/)

```{=html}
<!-- -->
```
2.  **主要过程**

> 选择一个与目标汽车模型最接近的已支持模型来进行修改，比如横向与纵向是否可控，纵向是否支持Stop&Go，横向可激活的最低车速等。还可考虑原车ACC供应商，相同的平台一般具有相似性，比如博某。选定已支持的模型然后对相应文件进行备份后可直接在其上进行修改，需要修改的文件都是重启直接生效的（关闭"Mark
> As
> Prebuilt"）。等基本实现各项控制功能后再考虑另建车型目录，相对来说会简单很多。

3.  **主要步骤：**

```{=html}
<!-- -->
```
1)  整理、编写DBC文件

    a.  [[How does openpilot work? - comma.ai
        blog]{.underline}](https://blog.comma.ai/how-does-openpilot-work/)

    b.  [[A panda and a cabana: How to get started with comma.ai -
        comma.ai
        blog]{.underline}](https://blog.comma.ai/a-panda-and-a-cabana-how-to-get-started-car-hacking-with-comma-ai/)

    c.  [[【DBC专题】-4-DBC文件格式解析 - 知乎
        (zhihu.com)]{.underline}](https://zhuanlan.zhihu.com/p/339090185)

> 思路：DBC是内部模块之间的CAN通信协议，不同品牌的DBC差别较大，但主要差别在于格式，所约定的通信内容大体上都是相似的。就像中文和英文，是两种不同的语言，但都有主谓宾，内容无外乎就是生老病死、衣食住行。先看懂一个已支持车型的dbc文件会帮助理解。

2)  修改软件代码

> 原理上只需要修改以下7个文件，即可完成最简单的移植，满足控制汽车模型的最低需求：（以修改subaru的文件为例）：
>
> \\openpilot\\opendbc\\subaru_global_2017_generated.dbc
>
> \\openpilot\\panda\\board\\safety\\safety_subaru.h(在重启时手机会自动编译panda的二进制文件，在界面上可进行操作将固件烧写到panda的stm32芯片中)
>
> \\openpilot\\selfdrive\\car\\subaru\\carcontroller.py
>
> \\openpilot\\selfdrive\\car\\subaru\\carstate.py
>
> \\openpilot\\selfdrive\\car\\subaru\\interface.py
>
> \\openpilot\\selfdrive\\car\\subaru\\subarucan.py
>
> \\openpilot\\selfdrive\\car\\subaru\\values.py
>
> \\openpilot\\opendbc\\can\\common.cc
>
> 其中common.cc的修改非必须，如果dbc中的信号名称为"CHECKSUM"，则生成can报文时，packer会自动调用common.cc中的subaru_checksum来计算校验码；如不改common.cc，在dbc中把校验码对应的信号名称改为别的，如"CheckSum"，然后在subarucan.py中进行计算。
>
> 在修改代码过程中，错误总是难免的，可将错误信息保存下来。用putty通过ssh连接到手机，输入以下命令：
>
> tmux capture-pane -S -32768 -b %1; tmux save-buffer -b %1 /tmp/bug_log

3)  参数整定

    a.  [[Tuning · commaai/openpilot Wiki
        (github.com)]{.underline}](https://github.com/commaai/openpilot/wiki/Tuning)

    b.  [[openpilot/README.md at kegman-ultimate · kegman/openpilot
        (github.com)]{.underline}](https://github.com/kegman/openpilot/blob/kegman-ultimate/README.md)

    c.  [[commaai/PlotJuggler: The Time Series Visualization Tool that
        you deserve.
        (github.com)]{.underline}](https://github.com/commaai/PlotJuggler)，可以用PlotJuggler解析rlog.bz2数据文件（dp的位于"/data/media/0/fakedata/"），对照曲线来调参，事半功倍。

```{=html}
<!-- -->
```
4.  **相关资料**

```{=html}
<!-- -->
```
a)  [[快速入门 · Openpilot 中文 Wiki
    (dragonpilot.cn)]{.underline}](http://www.dragonpilot.cn/)非常有用！

b)  [[comma.ai blog]{.underline}](https://blog.comma.ai/)
    有一些blog对移植非常有用，尤其是早期的。

c)  [[Home · commaai/openpilot Wiki
    (github.com)]{.underline}](https://github.com/commaai/openpilot/wiki/)
    openpilot的官方wiki
