##利用Awk工具 分析统计nginx日志

###1.知识点：

####1.1 数组

数组是用来存储一系列值的变量，可通过索引来访问数组的值。

Awk中数组称为关联数组，因为它的下标（索引）可以是数字也可以是字符串。

下标通常称为键，数组元素的键和值存储在Awk程序内部的一个表中，该表采用散列算法，因此数组元素是随机排序。

数组格式：array[index]=value

####1.2 AWK取列


awk同sed命令类似，只不过sed擅长取行，awk命令擅长取列。

列之间是以空格分隔计数。

	$clientRealIP                   第1列：$1
	$remote_addr                    第2列：$2
	[$time_local] 	                第6列：$6

	$request（方法）                 第9列：$9
	$request（URL）                 第10列：$10
	$request（协议）                 第11列：$11

    $status                         第12列：$12 
	$body_bytes_sent                第13列：$13 
	$http_referer                   第14列：$14 
    $http_user_agent                第15列 至 倒数5列
	$http_x_forwarded_for	        倒数4列：$(NF-3)
	$upstream_addr                  倒数3列：$(NF-2)
	$upstream_response_time         倒数2列：$(NF-1)
	$request_time                   倒数1列：$(NF)


	cat access.log | awk '{print $1 " " $3 " " $6 " " $9 " " $10 " " $11 " " $12 " " $13 " " $14 " #### " " "  $(NF-3) " "  $(NF-2) $(NF-1) " "  $(NF)}'|sort -nr|head -10

####1.3 sort排序

sort 命令对 File 参数指定的文件中的行排序，并将结果写到标准输出。如果 File 参数指定多个文件，那么 sort 命令将这些文件连接起来，并当作一个文件进行排序。

**sort语法**

	[root@www ~]# sort [-fbMnrtuk] [file or stdin]
	选项与参数：
	-f  ：忽略大小写的差异，例如 A 与 a 视为编码相同；
	-b  ：忽略最前面的空格符部分；
	-M  ：以月份的名字来排序，例如 JAN, DEC 等等的排序方法；
	-n  ：使用『纯数字』进行排序(默认是以文字型态来排序的)；
	-r  ：反向排序；
	-u  ：就是 uniq ，相同的数据中，仅出现一行代表；
	-t  ：分隔符，默认是用 [tab] 键来分隔；
	-k  ：以那个区间 (field) 来进行排序的意思

####1.4 uniq去重

 uniq命令可以去除排序过的文件中的重复行，因此uniq经常和sort合用。也就是说，为了使uniq起作用，所有的重复行必须是相邻的。

**uniq语法**
	
	[root@www ~]# uniq [-icu]
	选项与参数：
	-i   ：忽略大小写字符的不同；
	-c  ：进行计数
	-u  ：只显示唯一的行

####1.5 cut提取文本列

cut命令可以从一个文本文件或者文本流中提取文本列。

**cut语法**

	[root@www ~]# cut -d'分隔字符' -f fields <==用于有特定分隔字符
	[root@www ~]# cut -c 字符区间            <==用于排列整齐的信息
	选项与参数：
	-d  ：后面接分隔字符。与 -f 一起使用；
	-f  ：依据 -d 的分隔字符将一段信息分割成为数段，用 -f 取出第几段的意思；
	-c  ：以字符 (characters) 的单位取出固定字符区间；

####1.6 wc统计

统计文件里面有多少单词，多少行，多少字符。

**wc语法**

	[root@www ~]# wc [-lwm]
	选项与参数：
	-l  ：仅列出行；
	-w  ：仅列出多少字(英文单字)；
	-m  ：多少字符；


###2. Nginx日志

