# 2020.04.02更新：

最近正好用到了这个，贴个能用的代码帮助大家理解吧。该代码已通过仿真测试，可以与Xilinx的AXI BRAM Controller一起使用。

该模块的主要功能是利用AXI4协议对内存进行先写后读的反复操作。代码如下：

```
`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: 
// 
// Create Date: 01/7/2020 10:41:11 AM
// Design Name: 
// Module Name: AXI_read_write
//////////////////////////////////////////////////////////////////////////////////
module AXI_read_write #(
    parameter integer UPPER_WIDTH = 8,
    parameter integer DATA_WIDTH = 64
)
(
    //==========input===============//
    M_AXI_ACLK    ,
    M_AXI_ARESETN ,

    M_AXI_awready ,
    M_AXI_arready ,
    
    M_AXI_rdata   ,
    
    M_AXI_wready  ,
    M_AXI_rlast   ,
    M_AXI_rvalid  ,
    
    M_AXI_bvalid  ,
    //==========output===============//
    M_AXI_awvalid ,
    M_AXI_arvalid ,
    
    M_AXI_wdata   ,
    
    M_AXI_rready  ,
    M_AXI_wlast   ,
    M_AXI_wvalid  ,
    
    M_AXI_awaddr  ,
    M_AXI_araddr  ,

    M_AXI_bready  
    );

//port--------------------------------------
input wire M_AXI_ACLK    ;
input wire M_AXI_ARESETN ;

input wire M_AXI_awready ;
input wire M_AXI_arready ;
output reg M_AXI_awvalid ;
output reg M_AXI_arvalid ;

// input wire [1:0] M_AXI_BRESP;
// input wire [1:0] M_AXI_RRESP;
input wire [DATA_WIDTH-1:0] M_AXI_rdata;
output reg [DATA_WIDTH-1:0] M_AXI_wdata;

input wire M_AXI_wready ;
input wire M_AXI_rlast  ;
input wire M_AXI_rvalid ;
//---------------------------
output reg M_AXI_rready ;
output reg M_AXI_wlast  ;
output reg M_AXI_wvalid ;

output reg [UPPER_WIDTH-1:0] M_AXI_awaddr;
// output reg [7:0] M_AXI_AWLEN = 8'b0;
// output reg [2:0] M_AXI_AWSIZE = 3'b011;//8 byte
//----------------------------
output reg [UPPER_WIDTH-1:0] M_AXI_araddr;
// output reg [7:0] M_AXI_ARLEN = 8'b0;
// output reg [2:0] M_AXI_ARSIZE = 3'b011;//8 byte
//output reg [1:0] M_AXI_ARBURST = 2'b0;//FIXED

input wire M_AXI_bvalid;
output reg M_AXI_bready;

reg [3:0] cnt;
// output reg [DATA_WIDTH/8-1:0] M_AXI_WSTRB;//用于小块传输

always @(posedge M_AXI_ACLK or negedge M_AXI_ARESETN) begin
    if (!M_AXI_ARESETN) M_AXI_bready <= 1'b0;
    else if (M_AXI_awready) M_AXI_bready <= 1'b1;
    else if (M_AXI_wlast) M_AXI_bready <= 1'b0;
    else M_AXI_bready <= M_AXI_bready;
end

