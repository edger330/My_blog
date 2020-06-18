

# 前言 :

随着[**NGS**](https://en.wikipedia.org/wiki/DNA_sequencing#Next-generation_methods)(Next Generation Sequencing)技术日益成熟，如今一台设备一天可产生100G的碱基匹配数据。其产生的数据为随机片段化DNA碱基对，我们称其为short read。为了能够重组出原DNA序列，我们需要将大量的(millions of)short read 匹配到参考基因序列上。如今NGS系统的一个瓶颈即为short read 的匹配问题。下文中我们主要介绍一种基于哈希索引的短序列匹配算法的主要思路。

    `(鉴于本人水平有限，如有错误欢迎指正！)`

# 1、方法原型

该算法的基本原型为BFAST~[1]~，其主要有四个主要步骤 :

---------

## <1>  建立参考序列索引  (*Index the Reference*)

>以参考序列(Reference)为**AATCCCGATTACAGGGATT**为例
>
>则可建立下列索引 :
>
>        start            strings
>          0        AATCCCGATTACAGGGATT
>          1         ATCCCGATTACAGGGATT
>          2          TCCCGATTACAGGGATT
>          3           CCCGATTACAGGGATT
>          4            CCGATTACAGGGATT
>          5             CGATTACAGGGATT
>          6              GATTACAGGGATT
>          7               ATTACAGGGATT
>          8                TTACAGGGATT

-------

## <2>  建立CALs Table  (*candidate alignment locations*)

>在这一步骤中，我们需要一个**hash musk**，在这里我们取**musk = 10100000011**，则有 :
>
>
>        order    start      1 0 1 000000 1  1              substr
>          0        1       [A]T[C]CCGATT[A][C]AGGGATT       ACAC
>          1        7       [A]T[T]ACAGGG[A][T]T             ATAT
>          2        0       [A]A[T]CCCGAT[T][A]CAGGGATT      ATTA
>          3        5       [C]G[A]TTACAG[G][G]ATT           CAGG
>          4        3       [C]C[C]GATTAC[A][G]GGATT         CCAG
>          5        4       [C]C[G]ATTACA[G][G]GATT          CGGG
>          6        6       [G]A[T]TACAGG[G][A]TT            GTGA
>          7        8       [T]T[A]CAGGGA[T][T]              TATT
>          8        2       [T]C[C]CGATTA[C][A]GGGATT        TCCA
>
>具体建表过程为 :
>
>先将第一步得到的strings与musk做与运算，得到substr，对substr做**字典排序**，排成从上到下order依次增加的顺序。最后找到substr对应的string的start位置，完成order到start的映射。

------------

## <3>建立Hash Table

>这里作为例子只建立碱基长度为2(2-mer)的Hash Table，建表如下 :
>
>        No.    2-mer    order start    order end
>         1       AA          -             -
>         2       AC          0             0
>         3       AG          -             -
>         4       AT          1             2
>         5       CA          3             3
>         6       CC          4             4
>         7       CG          5             5
>         8       CT          -             -
>         9       GA          -             -
>         10      GC          -             -
>         11      GG          -             -
>         12      GT          6             6
>         13      TA          7             7
>         14      TC          8             8
>         15      TG          -             -
>         16      TT          -             -
>
>具体建表过程为 :
>
>先单列出碱基长度为2(2-mer)的所有碱基情况，并将其按**字典顺序**排列，即可得到上表中的(2-mer)列。我们以2-mer中的元素为前缀，查找前缀为2-mer列中元素的CALs Table中的substr的order。
>
>        例如 : 当2-mer为AT时，我们在CALs Table中的substr中查找前缀为AT的字符串，
>        结果找到了order为1和2的substr的前缀为AT，则其order start为1，order end为2.
>
>        当2-mer为AA时，我们发现在CALs Table中的substr中查找不到前缀为AA的字符串，
>        则其order start为 - ，order end为 - .
>

---------

## <4>进行匹配

>具体的匹配规则可参见下表，表中我们令**short read = ATCCCGATTATAG**，则 :
>
>        offset      A T C C C G A T T A T A G      match?
>          0         A   C             A T            x
>          1           T   C             T A          x
>          2             C   C             A G        o
>
>具体匹配过程为 : 先将short read的第一位与musk(10100000011)的第一位对齐，然后做与运算得到**read substr = ACAT**。如上图中所示，第一步我们得到了ACAT，取其前两个字符，即为AC，我们在Hash Table中查找AC，发现其order start = 0，order end = 0。
>
>我们利用已知的order start 和order end信息，在CALs Table中查找order，得到该order start和order end对应的substr为ACAC，拿得到的ACAC与read substr做比较，发现最后一位不匹配，匹配失败。
>
>-----------
>
>此时我们开始第二次匹配，将short read的第二位与musk(10100000011)的第一位对齐，注意到此时的offset为1，即short read相较于musk偏移了一位。之后做与运算得到**read substr = TCTA**，取前两个字符TC并在Hash Table中查找，得到order start = 8，order end = 8。
>
>利用order start 和order end，在CALs Table中查找order，得到对应的substr为TCCA，拿TCCA与此时的read substr比较，发现第三位不匹配，匹配失败。
>
>------------
>
>接着开始第三次匹配，将short read的第三位与musk(10100000011)的第一位对齐，注意到此时的offset为2，得到**read substr = CCAG**，同理在Hash Table中查找CC，发现其order start = 4，order end = 4。
>
>利用order start 和order end，在CALs Table中查找order，得到对应的substr为CCAG，拿CCAG与此时的read substr比较，匹配成功。

--------

# 2、方法优化

优化的主要思路是采用FPGA对BFAST进行加速~[2]~，并在一些细节上进行修正。主要方法有 :

## <1>  Seeds并行比较  (*Parallel Comparison of the Seeds*)

这一方面的优化主要是利用FPGA的并行计算的优势，通过在FPGA片上堆积资源，构建比较器阵列(*Comparator Array*)，从而达到Seeds的并行比较的效果。相较于CPU，其具体的加速实现机制为 :

>### A.排序建表(*Sorting*)
>
>![桶排序](https://github.com/edger330/My_blog/tree/master/img/5791357-2bf666e568ae71e2.png?raw=true)
>
>由上图所示，对于长度为100bp的short read片段，其存储方式为1个bp使用3bit二进制数据存储，故其长度为100 x 3b(b表示为bit)，同时传入到该模块的还有24bit长度的short read ID，该ID用来唯一标识该short read。
>
>首先我们利用hash musk对short read进行与操作，得到长度为14bit的Addr和长度为32bit的Key(其位拼接之后合称seed)。再将Addr按照字典顺序(*lexical order*)进行排序，查找到该seed的Addr在bucket table中的位置，然后将该seed的Key挂载在对应的bucket中。**其中一个bucket大小为16 x (32+7+1+24)bit，32bit表示Key的长度，7bit表示该Key的seed产生时的offset，1bit用于之后标记该Key是否match，24bit用于表示该Key的seed来源于的short read的ID。**当bucket满了之后，将bucket中的数据移至write buffer中，当write buffer达到P words大小时，将其写入到外部DRAM中。这样，即使用FPGA完成了bucket sort，并将结果存入到外部的DRAM中，当我们需要取出Key时，我们先通过bucket table查找到当前Addr对应的Key存储在DRAM中的位置，再将相同Addr的Key一并取出。
>
>                              <----note---->:
>        1、在此处我们取的hash musk的Key size为23，即hash musk中的1的个数有23个，
>        经过与操作之后，原本的100个碱基变成了23个(23 x 2bit)，人为地将23bp拆成7+16，
>        令前7bp(7 x 2bit = 14bit)为Addr，后16bp(16 x 2bit = 32bit)为Key。
>
>        2、counter的作用是当对short read进行移位操作时，每偏移一次，counter++。
>        由于长度为100的short read和Key size为23的hash musk移位与运算时，
>        最多产生100 - 23 + 1 = 78条seed，故counter的最大值为78，所以        
>        此处取counter为7位，最大值可达128。
>
>------------------
>
>### B.Key值并行比较(*Parallel Key Comparison*)
>
>![比较器阵列](https://github.com/edger330/My_blog/tree/master/img/5791357-f7adb3d3d8ace85d.png?raw=true)
>
>由上图所示，Key1表示short read中产生的Key，Key2表示由reference生成的CAL表中的Key。匹配时第一次匹配Key1.1和Key2.1，第二次匹配Key1.1和Key2.2以及Key1.2和Key2.1，......，第P次匹配Key1.1和Key2.P；匹配Key1.2和Key2.P-1；......；匹配Key1.P和Key2.1。如此达到并行比较Key值的效果，这样将比较的时间复杂度由O(mn)降低到O(m+n)。其具体的比较过程参见参考资料[4]中的Fig.6。比较器阵列的输出为short read ID，seed在short read中的位置，seed在reference genome中的位置。
>

--------------

## <2>  减少CAL表中候选位置  (*Reducing the number of candidate locations in the CAL table*)

由于至少只需要short read的一个seed能给出正确的位置就足以完成匹配，故我们不必将reference genome中的所有seeds注册到CAL表中。事实上，在BFAST算法中，对于那些能够匹配到多于8个候选位置的seeds将不会被注册到CAL table中。在[2]中主要论述了以下方法来减少CAL表中的候选位置 :

    1)在reference genome中出现次数超过32次的seed将不会被注册至CAL表中。

    2)所有的reference genome中的seeds从顶部开始，将被分成L个seeds一组。

    3)在每组中，从最后一个seed开始扫描，只有第一个满足注册条件的seed将会被注册到CAL table中。

    4)当第n个group没有能够注册的seed时，对于第(n+1)个group从第一个seed开始扫描，
      第一个满足注册条件的seed将被注册到CAL table中。

    -----------------------(上述方法可将CAL表压缩为原来的1/L)---------------------

    5)我们在比较Key值时，允许那些在reference genome中出现了超过32次的seeds
      在CAL table中查找时存在一个碱基的替换。之所以选择这些seeds，
      源于它们在reference genome出现过多次，这代表它们在short read中也出现了多次。

    6)如果其相邻的seeds，距离为Dm (distance from the key on the genome) 已经在table中注册，
      那么对于任何一个这样的Key，如果其能够匹配到超过Tm个seed，则其将被移出CAL table。

    ----------(上述方法旨在删除那些short read中能够匹配到多个候选位置的seeds，
               同时保持已注册的候选位置之间保持最大距离。-----------

-----------

### 主要参考 :

1、[*BFAST: An Alignment Tool for Large Scale Genome Resequencing.*](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0007767)

2、[*An Acceleration Method of Short Read mapping using FPGA.*](http://ieeexplore.ieee.org/abstract/document/6718385/)

3、[*FPGA acceleration of short read mapping based on sort and parallel comparison.*](http://ieeexplore.ieee.org/abstract/document/6927404/)

4、[*A Fast and Accurate FPGA System for Short Read Mapping Based on Parallel Comparison on Hash Table*](https://www.jstage.jst.go.jp/article/transinf/E100.D/5/E100.D_2016EDP7262/_article/-char/ja/)

