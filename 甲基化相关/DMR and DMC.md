### DMC和DMR
DMC(differentially methylated cytosines)就是差异甲基化位点
DMR(differentially methylated region)就是差异甲基化区域

### methylKit获得DMC和DMR
methylKit我之前写过，具体用法看前面[[methyLkit入门使用]] and [[methyLkit进阶使用]]
methylKit适用于有重复的情况，而没有重复的情况这个R包就会自动调用fast.fisher()，似乎跟fisher检测有关，但是算出来的准确度并不好

### 用fisher检验来获得DMC和DMR
#### DMC
##### 1.得到CX_report
首先通过bismark_methylation_extractor获得CX_report文件，文件格式形如:
```
chr		location	+/-		C_count		T_count		methylation_type	 alignment
//染色体		位点		正负链		测得C总数	测得T总数	甲基化类型		序列
```

例如:
```
Chr1    1001    +       0       0       CHH     CTA
Chr1    1006    +       0       0       CHH     CCC
Chr1    1007    +       0       0       CHH     CCT
```

这个文件并不能直接使用，要过滤一遍测序是深度，一般选择测序深度大于等于5的位点(可以根据实际情况灵活更改)

##### 2.fisher检验得到DMC
R语言里面有个fisher.test()的函数，其中比较关键的参数是x和y.

x可以是一个二维maritx，也可以是一个数，而y只能是一个数，且x是二维maritx时将忽略y这个参数

在这里，我们设实验组A位点的C_count为C_count1，T_count为T_count1，同理对照组A位点的C_count为C_count2，T_count为T_count2.

那么对于A位点，应该有如下检验 `fisher.test(maritx(c(C_count1,T_count1,C_count2,T_count2),nrow=2))`

一般来说，如果pvalue<0.05则说明差异显著，说明A位点在实验组和对照组在甲基化方面存在差异，也就是说是差异甲基化位点 .根据实际情况，这里的pvalue的标准也不一定非要按这个来，可以按照实际情况来定

以上是对一个位点，判断其是否是差异甲基化位点.而其他的位点实际上也是如此判断.


#### 下面是对上面思路的实现，仅供参考
```
mergeCX.sh
//合并对照组和实验组的CX_report.txt，并且淘汰掉测序深度小于等于4的位点
awk '{if(NR==FNR && $4+$5>4){a[$1","$2]=$1"\t"$2"\t"$6"\t"$4"\t"$5}else if($1","$2 in a && $4+$5>4){print a[$1","$2],$4,$5}}' experimentalGroup.CX_report.txt controlGroup.CX_report.txt > merged.txt

`bash mergeCX.sh` 得到merged.txt

//fisherTest.r

data=read.table("merged.txt")
colnames(data)=c("chrom","position","context","C_count_1","T_count_1","C_count_2","T_count_2")

fun.fisher=function(c1,t1,c2,t2)
{
	fisher.test(maritx(c(c1,t1,c2,t2),nrow=2))$p.value
}
data$pvalue=mapply(fun.fisher,C_count_1,T_count_1,C_count_2,T_count_2)
data$qvalue=p.adjust(data$pvalue,method="fdr")
write.table(data,"fisherResult.txt",quote=F,sep="\t",row.name=F)

`Rscript fisherTest.r` 得到fisherResult.txt，筛选最后一列的qvalue(pvalue的修正值)
```

#### DMR
##### 1.获得一个统计各个区域中C_count和T_count的bed文件

写出一个根据CX_report文件，统计出给定区域内C_count和T_count的脚本，文件格式形如:

`chr region_start_site region_end_site C_CG_count T_CG_count C_CHG_count T_CHG_count C_CHH_count T_CHH_count`

`染色体 区域开始位点 区域结束位点 区域中CG测得C的总数 区域中CG测得T的总数 区域中CHG测得C的总数 区域中CHG测得T的总数 区域中CHH测得C的总数 区域中CHH测得T的总数` 例如:

```
//bed文件解释 第一列是染色体，第二列是区域开始位点-1，第三列是区域结束位点
//所以此例中，第一行代表的是从1-100这个区域
Chr7    0       100     0       0       0       0       0   0
Chr7    50      150     0       0       0       0       0   0
Chr7    100     200     0       0       0       0       0   0
Chr7    150     250     0       0       0       0       0   0
```

脚本代码量略大（之前的实现是用python，感觉有点费力不讨好的感觉，现在学习了pandas，并且熟悉了awk和bedtools，现在写这个会简单很多，实现方法也会好很多）

##### 2.对CX_report使用binom(二项式)检验，获得一个mC统计文件

对一个样品的某个位点的数据进行binom检验之前，需要获得这个样品数据的错误率.因为理论上叶绿体基因组上不应该有甲基化，所以在叶绿体上测得的总的C_count是出错的，而`C_count/(C_count+T_count)`就是这个样本的错误率.错误是由测序本身产生

