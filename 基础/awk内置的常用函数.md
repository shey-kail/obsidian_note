### 与其他程序交互
awk 如果要跟bash进行交互，比如bash脚本中，上一步计算出来了一个数字，需要而下面的awk需要这个数字来做下一步的处理，则可以如此：
```bash

##这个python脚本计算出来了一个数字，把这个数字储存到变量a中

a=$(python3 ./demo.py)

##awk 用-v这个参数，把环境变量中的a读入awk的变量aaa中。

awk -v aaa=${a} 'BEGIN{print aaa}' 
```

### 函数

#### sqrt(num) 取平方根
```
awk 'BEGIN{sqrt(1024.000000)}'
# 结果：32.000000
```

#### int(num) 取整
```
awk 'BEGIN{int(5.12345)}'
# 结果：5
```

#### rand() 返回任意数字 n，其中 0 <= n < 1。 	
```
awk 'BEGIN {
  print "Random num1 =" , rand()
  print "Random num2 =" , rand()
  print "Random num3 =" , rand()
}'
# 结果：
# Random num1 = 0.237788
# Random num2 = 0.291066
# Random num3 = 0.845814
```

#### srand([int]) 设置rand()的随机数种子
```
awk 'BEGIN {
  param = 10

  printf "srand() = %d\n", srand()
  printf "srand(%d) = %d\n", param, srand(param)
}'

# 结果
# srand() = 1
# srand(10) = 1417959587
```

#### sub(before, after, str ) 字符串替换,只换一次
```
awk 'BEGIN {
    str = "Hello, World"

    print "String before replacement = " str

    sub("World", "Jerry", str)

    print "String after replacement = " str
}'
# 结果：
# String before replacement = Hello, World
# String after replacement = Hello, Jerry
```

#### gsub() 全局替换
```
awk 'BEGIN {
    str = "Hello, World, World"

    print "String before replacement = " str

    gsub("World", "Jerry", str)

    print "String after replacement = " str
}'
# 结果：
# String before replacement = Hello, World, World
# String after replacement = Hello, Jerry, Jerry
```

#### substr(str, start, len) 截取字串
```
awk 'BEGIN {
    str = "Hello, World !!!"
    subs = substr(str, 1, 5)

    print "Substring = " subs
}'

# 结果
# Substring = Hello
```

#### index(String1, String2) 查找string2在string1中的位置，如果没有则返回0
```
awk 'BEGIN {
    str = "One Two Three"
    subs = "Two"

    ret = index(str, subs)

    printf "Substring \"%s\" found at %d location.\n", subs, ret
}'
# 结果：
# Substring "Two" found at 5 location.
```

#### length(String) 返回字符串长度（以字符为单位）
```
awk 'BEGIN {
    str = "Hello, World !!!"

    print "Length = ", length(str)
}'
# 结果：
Length = 16
```

#### blength(String) 返回字符串长度（以字节为单位）
```
awk 'BEGIN {
    str = "Hello, World !!!"

    print "Length = ", blength(str)
}'
# 结果：
Length = 16
```

### split(String,arr,[Ere]) 将字符串分割，并且放到arr这个数组里面
```
awk 'BEGIN {
    str = "One,Two,Three,Four"

    split(str, arr, ",")

    print "Array contains following values"

    for (i in arr) {
        print arr[i]
    }
}'
# 结果：
# Array contains following values
# One
# Two
# Three
# Four
```

#### tolower(String) 变小写
```
awk 'BEGIN {
    str = "HELLO, WORLD !!!"

    print "Lowercase string = " tolower(str)
}'
# 结果：
# Lowercase string = hello, world !!!
```

#### toupper(String) 变大写
```
awk 'BEGIN {
    str = "hello, world !!!"

    print "Uppercase string = " toupper(str)
}'
# 结果：
# Uppercase string = HELLO, WORLD !!!
```

