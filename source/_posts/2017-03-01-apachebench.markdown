---
layout:     post
title:      "使用 Apache AB 对服务器进行压力测试"
subtitle:   ""
date:       2017-03-01 12:00:00
author:     "ZWY"
header-img: "img/in-post/3-1/banner.jpg"
catalog: true
tags:
    - 压测
    - 教程
    - 服务器
---

# ab
**ab（Apache Bench）**是apache自带的压力测试工具。ab非常实用，它不仅可以对apache服务器进行网站访问压力测试，也可以对或其它类型的服务器进行压力测试。比如nginx、tomcat、IIS等。

# 安装
我使用的环境为**Ubuntu-14.04**,输入命令：
`apt-get install apache2-utils`
<br>安装成功后就可以开始进行测试。

<!-- more -->

# 测试
测试命令的格式为：
<br> `ab [options] [http://]hostname[:port]/path`
<br> options为参数，有以下几种：
- -A auth-username:password
    对服务器提供BASIC认证信任。 用户名和密码由一个:隔开，并以base64编码形式发送。 无论服务器是否需要(即, 是否发送了401认证需求代码)，此字符串都会被发送。
- -c concurrency
    一次产生的请求个数。默认是一次一个。
* -C cookie-name=value
    对请求附加一个Cookie:行。 其典型形式是name=value的一个参数对。 此参数可以重复。
* -d
    不显示"percentage served within XX [ms] table"的消息(为以前的版本提供支持)。
* -e csv-file
    产生一个以逗号分隔的(CSV)文件， 其中包含了处理每个相应百分比的请求所需要(从1%到100%)的相应百分比的(以微妙为单位)时间。 由于这种格式已经“二进制化”，所以比'gnuplot'格式更有用。
* -g gnuplot-file
    把所有测试结果写入一个'gnuplot'或者TSV (以Tab分隔的)文件。 此文件可以方便地导入到Gnuplot, IDL, Mathematica, Igor甚至Excel中。 其中的第一行为标题。
* -h 显示使用方法。
* -H custom-header 对请求附加额外的头信息。
    此参数的典型形式是一个有效的头信息行，其中包含了以冒号分隔的字段和值的对 (如, "Accept-Encoding: zip/zop;8bit").
* -i 执行HEAD请求，而不是GET。
* -k 启用HTTP KeepAlive功能
    在一个HTTP会话中执行多个请求。 默认时，不启用KeepAlive功能. -n requests 在测试会话中所执行的请求个数。 默认时，仅执行一个请求，但通常其结果不具有代表意义。
* -p POST-file包含了需要POST的数据的文件.
* -P proxy-auth-username:password
    对一个中转代理提供BASIC认证信任。 用户名和密码由一个:隔开，并以base64编码形式发送。 无论服务器是否需要(即, 是否发送了401认证需求代码)，此字符串都会被发送。
* -q    如果处理的请求数大于150， ab每处理大约10%或者100个请求时，会在stderr输出一个进度计数。 此-q标记可以抑制这些信息。
* -s    用于编译中(ab -h会显示相关信息)使用了SSL的受保护的https， 而不是http协议的时候。此功能是实验性的，也是很简陋的。最好不要用。
* -S 不显示中值和标准背离值。而且在均值和中值为标准背离值的1到2倍时，也不显示警告或出错信息。 默认时，会显示 最小值/均值/最大值等数值。(为以前的版本提供支持).
* -t timelimit 测试所进行的最大秒数。其内部隐含值是-n 50000。 它可以使对服务器的测试限制在一个固定的总时间以内。默认时，没有时间限制。
* -T content-type POST数据所使用的Content-type头信息。
* -v verbosity 设置显示信息的详细程度
    4或更大值会显示头信息， 3或更大值可以显示响应代码(404, 200等), 2或更大值可以显示警告和其他信息。
