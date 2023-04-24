1. samtools markdup, 最传统的最慢的莫过于此
```bash
samtools sort -m 4G -@ ${thread} -n example.bam -o namesort.bam
samtools fixmate -m -@ ${thread} namesort.bam fixmate.bam
samtools sort -m 4G -@ ${thread} fixmate.bam -o positionsort.bam 
samtools markdup positionsort.bam markdup.bam

一些很有用的参数：
samtools sort :
	-l INT     Set compression level, from 0 (uncompressed) to 9 (best)
	-m INT     Set maximum memory per thread; suffix K/M/G recognized [768M]
	-n         Sort by read name (not compatible with samtools index command)
	-o FILE    Write final output to FILE rather than standard output
	-T PREFIX  Write temporary files to PREFIX.nnnn.bam
	-@, --threads INT Number of additional threads to use [0]
	--write-index Automatically index the output files [off]
samtools fixmate :
	-m           Add mate score tag
	-@, --threads INT Number of additional threads to use [0]
samtools markdup :
	-@, --threads INT Number of additional threads to use [0]
	--write-index Automatically index the output files [off]
```
用了四步才能完成markdup。

2. sambamba markdup，这个挺快的
```bash
samtools sort -m 4G -@ ${thread} example.bam -o positionsort.bam 
sambamba markdup -t ${thread} -l 6 -p --tmpdir=/data/Users/shey/sambamba_tmp/ positionsort.bam markdup.bam

一些很有用的参数：
sambamba markdup :
	 -t, --nthreads=NTHREADS
	 -l, --compression-level=N  (from 0 to 9)
	 -p, --show-progress
				show progressbar in STDERR
	 --tmpdir=TMPDIR  specify directory for temporary files
```
但sambamba（在1.0.0版本）有个毛病，生成bam的时候，都会同步生成bai格式的index文件，但是如果你的染色体大于2<sup>28</sup>的时候，bai的生成就会出错，进而让程序崩溃。我找了找，没找到合适的解决办法。

3. picard MarkDuplicate 
```bash
samtools sort input.bam -o positionsort.bam
# 第二步，运行MarkDuplicate命令
picard MarkDuplicate \
I=positionsort.bam \
O=markdup.bam \
M=markdup.metrc.csv
```
比sambamba慢一点，还好。不过我最近也遇到了染色体大于2<sup>28</sup>的时候报错的情况.

4. 用biobambam2软件包
```bash
bamsort SO=coordinate sortthreads=10 markduplicates=1 level=9 <input.bam >output.bam
bammarkduplicates markthreads=10 level=9 <input.bam >output.bam
bamsormadup SO=coordinate threads=10 level=9 <input.bam >output.bam

参数：
	SO=(coordinate, queryname)   sorting order (coordinate, queryname, hash, tag, queryname_HI or queryname_lexicographic)
	level=(0-9)                  compression settings for output bam file
```
最后用的这个，没有染色体大于2<sup>28</sup>的时候报错，而且可以非常好的融合在管道里面，而且还挺快，sort + markdup一步到位。但是bamsormadup在用多线程的时候，非常吃I/O，一定要注意。但是如果我用管道把bamsormadup放到bwa以及samtools的下游，那么出来的bam将会直接是markdup过的，这就很爽。如下
```bash
bwa mem -t $bwa_thread $ref $fq1 $fq2 | \
            samtools view -b -h -q 30 | \
            bamsormadup > ${output}/$sampleid.markdup.bam
```
而且，用这个流程跑出来的makrdup.bam跟sambamba出来的一样。

5. samblaster(未测试)
最近才发现的一个软件，输入是sam文件而不是bam，可以跟在bwa后面，然后用samtools view后再samtools sort。除了markdup，还能提取split reads 和 discordants read pairs。以下是来自官网的说明
* 直接把bwa mem的结果作为输入，markdup后，转为bam
```bash
bwa mem <idxbase> samp.r1.fq samp.r2.fq | samblaster | samtools view -Sb - > samp.out.bam
```

* 当bwa mem 加了`-M` 参数时，samblaster也得加`-M`
```bash
bwa mem -M <idxbase> samp.r1.fq samp.r2.fq | samblaster -M | samtools view -Sb - > samp.out.bam
```

关于提取discordant read pairs 和 split read alignments，看这里[[提取discordant read pairs 和 split read alignments]]