####2.1 日志格式：

    log_format  nohost '$host $remote_addr [$time_local] "$request" '
                       '"$status" $body_bytes_sent "$http_referer" '
                       '"$http_user_agent" "$http_x_forwarded_for"';

    log_format  main  '$remote_addr - $remote_user [$time_local] "$ssl_protocol/$ssl_cipher" "$request" '
                      '"$status" $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    log_format  access '$clientRealIP - $remote_addr - $remote_user [$time_local] "$ssl_protocol/$ssl_cipher" "$request" '
                       '"$status" $body_bytes_sent "$http_referer" '
                       '"$http_user_agent" "$http_x_forwarded_for"'
					   '$upstream_addr $upstream_response_time $request_time';

####2.2 日志记录：

    173.245.94.202 - 42.81.56.5 - - [07/Oct/2019:06:25:57 +0800] "-/-" "GET /d6/de/c4240a120542/page.htm HTTP/1.1" "200" 28601 "http://www.cpu.edu.cn/" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.2)" "173.245.94.202"202.119.191.207:80 0.008 0.005 

###3. 有关IP的统计（$clientRealIP）

####3.1 统计日志中访问最多的20个IP

**思路：**对第一列进行去重，并输出出现的次数

**方法1：**

    awk '{a[$1]++}END{for(i in a)print a[i],i|"sort -k1 -nr|head -n20"}' access.log > top.log

**方法2：**

    awk '{print $1}' access.log |sort |uniq -c |sort -k1 -nr |head -n20 > top.log

**说明：**a[$1]++ 创建数组a，以第一列作为下标，使用运算符++作为数组元素，元素初始值为0。处理一个IP时，下标是IP，元素加1，处理第二个IP时，下标是IP，元素加1，如果这个IP已经存在，则元素再加1，也就是这个IP出现了两次，元素结果是2，以此类推。因此可以实现去重，统计出现次数。

####3.2 统计日志中访问大于100次的IP

**方法1：**

    awk '{a[$1]++}END{for(i in a){if(a[i]>100)print i,a[i]}}' access.log

**方法2：**

    awk '{a[$1]++;if(a[$1]>100){b[$1]++}}END{for(i in b){print i,a[i]}}' access.log

 
**说明：**方法1是将结果保存a数组后，输出时判断符合要求的IP。方法2是将结果保存a数组时，并判断符合要求的IP放到b数组，最后打印b数组的IP。

####3.3 统计2019年10月7日6：30到24:00内访问最多的10个IP

**思路：**先过滤出这个时间段的日志，然后去重，统计出现次数

    awk '$6>="[07/Oct/2019:06:30:00" && $6<="[07/Oct/2019:23:59:59" {a[$1]++}END{for(i in a)print a[i],i|"sort -k1 -nr|head -n10"}' access.log
 
**说明：**前提开始时间与结束时间日志中必须存在

####3.4 统计某一个IP访问了哪些页面：

	grep ^202.119.189.160 access.log | awk '{print $1,$10}'

####3.5 统计某一个IP连接数最高的IP都在干些什么

	cat access.log | grep "202.119.189.160" | awk '{print $10}' | sort | uniq -c | sort -nr | head -n 10

####3.6 统计2019年10月7日14时这一个小时内有多少IP访问:

	awk '{print $6,$1}' access.log | grep '07/Oct/2019:14' | awk '{print $2}'| sort | uniq | wc -l

####3.7 统计以小时单位里ip连接数最多的10个时段

	awk -vFS="[:]" '{gsub("-.*","",$1);num[$2" "$1]++}END{for(i in num)print i,num[i]}' access.log | sort -n -k 3 -r | head -10

####3.8 统计访问次数最多的几个分钟

	 awk '{print $1}' access.log |cut -c 14-18|sort|uniq -c|sort -nr|head -10

####3.9 统计每个IP访问的页面数进行从小到大排序：

    awk '{++S[$1]} END {for (a in S) print a,S[a]}' access.log | sort -n -t ' ' -k 2

###4. 有关状态码的统计（$status）

####4.1 统计http status

**方法1：**

	cat access.log |awk '{counts[$(12)]+=1}; END {for(code in counts) print code, counts[code]}'

