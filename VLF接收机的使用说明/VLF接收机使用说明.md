# VLF接收机使用说明

## 1 准备工作

### 1.1 天线以及电源连接

接收机外壳上的品字插座需要接到**12V**电源上，注意是**12V**，不是**220V**

将两路磁天线和北斗天线接至机箱相应的SMA接口，机箱USB口接外置的硬盘柜，注意硬盘盒也需要**12V**供电

### 1.2 网络连接

**确保设备与电脑处于同一网段**，

可以是设备接到路由器上，电脑通过有线或者无线的方式连接至该网络，这时的设备是联网了的（可以访问外网，时间等信息也会自动进行同步）

也可以选择将设备直接接到电脑的网口，此时设备没有与外部的网络连接，但是可以与本地电脑进行网络传输（不推荐）

确保硬件连接好后，可以在本地主机上ping一下pynq，如果能ping通，说明网络连接没有问题，PYNQ正常启动

``` shell
PS C:\Users\43601> ping pynq

正在 Ping pynq.local [240e:36f:9c7:8931:fce3:e0ff:fe1d:fbf] 具有 32 字节的数据:
来自 240e:36f:9c7:8931:fce3:e0ff:fe1d:fbf 的回复: 时间=1ms
来自 240e:36f:9c7:8931:fce3:e0ff:fe1d:fbf 的回复: 时间=2ms
来自 240e:36f:9c7:8931:fce3:e0ff:fe1d:fbf 的回复: 时间=2ms
来自 240e:36f:9c7:8931:fce3:e0ff:fe1d:fbf 的回复: 时间=2ms

240e:36f:9c7:8931:fce3:e0ff:fe1d:fbf 的 Ping 统计信息:
    数据包: 已发送 = 4，已接收 = 4，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 1ms，最长 = 2ms，平均 = 1ms
```

**P.S. 更推荐将设备连到支持DHCP的路由器上使用，与电脑直接连受电脑网络环境的影响，可能不稳定**



## 2 通过SSH连接

打开**Putty**，使用**SSH**进行连接，**Host Name**填写**pynq**，如图所示：

<img src="typora_img/image-20241107131838549.png" alt="image-20241107131838549" style="zoom:50%;" /><img src="typora_img/image-20241107132415633.png" alt="image-20241107132415633" style="zoom:60%;" />

可以通过**Save**将设置保存下来，以后只需要双击**Default Settings**里的**pynq**就行

点击**Open**，会出现设备的命令行窗口，需要登录使用，用户名和密码均为` xilinx `，这里输入密码的时候不会显示，输入完成后回车即可

这样就登入了接收机的系统，之后可以使用命令行工具来对系统进行操作

此时可以用ifconfig查看设备当前的ip地址

``` bash
xilinx@pynq:~/pynq_dma$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.3.137  netmask 255.255.255.0  broadcast 192.168.3.255
        inet6 fe80::fce3:e0ff:fe1d:fbf  prefixlen 64  scopeid 0x20<link>
        inet6 240e:36f:9c7:8931:fce3:e0ff:fe1d:fbf  prefixlen 64  scopeid 0x0<global>
        ether fe:e3:e0:1d:0f:bf  txqueuelen 1000  (Ethernet)
        RX packets 478264  bytes 50401738 (50.4 MB)
        RX errors 472  dropped 0  overruns 231  frame 241
        TX packets 1068519  bytes 1075646665 (1.0 GB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 29  base 0xb000

eth0:1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.2.99  netmask 255.255.255.0  broadcast 192.168.2.255
        ether fe:e3:e0:1d:0f:bf  txqueuelen 1000  (Ethernet)
        device interrupt 29  base 0xb000

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 228  bytes 24673 (24.6 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 228  bytes 24673 (24.6 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

比如这里设备的IPV4地址就是192.168.3.137，也可以看看本地主机的ip地址，在终端中输入：

``` shell
PS C:\Users\43601> ipconfig.exe

