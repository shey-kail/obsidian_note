# SnpEff使用方法

> SnpEff 软件通过基因组结构注释数据（GTF文件），对VCF文件中的SNP/InDel信息进行注释，即主要解释了SNP/InDel是否能够对编码蛋白基因造成影响。

最近在给课题组做一些突变体基因定位的工作（BSA混池测序），得到了最终的VCF文件，然后最终将得到的SNP/InDel注释出来。

#### 1.SnpEff的下载

普通下载（推荐）
```shell
#软件下载
wget https://downloads.sourceforge.net/project/snpeff/snpEff_latest_core.zip
#解压缩
unzip snpEff_latest_core.zip -d ~
```

conda下载
```shell
conda activate mutmap
conda install -y snpeff
#conda下载的话，需要自己寻找snpeff软件包的位置
```

#### 2.SnpEff使用

SnpEff软件的主要程序就是snpEff.jar，该软件需要Java运行程序。SnpEff使用最多的程序就是build和eff,build适用于数据库的构建，eff适用于对SNP/InDel进行注释。

##### 2.1构建SnpEff数据库

SnpEff软件的运行，首先需要基因组fasta序列信息和GTF注释信息，来构建数据库。

配置文件步骤如下：
```shell
1.在~/snpEff/目录中，创建一个文件夹：data
2.在~/snpEFF/data目录下，创建两个文件夹
    AT_10/   genomes/
    这两个文件夹中，分别放置了GTF文件和基因组文件
    genes.gtf sequences.fa
3.编辑~/snpEff/snpEff.config文件
    在文件的最后一行添加信息：
    AT_10.genome: AT
```

构建数据库步骤如下：
```shell
java -jar ~/snpEff/snpEff.jar build -c ~/snpEff/snpEff.config -gtf22 -v AT_10

#参数说明
java -jar: Java环境下运行程序
-c snpEff.config配置文件路径
-gtf22 设置输入的基因组注释信息是gtf2.2格式
-gff3 设置输入基因组注释信息是gff3格式
-v 设置在程序运行过程中输出的日志信息
最后的AT_10参数 设置输入的基因组版本信息，和~/snpEff/snpEff.config配置文件中添加的信息一致
```

##### 2.2使用SnpEff进行注释
最终会产生四个文件 positive.snp.eff.vcf positive.html positive.csv positive.genes.txt

可以在positive.snp.eff.vcf文件中，分析自己的后续基因位点了。