**方法2：**

	cat access.log |awk '{print $12}'|sort|uniq -c|sort -rn

####4.2 统计每个IP访问状态码数量（$status）

    awk '{a[$1" "$12]++}END{for(i in a)print i,a[i]}' access.log

####4.3 统计访问状态码为503的IP及出现次数

    awk '{if($12~/503/)a[$1" "$12]++}END{for(i in a)print i,a[i]}' access.log

####4.4 统计503错误的页面和数量

    awk '($12 ~ /503/)' access.log | awk '{print $12}' | sort | uniq -c | sort -rn > 503.log

####4.5 统计403的连接

	awk '($12 ~/403/)' access.log | awk '{print $1,$10}' | sort

####4.6 统计503的连接

	awk '($12 ~/503/)' access.log | awk '{print $1,$10}' | sort

###5. 有关页面的统计（$request）

####5.1 统计访问最多的前10个页面

    awk '{a[$10]++}END{for(i in a)print a[i],i|"sort -k1 -nr|head -n10"}' access.log > top.log

####5.2 统计访问次数最多的20个文件或页面：

    cat access.log|awk '{print $10}'|sort|uniq -c|sort -nr | head -20

####5.3 分析日志文件下 2019-10-07下午2点一个小时 访问页面最高 的前20个 URL 并排序

	cat access.log |grep '07/Oct/2019:14' | awk '{print $10}'|sort|uniq -c|sort -nr|head -20

####5.4 统计列出输出大于200000byte(约200kb)的页面以及对应页面发生次数

	cat access.log |awk '($13 > 200000){print $10}'|sort -n|uniq -c|sort -nr|head -100

####5.5 统计受访问页面的URL地址中 含有/_upload/路径的 IP 地址

	cat access.log | awk '($10~_upload){print $1,$10}'|sort|uniq -c|sort -nr

####5.6 统计 2019/10/7的12点-14点的时间段 访问/_upload/路径的IP，并倒序排列

	cat access.log | egrep '07/Oct/2019:12|07/Oct/2019:14' | awk '($10~_upload) {print $1,$10}'|sort|uniq -c|sort -nr

####5.7 统计某一个页面被访问的次数：

    grep "/index.php" access.log | wc -l

###6. 有关页面耗时的统计（$request_time ）

####6.1  统计列出传输时间超过 30 秒的文件

	cat access.log |awk '($NF > 30){print $10}'|sort -n|uniq -c|sort -nr|head -20

####6.2 统计访问路径含有/_upload/，最耗时的一百个页面

	cat access.log |awk '($10~_upload){print $NF " " $1 " " $10}'|sort -nr|head -100

####6.3 统计最耗时的包含访问/_upload/路径页面(超过60秒的)，以及对应页面发生次数

	cat access.log |awk '($NF > 60 && $10~_upload){print $10}'|sort -n|uniq -c|sort -nr|head -100

###7. 有关流量的统计（$body_bytes_sent）

####7.1 统计网站流量（单位：G)

	cat access.log |awk '{if($9~/GET/) count+=$13} END {print "client_request="count/1024/1024/1024}'

####7.2 统计每个URL访问内容的总大小（单位：k)

	awk '{a[$10]++;size[$10]+=$13} END {for(i in a) print a[i],size[i]/1024,i}' access.log

###8. 有关跳转页面的统计（$http_referer）

####8.1 统计通过子域名访问次数，依据referer来计算，稍有不准

	cat access.log | awk '{print $14}' | sed -e ' s/http:\/\///' -e ' s/\/.*//' | sort | uniq -c | sort -rn | head -20

###9. 有关跳转页面的统计（$http_user_agent） 

####9.1 统计去掉搜索引擎统计的页面

	awk '{print $15,$1}' access.log  | grep ^\"Mozilla | awk '{print $1}' |sort | uniq | wc -l










	


