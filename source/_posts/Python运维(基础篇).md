---
title: Python运维(基础篇)
date: 2019-04-08 21:30:04
tags: Python运维
---


## 第一部分 基础篇

### 系统基础信息模块

#### psutil
- shell `free -m | ag Mem | awk '{print $2}'`


```python
import psutil
mem = psutil.virtual_memory()
mem.total, mem.used
```




    (8229679104, 2548932608)



##### 系统信息


```python
# CPU利用率： User Time, System Time, Wait IO, Idle
psutil.cpu_times()
psutil.cpu_times().user
psutil.cpu_count(logical=False)
```




    2




```python
# 内存
psutil.virtual_memory()
psutil.swap_memory()
```




    sswap(total=8444178432, used=0, free=8444178432, percent=0.0, sin=0, sout=0)




```python
# 磁盘
psutil.disk_partitions()
psutil.disk_usage('/home')
psutil.disk_io_counters(perdisk=True)
```




    {'sda': sdiskio(read_count=78381, write_count=88833, read_bytes=2100783104, write_bytes=3724104704, read_time=37900, write_time=337344, read_merged_count=12656, write_merged_count=125377, busy_time=92668),
     'sda1': sdiskio(read_count=132, write_count=2, read_bytes=5841920, write_bytes=1024, read_time=92, write_time=0, read_merged_count=505, write_merged_count=0, busy_time=68),
     'sda2': sdiskio(read_count=78102, write_count=85876, read_bytes=2090419200, write_bytes=3724103680, read_time=37708, write_time=331428, read_merged_count=12151, write_merged_count=125377, busy_time=86976),
     'sda3': sdiskio(read_count=79, write_count=0, read_bytes=2834432, write_bytes=0, read_time=80, write_time=0, read_merged_count=0, write_merged_count=0, busy_time=72),
     'sdb': sdiskio(read_count=19804, write_count=10235, read_bytes=2424295424, write_bytes=2018324480, read_time=113548, write_time=9251732, read_merged_count=206, write_merged_count=482537, busy_time=135780),
     'sdb1': sdiskio(read_count=19717, write_count=10218, read_bytes=2422161408, write_bytes=2018324480, read_time=113024, write_time=9249532, read_merged_count=206, write_merged_count=482537, busy_time=133096),
     'sdc': sdiskio(read_count=803, write_count=0, read_bytes=26894336, write_bytes=0, read_time=4760, write_time=0, read_merged_count=0, write_merged_count=0, busy_time=2952),
     'sdc1': sdiskio(read_count=715, write_count=0, read_bytes=24723456, write_bytes=0, read_time=4104, write_time=0, read_merged_count=0, write_merged_count=0, busy_time=2376)}




```python
# 网络
psutil.net_io_counters(pernic=True)
```




    {'enp3s0': snetio(bytes_sent=0, bytes_recv=0, packets_sent=0, packets_recv=0, errin=0, errout=0, dropin=0, dropout=0),
     'lo': snetio(bytes_sent=51166695, bytes_recv=51166695, packets_sent=8569, packets_recv=8569, errin=0, errout=0, dropin=0, dropout=0),
     'wlp4s0': snetio(bytes_sent=130920677, bytes_recv=1928265793, packets_sent=1100800, packets_recv=1367419, errin=0, errout=0, dropin=0, dropout=0)}




```python
import datetime

# 其他
psutil.users()
datetime.datetime.fromtimestamp(psutil.boot_time()).strftime("%Y-%m-%d %H:%M:%S")
```




    '2019-04-05 10:18:53'



##### 进程管理


```python
psutil.pids()
p = psutil.Process(2271)
p.name()
p.exe()
p.cwd()
p.status()
datetime.datetime.fromtimestamp(p.create_time()).strftime("%Y-%m-%d %H:%M:%S")
p.uids()
p.gids()
p.cpu_times()
p.cpu_affinity() # 亲和度
p.memory_percent()
p.memory_info()
p.io_counters()
p.connections() # 返回socket namedtuples
p.num_threads()

# popen
from subprocess import PIPE
pop = psutil.Popen(["/usr/bin/python", "-c", "print('hallo')"], stdout=PIPE)
pop.name()
pop.username()
pop.communicate()
```




    (b'hallo\n', None)



