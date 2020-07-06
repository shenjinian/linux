### 手机视频下载方法二：没有m3u8索引目录，直接根据TS文件顺序暴力下载

一年一度的高考又如期而至，招生宣传工作又开始了。

今年由于疫情的原因，以往面对面的线下宣传改成线上，各种直播讲座紧锣密鼓的开展。

现在各种线上直播平台都是手机端，不支持PC端，更不支持下载

如果感觉某个老师的讲座不错，想把它搬运到其他平台，就要大费周折。因为现在视频禁止下载的限制千奇百怪，总会遇到各种新问题。

前面已经写过一篇文章，大概思路：利用Fiddler手机抓包 ，抓m3u8的索引目录，再用专用的m3u8的下载工具，下载即可，基本是万能办法，可以下载任何HLS的视频，除非是找不到m3u8的索引目录，就像今天这个视频一样。

废话不说，直接上链接

https://wap.yzwb.net/live_casting.html?news_id=338427&kind=9&deviceid=null&version=new&from=singlemessage

显示手机访问，只能利用Fiddler手机抓包，抓包的记录

```
.......
http://recordcdn.quklive.com/broadcast/activity/1583661058304900/20200310/193432_18.ts
http://recordcdn.quklive.com/broadcast/activity/1583661058304900/20200310/193449_19.ts
http://recordcdn.quklive.com/broadcast/activity/1583661058304900/20200310/193505_20.ts
http://recordcdn.quklive.com/broadcast/activity/1583661058304900/20200310/193522_21.ts
http://recordcdn.quklive.com/broadcast/activity/1583661058304900/20200310/193539_22.ts
http://recordcdn.quklive.com/broadcast/activity/1583661058304900/20200310/193555_23.ts
http://recordcdn.quklive.com/broadcast/activity/1583661058304900/20200310/193612_24.ts
http://recordcdn.quklive.com/broadcast/activity/1583661058304900/20200310/193629_25.ts
http://recordcdn.quklive.com/broadcast/activity/1583661058304900/20200310/193645_26.ts
http://recordcdn.quklive.com/broadcast/activity/1583661058304900/20200310/193702_27.ts
http://recordcdn.quklive.com/broadcast/activity/1583661058304900/20200310/193719_28.ts
http://recordcdn.quklive.com/broadcast/activity/1583661058304900/20200310/193735_29.ts
http://recordcdn.quklive.com/broadcast/activity/1583661058304900/20200310/193752_30.ts
..........
```

由于没有抓到m3u8的索引文件，只有ts文件，怎么办？

研究了好长时间ts文件的命名规则，以为是有规律，发现这命名是个分段函数，两个文件一个区间的分段。

浪费了不少时间，直接暴力穷举吧！

直接上代码

```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
# Created by ShenJinian on: 2020-07-05 16:41:12

import os

import requests


def savefile(file_url, file_name, work_dir):
    # 配置headers防止被墙，一般问题不大
    headers = {
        'user-agent':
        'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.116 Safari/537.36'
    }
    r = requests.get(file_url, headers=headers)
    if r.status_code == 200:
        with open(work_dir + '/file_tmp/' + file_name, 'wb') as f:
            f.write(r.content)
    return r.status_code


if __name__ == "__main__":
    #指定工作目录
    work_dir = os.getcwd() + '/m3u8_download/'
    print(work_dir)

    # 用来保存ts文件
    file_dir = os.path.join(work_dir, 'file_tmp')
    if not os.path.exists(file_dir):
        os.mkdir(file_dir)

    #URL分解
    file_url_1 = "http://recordcdn.quklive.com/broadcast/activity/1583661058304900/20200310/"
    file_url_2 = "193008"
    file_url_3 = '_'
    file_url_4 = '0'
    file_url_5 = '.ts'

    while True:
        file_url = file_url_1 + file_url_2 + file_url_3 + file_url_4 + file_url_5
        file_name = file_url_2 + file_url_3 + file_url_4 + file_url_5

        code = savefile(file_url, file_name, work_dir)
        if code == 200:
            file_url_4 = str(int(file_url_4) + 1)
            print("%s 下载完成" % file_name)
        else:
            file_url_2 = str(int(file_url_2) + 1)
            print("%s 请求返回错误，错误码为： %d" % (file_name, code))

```

根据逐次递增的穷举了ts的文件名，拼接成url直接下载，再使用ffmpeg把所有ts文件拼接成mp4文件即可。

这个方法

优点：简单有效

缺点：稍微有点慢

疑惑：为啥这个视频没有m3u8的索引呢？如果没有索引，浏览器是如何识别ts文件的呢？很是疑惑！



后续...

以为视频下载好就结束，怕有遗漏，又抓包看了一下，咦？神奇的一幕发生了

```
http://recordcdn.quklive.com/broadcast/activity/1583661058304900/record.m3u8
```

竟然又索引文件，只是上次抓包，按文件类型排序，没注意到。

哎！这个脑袋。。。。

上面这段代码全当重做了一个ts文件下载的轮子吧！