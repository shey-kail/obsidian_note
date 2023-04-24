### quick solve
问题：出现报错：**Value was put into PairInfoMap more than once**

解决方案：
```
#修复Value was put into PairInfoMap more than once
samtools view your.sorted.bam -f 0x2 -@ 10 -b -o your.sorted.repaired.bam
#允许jvm占100G的内存，保证别挂
java -Xmx100G -Dsamjdk.use_async_io_read_samtools=false -Dsamjdk.use_async_io_write_samtools=true -Dsamjdk.use_async_io_write_tribble=false -Dsamjdk.compression_level=2 -jar /data/Users/pangxi/miniconda/envs/bio-soft/share/gatk4-4.2.6.1-1/gatk-package-4.2.6.1-local.jar MarkDuplicates -I NFS14.sorted.repaired.bam -M NFS14.csv -O NFS14.sorted.rmdup.bam
```

# 嫌太长不看版：
# 坑死了！！！！
我用gatk的MarkDuplicates来markdup，但是他报错说：**Value was put into PairInfoMap more than once**因为我的测序数据出了一点问题，他之前在bwa mapping的时候报错说readname不匹配啥的

当时我用bbmap的repair.sh修复后，mapping并sort成功

但是后面markdup的时候，报上面的恶心错误，焯！

造成原因貌似是修复过的fastq文件有两个重复的reads，貌似是1-fq和2-fq里面有些reads在这俩文件中分别出现了两次。

那么如何解决呢，gatk论坛里面有个大哥说：用：

```
samtools view your.sorted.bam -f 0x2 -@ 10 -b -o your.sorted.repaired.bam
```

然后再gatk MarkDuplicates。对，gatk他这次没报错，他报了一堆:
```
INFO 2021-06-15 02:30:37 MarkDuplicates Tracking 238408 as yet unmatched pairs. 238408 records in RAM.INFO 2021-06-15 02:30:49 MarkDuplicates Read 2,000,000 records. Elapsed time: 00:00:27s. Time for last 1,000,000: 12s. Last read position: CM022019.1:180,050
INFO 2021-06-15 02:30:49 MarkDuplicates Tracking 469361 as yet unmatched pairs. 469361 records in RAM.
INFO 2021-06-15 02:31:00 MarkDuplicates Read 3,000,000 records. Elapsed time: 00:00:38s. Time for last 1,000,000: 10s. Last read position: CM022019.1:238,206
INFO 2021-06-15 02:31:00 MarkDuplicates Tracking 742692 as yet unmatched pairs. 742692 records in RAM.
INFO 2021-06-15 02:31:12 MarkDuplicates Read 4,000,000 records. Elapsed time: 00:00:50s. Time for last 1,000,000: 12s. Last read position: CM022019.1:346,940
INFO 2021-06-15 02:31:12 MarkDuplicates Tracking 1059134 as yet unmatched pairs. 1059134 records in RAM.
INFO 2021-06-15 02:31:22 MarkDuplicates Read 5,000,000 records. Elapsed time: 00:00:59s. Time for last 1,000,000: 9s. Last read position: CM022019.1:443,432
INFO 2021-06-15 02:31:22 MarkDuplicates Tracking 1309709 as yet unmatched pairs. 1309709 records in RAM.
```
然后进程消失，也没出现rmdup后的bam文件。。。

焯！ 

看他这个日志的意思吧，应该是想要往内存里写东西，但是然后他就没了。那么是不是他没内存了导致他没了的呢？多给jvm点内存试试：
```
# -Xmx100G 就是允许jvm占更多的内存
java -Xmx100G -Dsamjdk.use_async_io_read_samtools=false -Dsamjdk.use_async_io_write_samtools=true -Dsamjdk.use_async_io_write_tribble=false -Dsamjdk.compression_level=2 -jar /data/Users/pangxi/miniconda/envs/bio-soft/share/gatk4-4.2.6.1-1/gatk-package-4.2.6.1-local.jar MarkDuplicates -I NFS14.sorted.repaired.bam -M NFS14.csv -O NFS14.sorted.rmdup.bam
```

试了试，成了！yeah！

> https://gatk.broadinstitute.org/hc/en-us/community/posts/4408717387803-SAMException-Value-was-put-into-PairInfoMap-more-than-once

> https://gatk.broadinstitute.org/hc/en-us/community/posts/1260803855350-Picard-MarkDuplicates-get-killed
