我的方法是用bcftools consensus
```
bcftools consensus -f ref.fa output.vcf.gz -o out.fa
```
假如你只是对某个区间感兴趣的话，参考下面这条命令:
```
samtools faidx ref.fa 8:11870-11890 | bcftools consensus output.vcf.gz -o out.fa  
```
当然多个区间也是可以的:
```
samtools faidx -r regions.bed ref.fa | bcftools consensus output.vcf.gz -o out.fa
```