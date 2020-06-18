
# 1.基本算法(摘自维基百科)：

## 1.1空位权值恒定模型算法

![空位权值恒定模型算法](./img/5791357-1047eb2e05f5712c.png?raw=true)

## 1.2通用算法

![通用模型算法](./img/5791357-df98a12a770d659e.png?raw=true)

其中H(i,j)是最终的得分矩阵。F(i,j)和E(i,j)矩阵分别用来存储在两条比对序列上开辟空位延伸比对的消耗(cost)。o代表第一个空位的罚分值，e代表延伸时的罚分值。
***(原文内容：In (3) and (4), o denotes the gap opening penalty, while e represents the gap extension penalty. The matrices F(i, j) and E(i, j) contain the trace of opening and extending a gap, respectively. F(i, j) stores the cost for opening a gap and extending a gap on sequence x, while E(i, j) stores the cost for opening a gap and extending a gap on sequence y)***

# 2.举例说明：

**比对序列为：A = TGTTACGG，B = GGTTGACTA**

## 2.1确定置换矩阵和空位罚分办法

```
置换矩阵s(ai,bj)= {+3，ai==bj }
　　   　　　　　　{-3，ai!=bj }
```
即碱基匹配时分数+3，不匹配时分数-3

    空位罚分Wk=(k-1)+2

即第一个空位得分-2，随后得分递增减１，即连续两个空位得分-3，连续三个空位得分-4...

## 2.2创建矩阵并初始化

需要创建的矩阵有H(得分矩阵)，E(B序列空位延伸罚分矩阵)，F(A序列空位延伸罚分矩阵)。

在初始化时一般会在H矩阵的最左上方空位赋一个初值(8)，右侧值和下侧值为拓展空位罚分的数值。

***(H矩阵初始化)***
```
              (Matrix　H)
            |T |G |T |T |A |C |G |G |   (A)
   ------------------------------------------
         |8 |6 |5 |4 |3 |2 |1 |0 |0 |
    G    |6 |x |x |x |x |x |x |x |x |
    G    |5 |x |x |x |x |x |x |x |x |
    T    |4 |x |x |x |x |x |x |x |x |
    T    |3 |x |x |x |x |x |x |x |x |
    G    |2 |x |x |x |x |x |x |x |x |
    A    |1 |x |x |x |x |x |x |x |x |
    C    |0 |x |x |x |x |x |x |x |x |
    T    |0 |x |x |x |x |x |x |x |x |
    A    |0 |x |x |x |x |x |x |x |x |
    
   (B)
```
***(E矩阵初始化)***
```
              (Matrix　E)
            |T |G |T |T |A |C |G |G |   (A)
   ------------------------------------------
         |  |  |  |  |  |  |  |  |  |
    G    |  |0 |0 |0 |0 |0 |0 |0 |0 |
    G    |  |x |x |x |x |x |x |x |x |
    T    |  |x |x |x |x |x |x |x |x |
    T    |  |x |x |x |x |x |x |x |x |
    G    |  |x |x |x |x |x |x |x |x |
    A    |  |x |x |x |x |x |x |x |x |
    C    |  |x |x |x |x |x |x |x |x |
    T    |  |x |x |x |x |x |x |x |x |
    A    |  |x |x |x |x |x |x |x |x |
    
   (B)
```
***(F矩阵初始化)***
```
              (Matrix　F)
            |T |G |T |T |A |C |G |G |   (A)
   ------------------------------------------
         |  |  |  |  |  |  |  |  |  |    
    G    |  |0 |x |x |x |x |x |x |x |
    G    |  |0 |x |x |x |x |x |x |x |
    T    |  |0 |x |x |x |x |x |x |x |
    T    |  |0 |x |x |x |x |x |x |x |
    G    |  |0 |x |x |x |x |x |x |x |
    A    |  |0 |x |x |x |x |x |x |x |
    C    |  |0 |x |x |x |x |x |x |x |
    T    |  |0 |x |x |x |x |x |x |x |
    A    |  |0 |x |x |x |x |x |x |x |
    
   (B)
```

## 2.3打分

由通用算法的计算式知：
**E(0,1)=max(H(0,0)-2=8-2,E(0,0)-1=0-1)=6**
**E(1,1)=max(H(1,0)-2=6-2,E(1,0)-1=0-1)=4**
**F(1,0)=max(H(0,0)-2=8-2,F(0,0)-1=0-1)=6**
**F(1,1)=max(H(0,1)-2=6-2,F(0,1)-1=0-1)=4**
**H(1,1)=max(0,H(0,0)=8-3,E(1,1),F(1,1))=5**
**......**

第一轮打分后：

