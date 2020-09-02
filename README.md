#串口DMA方式发送&接收
该工程为寄存器版本，不过在库函数中仍可添加使用

使用资源：
| Model  | DMA               | I/O       |
| ------ |:-----------------:|:---------:|
| USART1 | DMA2_7_CH4 (USART1_TX)<br>DMA2_5_CH4 (USART1_RX) | PA9<br>PA10  |
| USART2 | DMA1_6_CH4 (USART2_TX)<br>DMA1_5_CH4 (USART2_RX) | PA2<br>PA3   |
| USART3 | DMA1_3_CH4 (USART3_TX)<br>DMA1_1_CH4 (USART3_RX) | PB10<br>PB11 |
| UART4  | DMA1_4_CH4 (USART4_TX)<br>DMA1_2_CH4 (USART4_RX) | PA0<br>PA1   |
| UART5  | DMA1_7_CH4 (USART5_TX)<br>DMA1_0_CH4 (USART5_RX) | PC12<br>PD2  |
| USART6 | DMA2_6_CH5 (USART6_TX)<br>DMA2_1_CH5 (USART6_RX) | PC6<br>PC7   |

【中断】:6个串口空闲接收中断，6个DMA接收完成中断（共12个，设置为组2，优先级(3,3)~(1,0)）

笔者使用的是STM32F407VET6，共包含6路串口，页尾处程序已将全部串口的DMA收发配置完成，本文仅以串口1为例进行讲解。（查看代码可直接跳至第二节或页尾处下载）

##1 STM32F4 DMA 简介  
DMA，全称为：Direct Memory Access，即直接存储器访问。DMA 传输方式无需 CPU 直接控制传输，也没有中断处理方式那样保留现场和恢复现场的过程，通过硬件为 RAM 与 I/O 设备开辟一条直接传送数据的通路，能使 CPU 的效率大为提高。  
STM32F4 最多有 2 个 DMA 控制器（DMA1 和 DMA2），共 16 个数据流（每个控制器 8 个），每一个 DMA 控制器都用于管理一个或多个外设的存储器访问请求。每个数据流总共可以有多达 8个通道（或称请求）。每个数据流通道都有一个仲裁器，用于处理 DMA 请求间的优先级。  

它可以处理一下事务：

1.外设到储存器的传输
2.储存器到外设的传输
3.储存器到储存器的传输

> **<font color=red>【注】</font>**：DMA1 控制器 AHB 外设端口与 DMA2 控制器的情况不同，不连接到总线矩阵，因此，仅 DMA2 数据流能够执行存储器到  

存储器的传输。

其中，数据流的多通道选择，是通过 DMA_SxCR 寄存器控制的，如图1所示：

<p align="center"><img src="Pictures/1 DMA2数据流映射表.png" width="90%"\></p>
<p align="center" style="color:orange; font-size:14px; color: #999; " >图1.1 自定义树的整体结构</p>

```
typedef struct TNode  //结点结构
{
	char         *name;		//结点名称 
	TElemType    data;		//结点数据
	struct TNode *parent;	                //双亲节点指针 
	LinkList     child;		//子节点指针 
}TNode, *Tree;
```


