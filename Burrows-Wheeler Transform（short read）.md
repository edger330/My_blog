

## 前言 :

**Burrows–Wheeler Transform**（简称BWT，也称作**块排序压缩**），是一个被应用在[数据压缩](https://zh.wikipedia.org/wiki/%E6%95%B0%E6%8D%AE%E5%8E%8B%E7%BC%A9 "数据压缩")技术（如[bzip2](https://zh.wikipedia.org/wiki/Bzip2 "Bzip2")）中的[算法](https://zh.wikipedia.org/wiki/%E7%AE%97%E6%B3%95 "算法")。该算法于1994年被[Michael Burrows](https://zh.wikipedia.org/w/index.php?title=Michael_Burrows&action=edit&redlink=1)和[David Wheeler](https://zh.wikipedia.org/w/index.php?title=David_Wheeler&action=edit&redlink=1)在位于加利福尼亚州帕洛阿尔托的[DEC系统研究中心](https://zh.wikipedia.org/w/index.php?title=DEC%E7%B3%BB%E7%BB%9F%E7%A0%94%E7%A9%B6%E4%B8%AD%E5%BF%83&action=edit&redlink=1)发明<sup>[[1]](https://zh.wikipedia.org/wiki/Burrows-Wheeler%E5%8F%98%E6%8D%A2#cite_note--1)</sup>。

当一个[字符串](https://zh.wikipedia.org/wiki/%E5%AD%97%E7%AC%A6%E4%B8%B2 "字符串")用该算法转换时，算法只改变这个字符串中字符的顺序而并不改变其字符。如果原字符串有几个出现多次的[子串](https://zh.wikipedia.org/wiki/%E5%AD%90%E4%B8%B2 "子串")，那么转换过的字符串上就会有一些连续重复的字符，这对压缩是很有用的。该方法能使得基于处理字符串中连续重复字符的技术（如[MTF变换](https://zh.wikipedia.org/wiki/Move-to-front_transform "Move-to-front transform")和[游程编码](https://zh.wikipedia.org/wiki/%E6%B8%B8%E7%A8%8B%E7%BC%96%E7%A0%81 "游程编码")）的编码更容易被压缩。

（上述摘自维基百科）

------------

本篇文章主要讲解BWT的具体算法，具体参考论文 :

>[Li H. and Durbin R. (2009) Fast and accurate short read alignment with Burrows-Wheeler Transform. Bioinformatics, 25:1754-60. [PMID: 19451168]](https://academic.oup.com/bioinformatics/article/25/14/1754/225615)

主要目的是理清算法的具体思路，进而了解BWA-MEM算法。

     (`限于本人水平有限，如有错误，欢迎指正！`)

--------------

## 1.BWA的主要流程 :

### (1)后缀数组

设源字符串为X，X = google $($表示空串)，


设将X进行单字符字典排序后的字符串为Y，Y = $eggloo，
我们先将X，Y生成如下字符串数组:

>     pos      循环左移                     pos   映射     
>              (数组1)                                    (数组2)
>      0      [g]oogle$          |          0    (6)    [e]$goog{l}  <--
>      1      [o]ogle$g          |          1    (5)    [g]le$go{o}  <--
>      2      [o]gle$go          |          2    (3)    [g]oogle{$}  <--
>      3      [g]le$goo          |          3    (0)    [l]e$goo{g}  <--
>      4      [l]e$goog          |          4    (4)    [o]gle$g{o}  <--
>      5      [e]$googl          |          5    (2)    [o]ogle${g}  <-- 
>      6      [$]google          |          6    (1)    [$]googl{e}  <--

取字符串数组2的每个元素的最后一个字符构成BWT变换字符串，令其为B，则B = lo$goge .

>### note：
>1、观察数组1，当pos=0时，字符串为google$，当pos等于n时，即代表将原字符串向左循环移动几位。我们可以发现，从上到下，数组的每个元素的第一个字符连起来也是google$。
>
>------
>
>2、我们将原pos做一个映射，使得生成的新的字符串数组(数组2)的每个元素的第一个字符连起来是$eggloo(Y)，这样就生成了后缀数组，且称后缀数组的每个元素的最后一个字符连起来形成的字符串(B)为BWT字符串。
>
>-------------
>
>3、在源字符串末尾添加$可以避免以下情况:
>
>        X = abab
>
>        映射得到的数组:
>
>      pos
>       0     [a]bab
>       1     [a]bab
>       2     [b]aba
>       3     [b]aba
>
>  当查找子串'aba'时就会发生错误，按上式形成的后缀数组，在pos=0和pos=1时都可查找到对应子串，而实际上子串只能出现在pos=1处，添加$符号分析即可知。
>
--------

### (2)查找

如果W是源字符串X的一个子串， __-R-(W)__ 表示区间的下界， __+R+(W)__ 表示区间的上界。

    例如 : 在X中查找g，在后缀数组中(数组2)对应的pos区间为[2,3]，则 : -R-(W) = 2，+R+(W) = 3.

令aW为在W之前加上字符a形成的新字符串，则 :

    -R-(aW) = C(a) + O(a,-R-(W) - 1) + 1          (1)
    +R+(aW) = C(a) + O(a,+R+(W))                  (2)

__当且仅当-R-(W) <= +R+(W)时，aW也是源字符串X的子串。__

>### note :
>
>C(a)和O(a, R(W))是根据BWT算法建立的表格查找函数
>
>其中C(a)的构造方式为 :
>
>先将X序列转换为Y序列，以google$为例，转换成$eggloo，则其中有$，e，g，l，o五种字符(__但建立C(a)时不考虑$，具体参见论文2.4节的第一句话__)，C(a)的值即为每种字符在Y序列第一次出现的位置，建立对应关系如下 :
>
>                         (Table1)
>         c:      |  e  |  g  |  l  |  o  |
>        C(c):    |  0  |  1  |  2  |  4  |
>                           
>
>----------
>
>O(a,R(W))的构造方式为 :
>
>表格横向表示字符在B串中的序号，纵向表示字符的种类，值表示字符在B串中当前位置之前出现的次数。以google$为例，其对应的B串为elo$gog。建立表格如下 :
>
>                            (Table2)
>            |  e  |  l  |  o  |  $  |  g  |  o  |  g  |
>            |  0  |  1  |  2  |  3  |  4  |  5  |  6  |
>        -----------------------------------------------
>        $   |  0  |  0  |  0  |  1  |  1  |  1  |  1  |   
>        e   |  1  |  1  |  1  |  1  |  1  |  1  |  1  |   
>        g   |  0  |  0  |  0  |  0  |  1  |  1  |  2  |   
>        l   |  0  |  1  |  1  |  1  |  1  |  1  |  1  |   
>        o   |  0  |  0  |  1  |  1  |  1  |  2  |  2  |   
>
>        例如(O(g,1)代表g字符行的第二个值，则为0；O(g,5)代表g字符行的第五个值，则为1 .)
>


--------

### (3)终止

终止一般有两种情况：Exact Match 和 Inexact Match

>1 · Exact Match
>
>    只要出现一个不匹配就终止。
>
>        例如源字符串为X = google$，查找子串W = gll.
>        先查找字符'l'，继续，
>        再查找字符串'll'，终止。
>
>--------------
>
>2 · Inexact Match
>
>    允许设定一个错误匹配的值z，在这个范围内都允许继续查找。
>
>        例如源字符串为X = google$，查找子串W = gll，z = 1.
>        先查找字符'l'，继续，
>        再查找字符串'll'，-R-(W) >= +R+(W)，z = z - 1 >=0，继续，
>        再查找字符串'gll'，-R-(W) <= +R+(W)，查找完毕，终止。
>


--------

## 2.算法流程实例 :

>例如 : __现有源字符串X = google$，需要查找子串W = og。__

    解：首先查找字符'g'，由经字典排序的字符串Y = $eggloo知
    
    g在后缀数组(数组2)中的区间(pos)为[2,3]，故-R-(g)  = 2，+R+(g) = 3.

    再查找'og'。由式子(1)(2)来计算-R-(og)，+R+(og)：
    ------------------------------------------------------------
        -R-(og) = C(o) + O(o,-R-(g) - 1) + 1          (1)
        +R+(og) = C(o) + O(o,+R+(g))                  (2)
    ------------------------------------------------------------
    C(o)通过查找Table1可知，其值为4.

    O(o,-R-(g) - 1) = O(o,1)
    O(o,+R+(g) = O(o,3)

    查找Table2可知，O(o,1) = 0，O(o,3) = 1.

    计算得 -R-(og) = 5 <= +R+(og) = 5，故'og'是源字符串的子串，且查到其后缀数组区间为[5,5]。

    对照后缀数组(数组2)的pos = 5的那一行的字符串，发现'og'正是该字符串的前两个字符。

----------

### 参考资料 :

>https://academic.oup.com/bioinformatics/article/25/14/1754/225615
>
>http://bio-bwa.sourceforge.net/
