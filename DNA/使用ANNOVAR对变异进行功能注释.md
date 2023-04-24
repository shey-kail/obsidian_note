https://zhuanlan.zhihu.com/p/375264206


### 第一步下载数据库 refGene
```
annotate_variation.pl -downdb -buildver hg19 -webfrom annovar refGene humandb/annotate_variation.pl
```

### 第二步 注释 （这里没有指定注释的数据库，因为 annotate_variation.pl 默认的参数是 –geneanno -dbtype refGene）
```
annotate_variation.pl -out ex1 -build hg19 example/ex1.avinput humandb/
```

### 第三步 查看结果
```
cat ex1.variant_function
```
```
UTR5 ISG15(NM_005101:c.-33T>C) 1 948921 948921 T C comments: rs15842, a SNP in 5' UTR of ISG15
UTR3 ATAD3C(NM_001039211:c.*91G>T) 1 1404001 1404001 G T comments: rs149123833, a SNP in 3' UTR of ATAD3C
splicing NPHP4(NM_001291593:exon19:c.1279-2T>A,NM_001291594:exon18:c.1282-2T>A,NM_015102:exon22:c.2818-2T>A) 1 5935162 5935162 A T comments: rs1287637, a splice site variant in NPHP4
intronic DDR2 1 162736463 162736463 C T comments: rs1000050, a SNP in Illumina SNP arrays
intronic DNASE2B 1 84875173 84875173 C T comments: rs6576700 or SNP_A-1780419, a SNP in Affymetrix SNP arrays
intergenic LOC645354(dist=11566),LOC391003(dist=116902) 1 13211293 13211294 TC - comments: rs59770105, a 2-bp deletion
intergenic UBIAD1(dist=55105),PTCHD2(dist=135699) 1 11403596 11403596 - AT comments: rs35561142, a 2-bp insertion
intergenic LOC100129138(dist=872538),NONE(dist=NONE) 1 105492231 105492231 A ATAAA comments: rs10552169, a block substitution
exonic IL23R 1 67705958 67705958 G A comments: rs11209026 (R381Q), a SNP in IL23R associated with Crohn's disease
exonic ATG16L1 2 234183368 234183368 A G comments: rs2241880 (T300A), a SNP in the ATG16L1 associated with Crohn's disease
exonic NOD2 16 50745926 50745926 C T comments: rs2066844 (R702W), a non-synonymous SNP in NOD2
exonic NOD2 16 50756540 50756540 G C comments: rs2066845 (G908R), a non-synonymous SNP in NOD2
exonic NOD2 16 50763778 50763778 - C comments: rs2066847 (c.3016_3017insC), a frameshift SNP in NOD2
exonic GJB2 13 20763686 20763686 G - comments: rs1801002 (del35G), a frameshift mutation in GJB2, associated with hearing loss
exonic CRYL1,GJB6 13 20797176 21105944 0 - comments: a 342kb deletion encompassing GJB6, associated with hearing loss
```

