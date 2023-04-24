**OrthoFinder**的work flow见下图
![[Pasted image 20230313100634.png]]

OrthoFinder所需的输入数据很简单，把每个物种的蛋白序列放进单独的fasta文件中，然后把这些fasta文件放到一个目录下。fasta文件命名为对应的物种名。

```bash
orthofinder -f Dataset_ directory
```

如果你想更改线程数，使用-t参数即可修改。默认的比对工具是DIAMOND，你也可以通过-S指定blast等其他工具。其他参数详情可以运行 orthofinder -h 看到。

而一般情况下，我们只需要得到同源群（orthogroups），所以要这样：

```bash
orthofinder -og -f Dataset_ directory
```

这样输出出来的结果文件有：

```text
Blast*.txt
Orthogroups.tsv (Orthogroups.csv works to, be sure to specify this file using the -og flag)
SequenceIDs.txt
SpeciesIDs.txt
```

Orthogroups.tsv就是我们要的同源群