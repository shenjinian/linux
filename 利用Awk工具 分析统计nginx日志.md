##利用Awk工具 分析统计nginx日志

###知识点：

####1）数组

数组是用来存储一系列值的变量，可通过索引来访问数组的值。

Awk中数组称为关联数组，因为它的下标（索引）可以是数字也可以是字符串。

下标通常称为键，数组元素的键和值存储在Awk程序内部的一个表中，该表采用散列算法，因此数组元素是随机排序。

数组格式：array[index]=value


###1 Nginx日志分析

**日志格式：**

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


**日志记录：**

    173.245.94.202 - 42.81.56.5 - - [07/Oct/2019:06:25:57 +0800] "-/-" "GET /d6/de/c4240a120542/page.htm HTTP/1.1" "200" 28601 "http://www.cpu.edu.cn/" "Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.2)" "173.245.94.202"202.119.191.207:80 0.008 0.005 



####1）统计日志中访问最多的20个IP

**思路：**对第一列进行去重，并输出出现的次数

**方法1：**

    awk '{a[$1]++}END{for(i in a)print a[i],i|"sort -k1 -nr|head -n20"}' access.log > top.log

**方法2：**

    awk '{print $1}' access.log |sort |uniq -c |sort -k1 -nr |head -n20 > top.log

**说明：**a[$1]++ 创建数组a，以第一列作为下标，使用运算符++作为数组元素，元素初始值为0。处理一个IP时，下标是IP，元素加1，处理第二个IP时，下标是IP，元素加1，如果这个IP已经存在，则元素再加1，也就是这个IP出现了两次，元素结果是2，以此类推。因此可以实现去重，统计出现次数。

####2）统计日志中访问大于100次的IP

**方法1：**

    awk '{a[$1]++}END{for(i in a){if(a[i]>100)print i,a[i]}}' access.log

**方法2：**

    awk '{a[$1]++;if(a[$1]>100){b[$1]++}}END{for(i in b){print i,a[i]}}' access.log

 

**说明：**方法1是将结果保存a数组后，输出时判断符合要求的IP。方法2是将结果保存a数组时，并判断符合要求的IP放到b数组，最后打印b数组的IP。

####3）统计2019年10月7日6：30到24:00内访问最多的10个IP

**思路：**先过滤出这个时间段的日志，然后去重，统计出现次数

    awk '$6>="[07/Oct/2019:06:30:00" && $6<="[07/Oct/2019:23:59:59" {a[$1]++}END{for(i in a)print a[i],i|"sort -k1 -nr|head -n10"}' access.log
 
**说明：**前提开始时间与结束时间日志中必须存在

####4）统计当前时间前一分钟的访问数

**思路：**先获取当前时间前一分钟对应日志格式的时间，再匹配统计
 
    date=$(date -d '-1 minute' +%d/%b/%Y:%H:%M);awk -vdate=$date '$0~date{c++}END{print c}' access.log
    date=$(date -d '-1 minute' +%d/%b/%Y:%H:%M);awk -vdate=$date '$6>="["date":00" && $6<="["date":59"{c++}END{print c}' access.log
    grep -c $(date -d '-1 minute' +%d/%b/%Y:%H:%M) access.log
 
**说明：**date +%d/%b/%Y:%H:%M --> 09/Apr/2016:01:55

####5）统计访问最多的前10个页面（$request）

    awk '{a[$10]++}END{for(i in a)print a[i],i|"sort -k1 -nr|head -n10"}' access.log > top.log

####6）统计每个URL访问内容的总大小（$body_bytes_sent）

    awk '{a[$13]++;size[$13]+=$10}END{for(i in a)print a[i],size[i],i}' access.log

####7）统计每个IP访问状态码数量（$status）
    awk '{a[$1" "$12]++}END{for(i in a)print i,a[i]}' access.log

####8）统计访问状态码为503的IP及出现次数
    awk '{if($12~/503/)a[$1" "$12]++}END{for(i in a)print i,a[i]}' access.log

####9）统计503错误的页面和数量
    awk '($12 ~ /503/)' access.log | awk '{print $12}' | sort | uniq -c | sort -rn > 503.log




