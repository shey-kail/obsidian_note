常用的命令：
```bash
bcftools mpileup -m 3 -C 50 -E -q 10 -Q 20 -f $reference  -b bam_name_list -a "DP,AD,ADF,ADR,SP,INFO/AD,INFO/ADF,INFO/ADR" -p | bcftools call -f "GQ,GP" -m --thread 10 -v -Oz -o allsamplebcftools.vonly.vcf.gz
```
bcftools mpileup虽然也会输出VCF格式的文件，但是并不直接进行snp calling。下面的命令可以生成VCF格式的文件
```
bcftools mpileup mpileup.1.bam --fasta-ref mpileup.ref.fa >mpileup.vcf
```
直接生成的VCF文件内容如下
```
#CHROM POS ID REF ALT QUAL FILTER INFO FORMAT HG00100
17 1 . A <*> 0 . DP=5; PL
17 2 . A <*> 0 . DP=5; PL
17 3 . G <*> 0 . DP=5; PL
17 4 . C <*> 0 . DP=5; PL
17 5 . T <*> 0 . DP=5; PL
```
里面的每一条记录并不是一个SNP位点，而是染色体上每个碱基的比对情况的汇总。这种信息官方称之为genotype likelihoods。
而我这里出现的每一个参数的意义分别是：
```
bcftools mpileup :

-m,--min-ireads INT
Minimum number gapped reads for indel candidates INT[1]
gapped read的最小为多少时，才把这个位点认定成indel

-C,--adjust-MQ INT 
Coefficient for downgrading mapping quality for reads containing excessive mismatches
这个参数是降低含有过度失配的reads的mapping质量值的，bcftools中建议在mapping质量过高的时候，把这一项设定为50，并且一般对bwa-backtrack算法比较好使，然而我这里更多的是用的mem

-E，--redo-BAQ
Recalculate BAQ on the fly, ignore existing BQ tags
动态重新计算BAQ，忽略现有BQ标签

-q, --min-MQ INT
Minimum mapping quality for an alignment to be used [0]
最小的mapping质量值，小于这个值的比对将不被考虑

-Q，--min-BQ INT
Minimum base quality for a base to be considered [13]
碱基的最低质量值，低于这个质量的碱基将不被接受

-f, --fasta-ref FILE
参考基因组文件

-b，--bam-list FILE
List of input alignment files, one file per line

-p, --per-sample-mF
Apply -m and -F thresholds per sample to increase sensitivity of calling. By default both options are applied to reads pooled from all samples.
每个样本应用-m和-F阈值以提高调用的灵敏度。默认情况下，这两个选项都应用于从所有样本合并的读取。

-r, --regions CHR|CHR:POS|CHR:FROM-TO|CHR:FROM-[,…]
Only generate mpileup output in given regions. Requires the alignment files to be indexed. If used in conjunction with -l then considers the intersection; see Common Options

-a, --annotate LIST
Comma-separated list of FORMAT and INFO tags to output. (case-insensitive, the "FORMAT/" prefix is optional, and use "?" to list available annotations on the command line) [null]:
FORMAT/AD   .. Allelic depth (Number=R,Type=Integer)
FORMAT/ADF  .. Allelic depths on the forward strand (Number=R,Type=Integer)
FORMAT/ADR  .. Allelic depths on the reverse strand (Number=R,Type=Integer)
FORMAT/DP   .. Number of high-quality bases (Number=1,Type=Integer)
FORMAT/SP   .. Phred-scaled strand bias P-value (Number=1,Type=Integer)
FORMAT/SCR  .. Number of soft-clipped reads (Number=1,Type=Integer)
INFO/AD     .. Total allelic depth (Number=R,Type=Integer)
INFO/ADF    .. Total allelic depths on the forward strand (Number=R,Type=Integer)
INFO/ADR    .. Total allelic depths on the reverse strand (Number=R,Type=Integer)
INFO/SCR    .. Number of soft-clipped reads (Number=1,Type=Integer)
FORMAT/DV   .. Deprecated in favor of FORMAT/AD; Number of high-quality non-reference bases, (Number=1,Type=Integer)
FORMAT/DP4  .. Deprecated in favor of FORMAT/ADF and FORMAT/ADR; Number of high-quality ref-forward, ref-reverse,
               alt-forward and alt-reverse bases (Number=4,Type=Integer)
FORMAT/DPR  .. Deprecated in favor of FORMAT/AD; Number of high-quality bases for each observed allele (Number=R,Type=Integer)
INFO/DPR    .. Deprecated in favor of INFO/AD; Number of high-quality bases for each observed allele (Number=R,Type=Integer)
```

bcftools call才是真正的call snp的程序
```
bcftools call : 
-m,--multiallelic-caller
alternative model for multiallelic and rare-variant calling designed to overcome known limitations in -c calling model (conflicts with -c)
多等位基因和罕见变体调用的替代模型，旨在克服-c调用模型中的已知限制（与-c冲突）

-f, --format-fields list
comma-separated list of FORMAT fields to output for each sample. Currently GQ and GP fields are supported. For convenience, the fields can be given as lower case letters. Prefixed with "^" indicates a request for tag removal of auxiliary tags useful only for calling.
加入GQ（基因型质量）和GP信息

--thread
Use multithreading with INT worker threads. The option is currently used only for the compression of the output stream, only when --output-type is b or z. Default: 0.
该选项当前仅用于输出流的压缩，仅当--output类型为b或z时。默认值：0。其实就是在压缩的时候用多线程，call snp的时候用不了多线程。

-v,--variants-only
output variant sites only
只输出变异位点

-O [b|u|z|v]
输出文件的类型：b（bcf）、u（不压缩的bcf）、z（vcf.gz）、v(vcf)

-o FILE
输出文件名
```

因为实际上，bcftools call snp的时候不能多线程，所以我们最好是用-r参数，指定一下染色体，然后并行跑多个，如下
```bash
for i in chr{1..7}{A,B}
do
	{
	chrom=$i
	bcftools mpileup -m 3 -C 50 -E -q 10 -Q 20 -f $reference  -b bam_name_list -a "DP,AD,ADF,ADR,SP,INFO/AD,INFO/ADF,INFO/ADR" -p -r ${chrom} | bcftools call -f "GQ,GP" -m --thread 10 -v -Oz -o ${chrom}.vonly.vcf.gz 
	} &
done
```