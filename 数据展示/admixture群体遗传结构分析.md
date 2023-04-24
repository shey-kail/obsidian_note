
群体遗传结构(Structure)指遗传变异在物种或群体中的一种非随机分布。按照地理分布或其他标准可将一个群体分为若干亚群，处于同一亚群内的不同个体亲缘关系较高，而亚群与亚群之间则亲缘关系稍远。群体结构分析有助于理解进化过程，并且可以通过基因型和表型的关联研究确定个体所属的亚群。

一般进行群体分析所使用的软件为STRUCTURE，但是STRUCTURE的运行速度较慢，如今Admixture凭借其高速的运算速度逐渐成为群体遗传结构分析的主流软件。

在众多的涉及到亲缘关系分析文章中几乎都会提到群体遗传结构分析，下面是几个例子：

![image_2022-03-31-22-15-35](image_2022-03-31-22-15-35.png)

图片摘自文章Genomicanalyses identify distinct patterns of selection in domesticated pigs and Tibetanwild boars。本分析利用全世界广泛分布的野猪或者家猪共103 只，来鉴定和藏猪遗传关系最近的品种。图片中每一列表示一个个体，其中不同颜色片段的长度表示该个体基因组中某个祖先所占的比例。图片左侧K=2 到9 表示本次研究假定的祖先群体个数从2一直到9；图片上下横坐标分别表明了群体名称以及地理分布。当K=2 时，欧洲群体和亚、非洲群体明显分开；K=3 时，来自东南亚群岛的猪属的4 个个体以及1 个非洲疣猪个体与剩余的亚洲个体分开；K=4 时，藏猪和亚洲野猪与亚洲的驯化猪分开；K=5 时，103 个个体明显分为5 个群体，欧洲猪、藏猪、中国西南地区的驯化猪、中国东南地区驯化猪和亚洲野猪、猪属的4 个个体和1 个非洲疣猪，其中的藏猪基因组中包含多个祖先成分，这可能是由于祖先本身的多态性，或者是近期藏猪与邻近的驯化猪发生了杂交而导致了基因渗入所造成的。

通过这样的图，我们就可以明显看到来自不同地点的不同种群所包含的亚群或祖先的个数以及相似率。

Single nucleotide polymorphism profilesreveal an admixture genetic structure of grapevine germplasm from Calabria,Italy, uncovering its key role for the diversification of cultivars in theMediterranean Basin。横坐标表示不同样本，纵坐标表示每个样本所包含的亚群或祖先的个数、种类以及比例。

![image_2022-03-31-22-16-03](image_2022-03-31-22-16-03.png)

这样的题是不是很明显很直观又觉得“高大上”呢？其实这样的图一点都不难做，只要按照以下步骤，就可以得到这样的图了。

Admixture使用步骤：

**1.****输入文件**

Admixture的输入文件格式有以下三种：PLINK(.bed)，PLINK(.ped)或者EIGENSTRAT(.geno)，最常用的就是Plink产生的（.bed）文件了。

采用PLINK 进行群体结构分析。首先创建PLINK 的输文件-Ped 文件，然后利用Admixture软件构建群体遗传结构和群体世系信息。

PLINK（http://pngu.mgh.harvard.edu/~purcell/plink/）。

首先，我们将已经有的vcf文件处理成Plink可以分析的格式，这一步需要用到vcftools(linux系统下可以很容易地下载安装），代码如下：

这一步的输出结果为：plink.ped和plink.map，ped文件和map文件在后续处理中缺一不可。

**2.****过滤****SNP****文件**

使用上一步产生的.ped和.map文件，用Plink进行SNP过滤,代码如下：

输出为：QC.bed(binary file,genotype information)、QC.fam(first six columns of plink.bed)、QC.bim(extended MAP file:two extra cols=allelenames)，该步就得到了Admixture可以输入的bed文件来进行群体结构分析和作图。

K是样本所包含的亚群或者祖先数，如若不知道理想的K值，可以设定K=1,2,3,4,5，用admixture进行计算：

**3.****提取****CV****值**

提取CV值后，可以得到上一步得到的不同K值的错误率，一般认为CV error最小值为最佳K值。

![image_2022-03-31-22-16-42](image_2022-03-31-22-16-42.png)

如果觉得数字不是很明显，也可以通过绘制直线图直观进行K值的选择，如下图所示，K=3时，Cross-validation error值最小。

![image_2022-03-31-22-17-01](image_2022-03-31-22-17-01.png)

4.使用R画图

最后一步！就是拿着我们的数据来做图了。一般该步使用R来进行图片绘制，R是一款强大的绘图计算程序，在Linux系统和Windows系统下都能使用，代码很简单，只需要如下几行：

![image_2022-03-31-22-17-24](image_2022-03-31-22-17-24.png)

![image_2022-03-31-22-17-46](image_2022-03-31-22-17-46.png)

至此，群体结构分析的图就得到啦！可以根据自己文章的需要对图片进行润色，在文章中加入这样的图，相信会为文章增色不少。

参考：admixture-manual