* -V 显示版本号并退出。
* -w 以HTML表的格式输出结果。默认时，它是白色背景的两列宽度的一张表。
* -x <table>-attributes 设置<table>属性的字符串。 此属性被填入<table 这里 >.
* -X proxy[:port] 对请求使用代理服务器。
* -y <tr>-attributes 设置<tr>属性的字符串.
* -z <td>-attributes 设置<td>属性的字符串

参数很多，但我们通常使用到的是 **-n** 和 **-c**,即**总的请求数** 和**同时并发数**。
<br>在命令行输入：
`ab -n 100 -c 20 http://www.test.com/`
即对此域名的服务器进行**并发数20，总的请求为100**的压力测试。

# 实例
首先，在我本机运行一个tornado的http服务器的demo，然后输入测试命令：
<br>`ab -n 2000 -c 100 http://localhost:9998/`
![img](/img/in-post/3-1/3-1-1.png)
然后可以看到输出的测试结果：
```
Server Software:        TornadoServer/4.4.1
Server Hostname:        localhost
Server Port:            9998
//这段是服务器信息，可以看到使用的是Tornado，地址为本地，端口为9998

Document Path:          /
Document Length:        790 bytes
//测试的页面和页面大小

Concurrency Level:      100
//测试的并发数
Time taken for tests:   4.746 seconds
//测试持续时长
Complete requests:      2000
//测试的请求数
Failed requests:        0
//失败的请求数
Total transferred:      1972000 bytes
//整个过程中数据量
HTML transferred:       1580000 bytes
//整个过程中HTML数据量
Requests per second:    421.43 [#/sec] (mean)
//平均每秒处理请求数
Time per request:       237.288 [ms] (mean)
//平均每个请求处理时长（并发数为一个整体，即100个并发同时请求）
Time per request:       2.373 [ms] (mean, across all concurrent requests)
//平均每个请求处理时长(对于并发请求，cpu实际上并不是同时处理的，而是按照每个请求获得的时间片逐个轮转处理的，所以基本上第一个Time per request时间约等于第二个Time per request时间乘以并发请求数)
Transfer rate:          405.79 [Kbytes/sec] received
//平均每秒网络上的流量，可以帮助排除是否存在网络流量过大导致响应时间延长的问题

Connection Times (ms)
            min  mean[+/-sd] median   max
Connect:        0   17  75.6      0     443
Processing:     6  209 350.2     98    1720
Waiting:        6  208 349.5     98    1716
Total:          6  226 420.4     98    2156
//网络上消耗的时间的分解

Percentage of the requests served within a certain time (ms)
50%     98
66%    108
75%    114
80%    121
90%    507
95%    534
98%   2058
99%   2093
100%   2156 (longest request)
//整个场景中所有请求的响应情况。在场景中每个请求都有一个响应时间，其中50％的用户响应时间小于98毫秒，最大的响应时间小于2156毫秒。
```
# 几个重要指标

* **吞吐率（Requests per second）**
<br>概念：服务器并发处理能力的量化描述，单位是reqs/s，指的是某个并发用户数下单位时间内处理的请求数。某个并发用户数下单位时间内能处理的最大请求数，称之为最大吞吐率。
计算公式：总请求数 / 处理完成这些请求数所花费的时间，即
Request per second = Complete requests / Time taken for tests

* **并发连接数（The number of concurrent connections）**
<br>概念：某个时刻服务器所接受的请求数目，简单的讲，就是一个会话。

* **并发用户数（The number of concurrent users，Concurrency Level）**
<br>概念：要注意区分这个概念和并发连接数之间的区别，一个用户可能同时会产生多个会话，也即连接数。

* **用户平均请求等待时间（Time per request）**
<br>计算公式：处理完成所有请求数所花费的时间/ （总请求数 / 并发用户数），即
Time per request = Time taken for tests /（ Complete requests / Concurrency Level）

* **服务器平均请求等待时间（Time per request: across all concurrent requests）**
<br>计算公式：处理完成所有请求数所花费的时间 / 总请求数，即
Time taken for / testsComplete requests
可以看到，它是吞吐率的倒数。
同时，它也=用户平均请求等待时间/并发用户数，即
Time per request / Concurrency Level
