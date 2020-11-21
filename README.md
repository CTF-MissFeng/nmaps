# nmaps

> Nmaps是采用Golang语言编写的新一代端口及指纹扫描器，利用Golang语言特性使其扫描更快、跨平台等特点，内置并精简了nmap指纹库，以此摆脱调用nmap进行指纹识别。

## 初衷

> nmaps是后期开发信息搜集综合平台的子模块（不需要识别所有端口指纹，但常见服务能识别出来），在以往进行端口探测时，往往是调用masscan/nmap进行探测，对于大批量资产探测会导致识别不精确、速度慢、调用不方便等缺点，或进行内网横向探测时能单文件跨平台使用。

## 原理

- 端口存活探测：非root权限使用CONNECT方式扫描、root权限使用SYN方式扫描，对于批量资产探测，借鉴了masscan核心扫描算法及naabu开源项目

  > Masscan first stores the targets as a "list of ranges". IP address
  > ranges are stored in one structure, and port ranges are stored
  > in another structure.
  > Then, a single index variable is used to enumerate the set of all 
  > IP:port combinations. The scan works by simply incrementing the 
  >
  >  index variable from 0 to the total number of probes (the 'range').
  >  Then, before the enumeration step, the index is permuted into another
  >  random index within the same range, in a 1-to-1 mapping. In other
  >  words, the algorithm is theoretically reversable: given the output
  >  of the permutation function, we can obtain the original index.

- 指纹识别：使用nmap开源的指纹库，并去除了UDP及探测等级为8以上的部分不常用服务规则，对于单个端口的指纹识别，先并发探测等级优先的规则库，若未匹配到，则在并发探测下一等级规则库（由于是并发探测，故小部分服务会有多个匹配结果，导致匹配结果不精确），若还是未匹配到相应指纹，则使用常规端口对应的服务进行判断（可能会有误报情况）

## 命令行说明

```
Usage of ./nmap:
  -c int
        扫描并发工作协程数(切勿太大,造成网络波动) (default 25)
  -debug
        输出debug调试信息
  -exclude-file string
        指定包含要从扫描中排除的目标,以换行符分隔的文件 (ip, cidr)
  -exclude-hosts string
        指定要从扫描中排除的目标以逗号分隔列表 (ip, cidr)
  -exclude-ports string
        要从枚举中排除的端口
  -host string
        指定扫描的主机地址
  -iL string
        指定包含待扫描的主机文本文件
  -interface string
        指定用于端口扫描的网络接口
  -interface-list
        列出可用接口和公有IP
  -json
        以json格式保存
  -nC
        在输出中不使用彩色输出
  -nmaptimeout int
        端口指纹识别socket连接超时时间 (default 7)
  -o string
        指定保存扫描结果
  -p string
        指定待扫描的端口范围 (80, 80,443, 100-200, (-p - 将全端口扫描)
  -ping
        扫描前使用ping探针验证主机存活
  -ports-file string
        指定包含要枚举的端口文件
  -rate int
        端口扫描探测请求的速率 (default 1000)
  -retries int
        端口扫描探测的重试次数 (default 1)
  -silent
        仅在输出中显示找到的端口
  -source-ip string
        指定在TCP数据包中使用的SourceIP
  -timeout int
        扫描超时前等待的毫秒数 (default 700)
  -top-ports string
        指定扫描的常规端口 (top 100/1000
  -v    显示详细输出
  -verify
        扫描出端口后使用TCP连接二次验证
  -version
        显示版本信息
  -warm-up-time int
        扫描阶段之间的间隔时间(秒) (default 2)
```

## 演示

> 示例IP为fofa随机查询，不针对特定IP，若造成了困恼请联系我进行删除

> 非root权限使用CONNECT方式扫描，适用于Macos、Windows，但推荐使用Linux服务器进行批量扫描