//Read-Write Control
always @(posedge M_AXI_ACLK or negedge M_AXI_ARESETN) begin
    if (!M_AXI_ARESETN) cnt <= 'b0;
    else if (cnt == 'd11) cnt <= 'b0;
    else cnt <= cnt + 'b1;
end

//write data
always @(posedge M_AXI_ACLK or negedge M_AXI_ARESETN) begin
    if (!M_AXI_ARESETN) M_AXI_awaddr  <= 'b0;
    else if (cnt == 'd1) M_AXI_awaddr  <= M_AXI_awaddr + 'h4;   
    else M_AXI_awaddr <= M_AXI_awaddr;
end

always @(posedge M_AXI_ACLK or negedge M_AXI_ARESETN) begin
    if (!M_AXI_ARESETN) M_AXI_awvalid <= 'b0;
    else if ((cnt == 'd1) || (cnt == 'd2)) M_AXI_awvalid <= 'b1;
    else M_AXI_awvalid <= 'b0;
end

always @(posedge M_AXI_ACLK or negedge M_AXI_ARESETN) begin
    if (!M_AXI_ARESETN) begin
        M_AXI_wdata  <= 'hFFFFFFFF;
        M_AXI_wlast  <= 'b0;
    end
    else if (cnt == 'd3) begin
        M_AXI_wdata  <= M_AXI_wdata + 'hF;
        M_AXI_wlast  <= 'b1;
    end
    else begin
        M_AXI_wdata  <= M_AXI_wdata;
        M_AXI_wlast  <= 'b0;
    end
end

always @(posedge M_AXI_ACLK or negedge M_AXI_ARESETN) begin
    if (!M_AXI_ARESETN)  M_AXI_wvalid <= 'b0;
    else if (cnt == 'd3) M_AXI_wvalid <= 'b1;
    else                 M_AXI_wvalid <= 'b0;
end

//read data
always @(posedge M_AXI_ACLK or negedge M_AXI_ARESETN) begin
    if (!M_AXI_ARESETN)  M_AXI_araddr  <= 'b0;
    else if (cnt == 'd5) M_AXI_araddr  <= M_AXI_araddr + 'h4;
    else                 M_AXI_araddr  <= M_AXI_araddr;
end

always @(posedge M_AXI_ACLK or negedge M_AXI_ARESETN) begin
    if (!M_AXI_ARESETN)  M_AXI_arvalid <= 'b0;
    else if ((cnt == 'd5) || (cnt == 'd6)) M_AXI_arvalid <= 'b1;
    else                 M_AXI_arvalid <= 'b0;
end

always @(posedge M_AXI_ACLK or negedge M_AXI_ARESETN) begin
    if (!M_AXI_ARESETN) M_AXI_rready <= 'b0;
    else if (M_AXI_arready) M_AXI_rready <= 'b1;
    else if (M_AXI_rlast) M_AXI_rready <= 'b0;
    else M_AXI_rready <= M_AXI_rready;  
end
endmodule

```

-----------

# 前言：

最近需要对一个已经实现功能的IP核添加支持AXI总线接口的数据通信接口，于是研究了一下AXI4和AXI4-stream的接口，下文中大部分内容来自于其他博主，这里写此篇综合一下，以便日后查阅。

    (`由于本人水平有限，如有错误欢迎指正！`)

--------

# 1、初识AXI总线：

## 1.1、通信模型：

![AXI主设备与从设备之间的通信](https://github.com/edger330/My_blog/tree/master/img/5791357-e482ae3da2c20de1.png)

如上图所示，要通过AXI总线实现通信，我们需要一个主设备/上位机(*Master*)和一个从设备/下位机(*Slave*)，并通过AXI总线将其相连。我们可以将上图中的主设备假定为CPU，从设备假定为RAM。主设备和从设备的通信主要为了实现主设备对从设备的读写控制。

----------

## 1.2、AXI Interconnect：

![AXI Interconnect](https://github.com/edger330/My_blog/tree/master/img/5791357-0438441b293eb7f4.png)

有时我们需要实现一个主设备控制多个从设备，可以使用AXI Interconnect模块实现该功能。可以将其简单地认为是一个带仲裁功能的多路选择器(*MUX*)。在配置从设备的地址时(*Address*)时，注意设备地址不能重叠，且**地址分配时需要整块分配，而不是简单地跟在上个设备分配的地址之后继续分配**。

>举例如下：
>
>![地址分配错误](https://github.com/edger330/My_blog/tree/master/img/5791357-e165c839d07af771.png)
>
>如上图所示，给Slave1分配好地址之后，直接接在Slave1的地址之后给Slave2分配地址是不行的，因为Slave2的地址范围(*Address Range*)过大，从0x40001000分配最多只能分配到0x40001FFF，即最多分配4K，而现在需要分配2G，应将地址偏移(*Address Offset*)设为2G的边界(*boundary*)，即**地址偏移+地址范围=FFFFFFFF**，故此时地址偏移应该为0x80000000。
>

------

## 1.3、握手机制：

先放时序图中的图样说明图：

![时序图样说明图](https://github.com/edger330/My_blog/tree/master/img/5791357-91498b498c15fec4.png)

---------

AXI4和AXi4-stream都支持三种握手机制，但其具体的总线结构是不同的，详情在后文中会介绍。这三种握手机制分别是：

***VALID before READY：***

![VALID before READY Mode](https://github.com/edger330/My_blog/tree/master/img/5791357-a0baa31bc5359adf.png)

上图中的模式为VALID信号先于READY信号拉高，此时数据在VALID信号和READY信号为高时，在时钟上升沿触发，开始传输。

-------

***READY before VALID：***

![READY before VALID Mode](https://github.com/edger330/My_blog/tree/master/img/5791357-73ee6e70e1ff37aa.png)

上图中的模式为READY信号先于VALID信号拉高，此时数据在VALID信号和READY信号为高时，在时钟上升沿触发，开始传输。

-----

***VALID with READY：***

![VALID with READY Mode](https://github.com/edger330/My_blog/tree/master/img/5791357-004a0da09790f386.png)

上图中的模式为READY信号伴随着VALID信号拉高，此时数据在VALID信号和READY信号为高时，在时钟上升沿触发，开始传输。

-------

## 1.4、Burst：

![Burst概念](https://github.com/edger330/My_blog/tree/master/img/5791357-963f33ae2721858e.png)

按照传统的RAM的读写方式，给定一个Address，只能读取或者写入一个Data，但是在Burst模式下，给定一个Address，可以连续写入或者读取多组数据。

-------------

# 2、AXI4总线：

## 2.1、AXI4接口：

**主设备接口**

![Master Interface](https://github.com/edger330/My_blog/tree/master/img/5791357-80576aaad0e282a1.png)

---------

**从设备接口**

![Slave Interface](https://github.com/edger330/My_blog/tree/master/img/5791357-4e3732c995a01f40.png)


---------

## 2.2、AXI4读操作：

![读通道架构](https://github.com/edger330/My_blog/tree/master/img/5791357-6953c0bc92bd8a3b.png)

如上图所示，主设备向从设备通过读地址通道指定读数据地址及控制信号，从设备通过读数据通道将指定地址上的数据传输给主设备。

---------

![Read Burst 流程](https://github.com/edger330/My_blog/tree/master/img/5791357-75e582738422f57d.png)

>在实际代码中我们采用有限状态机(*FSM*)来实现对相关信号的控制，这里采用的是VALID before READY握手模式。
>
>        parameter IDLE         = 5'b00001;
>        parameter WAIT_START   = 5'b00010;
>        parameter SEND_ADDR    = 5'b00100;
>        parameter RECEIVE_DATA = 5'b01000;
>        parameter CHECK_CONT   = 5'b10000;
>
>1、当状态机的当前状态为WAIT_START时，master将ARVALID拉高。
>
>2、slave收到ARVALID信号后，将ARREADY拉高，持续到一次burst_len传完为止。master收到ARREADY拉高的信号后，将ARVALID拉低。
>
>3、ARADDR在ARVALID为高时给定对应地址。
>
>4、RREADY信号在收到RVALID信号为高时拉高，保持一个周期，读取出数据。
>
>5、RVALID信号由slave控制，具体控制模式参考slave模块的设计。
>
>6、当一次读取的最后一个数据包读取时将RLAST拉高，表示一次Burst读取完毕。

-------

**其中读操作的信号依赖关系如下：**

![读操作信号依赖](https://github.com/edger330/My_blog/tree/master/img/5791357-8f290b4281cf52e5.png)

如图可知，读操作的两个channel之间存在如下的依赖关系：必须等到ARVALID和ARREADY同时为High后，RVALID才能拉高。

---------------

## 2.3、AXI4写操作：

![写通道架构](https://github.com/edger330/My_blog/tree/master/img/5791357-3585cfacbdb1e56b.png)

如上图所示，主设备向从设备通过写地址通道指定写数据地址及控制信号，从设备通过写数据通道将指定数据写到从设备的指定地址上。待数据写入完成后，从设备通过写响应通道向主设备传递写响应信号，表明写入完成。

---------

![Write Burst 流程](https://github.com/edger330/My_blog/tree/master/img/5791357-60e34f3f46b0daea.png)

>1、当状态机的当前状态为WAIT_START时，master将AWVALID拉高。
>
>2、slave收到AWVALID信号后，将AWREADY拉高，持续到一次burst_len写完为止.master收到AWREADY拉高的信号后，将AWVALID拉低。
>
>3、AWADDR在AWVALID为高时给定对应地址。
>
>4、WREADY信号在收到WVALID信号为高时拉高，保持一个周期，写入数据。
>
>5、WVALID信号由slave控制，具体控制模式参考slave模块说明。
>
>6、当一次写入的最后一个数据包读取时将WLAST拉高，表示一次写入完毕。
>
>7、BRESP和BVALID都由slave控制，当收到WLAST信号时，BVALID拉高。
>
>8、BREADY可以一直拉高，也可以在AWREADY信号拉高后保持拉高。直到BVALID信号拉高时将其拉低即可。

------

**其中写操作的信号依赖关系如下：**

![写操作依赖](https://github.com/edger330/My_blog/tree/master/img/5791357-1aeca32943659356.png)

如图可知，ADDR和DATA两个channel之间不存在依赖关系，需要满足的是必须等到WVALID和WREADY同时为High，且最后一次传输完成后，BVALID才能拉高，表明写操作结束。

---------

# 3、AXI4-stream总线：

## 3.1、AXI4-stream接口：

![Interface](https://github.com/edger330/My_blog/tree/master/img/5791357-410be0d2ecd31d6a.png)

AXI4-Stream跟AXI4的区别在于AXI4-Stream没有ADDR接口，这样就不涉及读写数据的概念了，只有简单的发送与接收说法，减少了延时，允许无限制的数据突发传输规模。AXI4-Stream的核心思想在于流式处理数据。

**一个AXI-stream传输的时序图：**

![AXI-stream example](https://github.com/edger330/My_blog/tree/master/img/5791357-71219b178c505070.png)


**其中AXI-stream一般的数据传输过程如下：**

1、首先slave将TREADY信号拉高，表示自己可以接收信号。

2、当master将TDATA，TKEEP，TUSER准备就绪之后，将TVALID拉高，传输开始。

3、其中TKEEP满足TKEEP[x] is associated with TDATA[(8x+7):8x]，当其被拉高时表示这段数据必须传输到目的地。TSTRB表示该段信息是否有效。TUSER可以在传递时捎带用户信息。具体接口参照使用的AXI-stream接口器件，并非所有支持AXI-stream接口的器件都含有以上接口，其中的一些接口是可选的而不是必需的。

4、直到master将TLAST拉高，TVALID拉低，传输结束。

-------

### 主要参考：

1、[AMBA-AXI总线协议详解--赵中民的博客](http://blog.sina.com.cn/s/blog_13f7886010102x2iz.html)

2、[AXI4-Stream协议总结](https://blog.csdn.net/calvin790704/article/details/53942363)

3、[AXI4_specification.pdf](http://www.gstitt.ece.ufl.edu/courses/fall15/eel4720_5721/labs/refs/AXI4_specification.pdf)



