#RNA #STAR #mapping

## 预期

1. 建立index：
```bash
STAR --runThreadN $thread --runMode genomeGenerate --genomeDir $out_path --genomeFastaFiles $ref --sjdbGTFfile $gtf --sjdbOverhang 136 --limitGenomeGenerateRAM 40000000000
```
注意：--limitGenomeGenerateRAM 40000000000 加入是因为六倍体小麦基因组太大了，它当时给我报错让我改成一个大于38000000000多少的一个数字，然后我改成40000000000了

2. STAR mapping
```bash
STAR --runThreadN $thread  \
--runMode alignReads \
--readFilesCommand zcat \
--twopassMode Basic \
--outSAMtype BAM SortedByCoordinate \
--limitBAMsortRAM 100000000000 \
--outSAMattrIHstart 0 
--genomeDir $ref \
--readFilesIn $input_path/$sampleid/${sampleid}_clean_1.fq.gz $input_path/$sampleid/${sampleid}_clean_2.fq.gz \
--quantMode GeneCounts \
--outFilterIntronMotifs RemoveNoncanonical  \
--sjdbOverhang 136 \
--outFileNamePrefix $out_path/$sampleid/$sampleid 
```

ps: 
1. `--limitBAMsortRAM 100000000000 --outSAMattrIHstart 0` 这俩加入，也是六倍体小麦基因组太大了。
2. --sjdbOverhang 这个参数后面的数字，是(read_length-1)

## 后来，遇到了一些坑

1. 老给我报错，段错误，麻了。。。。后来发现，是版本的事。我TM。。。。。哎行吧。
2. 老是有内存问题，`--outSAMtype BAM SortedByCoordinate` 这个参数是直接输出sort后的bam，用STAR来sort bam是需要额外的内存的，这一块内存你得自己设定，有点类似于samtools sort -m

解决: 我在mapping的时候直接摆了，老子不输出sort后的bam了行了吧。bam直接unsort。以后想sort我再用sambamba，瞬间舒服了。所以，我的更正版的STAR-mapping的pipeline：

```bash
STAR --runThreadN $thread  \
--runMode alignReads \
--readFilesCommand zcat \
--twopassMode Basic \
--outSAMtype BAM Unsorted \
--outSAMattrIHstart 0 
--genomeDir $ref \
--readFilesIn $input_path/$sampleid/${sampleid}_clean_1.fq.gz $input_path/$sampleid/${sampleid}_clean_2.fq.gz \
--quantMode GeneCounts \
--outFilterIntronMotifs RemoveNoncanonical  \
--sjdbOverhang 136 \
--outFileNamePrefix $out_path/$sampleid/$sampleid 
--outReadsUnmapped Fastx
```
除了更改了`--outSAMtype`这个参数，也加入了 `--outReadsUnmapped Fastx`以输出没有mapping上的reads

> [官方说明书](../appendix/STARmanual.pdf)