```
./nmap -host 42.192.0.159 -top-ports 1000                                                                                                                                   (miniconda3)    100%    5.60G 

 _    _ __  __          _____   _____ 
|  \ | |  \/  |   /\   |  __ \ / ____|
|   \| | \  / |  /  \  | |__) | (___  
|      | |\/| | / /\ \ |  ___/ \___ \
| |\   | |  | |/ ____ \| |     ____) |
|_| \_ |_|  |_/_/    \_\_|    |_____/   V1.0


[INF] 当前非root权限运行CONNECT扫描(速度比SYN探测方式慢)
[INF] nmap指纹库加载成功，共计[42]个探针,[10652]条正则匹配
[INF] 主机[42.192.0.159]探测到[6]个存活端口 [3389,49153,49155,49152,49154,49156,]
[INF] 开始进行端口指纹识别，请稍后
[INF] 42.192.0.159:3389  [ssl]   
[INF] 42.192.0.159:49153  [msrpc] Microsoft Windows RPC  Windows
[INF] 42.192.0.159:49152  [msrpc] Microsoft Windows RPC  Windows
[INF] 42.192.0.159:49154  [msrpc] Microsoft Windows RPC  Windows
[INF] 42.192.0.159:49155  [msrpc] Microsoft Windows RPC  Windows
[INF] 42.192.0.159:49156  [msrpc] Microsoft Windows RPC  Windows
```



## 速度及准确度比较

### 1、nmap-126秒

> nmap -p- -Pn -sV 61.186.243.130

```
Nmap scan report for 61.186.243.130
Host is up (0.042s latency).
Not shown: 65506 closed ports
PORT      STATE    SERVICE            VERSION
23/tcp    filtered telnet
25/tcp    filtered smtp
80/tcp    filtered http
137/tcp   filtered netbios-ns
350/tcp   open     matip-type-a?
352/tcp   open     http               Microsoft IIS httpd 6.0
443/tcp   filtered https
445/tcp   filtered microsoft-ds
1433/tcp  open     ms-sql-s           Microsoft SQL Server 2008 R2 10.50.1600; RTM
1723/tcp  filtered pptp
3389/tcp  open     ssl/ms-wbt-server?
4444/tcp  filtered krb524
5554/tcp  filtered sgi-esphttp
8033/tcp  open     http               Apache Tomcat/Coyote JSP engine 1.1
8080/tcp  filtered http-proxy
8081/tcp  filtered blackice-icecap
8082/tcp  filtered blackice-alerts
8085/tcp  filtered unknown
8086/tcp  open     http               Microsoft IIS httpd 10.0
8088/tcp  filtered radan-http
8090/tcp  open     opsmessaging?
8099/tcp  open     http               Microsoft IIS httpd 7.5
8181/tcp  filtered intermapper
9090/tcp  open     http               Microsoft IIS httpd 10.0
9100/tcp  open     jetdirect?
9101/tcp  open     jetdirect?
10084/tcp filtered unknown
20002/tcp open     tcpwrapped
33400/tcp filtered unknown
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 126.03 seconds
```

### 2、nmaps-81秒

> ./nmaps -p - -c 100 -host 61.186.243.130
>
> 精简了http服务识别，因为后期信息搜集平台会针对http服务单独进行全面识别

```
 _    _ __  __          _____   _____ 
|  \ | |  \/  |   /\   |  __ \ / ____|
|   \| | \  / |  /  \  | |__) | (___  
|      | |\/| | / /\ \ |  ___/ \___ \
| |\   | |  | |/ ____ \| |     ____) |
|_| \_ |_|  |_/_/    \_\_|    |_____/   V1.0


[INF] 当前root权限运行TCP/ICMP/SYN扫描
[INF] nmap指纹库加载成功，共计[42]个探针,[10652]条正则,[8282]条TCP端口指纹
[INF] 主机[61.186.243.130]探测到[12]个存活端口 [8033,1433,8086,20002,9090,350,8099,8090,3389,352,9100,9101,]
[INF] 开始进行端口指纹识别，请稍后
[INF] 61.186.243.130:20002  [commtact-http]   
[INF] 61.186.243.130:9090  [http] Microsoft IIS httpd 10.0 Windows
[INF] 61.186.243.130:3389  [ssl]   
[INF] 61.186.243.130:8033  [http]  1.1 
[INF] 61.186.243.130:8086  [http] Microsoft IIS httpd 10.0 Windows
[INF] 61.186.243.130:8090  [opsmessaging]   
[INF] 61.186.243.130:9100  [jetdirect]   
[INF] 61.186.243.130:8099  [unknown]   
[INF] 61.186.243.130:9101  [jetdirect]   
[INF] 61.186.243.130:350  [matip-type-a]   
[INF] 61.186.243.130:1433  [ms-sql-s] Microsoft SQL Server 2008 R2 10.50.1600; RTM Windows
[INF] 61.186.243.130:352  [dtag-ste-sb]
```

