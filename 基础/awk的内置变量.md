### awk 的内置变量

##### ARGC 参数数量
* 执行`awk 'BEGIN {print "Arguments =", ARGC}' One Two Three Four`
* 得到`Arguments = 5` 

##### ARGV 参数数组
* 执行`awk 'BEGIN { for(i=0; i<ARGC-1; ++i) { printf "ARGV[%d]=%s\t", i, ARGV[i] } }' one two three four`
* 得到`ARGV[0]=awk	ARGV[1]=one		ARGV[2]=two		ARGV[3]=three`

##### CONVFMT 数字转换格式 （感觉用处不大的样子）
* 执行`awk 'BEGIN { print "Conversion Format =", CONVFMT }'`
* 得到`Conversion Format = %.6g`

##### ENVIRON 环境变量的关联字典（以字典的方式访问环境变量）
* 执行`awk 'BEGIN { print ENVIRON["USER"] }'`
* 得到`shey`（我当前的用户名）

##### FILENAME 当前文件名
* 执行`awk 'END {print FILENAME}' marks.txt`
* 得到`marks.txt`

##### FS 字段分隔符（也可以通过-F设定）
这里不多说了，知道-F是啥意思，这个就知道了，我一般不喜欢用这个

##### NF 字段数量（可以用来代表最后一列）
* 如果我要制空最后一列，那么可以这样做：
* `awk 'BEGIN{OFS="\t"} {NF="";print $0}' test.txt | sed 's/\t$//g'`

##### NR 和 FNR 当前记录的编号（总计处理的行数）和当前文件的记录的编号（当前文件处理的行数）
* 很熟悉 不多讲

##### OFMT 它表示输出格式编号，其默认值为%.6g 。
* 好像没什么用的样子

##### OFS 它表示输出字段分隔符，其默认值为space。 
* 常用，不多说 ，上面也有关于NF的例子说了

##### ORS 表示输出记录的分隔符，默认为换行符
* 一般谁没事改这玩意

##### RS 输入记录的分隔符，默认为换行符
* 同上，一般谁没事改这玩意

##### RLENGTH 表示match函数匹配的字符串的长度。AWK的匹配函数在输入字符串中搜索给定的字符串。
* 执行`awk 'BEGIN { if (match("One Two Three", "re")) { print RLENGTH } }'`
* 得到`2` （re的长度就是2）

##### RSTART 表示match函数匹配的字符串中的第一个位置。
* 执行`awk 'BEGIN { if (match("One Two Three", "Thre")) { print RSTART } }'`
* 得到`9` 

##### SUBSEP 表示数组下标的分隔符（不知道有啥用）

##### ARGIND 它表示正在处理的当前文件的ARGV中的索引。 
* 执行`awk '{ print "ARGIND   = ", ARGIND; print "Filename = ", ARGV[ARGIND] }' junk1 junk2 junk3`
* 得到 
```
ARGIND   =  1
Filename =  junk1
ARGIND   =  2
Filename =  junk2
ARGIND   =  3
Filename =  junk3
```




