种群核苷酸多样性，顾名思义指的就是核苷酸多样性，值越大说明核苷酸多样性越高。通常用于衡量群体内的核苷酸多样性，也可以用来推演进化关系。计算公式为：

![核苷酸多样性计算公式](pi.png.png)

计算群体的π值，可以理解成先把群体内每个样本两两求解，再求得群体的均值。

计算的软件最常见的是vcftools，也有对应的R包PopGenome。通常是选定某一基因组区域，设定好窗口大小，然后滑动窗口进行计算。

### 使用VCFTOOLS计算种群核苷酸多样性

#### 画窗计算核苷酸多样性（π）

```
vcftools --vcf YourDataNameWithId.vcf --keep groupid.txt --window-pi 10000 --out YourDataName_pi
``` 

--window-pi 指定窗口的大小，这里我设置了10000，具体大小根据基因组大小选择

--window-pi-step 还可以设置步长，这里没写

#### 计算单位点核苷酸多样性（π）
```
vcftools --vcf YourDataNameWithId.vcf --keep groupid.txt --site-pi --out YourDataName_pi
``` 

--site-pi  单位点计算


原文：https://www.jianshu.com/p/7b6701ae593b
