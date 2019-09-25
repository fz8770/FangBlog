# Linux 运维工程师经典面试题合集

1. 如何查看 HTTP 的并发请求数与其 TCP 连接状态？
   
```shell
etstat -n | awk '/^tcp/ {++b[$NF]} END {for(a in b) print a,b[a]}'
```

![](https://raw.githubusercontent.com/fz8770/FangBlog/master/source/%E8%BF%90%E7%BB%B4%E4%BA%BA%E7%94%9F/Linux%20%E8%BF%90%E7%BB%B4%E5%B7%A5%E7%A8%8B%E5%B8%88%E7%BB%8F%E5%85%B8%E9%9D%A2%E8%AF%95%E9%A2%98%E5%90%88%E9%9B%86.imgs/1.png)

Linux 服务器打开文件数也是影响并发的重要一环，具体可以查看该文件配置：/etc/security/limits.conf

当然同级目录下面的 limits.d 目录下的的配置文件也需要关注，也可以使用 ulimit -n 查看当前的配置数量。


2. 查看每个 IP 地址的连接数

```shell
netstat -n | awk '/^tcp/ {print $5}' | awk -F: '{print $1}' | sort | uniq -c | sort -rn
```

![](https://raw.githubusercontent.com/fz8770/FangBlog/master/source/%E8%BF%90%E7%BB%B4%E4%BA%BA%E7%94%9F/Linux%20%E8%BF%90%E7%BB%B4%E5%B7%A5%E7%A8%8B%E5%B8%88%E7%BB%8F%E5%85%B8%E9%9D%A2%E8%AF%95%E9%A2%98%E5%90%88%E9%9B%86.imgs/2.png)


3. 