Windows IP 配置

无线局域网适配器 WLAN:

   连接特定的 DNS 后缀 . . . . . . . :
   IPv6 地址 . . . . . . . . . . . . : 240e:36f:9c7:8931:bb:1c43:1d22:3
   IPv6 地址 . . . . . . . . . . . . : 240e:36f:9c7:8931:18e2:d8c1:c54b:ed25
   临时 IPv6 地址. . . . . . . . . . : 240e:36f:9c7:8931:44f6:1ab5:6cb6:65e1
   本地链接 IPv6 地址. . . . . . . . : fe80::f79f:d4e6:147e:9b30%3
   IPv4 地址 . . . . . . . . . . . . : 192.168.3.68
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
   默认网关. . . . . . . . . . . . . : fe80::2bb:1cff:fe43:1d22%3
                                       192.168.3.1
```



## 3 通过Samba实现文件共享

设备上运行了Samba文件共享服务，允许从网络访问PYNQ的主区域，便于和设备之间的文件传输

如图所示，在Windows的文件资源管理器顶部的地址栏中输入Host Name：

<img src="typora_img/image-20241110153810691.png" alt="image-20241110153810691" style="zoom:80%;" />

用户名和密码都是xilinx，可以选择`记住我的凭据`，以后就不用每次登入都输入用户名和密码

<img src="typora_img/image-20241107140658926.png" alt="image-20241107140658926" style="zoom:40%;" /><img src="typora_img/image-20241107141020279.png" alt="image-20241107141020279" style="zoom:60%;" />

点击确定，现在就可以直接访问PYNQ的主文件夹了，可以像访问本机的文件夹一样，对远程的文件进行查看、拷贝等操作

例如数据存放在`disk`文件夹中，可以直接从中对文件进行复制



## 4 数据存储

### 4.1 上位机数据保存

双击打开上位机软件

<img src="typora_img/abc7400e8e414493897097250766ff5b.png" alt="abc7400e8e414493897097250766ff5b" style="zoom:50%;" />

**界面上方 **可以选择数据的存储路径，默认存储路径为当前文件夹下

**右侧** 是对应设备的IP，电脑支持Host Name的话可以使用**pynq**进行连接，不支持的话可以使用设备的IPV4地址进行连接

点击`connect`即可连接，`Disconnect`断开连接

勾选`Save Enable`可以存储数据

连接之后，中间的坐标框会显示出当前接收到的波形，右侧的`CH1`、`CH2`可以勾选是否显示，以及改变显示的颜色

#### 4.1.1 寄存器配置

上位机支持对设备的一些功能进行配置

**界面底部** 可以查看当前配置的寄存器的数值，也可以通过修改寄存器的值来实现下采样、控制增益等功能

| 地址 | 功能                   | 值                                                           |
| ---- | ---------------------- | ------------------------------------------------------------ |
| 0x14 | 配置前级增益           | 以0x6000600为例，高三位与第三位分别表示两个通道的控制字；    |
| 0x18 | 配置采样率             | 1MSPS/(N+1)；以0x01为例，可以配置采样率为500k，默认为0x04，200k； |
| 0x1c | 配置ADC和PPS的数据来源 | 高四位为55aa的时候使用内部ADC数据，第四位为5aa5的时候使用内部PPS数据； |

### 4.2 设备本地数据保存

设备会在~/disk/disk1和~/disk/disk2下分别保存两份相同的文件，并且每天新建一个文件夹保存当天的数据

**设备保存数据的时间上限为180天，超过180天会将更早的数据删除**

### 4.3 数据格式

ADC的数据是16位有符号数，可以用int16将数据读出来，数据存储格式为：

|              |            帧头            |      GPS数据长度       |                           GPS数据                            |                           ADC数据                            |
| ------------ | :------------------------: | :--------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| **数据长度** |            2位             |          2位           |                           2 * N位                            |                                                              |
| **数据内容** |        0x55AA_A810         |         N - 0          | N<sub>1</sub>-0, N<sub>2</sub>-0, N<sub>3</sub>-0, ... ,N<sub>N</sub>-0 | A<sub>1</sub>-B<sub>1</sub>, A<sub>2</sub>-B<sub>2</sub>, A<sub>3</sub>-B<sub>3</sub>, ... |
| **注释**     | int16读出来是21930和-22512 | 第一位长度，第二位补零 |                    第一位数据，第二位补零                    |                   第一位A路，第二位B路 ...                   |

#### 4.3.1 数据读取参考代码

``` matlab
fid = fopen('GPS_20241107_182833.bin','r');
data = fread(fid,'int16');
gps_length = data(3);
header_length = gps_length * 2 + 4;
gps_data = data(5:2:header_length);
ch1 = data(header_length + 1:2:end);
ch2 = data(header_length + 2:2:end);
```

上面默认存下来的都是正确的数据，实际使用的时候可以校验下帧头等



## 5 常用操作

主要是使用Putty的时候会用到的一些操作

### 5.1 磁盘挂载

首先在未接硬盘之前，用fdisk -l指令查看一下当前设备的硬盘情况

``` bash
sudo fdisk -l
```

接上硬盘之后，再fdisk -l查看一下

``` bash
Device     Start         End     Sectors  Size Type
/dev/sda1   2048 31251759070 31251757023 14.6T Linux filesystem
```

这个/dev/sda1就是接上去的磁盘，可以根据大小判断一下

如果只有sda，说明当前磁盘还没有分区

/////////////////////////////////////////////////

接着，如果是新磁盘，需要先格式化一下，使用指令

``` bash
sudo mkfs.ext4 /dev/sda1
```

将磁盘格式化为ext4格式

挂载磁盘的指令

``` bash
sudo mount /dev/sda1 /home/xilinx/disk
```

#### 5.1.1 查看当前磁盘的挂载情况

``` bash
xilinx@pynq:~/pynq_dma$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        29G  5.6G   23G  21% /
devtmpfs        438M     0  438M   0% /dev
tmpfs           502M     0  502M   0% /dev/shm
tmpfs           502M  1.8M  501M   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           502M     0  502M   0% /sys/fs/cgroup
tmpfs           101M     0  101M   0% /run/user/1000
/dev/sda1        15T  705M   14T   1% /home/xilinx/disk/disk1
/dev/sdb1        15T  705M   14T   1% /home/xilinx/disk/disk2
```

可以看到当前/dev/sda1和/dev/sdb1已经分别挂载到了~/disk/disk1和~/disk/disk2下

### 5.2 Supervisor进程管理

接收机使用Supervisor来实现开机自启动以及进程的管理，要查看当前进程状态，使用指令

``` bash
xilinx@pynq:~$ sudo supervisorctl status
daq                              RUNNING   pid 1819, uptime 17:34:07
```

daq就是我们正在执行的数据采集进程，要想关闭进程，可以使用指令

``` bash
xilinx@pynq:~$ sudo supervisorctl stop daq
daq: stopped
xilinx@pynq:~$ sudo supervisorctl status
daq                              STOPPED   Nov 07 06:37 AM
```

再启动daq，可以使用指令

``` bash
xilinx@pynq:~$ sudo supervisorctl start daq
daq: started
```

开机自启动的脚本在/home/xilinx/pynq_dma文件夹下的start.sh里

**P.S. 有时候stop daq之后会发现程序还是在不断往disk里面写数据，如果再手动运行driver.py，可能设备会直接掉线，这时候可以用5.3中的办法手动关掉进程**

### 5.3 进程的查看与删除

使用top监视进程和Linux整体性能，找到pyton3还在运行

手动杀死进程：

``` bash
sudo pkill python3
```

之后就可以手动运行driver.py来进行代码的调试

### 5.4 不挂断进程的启动方式

``` bash
nohup sudo python3 -u driver.py > /home/xilinx/disk/test.log 2>&1 &
```

