#RNA #cufflinks #quantify #定量

这里只说定量的事：用cuffquant和cuffnorm

我的数据是非链特异性的，那么您在用cuffquant和cuffnorm的时候，应该加上以下参数：
```bash
--library-type fr-unstranded
```
这个参数表示您的文库是非链特异性的，没有方向性的，即理论上正负链各占0.5。

具体来说，您可以按照以下命令进行：

1.  使用cuffquant根据gtf文件和bam文件计算每个样本中的转录本表达水平，生成cxb文件。
```bash
cuffquant -o sample_quant -p 8 -u --library-type fr-unstranded AT.gff accepted_hits.bam
参数说明： 
-o ：指定结果输出目录：包含结果文件abundances.cxb 
-p ：指定线程数 
-u ：指定对比对上基因组上多个位置的reads进行统计分析。 
--library-type fr-unstranded：指定文库类型为非链特异性 
加上参考基因组注释文件：AT.gff 
最后加上Tophat2或STAR产生的该样本的比对结果文件：accepted_hits.bam
```

2.  使用cuffnorm根据gtf文件和多个样本的cxb文件计算每个样本中的基因和转录本表达水平，并进行标准化处理，生成txt文件。
```bash
cuffnorm -o cuffnorm_out -p 8 -L 0h_1,12h_CK1,12h_E1 --library-type fr-unstranded AT.gff /data/disk2/liyan/AT/sample_1_quant/abundances.cxb /data/disk2/liyan/AT/sample_2_quant/abundances.cxb /data/disk2/liyan/AT/sample_3_quant/abundances.cxb
参数说明： 
-o ：指定结果输出目录：包含多个结果文件，如genes.fpkm_table等 
-p ：指定线程数 
-L ：指定样本分组信息 
--library-type fr-unstranded：指定文库类型为非链特异性 
加上参考基因组注释文件：AT.gff 
最后加上多个样本的cxb文件
```

以上是一个简单的介绍，更多细节和参数请参考cufflinks的官方文档：http://cole-trapnell-lab.github.io/cufflinks/manual/