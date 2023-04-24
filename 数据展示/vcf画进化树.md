画进化树的软件默认一般是直接输入序列，但是如果序列很长，或者我本来就call snp完成了，那么就不大适合直接输入序列了。本篇将介绍如何在有vcf文件的情况下得到进化树

### 1. 利用VCF2Dis生成距离矩阵

[VCF2Dis](https://github.com/BGI-shenzhen/VCF2Dis)，是一款计算根据vcf文件计算距离矩阵的小工具

Usage: VCF2Dis -i <in.vcf>  -o  <p_dis.mat>
```
#1.0) Parameters can used as short letter
	  Such as : [-i] short for [-InPut], [-o] for [-OutPut],[-s] for [-SubPop], [-k] for [-KeepMF]

#2.1) To new all the sample p_distance matrix based VCF, run VCF2Dis directly
	   ./bin/VCF2Dis    -i  in.vcf.gz  -o p_dis.mat

#2.2) To new sub group sample p_distance matrix ; Put their sample name into File sample.list
	 ./bin/VCF2Dis  -InPut  chr1.vcf.gz chr2.vcf.gz  -OutPut p_dis.mat  -SubPop  sample.list

#3.0) Default use all site to join the Calculation. To run the bootstrap tree , can run muti-time with using part of site, Para [-Rand]
	 ./bin/VCF2Dis  -InPut  in.vcf.gz  -OutPut p_dis.mat   -Rand  0.25
```
操作
```
# 对所有样本进行计算距离矩阵
../bin/VCF2Dis  -InPut  in.vcf.gz       -OutPut p_dis.mat

# 对部分样本计算
../bin/VCF2Dis  -InPut  in.vcf.gz       -OutPut p_dissub.mat  -SubPop  sample.list
# 其中
head sample.list
S010
S033
S186
S123
S124
S011
```
得到的结果如下所示
![image_2022-03-31-22-18-55](image_2022-03-31-22-18-55.png)

3 构建树

    在线构建
    上传距离矩阵到在线网站, FastMe2.0。上传以后，选择Data type为Distance matrix。 然后点击最下方的execute & email results即可。邮箱也可不写。
    最终得到一个.nwk的文件，导入iTOl即可查看，如下所示
    image.png

    也可通过phylip进行构建树
    具体可以查看# 序列比对和构建进化树（clustalw和phylip）

参考

转化为newick格式

然后用[iTol](https://itol.embl.de/)来调整样式

### 2.phylip

#### vcf转phylip

转phylip格式的文件的参数很简单，如下
```
python vcf2phylip.py --input myfile.vcf  
```

[vcf2phylip.py](https://github.com/edgardomortiz/vcf2phylip)

[vcf2phylip.py](./appendix/vcf画进化树/vcf2phylip.py)

这个软件，还能转fasta，nexus，等

参数分别是
```
#转fasta
python vcf2phylip.py --input myfile.vcf --fasta 

#转nexus
python vcf2phylip.py --input myfile.vcf --nexus 
```
虽然，我不知道nexus有啥用

然后，利用phylip构建进化树

phylip 在命令行中可以根据提示输入参数，也可以用含有参数的文本导入参数。

#### phylip
phylip中有许多程序，大部分的程序运行方法相同，把infile作为默认的输入文件，输出结果写在outfile中。因此，在进行下一步分析前，需要重命名想要保存的文件。

> seqboot: 生成随机样本，用bootstrap和jack-knife方法。需要设置选项M
> 
> dnadist：DNA距离矩阵计算器。
> 
> neighbor：NJ法的使用
> 
> consense：用多重树构建一致树。

每个程序都需要设定参数，因此还需要新建par文件。

```bash
#cat seqboot.par
all.merge.snp.phy #设定输入文件的名称，否则输入默认的名为infile的文件
r #选择bootstrap
1000 #设置bootstrap的值，即重复的replicate的数目，通常使用1000或者100，注意此处设定好后，后续两步的M值也为1000或者100
y #yes确认以上设定的参数
9 #设定随机参数，输入奇数值。

#cat dnadist.par
seqboot.out #本程序的输入文件
t #选择设定Transition/transversion的比值
2.3628 #比值大小
m #修改M值
d #修改M值
1000 #设定M值大小
2 #将软件运行情况显示出来
y #确认以上设定的参数

#cat neighbor.par
dnadist.out #本程序的输入文件
m 
1000  #设定M值大小
9 #设定随机数，输入奇数值
y #确认以上设定的参数

# cat consense.par
nei.tree  #本程序的输入文件
y #确认以上设定的参数
```

再运行以下命令行即可
```bash
seqboot<seqboot.par &&mv outfile seqboot.out &&dnadist<dnadist.par &&  mv outfile dnadist.out && neighbor<neighbor.par && mv  outfile nei.out && mv outtree nei.tree  && consense<consense.par && mv outfile cons.out && mv outtree constree
```
最后将会得到constree文件，可将该文件改为*.tre文件，双击后在treeview中直接查看进化树的内容。

若要进行进一步的编辑，可使用iTOL在线的网站（[http://itol.embl.de/](https://link.jianshu.com?t=http%3A%2F%2Fitol.embl.de%2F)）进行编辑，以下即为我得到的一个进化树。

### 3.iqtree
vcf转fasta后，（[第二条](#vcf转phylip)里面说了）
```
iqtree -s fasta
```
这个是ML树，很准，但是模型敏感，但是iqtree会自动选择最佳模型，很好用！
然后再用itol美化就行啦