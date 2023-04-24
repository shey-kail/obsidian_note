检测基因组选择信号的方法有很多种，其中 XP-CLR 方法是常用的一种。XP-CLR 是陈华老师、Nick Patterson 和 David Reich 在 2010 年发表的方法，全称叫 the cross-population composite likelihood ratio test（跨群体复合似然比检验），是一种是基于选择扫荡（selective sweeep）的似然方法。选择扫荡可以增加群体之间的遗传分化，导致等位基因频率偏离中性条件下的预期值。

XP-CLR 利用了两个群体之间的多基因座等位基因频率差异（multilocus allele frequency differentiation）建立模型，使用布朗运动来模拟中性下的遗传漂移，并使用确定性模型来近似地对附近的单核苷酸多态性（SNPs）进行选择性扫描。

XP-CLR 不仅可以用在人类上，它在动植物驯化研究中也用得较多，比如玉米、大豆、家犬、牛等。

### 1. 安装
```
wget https://reich.hms.harvard.edu/sites/reich.hms.harvard.edu/files/inline-files/XPCLR.tar
tar xvf XPCLR.tar
cd XPCLR
cd src
make
make install
```

### 2. 输入文件
XP-CLR 的计算需要2个 geno 文件和 1 个.snp (map) 文件。
#### .geno file
一个群体的基因型数据放在一个 geno 文件中。每一行包含一个 SNP 的 genotype（0或1），每两列代表一个人。数据可以是 phased 的，也可以未 phased。如果是 phased 后的，每一列是一个 haplotype。如果是未 phased 的，同一个人的两个 alleles 可以随意在两列中排放。示例数据 CEU.9 和YRI.9用的是空格间隔符。
比如下面这个，代表了 3 个人的 2 个 SNPs：
```
1 0 1 1 9 9
1 1 1 0 0 0
```

生成这个文件

```
## 生成pop1亚群 geno文件
#样品名字变成两列方便plink转换生成.geno 文件：
awk '{print $1 "\t" $1}'  pop1.txt > pop1.keep.txt
plink --vcf $workdir/00.filter/clean.sorted.vcf.gz \
    --keep pop1.keep.txt  --chr 1  --out  Chr1.pop1  \
    --recode 01 transpose -output-missing-genotype 9  \
    --allow-extra-chr --set-missing-var-ids @:# --keep-allele-order
cut -d " " -f 5- Chr1.pop1.tped  | awk '{print $0" "}' > Chr1.pop1.geno



## 生成pop2亚群 geno文件
awk '{print $1 "\t" $1}'  $datadir/pop2.txt > pop2.keep.txt
plink --vcf $workdir/00.filter/clean.sorted.vcf.gz \
    --keep pop2.keep.txt  --chr 1  --out  Chr1.pop2  \
    --recode 01 transpose -output-missing-genotype 9  \
    --allow-extra-chr --set-missing-var-ids @:# --keep-allele-order
cut -d " " -f 5- Chr1.pop2.tped  | awk '{print $0" "}' > Chr1.pop2.geno
```
#### .snp file
每一行是一个 SNP的信息，每一列分别是 SNPName chr# GeneticDistance(Morgan) PhysicalDistance(bp) RefAllele TheOtherAllele。示例数据 9.xpclr.b36.snp 用的是 tab 间隔符。
比如：
```
rs10814410 9 0.000109 36587 C T
rs9408752  9  0.000938    91857  A  G
rs2810979  9  0.001323    152695  G  A
```

生成这个文件

```
## 生成snp位置信息文件
zcat clean.sorted.vcf.gz|awk '$1=="1" {print " "$1":"$2 "\tChr01\t" $2*0.00000018 "\t" $2 "\t"  $4 "\t" $5 }' >  Chr01.snp
## 运行xpclr分析
XPCLR  -xpclr  Chr1.pop1.geno Chr1.pop2.geno  Chr01.snp Chr01.out  -w1 0.005 200 2000 Chr01 -p0 0.95
## 替换染色体名称
sed 's/^0/Chr01/'  Chr01.out.xpclr.txt
```

### 3. 运行
```
XPCLR -xpclr genofile1 genofile2 mapfile outputFile -w1 0.005 500 10000 chrN -p1 0.95
```
-xpclr ：后面接是两个群体的 .geno 文件（genofile1 和 genofile2）、 .snp 文件（mapfile）、输出文件（outputFile）
-w1：后面接的参数依次为：gWin 是 Morgan 为单位的window size (一般可以设为 0.005)；snpWin 代表一个 window 中最大的 SNP 数量；gridSize 是 bp 为单位的两个 grid points 的间距；chrN 是染色体数。
-p：-p1 代表 phased 的数据，-p0 代表未 phased。
corrLevel：加权复合似然比检验中的 criterion，一般可设为0.95。

### 4.结果
得到的结果文件中，每一列分别代表 chr grid_ofSNPs_in_window physical_pos genetic_pos XPCLR_score max_s，XPCLR_score 是算出的 XP-CLR 分数。


### 5.补充关于phased的一些内容
SNP芯片标记测到的是一对同源染色体上的两个碱基，比如，一个SNP标记在一个个体当中的的结果是AA，在另一个个体当中的结果是TT，若两个SNP标记在同一条染色体上后，如果这个两个位点都是杂合的，一个是AT，另一个是AG，这个时候就有两种可能，要么AA是在同一个同源染色单体上（AA是一种单倍型，haplotype），要么AG（单倍型）是在同一个同源染色单体上，如果我们知道这个信息，那么这个基因型就是phased genotype, 如果我们不确定谁和谁在同一条同源染色单体上，那么这时测得的基因型就是unphased genotype.

###注意XPCLR的c语言版本年久失修，老是报错段错误

### 6.python版本
可以从conda安装
```
xpclr --format vcf --input chrom.misF.mafF.vcf.gz --samplesA ./NFS.txt --samplesB ./SFS1.txt --rrate 1.8e-9 --chr chr1A --maxsnps 200 --size 10000 --out ${i}.NFS_SFS2.out
```
