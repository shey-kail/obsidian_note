bwa是一款出色的二代测序DNA mapping工具，这玩意非常带劲，跑得超级慢，因为是穷举，会把所有的可能的mapping结果都穷举到sam文件里面，所以生成的sam也很大。

bwa支持gap，也支持mismatch，所以bwa出来的sam文件支持call SNP和call indel。

对于双端测序来说，bwa会搞一个window，一对reads往一个window上面mapping。所以不至于让一个read的insert过于离谱。

bwa详解：

1. 有三种算法：
* BWA-backtrack  如果说你的reads小于等于100bp，这样你就用这个方法
* BWA-SW  如果你的read是不符合上面的情况，那么就可以用这个。这个算法支持的read长度为70bp——1Mbp（这个支持分割比对）
* BWA-MEM  这个算法跟SW差不多，但是性能更强

2. 我用哪个？
对于70bp或更长的Illumina、454、Ion Torrent和Sanger reads、assembly contigs和BAC序列，BWA-MEM通常是首选算法。对于短序列，BWA-backtrack可能更好。当校准间隙频繁时，BWA-SW可能具有更好的灵敏度。

3. 我用mem或者SW算法时，出现了多个primary alignments？这个正常吗？
这个可能是由于结构变异(structural variations), 基因融合(gene fusion) 或者参考基因组的错误组装(reference misassembly)，但是这事原因并不确定。可以用-M参数把剩余的一些标记为secondary

4. 测序容错如何？
BWA-backtrack主要设计用于测序错误率低于2%的情况。尽管用户可以通过调整命令行选项要求它容忍更多错误，但它的性能会剧烈下降。请注意，对于Illumina读取，BWA-backtrack可以在对齐之前选择性地从3’-端修剪低质量的基，因此能够在尾部以高错误率对齐更多读取，这是Illumina数据的典型情况。

由于校准时间较长，BWA-SW和BWA-MEM都能容忍更多的错误。模拟表明，如果100bp校准的误差为2%，200bp校准的误差为3%，500bp校准的误差为5%，1000bp或更长校准的误差为10%，则它们可能工作良好。

5. bwa能否找嵌合reads（chimeric reads）？
BWA通常为每个read报告一次alignment，但如果read/contig是嵌合体，则可能输出两次或更多alignment。

6. 我发现一对reads中的一个有一个很高的mapping质量，但是这对中的另一个质量为0，这个对吗？
对的，质量值是相对于一个read来说的。

7. 我看到一个在染色体末端read被标记为unmapped（标志0x4）。发生了什么？
BWA内部将所有参考序列连接成一个长序列。read可以mapping到两个相邻参考序列（如chr1A和chr1B）的连接处。在这种情况下，BWA会将读取标记为unmapped，但您会看到pos、CIGAR和所有tags。更好的解决方案是选择一个替代位置或修剪末端alignment，但这在编程中相当复杂，目前尚未实现。

