# 先要干的事

参考软件官方说明，安装完成后，需要下载参考序列库

```bash
get_organelle_config.py --add embplant_pt,embplant_mt
```

这两个分别是植物质体和线粒体的模版

# 运行Demo

参考官网说明，下载测试数据，正反向各8Mb

```bash
wget https://github.com/Kinggerm/GetOrganelleGallery/raw/master/Test/reads/Arabidopsis_simulated.1.fq.gz
wget https://github.com/Kinggerm/GetOrganelleGallery/raw/master/Test/reads/Arabidopsis_simulated.2.fq.gz
```

按照官网说明，60秒可以组装好拟南芥这套数据

```bash
get_organelle_from_reads.py -1 Arabidopsis_simulated.1.fq.gz -2 Arabidopsis_simulated.2.fq.gz -t 1 -o Arabidopsis_simulated.plastome -F embplant_pt -R 10
```

![[Pasted image 20230218191114.png]]

主要输出结果，  
结果文件看起来有点复杂，没时间折腾，截图Manual。

![[Pasted image 20230218191133.png]]

查看 log 文件，看到有两个完整组装，即成环

![[Pasted image 20230218191151.png]]
 
上述图片中，我们也可以看到有两个.fasta文件，对应的，可以看看  

![[Pasted image 20230218191223.png]]
  
![[Pasted image 20230218191234.png]]

看了下manual，了解了下质体组装的内容，可以认为两者都是正确组装。使用时选择一个常用的即可。只是常用的是哪一个？这是一个问题。对于研究较多的物种，应是可以参考；研究较少的，或许考虑做个多序列比对，mauve，mummer等，投票决定。  

![[Pasted image 20230218191255.png]]
  

一个材料（注意就是一个植物或者一个叶片）中会同时存在两种组装，见文献

> Palmer, J. Chloroplast DNA exists in two orientations. Nature 301, 92–93 (1983). https://doi.org/10.1038/301092a0


# 运行实际测试数据

开 20 个线程试试

```bash
get_organelle_from_reads.py -1 NFS1_ch_1.fq.gz -2 NFS1_ch_2.fq.gz -t 10 -o wild_emmer_wheat/ -F embplant_pt -R 10

// -t 线程数
// -R Maximum extension rounds (suggested: >=2). Default: 15 (embplant_pt)
```

一共耗时 1415.10 s。速度不错，但是并没成环，而是产生了29个scaffold。

![[Pasted image 20230218192247.png]]
  
根据[github](https://github.com/Kinggerm/GetOrganelle/wiki/FAQ#what-should-i-do-with-incomplete-resultbroken-assembly-graph)的相关建议

```
1. reducing word size (`-w`) if your working environment could supply more memory. If no word size was given for the first run, it would be automatically estimated by the program and logged in the `get_org.log.txt`, where you can find the value to start reducing. For example, if the estimated word size was 105, I would try a 95 next. A reasonable value is usually between 65 and 105. Reducing word size will be more helpful when AW/AI seems stable/barely changed as rounds increase in the old run.

2. increasing input reads (`--reduce-reads-for-coverage` or `--max-reads`) if reads were reduced by any of these two options.

3. using a closely-related organelle genome as the seed (`-s`) if the target genome is animal mitogenome (highly recommended), or the quality of the reads is poor, or target coverage is highly uneven. For animal genome assembly or cases without closely-related seed, using the output of the previous run as the seed for a second run will be a good choice.

4. increasing rounds (`-R`). The suggested number of rounds is sufficient for most samples unless your sample has extremely shallow coverage and the seed is highly divergent from the target.
5. using wider and denser _k_-mer gradient (`-k`)
6. narrowing extending steps (`-J`/`-M`)
7. looking for help at [Discussion](https://github.com/Kinggerm/GetOrganelle/discussions) with the get_org.log.txt and the assembly graph (*.fastg or *.png with length/depth/name turned on) attached.
```

实测并没有什么改善，应该是测序深度太低了。

另外，如果成环了，可以用bandage进行可视化。（没成环也可以）

> 参考链接：https://www.jianshu.com/p/bab69c3889b5