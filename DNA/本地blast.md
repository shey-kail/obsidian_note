# BLAST（Basic local alignment search tool），即基于局部比对算法的搜索工具

# BLAST的基本步骤

1.  用makeblastdb为BLAST建立数据库
2.  选择BLAST工具，blastn, blastp, blastx, tblastn, tblastx等
3.  运行BLAST工具，在有必要的情况下还可以对输出结果进行修饰以得到自己想要的结果格式

## 第一步 BLAST数据库的构建
```bash
#格式化蛋白质数据库：
makeblastdb -in spinach.fa  -parse_seqids -hash_index  -out spinach_DB -dbtype prot
#格式化核酸数据库：
makeblastdb -in spinach.fa  -parse_seqids -hash_index  -out spinach_DB -dbtype nucl    
#一般带上-parse_seqids -hash_index 
```
将基因组和想要构建的数据库名字替换即可，一般我们做生物信息都会用本地BLAST，因此不介绍远程数据库。
  
## 第二步 选择BLAST工具
1.  **blastn** nucleotide to nucleotide
2.  **blastp** protein to protein
3.  **blastx** translated nucleotide to protein
4.  **tblastn** protein to translated nucleotide

#### blastp的常用参数

-query 后接查询序列的文件名称；  
-db 后接格式化好的数据库名称；  
-out 后接要输出的文件名称及格式

#### blastn的常用参数

-db: 指定blast搜索用的数据库  
-query: 用来查询的输入序列，fasta格式  
-out：输出结果文件  
-evalue: 设置e值cutoff  
-max_target_seqs: 设置最多的目标序列匹配数  
-num_threads：指定多少个cpu运行任务（依赖于你的系统，同于以前的-a参数）  
-outfmt format "7 qacc sacc evalue length pident" ：这个是新BLAST+中最叼的功能了，直接控制输出格式，不用再用parser啦， 7表示带注释行的tab格式的输出，可以自定义要输出哪些内容，用空格分格跟在7的后面，并把所有的输出控制用双引号括起来，其中qacc查询序列的acc,sacc表示目标序列的acc，evalue即是e值，length即是匹配的长度，pident即是序列相同的百分比，其他可用的特征（代码块中显示）如下：
```
Formatting options  
-outfmt <String>  
alignment view options:  
0 = pairwise,  
1 = query-anchored showing identities,  
2 = query-anchored no identities,  
3 = flat query-anchored, show identities,  
4 = flat query-anchored, no identities,  
5 = XML Blast output,  
6 = tabular,  
7 = tabular with comment lines,  
8 = Text ASN.1,  
9 = Binary ASN.1  
10 = Comma-separated values  
11 = BLAST archive (ASN.1),  
12 = Seqalign (JSON),  
13 = Multiple-file BLAST JSON,  
14 = Multiple-file BLAST XML2,  
15 = Single-file BLAST JSON,  
16 = Single-file BLAST XML2,  
17 = Sequence Alignment/Map (SAM),  
18 = Organism Report  
Options 6, 7, and 10 can be additionally configured to produce  
a custom format specified by space delimited format specifiers.  
一般来说用的就是6、7、10、17，因为要通过覆盖度和得分来筛选结果的话用这三种比较方便，而7的话是比6多了一些注释部分，结果部分是一样的。
```

一般来说用的就是6、7、10、17，因为要通过覆盖度和得分来筛选结果的话用这三种比较方便，而7的话是比6多了一些注释部分，结果部分是一样的。

```
qseqid    means Query Seq-id
qgi       means Query GI
qacc      means Query accesion
sseqid    means Subject Seq-id
sallseqid means All subject Seq-id(s), separated by a ';'
sgi       means Subject GI
sallgi    means All subject GIs
sacc      means Subject accession
sallacc   means All subject accessions
qstart    means Start of alignment in query    #这个位置在最起始为1，而不是0，注意这个位置和别的的计算未知的差别
qend      means End of alignment in query
sstart    means Start of alignment in subject    #同理，这个也是最起始为1，不是0
send      means End of alignment in subject
qseq      means Aligned part of query sequence
sseq      means Aligned part of subject sequence
evalue    means Expect value
bitscore  means Bit score
score     means Raw score
length    means Alignment length
pident    means Percentage of identical matches
nident    means Number of identical matches
mismatch  means Number of mismatches
positive  means Number of positive-scoring matches
gapopen   means Number of gap openings
gaps      means Total number of gaps
ppos      means Percentage of positive-scoring matches
frames    means Query and subject frames separated by a '/'
qframe    means Query frame
sframe    means Subject frame
```

## 第三步 运行BLAST

**blastp**

```bash
blastp -query spinach_1.fa -db spinach_DB -out spinach_blast
```

**blastn**  
自己经常用的命令如下所示

```bash
blastn -task blastn -db datebase_name -query input_file -outfmt 6 -out output_file
blastn -task blastn -db database_name -query input_file -out output_file -evalue evalue -max_target_seqs num_sequences -num_threads int_value -outfmt format format_string
blastn -task blastn -db database_name -query input_file -out output_file -evalue evalue -max_target_seqs num_sequences -num_threads int_value -outfmt format " 6 qseqid sseqid pident length mismatch gapopen qstart qend sstart send qseq sseq evalue bitscore"
```

一般默认的使用输出结果格式6的话输出的结果文件的数据如下所示：

```
query acc.ver, subject acc.ver, % identity, alignment length, mismatches, gap opens, q. start, q. end, s. start, s. end, evalue, bit score
```

**注**：第二条命令可以增加对应的qseq, sseq
