~~1. 建立bam索引的时候，报错：failed to create index for "NFS1.sorted.bam": Numerical result out of range~~

~~* 解决办法，安装一个0.1.13版本的samtools~~

~~2. samtools: error while loading shared libraries: libncurses.so.5: cannot open shared object file: No such file or directory~~

~~* 解决办法：conda install -c conda-forge ncurses （用conda-forge的ncurses）~~


1. 建立bam索引的时候，报错：failed to create index for "NFS1.sorted.bam": Numerical result out of range

* samtools index -c NSF1.sorted.bam

csi格式的索引支持的染色体长度比bai的长，所以用csi的就行了
