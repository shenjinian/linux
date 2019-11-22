### sed 替换字符串，awk和cut取行常用操作



#### 1.要处理的日志格式

```
10.3.3.148 - 10.3.3.148 - - [21/Nov/2019:*10:10:21* +0800] "TLSv1.2/ECDHE-RSA-AES128-GCM-SHA256" 111.cpu.edu.cn "GET /_css/tpl2/default/default.css HTTP/2.0" "200" 1496 "https://111.cpu.edu.cn/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36" "-"202.119.191.203:80 0.004 0.002  
```



#### 2.sed替换和删除

##### 2.1 替换

**把字符[替换成空格，[需要用\转义**

```
sed  's/\[/ /' host.*.cpu.edu.cn.access.log
```

**把字符]替换成空格，不需要转义**

```
sed  's/]/ /' host.*.cpu.edu.cn.access.log
```

**把字符""换成空格，不需要转义，g表示全部**

```
sed  's/"/ /g' host.*.cpu.edu.cn.access.log
```

##### 2.2 删除字符串

**把字符[删除，[需要用\转义**

```
sed  's/\[//' host.*.cpu.edu.cn.access.log
```

**把字符]删除，不需要转义**

```
sed  's/]//' host.*.cpu.edu.cn.access.log
```

**把字符""删除，不需要转义，g表示全部**

```
sed  's/"//g' host.*.cpu.edu.cn.access.log
```

**删除多个字符**

```
sed -e 's/\[//' -e 's/]//' -e 's/"//g' host.*.cpu.edu.cn.access.log
```

##### 2.3 删除行

**删除第一行**

```
sed -i '1d' access.log
```

**删除最后一行**

```
 sed -i '$d' access.log
```

**删除指定行**

```
 sed -i '3d' access.log
```

**删除包含特定字符的行**

```
sed -i '/line2/d' access.log
```

**删除注释行和空行**

Linux配置项注释多为'#'开头的行，当然也有以';'开头的，视情况而定

```
sed -i -c -e '/^#/d' config.conf           #sed去除注释行
sed -i -c -e '/^$/d' config.conf           #sed去除空行
sed -i -c -e '/^$/d;/^#/' config.conf      #sed去空行和注释行
```



#### 3.awk取行（间隔可以多个空格）

**3.1 取行**

```
awk '{print $1 " " $3 " " $6 " " $10 " " $9 " " $11 " " $8 " " $12 " " $13 " " $14 " " $15 " " $(NF-3) " " $(NF-2) " " $(NF-1) " " $(NF)}' host.*.cpu.edu.cn.access.log
```

**3.2 取行后的日志格式**

```
223.98.163.218 120.199.93.10 [22/Nov/2019:08:57:24 "GET yjsy.cpu.edu.cn /6288/list.htm "-/-" HTTP/1.1" "200" 3133 "://yjsy.cpu.edu.cn/6285/list.htm" 3.5.30729)" "223.98.163.218"202.119.191.203:80 0.004 0.005
```



#### 4. 日志合并

```
sed -e 's/\[//' -e 's/]//' -e 's/"//g' host.*.cpu.edu.cn.access.log|awk '{print $1 " " $3 " " $6 " " $10 " " $9 " " $11 " " $8 " " $12 " " $13 " " $14 " " $15 " " $(NF-3) " " $(NF-2) " " $(NF-1) " " $(NF)}' > access.log
```



#### 5. cut截取（间隔只能是一个空格）

**访问最多的页面top20**

```
cut -f5,6 -d' ' access.log |sort|uniq -c|sort -nr|head -20
```

**IP访问域名状态为503的top20**

```
grep "503" access.log|cut -f1,5 -d' '|sort|uniq -c|sort -nr|head -20
```

**传输时间超过 30 秒的前20个页面**

```
awk '($NF > 30)' access.log|cut -f15,1,5,6 -d' '|sort|uniq -c|sort -nr|head -20
```



