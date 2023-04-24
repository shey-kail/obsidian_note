#RNA #hisat2 #mapping

## 预期

1. 建立index
```bash
hisat2_extract_exons.py $gtf > exon.txt
hisat2_extract_splice_sites.py $gtf > splice_sites.txt
hisat2-build --ss splice_sites.txt --exon exon.txt -p ${thread} -f $fasta_filename $hisat2_path_prefix
#	--ss splice_sites.txt, 这个文件可以用hisat2_extract_splice_sites.py $gtf > splice_sites.txt生成
#	--exon exon.txt, 这个文件可以用hisat2_extract_exons.py $gtf > exon.txt
#	$hisat2_path_prefix应该设定为："/path/to/hisat2_index/"+"自己设置的前缀"
```
实例：
```bash
source /data/Users/pangxi/miniconda3/etc/profile.d/conda.sh
conda activate bio-soft

#最大线程数
thread=28
#fasta文件的位置
fasta_origin_file=../ref/neo_by_px_CS_iwgsc_v2.1.fa
#hisat2 index要生成在那个文件夹
hisat2_index_dir=../ref/hisat2-index/
#fasta要链接到hisat2 index文件夹中，这个链接在文件夹中的文件名。
fasta_filename=$(basename $fasta_origin_file)
#gtf文件
gtf=../ref/CS_iwgsc_v2.1_HC.gtf
#hisat2_index的prefix, 一般不用改，这里就设置成fasta的文件名（去掉后缀）
hisat2_prefix=$(basename $fasta_filename .fa)
#exons 文件名
exon_file=${gtf/.gtf}.exons.txt
#splice_sites 文件名
splice_sites_file=${gtf/.gtf}.splice_sites.txt

#生成hisat2 index文件夹和bam输出的文件夹
mkdir -p $hisat2_index_dir

#把fasta源文件链接到hisat2_index文件夹中
cd $hisat2_index_dir
ln -s ../$fasta_filename $fasta_filename
cd -

hisat2_extract_exons.py $gtf > $exon_file
hisat2_extract_splice_sites.py $gtf > $splice_sites_file

#开始建立index
echo "############################################################"
echo "############################################################"
echo "[INFO] build index in path : $hisat2_index_dir"
echo "############################################################"
echo "############################################################"
hisat2-build --ss ../$splice_sites_file --exon ../$exon_file -p ${thread} -f $fasta_filename $hisat2_index_dir/$hisat2_prefix
```

2. mapping
```bash
hisat2 -x $hisat2_index_dir/$hisat2_prefix  `#index目录` \
-p ${thread} --dta-cufflinks \
-1 ${fastq_dir}/${sampleid}${fastq1_suffix} \ 
-2 ${fastq_dir}/${sampleid}${fastq2_suffix} `#fastq.gz` | \  
samtools view -b -h | samtools sort -m 4G - -o $out_path/${sampleid}.sort.bam `#bam输出`
```
实例：
```bash
#最大线程数
thread=10
#fasta文件的位置
fasta_origin_file=../ref/neo_by_px_CS_iwgsc_v2.1.fa
#hisat2 index要生成在那个文件夹
hisat2_index_dir=../ref/hisat2-index/
#gtf文件的路径
gtf=../ref/CS_iwgsc_v2.1_HC.gtf
#bam文件的输出目录
out_path=../4-hisat2_mapping/
#fastq的路径
#../result/3-fastp/A1-1/A1-1_clean_1.fq.gz
fastq_dir=../result/3-fastp/
#fastq的后缀
fastq1_suffix=_clean_1.fq.gz
fastq2_suffix=_clean_2.fq.gz
# hisat2的index的前缀，一般不要改
hisat2_prefix=$(basename $fasta_filename .fa)
#开始mapping
for i in $(ls -d ${fastq_dir}/*)
do
        sampleid=$(basename $i ${fastq1_suffix})
        echo "############################################################"
        echo "############################################################"
        echo "[INFO] mapping $sampleid"
        echo "############################################################"
        echo "############################################################"
        hisat2 -x $hisat2_index_dir/$hisat2_prefix -p ${thread} --dta-cufflinks -1 ${fastq_dir}/${sampleid}${fastq1_suffix} -2 ${fastq_dir}/${sampleid}${fastq2_suffix} | samtools view -b -h | samtools sort -m 4G - -o $out_path/${sampleid}.sort.bam
done
```

## 后来，跟STAR一样，遇到坑了

建立index的时候，内存占用特别恐怖，所以，尽量用STAR吧