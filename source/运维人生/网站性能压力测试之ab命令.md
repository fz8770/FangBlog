
# 网站性能压力测试之ab命令

ab是Apache自带的压力测试工具。ab非常实用，它不仅可以对Apache服务器进行网站访问压力测试，也可以对其它类型的服务器进行压力测试。比如Nginx、Tomcat、IIS等。

## 一 ab原理

ab是apachebench命令的缩写。

ab的原理：ab命令会创建多个并发访问线程，模拟多个访问者同时对某一URL地址进行访问。它的测试目标是基于URL的，因此，它既可以用来测试apache的负载压力，也可以测试nginx、lighthttp、tomcat、IIS等其它Web服务器的压力。

ab命令对发出负载的计算机要求很低，它既不会占用很高CPU，也不会占用很多内存。但却会给目标服务器造成巨大的负载，其原理类似CC攻击。自己测试使用也需要注意，否则一次上太多的负载。可能造成目标服务器资源耗完，严重时甚至导致死机。

## 二 ab安装

```shell
yum install httpd-tools
```
命令执行完成后，就可以直接运行ab。

## 三 ab参数说明

-n：在测试会话中所执行的请求个数。默认时，仅执行一个请求。

-c：一次产生的请求个数。默认是一次一个。

-t：测试所进行的最大秒数。其内部隐含值是-n 50000，它可以使对服务器的测试限制在一个固定的总时间以内。默认时，没有时间限制。

-p：包含了需要POST的数据的文件。

-P：对一个中转代理提供BASIC认证信任。用户名和密码由一个:隔开，并以base64编码形式发送。无论服务器是否需要(即是否发送了401认证需求代码)，此字符串都会被发送。

-T：POST数据所使用的Content-type头信息。

-v：设置显示信息的详细程度-4或更大值会显示头信息，3或更大值可以显示响应代码(404,200等),2或更大值可以显示警告和其他信息。

-V：显示版本号并退出。

-w：以HTML表的格式输出结果。默认时，它是白色背景的两列宽度的一张表。

-i：执行HEAD请求，而不是GET。

-x：设置<table>属性的字符串。

-X：对请求使用代理服务器。

-y：设置<tr>属性的字符串。

-z：设置<td>属性的字符串。

-C：对请求附加一个Cookie:行。其典型形式是name=value的一个参数对，此参数可以重复。

-H：对请求附加额外的头信息。此参数的典型形式是一个有效的头信息行，其中包含了以冒号分隔的字段和值的对(如,"Accept-Encoding:zip/zop;8bit")。

-A：对服务器提供BASIC认证信任。用户名和密码由一个:隔开，并以base64编码形式发送。无论服务器是否需要(即,是否发送了401认证需求代码)，此字符串都会被发送。

-h：显示使用方法。

-d：不显示"percentage served within XX [ms] table"的消息(为以前的版本提供支持)。

-e：产生一个以逗号分隔的(CSV)文件，其中包含了处理每个相应百分比的请求所需要(从1%到100%)的相应百分比的(以微妙为单位)时间。由于这种格式已经“二进制化”，所以比'gnuplot'格式更有用。

-g：把所有测试结果写入一个'gnuplot'或者TSV(以Tab分隔的)文件。此文件可以方便地导入到Gnuplot,IDL,Mathematica,Igor甚至Excel中。其中的第一行为标题。

-i：执行HEAD请求，而不是GET。

-k：启用HTTP KeepAlive功能，即在一个HTTP会话中执行多个请求。默认时，不启用KeepAlive功能。

-q：如果处理的请求数大于150，ab每处理大约10%或者100个请求时，会在stderr输出一个进度计数。此-q标记可以抑制这些信息。


## 四 ab性能指标

在进行性能测试过程中有几个指标比较重要：

吞吐率（Requests per second）
概念：服务器并发处理能力的量化描述，单位是reqs/s，指的是某个并发用户数下单位时间内处理的请求数。某个并发用户数下单位时间内能处理的最大请求数，称之为最大吞吐率。
计算公式：总请求数 / 处理完成这些请求数所花费的时间，即
Request per second = Complete requests / Time taken for tests

并发连接数（The number of concurrent connections）
概念：某个时刻服务器所接受的请求数目，简单的讲，就是一个会话。

