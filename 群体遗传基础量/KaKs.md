### Ka/Ks的解释
在遗传学中，Ka/Ks表示的是两个蛋白编码基因的非同义替换率（Ka）和同义替换率（Ks）之间的比例。这个比例可以判断是否有选择压力作用于这个蛋白质编码基因。

Ka和Ks的计算公式：

Ka=发生非同义替换的SNP数/非同义替换位点数

Ks=发生同义替换的SNP数/同义替换位点数

另外，计算Ka/Ks时，不考虑start codon和stop codon。

但是！上面的计算方法并没有考虑不同碱基之间发生替换的速率的不同，比如，嘌呤之间替换的概率（A=>G）要高于嘌呤替换为嘧啶的概率（A=>C或T），也就是说转换（transition，嘌呤变嘌呤，嘧啶变嘧啶）发生的概率要高于颠换（transversion，嘌呤变嘧啶，嘧啶变嘌呤）发生的概率。很多计算方法都会考虑到这些替换发生概率的不同。

另外，两个物种分化时间的长短也会影响到Ka/Ks的比值。比如有一个位点，原来是A，后来变成了T，再后来又变成了C，虽然发生了两次替换，但最后仅有一次替换被用于计算替换率。再比如有一个位点，原来是A，后来变成T，但同时与它相对应的另一个序列的位点，也发生了A到T的替换，那么我们也是无法用上面的方法来计算替换率。对于这种复杂的情况，我们可以用最大可能性算法来计算最可能的替换率，这里不再详述。

一般来讲，因为非同义替换会造成氨基酸变化，可能会改变蛋白质的构象和功能，因此会造成适应性的变化，从而带来自然选择的优势或劣势（一般是劣势）。而同义替换没有改变蛋白质的组成，因此不受自然选择的影响（当然这里我们忽略密码子偏好性的影响），那么Ks就能反映进化过程的背景碱基替换率。Ka/Ks的比值就能说明这个基因是受到了何种选择。

一般情况下，在某个个体中偶然发生的一个碱基替换（突变），如果没有额外的好处或者坏处的话，慢慢地也就消失了。但是自然选择中会有很多巧合，某些突变就是很幸运地被保留了下来，并且被固定了

对于一个没有受到自然选择压力的基因来说，我们可以计算得到这样的结果：Ka/Ks=1。但实际情况下，这个比值都是远小于1的：Ka/Ks<<1，因为一般非同义替换带来的都是有害的性状，只有极少数情况下会造成进化上的优势。

当Ka/Ks>>1时，基因受到强烈正选择，这样的基因即为近期正在快速进化的基因，对于物种的进化有着非常重要的意义。

除了查找快速进化基因，Ka/Ks还能检测基因的功能性，因为假基因（pseudogene）的Ka/Ks比值通常比功能基因更高。

此介绍比较基础，还有更加详细的介绍：[点这里](https://mp.weixin.qq.com/s?__biz=MjM5NTUwODUwMw==&mid=2453919064&idx=1&sn=2b7dd82e49515337171df570be136658&chksm=b1400d2086378436314370eaf45a553b0f994f3b3fcf838d9608dc2b373eae0f6db2c5b31a86&scene=21#wechat_redirect)

### 那么Ka/Ks到底咋算呢？！

#### 1.MAGA7

MAGA7，是通过分别求取dn，ds的值，然后得到dn/ds。

将fasta格式的序列文件导入MAGA, 然后选择Distance——computer overall mean distance进入图一页面：仔细看下面的选项卡，在substitutions type 中选择 Syn-Nonsynonymous；Genetic code table 按照自己的序列选择；modle/method 选择 Nei-Gojobori method (No. of Differences)；Substitutions to include 选择要计算的dn或者ds。下一步，就能得到dn或者ds，两者相比得到结果。

![MAGA7](MAGA7.png)

如果在distance——computer pairwise#####，然后按照后面步骤操作，结果会得到一个两两比较的矩阵（三角），我还不知道这个要怎么用。

如果只计算dn/ds，第一种应该够用了。

#### 2.ParaAT
上面那个是GUI的计算，适合小批量的计算，大批量基因的不大适合，此方法参考自：https://zhuanlan.zhihu.com/p/82432544

ParaAT是中科院基因组所的张章教授课题组开发的工具，它整合了计算ka/ks所需的一整套分析，包括:

* 蛋白序列比对（可选 clustalw2 | t_coffee | mafft | muscle）根据蛋白比对结果回译成codon对应的核酸比对结果(Back-translated nucleotide alignments guided by amino acid alignments are more reliable and accurate than direct nucleotide alignments)
* 计算kaks值（KaKs_Calculator实现）

##### 下载ParaAT2.0 

[下载链接](https://bigd.big.ac.cn/tools/paraat) 解压后，“ParaAT.pl”是运行的脚本

##### 安装所需的依赖工具，依赖工具需要加入环境变量

1. 蛋白比对工具，推荐安装muscle，比对效果相对最好，比对速度快，但它比其它工具更消耗内存。

2. [KaKs_Calculator](http://bigd.big.ac.cn/tools/kaks)

##### 准备输入文件

1. 同源基因列表，文件格式如下：

	```
	geneA1  geneA2  geneA3
	geneB1  geneB2  geneB3
	geneC1  geneC2  geneC3
	geneD1  geneD2  geneD3
	```

	每一行表示一组同源基因，每一列表示每个物种对应的基因。gene ID之间用tab符隔开。

2. fasta格式的蛋白序列文件和核酸序列文件，注意gene ID要与同源基因列表文件中的ID一致；

3. 多线程运行，指定线程数量的文件。

	这个文件只需要写入一个数字即可，表示有多少个线程同时运行。 

* 注：三种示例文件可以在解压的安装包中找到，分别是：test.homologs, test.pep, test.cds, proc

##### 运行ParaAT

运行代码如下:

```
ParaAT.pl -h test.homologs -n test.cds -a test.pep -p proc -m muscle -f axt -g -k -o result_dir
```

-h, 指定同源基因列表文件

-n, 指定核酸序列文件

-a, 指定蛋白序列文件

-p, 指定多线程文件

-m, 指定比对工具

-g, 去除比对有gap的密码子

-k, 用KaKs_Calculator 计算kaks值

-o, 输出结果的目录

-f, 输出比对文件的格式

注：

1. 如果需要用PAML，Hyphy等工具分析kaks时，ParaAT也可以生成这些工具所需的输入格式（-f 参数）

2. 如果是细菌的序列，需要设置成细菌对应的Genetic Code used （-c 11）。其他物种同理，默认的是The Standard Code （-c 1）