#### IPy

##### IP地址、网段


```python
from IPy import IP

IP('10.0.0.0/8').version()
IP('::1').version()

ips = IP('192.168.0.0/16')
ips.len()

ip = IP('192.168.1.102')
ip.reverseNames()
ip.iptype()
ip.int()
ip.strHex()
ip.strBin()

IP('192.168.1.102').make_net('255.255.255.0')
IP('192.168.1.102/255.255.255.0', make_net=True)
IP('192.168.1.0-192.168.1.255', make_net=True)

# wantprefixlen
IP('192.168.1.0/24').strNormal(0)  # 无返回 '192.168.1.0'
IP('192.168.1.0/24').strNormal(1)  # prefix '192.168.1.0/24'
IP('192.168.1.0/24').strNormal(2)  # decimalnetmask '192.168.1.0/255.255.255.0'
IP('192.168.1.0/24').strNormal(3)  # lastIP '192.168.1.0-192.168.1.255'
```




    '192.168.1.0-192.168.1.255'



##### 多网络计算


```python
IP('10.0.0.0/24') < IP('12.0.0.0/24')
'192.168.1.102' in IP('192.168.1.0/24')
IP('192.168.1.0/24') in IP('192.168.0.0/16')

IP('192.168.0.0/23').overlaps('192.168.1.0/24')  # 1  存在重叠
IP('192.168.1.0/24').overlaps('192.168.2.0')  # 0  不存在重叠
```




    0



#### dnspython
- nslookup
- dig


```python
import dns.resolver

domain = 'google.com'

A = dns.resolver.query(domain, 'A')  # A  主机名转换为IP地址
for i in A.response.answer:
    for j in i.items:
        print('A: ', j)
print('--------------------------------')
        
MX = dns.resolver.query(domain, 'MX')  # MX  定义邮件服务器的域名
for i in MX:
    print('MX: ', i.preference, i.exchange)
print('--------------------------------')

NS = dns.resolver.query(domain, 'NS')  # NS  域名服务器及授权子域, 只限一级域名， 如： baidu.com
for i in NS.response.answer:
    for j in i.items:
        print('NS: ', j.to_text())
print('--------------------------------')

CNAME = dns.resolver.query('www.baidu.com', 'CNAME')  # CNAME  别名记录， 实现域名间的映射
for i in CNAME.response.answer:
    for j in i.items:
        print('CNAME: ', j.to_text())
```

    A:  172.217.161.174
    --------------------------------
    MX:  10 aspmx.l.google.com.
    MX:  40 alt3.aspmx.l.google.com.
    MX:  50 alt4.aspmx.l.google.com.
    MX:  20 alt1.aspmx.l.google.com.
    MX:  30 alt2.aspmx.l.google.com.
    --------------------------------
    NS:  ns2.google.com.
    NS:  ns1.google.com.
    NS:  ns3.google.com.
    NS:  ns4.google.com.
    --------------------------------
    CNAME:  www.a.shifen.com.


### 业务服务监控

#### 文件内容差异对比
```
d = difflib.Differ()  # 比较字符串
           .SequenceMatcher()  # 比较序列
d.compare(text1, text2)

d = difflib.HtmlDiff()  # 比较结果输出
d.make_file(text1, text2)
```

#### 目录差异对比
- cmp(f1, f2, shallow=True) 单文件, shallow=True忽略文件内容对比
- cmpfiles 多文件
  ```
  filecmp.cmpfiles(dir1, dir2, [指定文件])
  return (匹配， 不匹配， 错误)
  ```
- dircmp 目录
  ```
  filecmp.dircmp(dir1, dir2, ['test.py'])  # 忽略test.py
  ```
  - 方法
    - report()  指定目录
    - report_partial_closure()  当前及第一级子目录
    - report_full_closure()  递归所有
  - 属性
    - left
    - right
    - left_list 左目录中的文件及目录列表
    - right_list
    - common 共同存在的文件或目录
    - left_only
    - right_only
    - common_dirs
    - common_files
    - common_funny _两边都存在但不同类型或os.stat()记录错误的子目录_
    - same_files
    - diff_files
    - funny_files 两边都存在但无法比较的文件
    - subdirs `common_dirs`映射到新的字典类型的dircmp对象

