tabix 可以对NGS分析中常见格式的文件建立索引，从而加快访问速度，不仅支持VCF文件，还支持**BED, GFF，SAM**等格式。
下载地址：https://sourceforge.net/projects/samtools/files/tabix/

安装：
```
wget https://sourceforge.net/projects/samtools/files/tabix/tabix-0.2.6.tar.bz2
tar xjvf tabix-0.2.6.tar.bz2
cd tabix-0.2.6/
make
```

下载源代码，解压缩之后，编译即可。编译成功之后，会有两个可执行文件tabix和bgzip。 由于SNP位点数量巨大，对应VCF文件也非常的大，为例节省存储空间，最常见的做法就是压缩。bgzip 可以压缩VCF文件，用法如下`bgzip  view.vcf`

bgzip的压缩算法和gzip压缩算法有着相似之处，所以对于bgzip压缩的文件，解压缩时除了可以使用bgzip软件本身，还可以使用gunzip进行解压缩。

需要注意的是，两种算法虽然有相似之处，但是还是有本质区别的，在对VCF文件压缩时，**不可以使用gzip来代替bgzip。**

对于大型的VCF文件而言，如何快速访问其中的记录也是个难点。tabix可以对VCF文件构建索引，索引构建好之后，访问速度会快很多。tabix对VCF文件建立索引的用法如下`tabix -p vcf view.vcf.gz`

注意输入的VCF文件必须是使用bgzip压缩之后的VCF文件，生成的索引文件为view.vcf.gz.tbi, 后缀为.tbi。

构建好索引之后，可以快速的获取指定区域的记录，示例如下

1. 获取位于11号染色体的SNP位点 
`tabix view.vcf.gz 11`

2. 获取位于11号染色体上突变位置大于或者等于2343545的SNP位点
`tabix view.vcf.gz 11:2343545`

3. 获取位于11号染色体上突变位置介于2343540到2343596的SNP位点
`tabix view.vcf.gz 11:2343540-2343596`

很多操作VCF的软件都会识别tabix建立的索引，从而加快处理速度。很多大型项目VCF文件，也都会用bgzip压缩，然后建立tabix的索引。