并发用户数（The number of concurrent users，Concurrency Level）
概念：要注意区分这个概念和并发连接数之间的区别，一个用户可能同时会产生多个会话，也即连接数。

用户平均请求等待时间（Time per request）
计算公式：处理完成所有请求数所花费的时间/总请求数，即
Time per request=Time taken for tests/（Complete requests/Concurrency Level）

服务器平均请求等待时间（Time per request: across all concurrent requests）
计算公式：处理完成所有请求数所花费的时间 / 总请求数，即
Time taken for / testsComplete requests
可以看到，它是吞吐率的倒数。
同时，它也=用户平均请求等待时间/并发用户数，即
Time per request / Concurrency Level


## 五 ab实际使用

ab的命令参数比较多，我们经常使用的是-c和-n参数。

```shell

-n 100表示请求总数为100
-c 10表示并发用户数为10

下面表示处理100个请求并每次同时运行10次请求。

ab -n 100 -c 10 http://www.163.com/index.html

This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking www.163.com (be patient).....done


Server Software:        nginx  # 被测试的Web服务器软件名称
Server Hostname:        www.163.com   # 请求的URL主机名
Server Port:            80  # 监听端口

Document Path:          /index.html  # URL中的根绝对路径
Document Length:        697909 bytes # HTTP响应数据的正文长度

Concurrency Level:      10  # 并发用户数，我们设置的参数之一
Time taken for tests:   3.717 seconds # 压力测试消耗的总时间
Complete requests:      100  # 总请求数量，即并发数, 我们设置的参数之一
Failed requests:        0   # 表示失败的请求数量，指请求在连接服务器、发送数据等环节发生异常，以及无响应后超时的情况。含有2XX以外的状态码，则会在测试结果中显示另一个名为“Non-2xx responses”的统计项，用于统计这部分请求数，这些请求并不算在失败的请求中。
Write errors:           0   # 写入错误数
Total transferred:      69833401 bytes # 表示所有请求的响应数据长度总和，包括每个HTTP响应数据的头信息和正文数据的长度。不包括HTTP请求数据的长度，仅仅为web服务器流向用户PC的应用层数据总长度。
HTML transferred:       69790900 bytes # 表示所有请求的响应数据中正文数据的总和，也就是减去了Total transferred中HTTP响应数据中的头信息的长度。
Requests per second:    26.90 [#/sec] (mean)   # 吞吐率29.91[#/sec](mean) ，也叫QPS或者每秒请求数, 主要指标一
Time per request:       371.720 [ms] (mean)  # 用户平均请求等待时间, 3710/100/10,  主要指标二
Time per request:       37.172 [ms] (mean, across all concurrent requests)  # 单个用户请求一次的平均时间, 3717/100,  主要指标三
Transfer rate:          18346.24 [Kbytes/sec] received # 表示网络传输速度，计算公式：Total trnasferred/ Time taken for tests，统计很好的说明服务器的处理能力达到极限时，其出口宽带的需求量。

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       11   33  14.5     32      75
Processing:   189  318  57.5    318     453
Waiting:       12   37  16.2     36     107
Total:        240  351  57.4    348     473

# 用于描述每个请求处理时间的分布情况，这个处理时间是指前面的Time per request，即对于单个用户而言，平均每个请求的处理时间。
Percentage of the requests served within a certain time (ms)
  50%    348
  66%    377
  75%    389
  80%    407
  90%    433
  95%    455
  98%    465
  99%    473
 100%    473 (longest request)

```

对于大文件的请求测试，这个值很容易成为系统瓶颈所在。要确定该值是不是瓶颈，需要了解客户端和被测服务器之间的网络情况，包括网络带宽和网卡速度等信息。

显示结果:

![](https://blog-1251258764.cos.ap-shanghai.myqcloud.com/ab1.jpg)


## 其它使用场景

```shell
#ab进行app接口的压测：
ab -n 400 -c20  "http://www.xxx.com/api.php?sig=......"；

将需要压测的接口，用  " " ;

#ab进行post传参的压测
#将 parm.txt放在和ab.exe相同的文件夹中，parm.txt中存放的是需要post格式传递的参数。-T ：post请求的head头
ab -n 400 -c20  -p  parm.txt  -T "application/x-www-form-urlencoded" http://localhost:3000/login

```
