* 把discordant read pairs 和 split read alignments 单独输出
```bash
bwa mem <idxbase> samp.r1.fq samp.r2.fq | samblaster -e -d samp.disc.sam -s samp.split.sam | samtools view -Sb - > samp.out.bam
```

* 从已经存在的bam中，提取discordant read pairs and split read alignments 
```bash
samtools view -h samp.bam | samblaster -a -e -d samp.disc.sam -s samp.split.sam -o /dev/null
```