对于binom检验，R中有函数`binom.test()`，其中比较关键的参数是x、n和p

x:可以是一个数，也可以是长度为2的向量.当是一个数时代表成功次数，在这里就是测得甲基化C的个数;当是向量是，代表成功数和失败数，在这里也就是测得甲基化C的个数和测得未甲基化C的个数(T的个数).

n:当x是一个数时，n代表尝试次数，这里也就是(测得甲基化C的个数+测得T的个数);当x是长度为2的向量时，此参数被忽略.

p:假成功的概率，这里也就是错误率

_注意，这里最好把结果输出为bed的格式，虽然这里表示的只是一个位点.如果用bed表示chr1中第2000号位点，那么应该是_ `chr1 1999 2000 ……`

然后binom检验所得p值，小于0.01的(这个标准也可以根据实际需要进行调整)，认为是mC(甲基化胞嘧啶)

以上是对一个位点的检验思路，每个样本的CX_report.txt中的位点都要如此检验一遍

建议最后生成的文件格式，形如:

```
chr		site-1	site	C_count		T_count		methylation_type	pvalue
```

建议生成bed格式的原因是方便使用intersectBed，处理快而且不用再写脚本

##### 3.统计区域bed文件中，各个区域内所包含的mC个数，并以此进行筛选

这一步得出每个区域内所包含的mC的个数的目的，是要针对mC对区域进行筛选，选择出符合mC数量条件的区域

这里就可以使用intersectBed这个工具了，这个工具是bedtools里面包含的工具，可以用于统计一个区域内包含的小区域，但是并不能直接统计位点，这也就是前面建议将位点写成bed格式的区域的原因

注意:如果有区分序列环境的需要，则需要提前将mC的类型分开，这也就是上一条中建议最后生成的文件格式里面带有methylation_type的原因.

如果需要区分序列环境，则建议此步骤生成的文件格式形如:

```
chr		region_start_site	region_end_site	C_CG_count	T_CG_count	C_CHG_count		T_CHG_count		C_CHH_count		T_CHH_count		methylation_type
```

可以不用输出mC的数量，mC的数量在以后的分析中并没有什么用，只是在这里有筛选的作用

##### 4.按照甲基化类型的不同，将上一条的结果文件分开(方便按照序列环境分别比对)

说白了就是把上一条的结果文件中，methylation_type为CG的，CHG的和CHH的分别拿出来放到三个文件里面.

这里可以用`awk '{print > $10} bin.filtrated(mc).txt'`来做这件事

_在这一步，注意顺便可以把一些冗余的信息去掉，比如: CG文件里面的C_CHG_count，T_CHG_count，C_CHH_count，T_CHH_count这几列的都可以去掉_

##### 5.fisher检验得DMR

事实上这一步的本质根计算DMC时fisher检验是一样的，同样都是拿实验组和对照组的C_count和T_count去做检验，最大的不同仅仅是DMC中检验检验的是位点而这里检验的是区域