#### smtplib
- smtplib.SMTP方法
  - connect("smtp.163.com", "25")
  - login(user, pass)
  - sendmail(from, to, msg)
  - starttls()
  - quit()
- MIME
  - email.mime.multipart.MIMEMultipart
  - email.mime.audio.MIMEAudio
  - email.mime.image.MIMEImage
  - email.mime.text.MIMEText

#### Web服务质量
- pycurl.Curl()
  - close()
  - perform() 提交请求
  - setopt(opt, value)
    - .CONNECTTIMEOUT  连接等待时间
    - .TIMEOUT
    - .NOPROGRESS  进度条，非0则屏蔽
    - .MAXREDIRS  最大重定向数
    - .FORBID_REUSE
    - .FRESH_CONNECT  强制获取新的连接
    - .DNS_CACHE_TIMEOUT  保存DNS信息的时间， 默认120秒
    - .URL  指定请求的URL
    - .USERAGENT
    - .HEADERFUNCTION  返回的header定向到回调函数
    - .WRITEFUNCTION  返回的body定向到回调函数
    - .WRITEHEADER  返回的header定向到对象
    - .WRITEDATA  返回的html内容定向到对象
  - getinfo(opt)
    - .HTTP_CODE
    - .TOTAL_TIME
    - .NAMELOOKUP_TIME  DNS解析时间
    - .CONNECT_TIME
    - .PRETRANSFER_TIME
    - .STARTTRANSFER_TIME
    - .REDIRECT_TIME
    - .SIZE_UPLOAD
    - .SIZE_DOWNLOAD
    - .SPEED_UPLOAD  平均上传速度
    - .SPEED_DOWNLOAD
    - .HEADER_SIZE

### 定制业务质量报表

#### Excel
```
workbook = xlsxwriter.Workbook('test.xlsx')
worksheet = workbook.add_worksheet()
worksheet.set_column('A:A', 20)  # A列宽度20

bold = workbook.add_format({'bold': True})

worsheet.write('A1', 'hallo', bold)
worksheet.write(4, 0, '=SUM(A3:A4)', {'strings_to_numbers': True})

worksheet.insert_image('B2', 'some-image-path')
workbook.close()
```

#### rrdtool
- round robin  处理定量数据及当前元素指针
- installation
  ```
  sudo apt-get install librrd-dev libpython3-dev
  sudo pip3 install -i https://mirrors.aliyun.com/pypi/simple/ rrdtool
  ```
- create
- update
- graph
- fetch


```python
# %load /home/phil/rrdtool/create.py
#!/uar/bin/python3.5

import time
import rrdtool

# create
cur_time = str(int(time.time()))
rrd = rrdtool.create('Flow.rrd',
                     '--step', '300',  # 写频率
                     '--start', cur_time,
                     # COUNTER 递增， 600心跳值（若600没有收到值，UNKNOWN代替），最大值U表示不确定
                     'DS:eth0_in:COUNTER:600:0:U',
                     'DS:eth0_out:COUNTER:600:0:U',
                     # [RRA:[AVERAGE, MAX, MIN]:xff:steps:rows]
                     # xff定义为0.5表示一个CDP中的PDP如超一半为UNKNOWM，则该CDP标为UNKNOWM
                     'RRA:AVERAGE:0.5:1:600',
                     'RRA:AVERAGE:0.5:6:700',
                     'RRA:AVERAGE:0.5:24:775',
                     'RRA:AVERAGE:0.5:288:797',
                     'RRA:MAX:0.5:1:600',
                     'RRA:MAX:0.5:6:700',
                     'RRA:MAX:0.5:24:775',
                     'RRA:MAX:0.5:444:797',
                     'RRA:MIN:0.5:1:600',
                     'RRA:MIN:0.5:6:700',
                     'RRA:MIN:0.5:24:775',
                     'RRA:MIN:0.5:444:797')
if rrd:
    print(rrdtool.error())
```


```python
# %load /home/phil/rrdtool/update.py
#!/uar/bin/python3.5

import time
import rrdtool
import psutil

# update
total_input_traffic = psutil.net_io_counters()[1]
total_output_traffic = psutil.net_io_counters()[0]
starttime = int(time.time())
update = rrdtool.updatev('Flow.rrd', '%s:%s:%s' %
                        (str(starttime),
                        str(total_input_traffic),
                        str(total_output_traffic)))
print(update)  # 返回{'return_value': 0L} 则更新成功
```