***(H矩阵)***
```
              (Matrix　H)
            |T |G |T |T |A |C |G |G |   (A)
   ------------------------------------------
         |8 |6 |5 |4 |3 |2 |1 |0 |0 |
    G    |6 |5 |9 |x |x |x |x |x |x |
    G    |5 |3 |x |x |x |x |x |x |x |
    T    |4 |x |x |x |x |x |x |x |x |
    T    |3 |x |x |x |x |x |x |x |x |
    G    |2 |x |x |x |x |x |x |x |x |
    A    |1 |x |x |x |x |x |x |x |x |
    C    |0 |x |x |x |x |x |x |x |x |
    T    |0 |x |x |x |x |x |x |x |x |
    A    |0 |x |x |x |x |x |x |x |x |
    
   (B)
```
***(E矩阵)***
```
              (Matrix　E)
            |T |G |T |T |A |C |G |G |   (A)
   ------------------------------------------
         |  |  |  |  |  |  |  |  |  |  
    G    |  |0 |0 |0 |0 |0 |0 |0 |0 |
    G    |  |6 |4 |3 |2 |1 |0 |0 |0 |
    T    |  |5 |3 |7 |x |x |x |x |x |
    T    |  |4 |x |x |x |x |x |x |x |
    G    |  |3 |x |x |x |x |x |x |x |
    A    |  |2 |x |x |x |x |x |x |x |
    C    |  |1 |x |x |x |x |x |x |x |
    T    |  |0 |x |x |x |x |x |x |x |
    A    |  |0 |x |x |x |x |x |x |x |
    
   (B)
```
***(F矩阵)***
```
              (Matrix　F)
         |T |G |T |T |A |C |G |G |   (A)
   ------------------------------------------
    G    |0 |6 |5 |4 |3 |2 |1 |0 |
    G    |0 |4 |3 |x |x |x |x |x |
    T    |0 |3 |2 |x |x |x |x |x |
    T    |0 |2 |x |x |x |x |x |x |
    G    |0 |1 |x |x |x |x |x |x |
    A    |0 |0 |x |x |x |x |x |x |
    C    |0 |0 |x |x |x |x |x |x |
    T    |0 |0 |x |x |x |x |x |x |
    A    |0 |0 |x |x |x |x |x |x |
    
   (B)
```
继续计算直到完整填充......

***(H矩阵)***
```
              (Matrix　H)
            |T |G |T |T |A |C |G |G |   (A)
   ------------------------------------------
         |8 |6 |5 |4 |3 |2 |1 |0 |0 |
    G    |6 |5 |9 |7 |6 |5 |4 |4 |3 |
    G    |5 |3 |8 |6 |5 |4 |3 |7 |7 |
    T    |4 |8 |6 |11|9 |9 |8 |7 |6 |
    T    |3 |7 |x |x |x |x |x |x |x |
    G    |2 |5 |x |x |x |x |x |x |x |
    A    |1 |4 |x |x |x |x |x |x |x |
    C    |0 |3 |x |x |x |x |x |x |x |
    T    |0 |3 |x |x |x |x |x |x |x |
    A    |0 |x |x |x |x |x |x |x |x |
    
   (B)
```
***(E矩阵)***
```
              (Matrix　E)
         |T |G |T |T |A |C |G |G |   (A)
   ------------------------------------------
    G    |0 |0 |0 |0 |0 |0 |0 |0 |
    G    |6 |4 |3 |2 |1 |0 |0 |0 |
    T    |5 |3 |7 |5 |4 |3 |2 |2 |
    T    |4 |2 |6 |4 |3 |2 |1 |5 |
    G    |3 |6 |x |x |x |x |x |x |
    A    |2 |5 |x |x |x |x |x |x |
    C    |1 |4 |x |x |x |x |x |x |
    T    |0 |3 |x |x |x |x |x |x |
    A    |0 |2 |x |x |x |x |x |x |
         |0 |1 |x |x |x |x |x |x |
    
   (B)
```
***(F矩阵)***
```
              (Matrix　F)
         |T |G |T |T |A |C |G |G |   (A)
   ------------------------------------------
    G    |0 |6 |5 |4 |3 |2 |1 |0 |0 |
    G    |0 |4 |3 |7 |6 |5 |4 |3 |2 |
    T    |0 |3 |2 |6 |5 |4 |3 |2 |5 |
    T    |0 |2 |6 |5 |7 |9 |8 |7 |6 |
    G    |0 |1 |x |x |x |x |x |x |x |
    A    |0 |0 |x |x |x |x |x |x |x |
    C    |0 |0 |x |x |x |x |x |x |x |
    T    |0 |0 |x |x |x |x |x |x |x |
    A    |0 |0 |x |x |x |x |x |x |x |
    
   (B)
```

这里不再计算，留给读者自己验证……

--------
### 参考资料：

1.[Optimized and Portable FPGA-Based Systolic Cell Architecture for Smith–Waterman-Based DNA Sequence Alignment](http://www.kpubs.org/article/articleMain.kpubs?articleANo=E1ICAW_2016_v14n1_26)

2.[Wiki-en](https://en.wikipedia.org/wiki/Smith%E2%80%93Waterman_algorithm)