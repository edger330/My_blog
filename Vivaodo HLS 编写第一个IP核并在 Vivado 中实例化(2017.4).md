
## 前言:

Xilinx Vivado~®~ 高层次综合(*High Level Synthesis*)工具将C语言转换为寄存器传输级(*RTL*)实现，并能够综合到Xilinx现场可编程逻辑门阵列(*FPGA*)中。可以使用C，C++，SystemC或开放计算语言(*OpenCL™*)API C内核编写C规范，并且FPGA提供了一个大规模并行体系结构，在性能，成本和功耗方面优于传统处理器 ~[1]~。

> **其主要优势有 :**
>
>        1、提高硬件设计的开发效率。
>
>        2、提高软件设计的系统性能。
>
>        3、在C语言层面完成算法开发并验证。
>
>        4、使用优化指令完成C语言到HDL的多个实现。
>
>        5、创建可读性和可移植性强的C语言代码。

---------------------

## 1、新建第一个project。

![点击create new project后出现的对话框](https://github.com/edger330/My_blog/tree/master/img/5791357-68bebff0d26c8d4b.png?raw=true)

按照图中配置后点击next，将出现 :

![添加文件对话框](https://github.com/edger330/My_blog/tree/master/img/5791357-cb28bedd16d2ee06.png?raw=true)

这里可以先添加文件，也可以选择直接next。**如果选择next在Vivado HLS中编辑C源文件后要记得添加Top Function**。这里演示先添加源文件的方法 :

![添加C源文件](https://github.com/edger330/My_blog/tree/master/img/5791357-6813f2243737d0bc.png?raw=true)

先点击Add File添加源文件(包含.cpp和.h)，添加完成后，点击Top Function后的Browser按钮，即可弹出Select Top Function的对话框，这里我们的Top Function是hier_func.cpp，点击ok，然后点击next。

![添加C-testbench](https://github.com/edger330/My_blog/tree/master/img/5791357-3d168cdba139dc42.png?raw=true)

可以在这里添加C-testbench也可以直接点击next在之后再添加testbench，我在这里直接点击next。然后出现如下对话框 :

![选择器件](https://github.com/edger330/My_blog/tree/master/img/5791357-24ee626b80a73814.png?raw=true)

在这里是提醒我们选择器件，点击part后面的...按钮，则会出现 :

![器件型号选择](https://github.com/edger330/My_blog/tree/master/img/5791357-9e0a765d8cb773b5.png?raw=true)

这里可以在左边的part里选择FPGA芯片具体型号，也可以在右边选择开发板型号，我选择的是ZYNQ-7 ZC702开发板，点击ok继续，点击finish。

-------------

## 2、我的C源码 :

>### 1) : `hier_func.h`
>
>        #ifndef _HIER_FUNC_H_
>        #define _HIER_FUNC_H_
>        
>        #include <stdio.h>
>        
>        #define NUM_TRANS 40
>        
>        typedef int din_t;
>        typedef int dint_t;
>        typedef int dout_t;
>        
>        void hier_func(din_t A, din_t B, dout_t *C, dout_t *D);
>        
>        void sumsub_func(din_t *in1, din_t *in2, dint_t *outSum, dint_t *outSub);
>        
>        void shift_func(dint_t *in1, dint_t *in2, dout_t *outA, dout_t *outB);
>        
>        #endif
>
>可以看出.h文件；里主要做了常量参数声明，自定义数据类型名，声明主要函数等工作。
>

----------

>### 2) : `hier_func.cpp`
>
>        #include "hier_func.h"
>        
>        void sumsub_func(din_t *in1, din_t *in2, dint_t *outSum, dint_t *outSub)
>        {
>        	*outSum = *in1 + *in2;
>        	*outSub = *in1 - *in2;
>        }
>        void shift_func(dint_t *in1, dint_t *in2, dout_t *outA, dout_t *outB)
>        {
>        	*outA = *in1 >> 1;
>        	*outB = *in2 >> 2;
>        }
>        void hier_func(din_t A, din_t B, dout_t *C, dout_t *D)
>        {
>        	dint_t apb, amb;
>        	sumsub_func(&A,&B,&apb,&amb);
>        	shift_func(&apb,&amb,C,D);
>        }
>

可以看出，*func* sumsub_func主要输出两个输入的和与差、*func* shift_func主要输出in1右移一位的值以及in2右移2位的值。在hier_func中调用两个子函数，最后的输出结果满足计算式 :

    C = (int)(A+B)/2
    
    D = (int)(A-B)/4

-------

## 3、规范型C-testbench编写 :

     #include "hier_func.h"
     int main() {
     // Data storage
     	int a[NUM_TRANS], b[NUM_TRANS];  //输入
     	int c_expected[NUM_TRANS], d_expected[NUM_TRANS];  //应该得到的输出结果
     	int c[NUM_TRANS], d[NUM_TRANS];  //实际得到的输出结果
     
     //Function data (to/from function)
     	int a_actual, b_actual;
     	int c_actual, d_actual;
     
     // Misc
     	int retval=0, i, i_trans, tmp;
     	FILE *fp;
     
     // Load input data from files
     //载入输入数据A的输入样本文件

     	fp = fopen("tb_data/inA.dat","r");
     	for (i=0; i<NUM_TRANS; i++){
     		fscanf(fp, "%d", &tmp);
     		a[i] = tmp;
     	}
     	fclose(fp);

     //载入输入数据B的输入样本文件
     	fp=fopen("tb_data/inB.dat","r");
     	for (i=0; i<NUM_TRANS; i++){
     		fscanf(fp, "%d", &tmp);
     		b[i] = tmp;
     	}
     	fclose(fp);
     
     // Execute the function multiple times (multiple transactions)
     //依次从样本文件中取出数据进行运算
     	for(i_trans=0; i_trans<NUM_TRANS-1; i_trans++){
     	//Apply next data values
     		a_actual = a[i_trans];
     		b_actual = b[i_trans];
     		hier_func(a_actual, b_actual, &c_actual, &d_actual);
     
     	//Store outputs
        //存储输出结果
     		c[i_trans] = c_actual;
     		d[i_trans] = d_actual;
     	}

     	// Load expected output data from files
        //从标准输出结果文件中载入输出结果C
     	fp=fopen("tb_data/outC.dat","r");
     	for (i=0; i<NUM_TRANS; i++){
     		fscanf(fp, "%d", &tmp);
     		c_expected[i] = tmp;
     	}
     	fclose(fp);

        //从标准输出结果文件中载入输出结果D     
     	fp=fopen("tb_data/outD.dat","r");
     	for (i=0; i<NUM_TRANS; i++){
     		fscanf(fp, "%d", &tmp);
     		d_expected[i] = tmp;
     	}
     	fclose(fp);
     
     	// Check outputs against expected
        //比较得到的输出结果和标准输出结果
     	for (i = 0; i < NUM_TRANS-1; ++i) {
     		if(c[i] != c_expected[i]){
     			retval = 1;
     		}
     		if(d[i] != d_expected[i]){
     		    retval = 1;
     		}
     	}

     	// Print Results
     	if(retval == 0){
     		printf(" *** *** *** *** \n");
     		printf(" Results are good \n");
     		printf(" *** *** *** *** \n");
     	}
     	else{
          	printf(" *** *** *** *** \n");     
            printf(" Mismatch: retval=%d \n", retval);
         	printf(" *** *** *** *** \n");
     	}
     
     	// Return 0 if outputs are corret
	    return retval;
     }

从上述testbench中我们可以看出，在进行仿真是我们还需要加入4个文件，分别是inA.txt，inB.txt，outC.txt，outD.txt，其中inA和inB作为输入的测试数据，outC和outD作为inA和inB输入后应该得到的标准输出结果。输入数据用space或者enter隔开即可，这四个文件可以按照下图放置 :

![test file 放置位置](https://github.com/edger330/My_blog/tree/master/img/5791357-7250a3b595b733ff.png?raw=true)

--------

## 4、HLS综合以及仿真 :

点击Run Synthesis(*图中绿色三角形*)

![点击C综合按钮](https://github.com/edger330/My_blog/tree/master/img/5791357-a3657d5d79763d1e.png?raw=true)

之后在左侧选项卡中会出现impl和syn文件夹 :

![synthesis得到的文件](https://github.com/edger330/My_blog/tree/master/img/5791357-008ba708e58cea02.png?raw=true)

在impl文件夹下或者syn文件夹下的verilog文件夹下会生成.v文件，在vhdl文件夹下会生成.vhd文件。这里以.v文件进行分析，生成的.v文件如下 :

    // ==============================================================
    // RTL generated by Vivado(TM) HLS - High-Level Synthesis from C, C++ and SystemC
    // Version: 2017.4
    // Copyright (C) 1986-2017 Xilinx, Inc. All Rights Reserved.
    // 
    // ===========================================================
    
    `timescale 1 ns / 1 ps 

    (* CORE_GENERATION_INFO="hier_func,hls_ip_2017_4,            
    {HLS_INPUT_TYPE=cxx,HLS_INPUT_FLOAT=0,HLS_INPUT_FIXED=0,
    HLS_INPUT_PART=xc7vx690tffg1761-    
    2,HLS_INPUT_CLOCK=10.000000,
    HLS_INPUT_ARCH=others,HLS_SYN_CLOCK=1.514000,
    HLS_SYN_LAT=0,HLS_SYN_TPT=none,HLS_SYN_MEM=0,
    HLS_SYN_DSP=0,HLS_SYN_FF=0,HLS_SYN_LUT=78}" *)
    
    module hier_func (
        ap_start,
        ap_done,
        ap_idle,
        ap_ready,
        A,
        B,
        C,
        C_ap_vld,
        D,
        D_ap_vld
    );


        input   ap_start;
        output   ap_done;
        output   ap_idle;
        output   ap_ready;
        input  [31:0] A;
        input  [31:0] B;
        output  [31:0] C;
        output   C_ap_vld;
        output  [31:0] D;
        output   D_ap_vld;

       reg C_ap_vld;
       reg D_ap_vld;

       wire   [31:0] apb_fu_54_p2;
       wire   [30:0] tmp_1_i_fu_66_p4;
       wire   [31:0] amb_fu_60_p2;
       wire   [29:0] tmp_3_i_fu_81_p4;

       always @ (*) begin
           if ((ap_start == 1'b1)) begin
               C_ap_vld = 1'b1;
           end else begin
               C_ap_vld = 1'b0;
           end
       end

       always @ (*) begin
           if ((ap_start == 1'b1)) begin
               D_ap_vld = 1'b1;
           end else begin
               D_ap_vld = 1'b0;
           end
       end

       assign C = $signed(tmp_1_i_fu_66_p4);

       assign D = $signed(tmp_3_i_fu_81_p4);

       assign amb_fu_60_p2 = (A - B);

       assign ap_done = ap_start;

       assign ap_idle = 1'b1;

       assign ap_ready = ap_start;

       assign apb_fu_54_p2 = (B + A);

       assign tmp_1_i_fu_66_p4 = {{apb_fu_54_p2[31:1]}};

       assign tmp_3_i_fu_81_p4 = {{amb_fu_60_p2[31:2]}};

       endmodule //hier_func

可以看出，核心语句只有这四句话 :

       assign amb_fu_60_p2 = (A - B);

       assign apb_fu_54_p2 = (B + A);

       assign tmp_1_i_fu_66_p4 = {{apb_fu_54_p2[31:1]}};

       assign tmp_3_i_fu_81_p4 = {{amb_fu_60_p2[31:2]}};

----------

点击Run C SImulation 按钮(*绿色三角形左边的按钮*)

![Run C SImulation 按钮](https://github.com/edger330/My_blog/tree/master/img/5791357-88fd0e2911df66ff.png?raw=true)

出现下图对话框，直接选择ok，这里主要选择仿真选项和配置argument，没有特殊需要直接选择ok即可。

![C SImulation configuration](https://github.com/edger330/My_blog/tree/master/img/5791357-a0b28cf3d0f2c87f.png?raw=true)

仿真结束后，则会弹出 : hier_func_csim.log，其内容为 :

    INFO: [SIM 2] *************** CSIM start ***************
    INFO: [SIM 4] CSIM will launch GCC as the compiler.
       Compiling ../../../testbench.cpp in debug mode
       Compiling ../../../hier_func.cpp in debug mode
       Generating csim.exe
     *** *** *** *** 
     Results are good 
     *** *** *** *** 
    INFO: [SIM 1] CSim done with 0 errors.
    INFO: [SIM 3] *************** CSIM finish ***************

可以看出CSim done with 0 errors，仿真通过。

---------

## 5、导出IP至Vivado :

点击下图中的Export 按钮，

![Export 按钮](https://github.com/edger330/My_blog/tree/master/img/5791357-9ec56536cef3b5da.png?raw=true)

则会弹出下图对话框 :

![Export RTL 对话框](https://github.com/edger330/My_blog/tree/master/img/5791357-b19f9a0a77328f47.png?raw=true)

按照需求配置后点击ok，Export完成后，则会弹出 :

![Export Report](https://github.com/edger330/My_blog/tree/master/img/5791357-a07ec39389c3aa4b.png?raw=true)

这个时候我们发现在solution/impl/ip目录下生成了一个.zip文件 :

![zip 文件](https://github.com/edger330/My_blog/tree/master/img/5791357-d6ede2901974c807.png?raw=true)

这个zip就是之后导入时需要用到的文件，现在我们打开Vivado 2017.4，新建一个Project，**在选择器件型号时记得选择与HLS中相同的器件型号。**

![New Project Summary](https://github.com/edger330/My_blog/tree/master/img/5791357-956797688c2b5ce3.png?raw=true)

点击Finish，完成新建工程之后，点击界面左侧的Settings :

![](https://github.com/edger330/My_blog/tree/master/img/5791357-5c83268f77615fa9.png?raw=true)

在弹出的页面下添加IP repositories。(*添加路径为在HLS工程中的/solution1/impl/ip*)

![添加ip](https://github.com/edger330/My_blog/tree/master/img/5791357-faa13c32749d59d3.png?raw=true)

之后在Block Design中就可以添加IP了 :

![](https://github.com/edger330/My_blog/tree/master/img/5791357-78bb103aa7db0536.png?raw=true)

![](https://github.com/edger330/My_blog/tree/master/img/5791357-de64d64e2dd08e98.png?raw=true)

-------

### 参考资料 :

1、[Vivado Design Suite User Guide - High-Level Synthesis](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2018_1/ug902-vivado-high-level-synthesis.pdf)

2、[Vivado Design Suite Tutorial - High-Level Synthesis](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2018_1/ug871-vivado-high-level-synthesis-tutorial.pdf)