`5  *    * * *   phil    /usr/bin/python3.5 /home/phil/rrdtool/update.py > /dev/null 2>&1`


```python
# %load /home/phil/rrdtool/graph.py
#!/uar/bin/python3.5

import time
import rrdtool

title = "Server network traffic flow (" + \
        time.strftime('%Y-%m-%d', time.localtime(time.time())) + ")"
rrdtool.graph("Flow.png",
              "--start", "-1d",
              "--vertical-label=Bytes/s",
              "--x-grid", "MINUTE:12:HOUR:1:HOUR:1:0:%H",
              # MINUTE:12 每12分钟一根次要格线
              # HOUR:1 每1小时一根主要格线
              # HOUR:1 1小时输出一个label
              # 0:%H 0表示数字对齐格线， %H表示标签以小时显示
              "--width", "650",
              "--height", "230",
              "--title", title,
              "DEF:inoctets=Flow.rrd:eth0_in:AVERAGE",  # 数据源DS, CF
              "DEF:outoctets=Flow.rrd:eth0_out:AVERAGE",
              "CDEF:total=inoctets,outoctets,+",
              "LINE1:total#FF8833:Total traffic",  # 线条方式
              "AREA:inoctets#00FF00:In traffic",  # 面积方式
              "LINE1:outoctets#0000FF:Out traffic",
              "HRULE:6144#FF0000:Alarm value\\r",  # 水平线， 阀值6.1k
              "CDEF:inbits=inoctets,8,*",  # *8， 换算成bit
              "CDEF:outbits=outoctets,8,*",
              "COMMENT:\\r",  # 换行符
              "COMMENT:\\r",
              "GPRINT:inbits:AVERAGE:Avg In traffic\: %6.2lf %Sbps",
              "COMMENT:  ",
              "GPRINT:inbits:MAX:Max In traffic\: %6.2lf %Sbps",
              "COMMENT:  ",
              "GPRINT:inbits:MIN:Min In traffic\: %6.2lf %Sbps\\r",
              "COMMENT:  ",
              "GPRINT:inbits:AVERAGE:Avg In traffic\: %6.2lf %Sbps",
              "COMMENT:  ",
              "GPRINT:inbits:MAX:Max In traffic\: %6.2lf %Sbps",
              "COMMENT:  ",
              "GPRINT:inbits:MIN:Min In traffic\: %6.2lf %Sbps\\r"
)
```

#### scapy

- installation
  ```
  sudo apt-get install tcpdump
                       graphviz
                       imagemagick
                       python-matplotlib
                       python-cryptography
                       python-pyx
  sudo pip3 install -i https://mirrors.aliyun.com/pypi/simple cryptography scapy
  ```
- usage
  ```
  import os
  import sys
  import time
  import subprocess
  import warnings
  import logging
  from scapy.all import traceroute
  
  warnings.filterwarnings("ignore", category=DeprecationWarning)  # 屏蔽无用警告信息
  logging.getLogger("scsapy.runtime").setLevel(logging.ERROR)  # 屏蔽模块IPv6多余警告
  
  domain = 'www.baidu.com'
  dport = [80]
  
  res, unans = traceroute(domain, dport=dport, retry=-2)
  res.graph(target="> test.svg")
  time.sleep(1)
  subprocess.Popen("/usr/bin/convert test.svg test.png", shell=True)
  ```
  > todo: 
          PermissionError: [Errno 1] Operation not permitted
          No module named 'traceroute'

### 系统安全

#### ClamAV
- pyClamd
  - ClamdNetworkSocket()
    - __init__()
    - contscan_file()
    - multiscan_file()
    - scan_file()  # 发现病毒将终止
    - shutdown()
    - stats()
    - reload()
    - EICAR()  # 生成具有病毒特征的字符串
  - ClamdUnixSocket()

#### nmap
- PortScanner()
  - scan()
  - command_line()  # 返回nmap命令行
  - scaninfo()
  - all_hosts()
- PortScannerHostDict()
  - hostname()
  - state()
  - all_protocals()
  - all_tcp()
  - tcp()


