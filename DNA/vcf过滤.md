师兄的过滤方案：
```
#删去indels
vcftools --vcf allsamplebcftools.vonly.vcf --recode --remove-indels --stdout > allsamplebcftools.vonly.snp.vcf

#保留maf(最小等位基因频率)>0.05，一个基因座上最多有两个等位基因，而且基因型明确的snp
bcftools view allsamplebcftools.vonly.snp.vcf -i  'maf>0.05 && F_PASS(GT!=mis)>0.8 && N_ALT=1' --threads 4 -Oz -o allsamplebcftools.vonly.snp.misF.mafF.vcf.gz
```

其实这里可以这样：
```
bcftools view allsamplebcftools.vonly.vcf -i 'maf>0.05 && F_PASS(GT!="./.")>0.8 && N_ALT=1 && TYPE=="SNP"' --threads 4 -Oz -o allsamplebcftools.vonly.snp.misF.mafF.vcf.gz
```

其实，过滤方案还可以有：
```
MQ：mapping quality
DP：总的Raw read depth
GQ：Genotype quality
QUAL：ref quality（是指ref的确信程度）
```