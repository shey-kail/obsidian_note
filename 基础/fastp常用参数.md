fastp -z 6 -L -w 10 -e 30 -f 5 -F 5 -t 5 -T 5 -i input_1.fq.gz -I input_2.fq.gz -o input_1.fq.gz -O input_2.fq.gz


-w : 线程数
-e : 一条read的平均质量如果小于30，那么就之间扔掉这个read
-f : 对-i后面的文件，剪裁前5个碱基
-F : 对-I后面的文件，剪裁前5个碱基
-t : 对-i后面的文件，剪裁后5个碱基
-T : 对-I后面的文件，剪裁后5个碱基
-i : input 文件，_1.fq.gz 
-I : input 文件，_2.fq.gz 
-o : output 文件，_1.fq.gz 
-O : output 文件，_2.fq.gz 
-L : --disable_length_filtering 禁用reads长度过滤，要是经过qc发现长度都没啥问题，就可以-L
-z, --compression compression level for gzip output (1 ~ 9). 1 is fastest, 9 is smallest, default is 4. (int [=4])
