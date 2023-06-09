### 介绍
在群体遗传学中衡量群体间的遗传分化的程度的指标有许多种，较为常见的就是**遗传分化指数（Fst）**，fst是由F统计量演变而来，F统计量主要有三种（FIS，FIF，FST）。Fst是针对一对等位基因，如果基因座上存在复等位基因，则需要用Gst衡量，基因差异分化系数（gene differentiation coefficient，Gst）。假定有s个地方群体，第k个地方群体相对大小为wk，第k个地方群体中第i个等位基因频率为qk(i)，杂合体频率观察值为hk，那么整个群体中观察到的杂合体频率平均值HI，地方群体为理想群体的期望杂合体频率平均值HS，整个群体为理想群体的期望杂合体频率HT，分别为：

FIS，是HI相对于HS减少量的比值，即地方群体的平均近交系数。FIS是群体遗传学中的另一个统计指标，用于衡量群体内的近交程度。FIS的值从-1到1变化，0表示群体内是随机交配的，杂合度符合哈温平衡；正值表示群体内存在近交或者自交，杂合度低于哈温平衡；负值表示群体内存在异交或者超级杂合子，杂合度高于哈温平衡。FIS可以从基因型数据计算得到

FST，是HS相对于HT减少量的比值，即有亲缘关系地方群体间的平均近交系数。

FIT，是HI相对于HT减少量的比值，即整个群体的平均近交系数。

**Fst值的取值范围是【0,1】，最大值为1表明两个群体完全分化，最小值为0表明群体间无分化。**

**在实际的研究中Fst值为0--0.05时说明群体间遗传分化很小，可以不做考虑；**

**为0.05--0.15时，表明群体间存在中等程度的遗传分化；**

**为0.15--0.25时群体间存在较大的遗传分化；**

**为0.25以上的时候群体间就存在很大的遗传分化了。**

### 计算
**计算FST值有两种情况：一是snp单点计算**

```
vcftools --vcf test.vcf --weir-fst-pop population_1.txt --weir-fst-pop population_2.txt --out P_1_2
```

其中--vcf 是输入所需要计算的群体的输入文件，注意是vcf格式的

--weir-fst-population 这个命令是输入第一个群体文件，注意是txt文件格式。即population_1.txt，此文件只包含一列，就是群体个体的ID。population_2.txt也是一样的，是第二个群体的个体的ID。

**第二种情况就是按照区域（窗口式）计算**

```
vcftools --vcf test.vcf --weir-fst-pop population_1.txt --weir-fst-pop population_2.txt --out P_1_2 --fst-window-size 500000 --fst-window-step 50000
```

这个窗口式的计算，就是在后面加上窗口的大小和步长，例如我上述的--fst-window-size 500000 --fst-window-step 50000  窗口设置为500kb，步长设置为50kb。这个窗口的设置没有一个固定的标准和要求，都是按照自己的需要而定。

一般认为Fst值在top 1%的位点作为正向选择的位点