这里就不再赘述了，详情看上面[[#2 fisher检验得到DMC]]


#### 下面是对上面思路的实现，仅供参考

##### 按照测序深度进行过滤

```
//filtrateCX_report.sh
for i in experimentalGroup controlGroup
	do
		awk '{if($4+$5>4){print $0}}' OFS="\t" $i.CX_report.txt > $i.filtrated.CX_report.txt
	done
```

`bash filtrateCX_report.sh`得到experimentialGroup.filtrateCX_report.txt和controlGroupCX_report.txt


##### 统计每个区域，得到bin.bed文件

[[#1 获得一个统计各个区域中C_count和T_count的bed文件]] 里面涉及的脚本文件CX2Bed.py代码量较大而且可读性较差，所以就不放到这里，点击下方链接获取![[../appendix/CX2Bed.py]]

`python3 CX2Bed.py`得到experimentialGroup.bin.bed和controlGroup.bin.bed，两者形如:

```
//chr	start_site	end_site	C_CG_count	T_CG_count	C_CHG_count	T_CHG_count	C_CHH_count	T_CHH_count
Chr7	0		100		0		0		0		0		0		0
```


##### binom检验得到mC
```
// getErrorRate.sh
for i in experimentalGroup controlGroup
	do
		echo $i
		awk '{if($1~/ChrPt/){a+=$4;b+=$5} END{print a/(a+b)}}' $i.CX2Bed.txt >> errorRate.txt
	done
```

`bash getErrorRate.sh`得到errorRate.txt

```
// binomTest.r 这个脚本需要两个参数:inputFilePath outputFilePath errorRate
args=commandArgs(T)

data=read.table(args[1])

colnames=c("chr","location","+/-","C_count","T_count","methylation_type","alignment")

fun.binomTest=function(meth,total,errorRate)
{
	fun.binom.test(meth,total,p=errorRate)$p.value
}

meth=data$C_count
total=data$C_count+data$T_count
errorRate=as.numeric(args[3])

data$locForward=data$location-1
data$pvalue=mapply(fun.binomTest,meth,total,errorRate)

data_output=data[,c(1,8,2:7)]
write.table(data_output,args[2],sep="\t",quote=F,col.name=F,row.name=F)

// get_mC.py
import os

errorRateFile=open("errorRate.txt")
errorRateLine=errorRate.readlines()

errorRateDict={}
taglist=[]

for i in range(len(errorRateLine),step=2)
	tag=errorRateLine[i]
	taglist.append(tag)
	errorRate=errorRateLine[i+1]
	errorRateDict[tag]=errorRate
	
for j in taglist:
	foo="Rscript binomTest.r {0}.filtrated.CX_report.txt {0}_mC.txt {1}".format(l,errorRateDict[l])
	print foo
	os.system(foo)
```


`python get_mC.py`得到experimentialGroup_mC.txt和controlGroup_mC.txt，两者形如:

```
//chr	start_site	end_site	+/-		C_count		T_count		methylation_type	 alignment	pvalue
Chr1	1062		1063		+		8		0		CHH			CAA		7.69070904377531e-13
```

##### 根据mC，对区域进行过滤
```
//filtrateBin.sh
for i in experimentalGroup controlGroup
	do
		intersectBed -a $i.bin.txt -b $i_mC.txt -wo |\
		cut -f 1,2,3,4,5,6,7,8,9,16 | sort | uniq -c |\
		awk '{if($1>4){print $2,$3,$4,$5,$6,$7,$8,$9,$10}}' $i.bin.bed > $i.filtratedBin.bed
	done
```

这里是按照mC>4的标准对区域进行筛选，这个标准可以改

`bash filtrateBin.sh`可以得到experimentalGroup.filtratedBin.bed和controlGroup.filtratedBin.bed

文件格式形如:

```
//chr	start_site	end_site	C_CG_count	T_CG_count	C_CHG_count	T_CHG_count	C_CHH_count	T_CHH_count		methylation_type
```

##### 按照methylation_type，分割filtratedBin文件
```
//splitBin.sh
mkdir experimentalGroup controlGroup
for i in experimentalGroup controlGroup 
	do
		mv $i.filtratedBin.bed $i
		awk '{print >$10}' ./$i.filtratedBin.bed 
	done
```

`bash splitBin.sh`得到experimentialGroup/CG，experimentialGroup/CHG，experimentialGroup/CHH和controlGroup/CG，controlGroup/CHG，controlGroup/CHH

这里写脚本的时候注意要把结果文件放到两个文件夹，以防覆盖


##### 实验组CG，CHG，CHH与对照组的合并，并做fisher检验
```
//cleanCGCHGCHH.sh
for i in experimentialGroup controlGroup 
	do
		awk '{print $1,$2,$3,$4,$5}' ./$i/CG > ./$i/CG_clean
		awk '{print $1,$2,$3,$6,$7}' ./$i/CG > ./$i/CHG_clean
		awk '{print $1,$2,$3,$8,$9}' ./$i/CG > ./$i/CHH_clean
	done
```

`bash cleanCGCHGCHH.sh`得到CG_clean，CHG_clean，CHH_clean

```
//mergeExperimentalControl.sh
for j in CG CHG CHH
	do
		awk '{if(NR=FNR){a[$1","$2]=$0}else{if($1","$2 in a){print a[$1","$2],$4,$5}}}' experimentialGroup/$j controlGroup/$j > $j_merge
	done
```


`bash mergeExperimentalControl.sh`得到CG_merge，CHG_merge，CHH_merge

```
//fisherTest.r 有两个参数:inputFilePath outputFilePath 
args=commandArgs(T)

data=read.table(args[1])
colnames(data)=c("chrom","position","context","C_count_1","T_count_1","C_count_2","T_count_2")

fun.fisher=function(c1,t1,c2,t2)
{
	fisher.test(maritx(c(c1,t1,c2,t2),nrow=2))$p.value
}
data$pvalue=mapply(fun.fisher,C_count_1,T_count_1,C_count_2,T_count_2)
data$qvalue=p.adjust(data$pvalue,method="fdr")
write.table(data,args[2],quote=F,sep="\t",row.name=F)

```


```
//bash中直接执行
Rscript fisherTest.r CG_merge CG.fisherResult.txt
Rscript fisherTest.r CHG_merge CHG.fisherResult.txt
Rscript fisherTest.r CHH_merge CHH.fisherResult.txt
```


# 注意，Rscript执行的R程序基本都是做统计学检验，检验挺慢，特别是fisher检验，耐心等待
