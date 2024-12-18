---
title: Python读取服务器状态
date: 2024-11-18 09:53:11
tags:
---

# Python 读取服务器状态

## 连接服务器

### 1. paramiko -通过SSH连接服务器

直接使用[SSH协议](https://so.csdn.net/so/search?q=SSH协议&spm=1001.2101.3001.7020)对远程服务器执行操作

#### 连接服务器

```python
import paramiko
from paramiko import SSHClient, AutoAddPolicy, RSAKey

def ssh_login_with_password(ip, username, password):
    try:
        client = SSHClient()
        client.set_missing_host_key_policy(AutoAddPolicy())
        client.connect(ip, username=username, password=password, timeout=5)
        client.close()
        return True
    except Exception as e:
        return False

def ssh_login_with_key(ip, username, key_path):
    try:
        client = SSHClient()
        client.set_missing_host_key_policy(AutoAddPolicy())
        key = RSAKey.from_private_key_file(key_path)
        client.connect(ip, username=username, pkey=key, timeout=5)
        client.close()
        return True
    except Exception as e:
        return False
```

#### 读取状态信息

```python
# 执行命令
stdin, stdout, stderr = client.exec_command('vmstat')

# 读取输出
output = stdout.read().decode('utf-8')
```



#### 处理状态信息

```python
# 处理状态信息
lines = output.strip().split('\n')
headers = lines[0].split()
data = lines[2].split()
status = dict(zip(headers, data))
```



#### 显示状态信息

```python
# 显示状态信息
for key, value in status.items():
    print(f'{key}: {value}')
```

## 获取进程和系统运行状态

### 1.psutil

#### 获取以下信息

- 进程信息：包括进程ID、进程名称、进程状态等。
- CPU信息：包括CPU数量、CPU使用率、每个CPU的使用率等。
- 内存信息：包括总内存、已使用内存、空闲内存等。
- 磁盘信息：包括磁盘使用率、磁盘IO等。
- 网络信息：包括网络接口、网络连接数等。
- 传感器信息：包括温度、湿度等信息。

#### 使用方式

- 获取进程信息：可以使用 psutil.Process(pid) 方法获取指定进程的信息，其中 pid 是进程ID。
- 获取系统信息：可以使用 psutil.virtual_memory() 和 psutil.cpu_percent(interval=1) 等方法获取系统信息。
- 监控磁盘信息：可以使用 psutil.disk_usage(‘/’) 方法获取磁盘使用率信息。
- 监控网络信息：可以使用 psutil.net_io_counters() 方法获取网络IO信息。
- 管理进程：可以使用 psutil.Process(pid).kill() 方法杀死指定进程

##### 查看操作系统相关信息

```python
import psutil
from datetime import datetime

Users = psutil.users()  # 查看当前登录的用户信息，列表类型。包括：name: 用户名  terminal: 终端  host: 主机地址  started: 登录时间  pid: 进程id
BootTime = datetime.fromtimestamp(psutil.boot_time())  # 查看系统的启动时间 2022-12-12 08:40:05

```



##### 查看CPU相关信息

```python
import psutil

CpuLogicCoreCount = psutil.cpu_count()  # CPU逻辑核心数 '8'
CpuPhysicalCoreCount = psutil.cpu_count(logical=False)  # CPU物理核心数 '4'
CPUPercent = psutil.cpu_percent(interval=1, percpu=True)  # CPU使用率，列表显示，interval表示刷新间隔，列表中的元素代表每个CPU逻辑核心 [6.1, 1.5, 4.6, 7.7, 10.8, 1.5, 1.5, 1.5]
CPUFreq = psutil.cpu_freq()  # CPU频率 scpufreq(current=3408.0, min=0.0, max=3408.0)

```



##### 查看内存相关信息

```python
import psutil
VirtualMemory = psutil.virtual_memory()  # 内存使用情况 svmem(total=8480059392, available=1381511168, percent=83.7, used=7098548224, free=1381511168) total: 总内存 available: 可用内存

```



##### 查看硬盘存储和IO相关信息

```python
import psutil
DiskPartitions = psutil.disk_partitions(all=True)
"""
[sdiskpart(device='C:\\', mountpoint='C:\\', fstype='NTFS', opts='rw,fixed'),
 sdiskpart(device='D:\\', mountpoint='D:\\', fstype='NTFS', opts='rw,fixed'),
 sdiskpart(device='E:\\', mountpoint='E:\\', fstype='NTFS', opts='rw,fixed')]
# 可以看到一共有三个盘符，fstype表示文件系统格式是NTFS，opts中的rw表示可读写
# 里面有一个参数 all, 默认为 False, 如果指定为 True, 那么返回的内容还会包含 /proc 等特殊文件系统的挂载信息，由于我这里是 Windows, 所以两者没区别
"""
DiskUsage = psutil.disk_usage('C:')  # 磁盘使用情况 sdiskusage(total=84821807104, used=66361847808, free=18459959296, percent=78.2)
DiskIOCounters = psutil.disk_io_counters(perdisk=False)  # 磁盘IO统计信息 sdiskio(read_count=4612339, write_count=1834298, read_bytes=116422007808, write_bytes=61427764224, read_time=12831, write_time=4321)
"""
read_count: 读次数                     write_count: 写次数
read_bytes: 读的字节数                 write_bytes: 写的字节数
read_time: 读时间                      write_time: 写时间
默认返回的是所有磁盘加起来的统计信息，我们可以指定 perdisk=True，则分别列出每一个磁盘的统计信息。
"""

```



##### 查看网络相关信息

```py
import psutil
NetIOCounters = psutil.net_io_counters(pernic=False)  # 网络 IO 统计信息 snetio(bytes_sent=873520263, bytes_recv=311085337, packets_sent=1091582, packets_recv=1259384, errin=0, errout=0, dropin=0, dropout=0)
"""
# bytes_sent: 发送的字节数
# bytes_recv: 接收的字节数
# packets_sent: 发送的包数据量
# packets_recv: 接收的包数据量
# errin: 接收包时, 出错的次数
# errout: 发送包时, 出错的次数
# dropin: 接收包时, 丢弃的次数
# dropout: 发送包时, 丢弃的次数
里面还有一个 pernic 参数, 如果为 True, 则列出所有网卡的信息
"""
NetIfAddrs = psutil.net_if_addrs()  # 以字典的形式返回网卡的配置信息, 包括 IP 地址、Mac地址、子网掩码、广播地址等等
"""
{'Loopback Pseudo-Interface 1': [snicaddr(family=<AddressFamily.AF_INET: 2>, address='127.0.0.1', netmask='255.0.0.0', broadcast=None, ptp=None),
                         snicaddr(family=<AddressFamily.AF_INET6: 23>, address='::1', netmask=None, broadcast=None, ptp=None)],
 '本地连接': [snicaddr(family=<AddressFamily.AF_LINK: -1>, address='10-E7-C6-2D-AB-2C', netmask=None, broadcast=None, ptp=None),
  snicaddr(family=<AddressFamily.AF_INET: 2>, address='10.36.23.50', netmask='255.255.255.0', broadcast=None, ptp=None),
  snicaddr(family=<AddressFamily.AF_INET6: 23>, address='fe80::d472:4d86:72f6:4da1', netmask=None, broadcast=None, ptp=None)]}
"""
NetIfStats = psutil.net_if_stats()  # 返回网卡的详细信息, 包括是否启动、通信类型、传输速度、mtu
"""
{'Loopback Pseudo-Interface 1': snicstats(isup=True, duplex=<NicDuplex.NIC_DUPLEX_FULL: 2>, speed=1073, mtu=1500),
 '本地连接': snicstats(isup=True, duplex=<NicDuplex.NIC_DUPLEX_FULL: 2>, speed=100, mtu=1500)}
"""
NetConnections = psutil.net_connections(kind='inet')  # 以列表的形式返回每个网络连接的详细信息
"""
[sconn(fd=-1, family=<AddressFamily.AF_INET: 2>, type=1, laddr=addr(ip='127.0.0.1', port=57096), raddr=addr(ip='127.0.0.1', port=57095), status='ESTABLISHED', pid=7456),
 sconn(fd=-1, family=<AddressFamily.AF_INET: 2>, type=1, laddr=addr(ip='127.0.0.1', port=57230), raddr=addr(ip='127.0.0.1', port=57229), status='ESTABLISHED', pid=20992),
 sconn(fd=-1, family=<AddressFamily.AF_INET: 2>, type=1, laddr=addr(ip='127.0.0.1', port=57184), raddr=addr(ip='127.0.0.1', port=57251), status='ESTABLISHED', pid=20992)]
...
里面接受一个参数, 默认是 "inet", 当然我们也可以指定为其它, 比如 "tcp"
"""
```

#### 2.cpuinfo

##### 专门用于获取 CPU 的详细信息,如型号、架构、频率等
