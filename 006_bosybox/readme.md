# <center>***Busybox编译构建***</center>

Busybox是嵌入式系统上广泛应用得到根文件系统。接下来的部分是Busybo的编译构建过程。

## 第1章 构建最小根文件系统

### 1.1 编译

编译Busybox分为3个过程：配置（`make menuconfig`）、编译（`make`）和安装（`make install`）

执行`make menuconfig`命令，选择一些常用的配置项。

### 1.2 配置

*设置TAB自动补全*

```
Busybox Settings --->
    Busybox Library Tuning --->
        [*] Tab completaion // 勾选
```

*设置连接选项，不使用静态链接。*

使用glibc时，如果静态编译Busybox会警告报错。

```
Busybox Settings --->
    Build Options --->
        [] Build Busybox as s static binary (no shared libs)    // 不勾选
```

*常用的shell命令*

Busybox支持自定义配置裁剪shell命令。

```
Archival Utilities --->
    [*] ar
    ...
    [*] gzip
    ...
```

*模块加载命令*

```
Linux Module Utilities --->
    [*] insmod
    ...
    [*] rmmod
    ...
```

*mdev设备热插拔命令*

```
Linux System Utilities --->
    [*] mdev
    ...
    [*] mount
    ...
    [*] umount
    ...
```

*网络配置命令*

```
Networking Utilities --->
    [*] ifconfig
    ...
```

### 1.3 编译

直接输入`make`，开始编译

### 1.4 安装

注意，千万不能直接`make install`。因为我们现在是交叉编译，直接`make install`会安装到PC机上。

1. 创建新目录 ：`mkdir ~/nfs_root/first_fs`

2. 安装到刚创建的目录中 ：`make CONFIG_PREFIX=~/nfs_root/first_fs install`

    安装完成之后，目录中包含以下内容：

    ```sh
    ls -l ~/nfs_root/first_fs/
    总计 12
    drwxrwxr-x 2 ding ding 4096  7月 17 21:43 bin
    lrwxrwxrwx 1 ding ding   11  7月 17 21:43 linuxrc -> bin/busybox
    drwxrwxr-x 2 ding ding 4096  7月 17 21:43 sbin
    drwxrwxr-x 4 ding ding 4096  7月 17 21:43 usr
    ```

3. 完成

### 1.5 构建根文件系统

#### 1.5.1 创建设备文件/dev/console和/dev/null

我们先查看一下PC机的这两个设备文件。可以看到，这两个设备都是字符设备，然后也显示了对应的设备号。

```sh
s -l /dev/console /dev/null 
crw--w---- 1 root tty  5, 1  7月 17 21:30 /dev/console
crw-rw-rw- 1 root root 1, 3  7月 17 21:30 /dev/null
```

1. 创建dev目录

    ```sh
    cd ~/nfs_root/first_fs/

    mkdir dev
    ```

2. 创建设备文件

    ```sh
    cd dev

    sudo mknod console c 5 1
    sudo mknod null c 1 3
    ```

3. 完成

#### 1.5.2 创建/etc/inittab文件

1. 创建文件

    ```sh
    ding@linux:~/nfs_root/first_fs$ mkdir etc
    ding@linux:~/nfs_root/first_fs$ cd etc/
    ding@linux:~/nfs_root/first_fs/etc$ touch inittab
    ```

2. inittab文件内容

最简单的配置文件，里面只执行/bin/sh程序，他的标准输入、标准输出和标准错误都定义到console中

```sh
console::askfirst:-/bin/sh
```

3. 完成

#### 1.5.3 安装C库

1. 在最小根文件系统创建lib目录

    ```sh
    mkdir ~/nfs_root/first_fs/lib
    ```

2. 进入编译器安装路径的/arm-linux/lib子目录
   
   ```sh
   cd /usr/local/arm/gcc-3.4.5-glibc-2.3.6/arm-linux/lib
    ding@linux:/usr/local/arm/gcc-3.4.5-glibc-2.3.6/arm-linux/lib$ ls -l
    总计 18628
    -rw-r--r-- 1 ding ding    1863  1月 22  2008 crt1.o
    -rw-r--r-- 1 ding ding    2604  1月 22  2008 crti.o
    -rw-r--r-- 1 ding ding    2208  1月 22  2008 crtn.o
    drwxr-xr-x 2 ding ding    4096  1月 22  2008 gconv
    -rw-r--r-- 1 ding ding    2212  1月 22  2008 gcrt1.o
    -rwxr-xr-x 1 ding ding  112886  1月 22  2008 ld-2.3.6.so
    lrwxrwxrwx 1 ding ding      11  1月 22  2008 ld-linux.so.2 -> ld-2.3.6.so
    drwxr-xr-x 2 ding ding    4096  1月 22  2008 ldscripts
    -rwxr-xr-x 1 ding ding   17586  1月 22  2008 libanl-2.3.6.so
    -rw-r--r-- 1 ding ding   13094  1月 22  2008 libanl.a
    lrwxrwxrwx 1 ding ding      11  1月 22  2008 libanl.so -> libanl.so.1
    lrwxrwxrwx 1 ding ding      15  1月 22  2008 libanl.so.1 -> libanl-2.3.6.so
    -rwxr-xr-x 1 ding ding    8750  1月 22  2008 libBrokenLocale-2.3.6.so
    -rw-r--r-- 1 ding ding    1142  1月 22  2008 libBrokenLocale.a
    lrwxrwxrwx 1 ding ding      20  1月 22  2008 libBrokenLocale.so -> libBrokenLocale.so.1
    lrwxrwxrwx 1 ding ding      24  1月 22  2008 libBrokenLocale.so.1 -> libBrokenLocale-2.3.6.so
    -rw-r--r-- 1 ding ding     744  1月 22  2008 libbsd-compat.a
    -rwxr-xr-x 1 ding ding 1435660  1月 22  2008 libc-2.3.6.so
    -rw-r--r-- 1 ding ding 2768280  1月 22  2008 libc.a
    -rw-r--r-- 1 ding ding    7848  1月 22  2008 libc_nonshared.a
    -rwxr-xr-x 1 ding ding   30700  1月 22  2008 libcrypt-2.3.6.so
    -rw-r--r-- 1 ding ding   23118  1月 22  2008 libcrypt.a
    lrwxrwxrwx 1 ding ding      13  1月 22  2008 libcrypt.so -> libcrypt.so.1
    lrwxrwxrwx 1 ding ding      17  1月 22  2008 libcrypt.so.1 -> libcrypt-2.3.6.so
    -rw-r--r-- 1 ding ding     195  1月 22  2008 libc.so
    lrwxrwxrwx 1 ding ding      13  1月 22  2008 libc.so.6 -> libc-2.3.6.so
    -rw-r--r-- 1 ding ding     205  1月 22  2008 libc.so_orig
    -rwxr-xr-x 1 ding ding   16665  1月 22  2008 libdl-2.3.6.so
    -rw-r--r-- 1 ding ding    7474  1月 22  2008 libdl.a
    lrwxrwxrwx 1 ding ding      10  1月 22  2008 libdl.so -> libdl.so.2
    lrwxrwxrwx 1 ding ding      14  1月 22  2008 libdl.so.2 -> libdl-2.3.6.so
    -rw-r--r-- 1 ding ding     744  1月 22  2008 libg.a
    drwxr-xr-x 2 ding ding    4096  1月 22  2008 libgcc_s.dir
    lrwxrwxrwx 1 ding ding      13  1月 22  2008 libgcc_s.so -> libgcc_s.so.1
    -rw-r--r-- 1 ding ding   63973  1月 22  2008 libgcc_s.so.1
    -rw-r--r-- 1 ding ding  457212  1月 22  2008 libiberty.a
    -rw-r--r-- 1 ding ding     605  1月 22  2008 libieee.a
    -rwxr-xr-x 1 ding ding  779096  1月 22  2008 libm-2.3.6.so
    -rw-r--r-- 1 ding ding 1134282  1月 22  2008 libm.a
    -rw-r--r-- 1 ding ding     884  1月 22  2008 libmcheck.a
    -rwxr-xr-x 1 ding ding   23010  1月 22  2008 libmemusage.so
    lrwxrwxrwx 1 ding ding       9  1月 22  2008 libm.so -> libm.so.6
    lrwxrwxrwx 1 ding ding      13  1月 22  2008 libm.so.6 -> libm-2.3.6.so
    -rwxr-xr-x 1 ding ding   94450  1月 22  2008 libnsl-2.3.6.so
    -rw-r--r-- 1 ding ding  123414  1月 22  2008 libnsl.a
    lrwxrwxrwx 1 ding ding      11  1月 22  2008 libnsl.so -> libnsl.so.1
    lrwxrwxrwx 1 ding ding      15  1月 22  2008 libnsl.so.1 -> libnsl-2.3.6.so
    -rwxr-xr-x 1 ding ding   37812  1月 22  2008 libnss_compat-2.3.6.so
    lrwxrwxrwx 1 ding ding      18  1月 22  2008 libnss_compat.so -> libnss_compat.so.2
    lrwxrwxrwx 1 ding ding      22  1月 22  2008 libnss_compat.so.2 -> libnss_compat-2.3.6.so
    -rwxr-xr-x 1 ding ding   21467  1月 22  2008 libnss_dns-2.3.6.so
    lrwxrwxrwx 1 ding ding      15  1月 22  2008 libnss_dns.so -> libnss_dns.so.2
    lrwxrwxrwx 1 ding ding      19  1月 22  2008 libnss_dns.so.2 -> libnss_dns-2.3.6.so
    -rwxr-xr-x 1 ding ding   52833  1月 22  2008 libnss_files-2.3.6.so
    lrwxrwxrwx 1 ding ding      17  1月 22  2008 libnss_files.so -> libnss_files.so.2
    lrwxrwxrwx 1 ding ding      21  1月 22  2008 libnss_files.so.2 -> libnss_files-2.3.6.so
    -rwxr-xr-x 1 ding ding   22938  1月 22  2008 libnss_hesiod-2.3.6.so
    lrwxrwxrwx 1 ding ding      18  1月 22  2008 libnss_hesiod.so -> libnss_hesiod.so.2
    lrwxrwxrwx 1 ding ding      22  1月 22  2008 libnss_hesiod.so.2 -> libnss_hesiod-2.3.6.so
    -rwxr-xr-x 1 ding ding   50291  1月 22  2008 libnss_nis-2.3.6.so
    -rwxr-xr-x 1 ding ding   56909  1月 22  2008 libnss_nisplus-2.3.6.so
    lrwxrwxrwx 1 ding ding      19  1月 22  2008 libnss_nisplus.so -> libnss_nisplus.so.2
    lrwxrwxrwx 1 ding ding      23  1月 22  2008 libnss_nisplus.so.2 -> libnss_nisplus-2.3.6.so
    lrwxrwxrwx 1 ding ding      15  1月 22  2008 libnss_nis.so -> libnss_nis.so.2
    lrwxrwxrwx 1 ding ding      19  1月 22  2008 libnss_nis.so.2 -> libnss_nis-2.3.6.so
    -rwxr-xr-x 1 ding ding   10046  1月 22  2008 libpcprofile.so
    -rwxr-xr-x 1 ding ding  102457  1月 22  2008 libpthread-0.10.so
    -rw-r--r-- 1 ding ding  150540  1月 22  2008 libpthread.a
    -rw-r--r-- 1 ding ding    1132  1月 22  2008 libpthread_nonshared.a
    -rw-r--r-- 1 ding ding     207  1月 22  2008 libpthread.so
    lrwxrwxrwx 1 ding ding      18  1月 22  2008 libpthread.so.0 -> libpthread-0.10.so
    -rw-r--r-- 1 ding ding     217  1月 22  2008 libpthread.so_orig
    -rwxr-xr-x 1 ding ding   83645  1月 22  2008 libresolv-2.3.6.so
    -rw-r--r-- 1 ding ding   95024  1月 22  2008 libresolv.a
    lrwxrwxrwx 1 ding ding      14  1月 22  2008 libresolv.so -> libresolv.so.2
    lrwxrwxrwx 1 ding ding      18  1月 22  2008 libresolv.so.2 -> libresolv-2.3.6.so
    -rwxr-xr-x 1 ding ding   47920  1月 22  2008 librt-2.3.6.so
    -rw-r--r-- 1 ding ding   67792  1月 22  2008 librt.a
    lrwxrwxrwx 1 ding ding      10  1月 22  2008 librt.so -> librt.so.1
    lrwxrwxrwx 1 ding ding      14  1月 22  2008 librt.so.1 -> librt-2.3.6.so
    -rwxr-xr-x 1 ding ding   18959  1月 22  2008 libSegFault.so
    -rw-r--r-- 1 ding ding 6731052  1月 22  2008 libstdc++.a
    drwxr-xr-x 2 ding ding    4096  1月 22  2008 libstdc++.dir
    -rwxr-xr-x 1 ding ding    1344  1月 22  2008 libstdc++.la
    lrwxrwxrwx 1 ding ding      18  1月 22  2008 libstdc++.so -> libstdc++.so.6.0.3
    lrwxrwxrwx 1 ding ding      18  1月 22  2008 libstdc++.so.6 -> libstdc++.so.6.0.3
    -rwxr-xr-x 1 ding ding 3862070  1月 22  2008 libstdc++.so.6.0.3
    -rw-r--r-- 1 ding ding  323516  1月 22  2008 libsupc++.a
    -rwxr-xr-x 1 ding ding    1252  1月 22  2008 libsupc++.la
    -rwxr-xr-x 1 ding ding   30636  1月 22  2008 libthread_db-1.0.so
    lrwxrwxrwx 1 ding ding      17  1月 22  2008 libthread_db.so -> libthread_db.so.1
    lrwxrwxrwx 1 ding ding      19  1月 22  2008 libthread_db.so.1 -> libthread_db-1.0.so
    -rwxr-xr-x 1 ding ding   14213  1月 22  2008 libutil-2.3.6.so
    -rw-r--r-- 1 ding ding    8676  1月 22  2008 libutil.a
    lrwxrwxrwx 1 ding ding      12  1月 22  2008 libutil.so -> libutil.so.1
    lrwxrwxrwx 1 ding ding      16  1月 22  2008 libutil.so.1 -> libutil-2.3.6.so
    -rw-r--r-- 1 ding ding     546  1月 22  2008 Mcrt1.o
    -rw-r--r-- 1 ding ding    1863  1月 22  2008 Scrt1.o
   ```

3. 拷贝所有的.so文件到根文件系统的lib目录

    cd -d选项，表示如果源文件是链接文件，复制后仍然是链接文件

    ```sh
    cp -d *.so* ~/nfs_root/first_fs/lib/
    ```

至此，最小根文件系统制作完成！

### 1.6 制作映像文件

1. 使用mkyaffs2image工具来把文件系统制作成映像文件。命令：mkyaffs2image 目录名 映像文件名

```sh
mkyaffs2image first_fs first_fs.yaffs2
```

2. 把first_fs.yaffs2烧录到2440中，启动如下：

```
Booting Linux ...

NAND read: device 0 offset 0x60000, size 0x200000

Reading data from 0x25f800 -- 100% complete.
 2097152 bytes read: OK
## Booting image at 30007fc0 ...
   Image Name:   Linux-2.6.22.6-g6d79e08f
   Created:      2024-07-14  13:21:23 UTC
   Image Type:   ARM Linux Kernel Image (uncompressed)
   Data Size:    1848688 Bytes =  1.8 MB
   Load Address: 30008000
   Entry Point:  30008000
   Verifying Checksum ... OK
   XIP Kernel Image ... OK

Starting kernel ...

Uncompressing Linux...................................................................................................................... done, booting the kernel.
Linux version 2.6.22.6-g6d79e08f (ding@linux) (gcc version 3.4.5) #2 Sun Jul 14 21:21:21 CST 2024
CPU: ARM920T [41129200] revision 0 (ARMv4T), cr=c0007177
Machine: SMDK2440
Memory policy: ECC disabled, Data cache writeback
CPU S3C2440A (id 0x32440001)
S3C244X: core 400.000 MHz, memory 100.000 MHz, peripheral 50.000 MHz
S3C24XX Clocks, (c) 2004 Simtec Electronics
CLOCK: Slow mode (1.500 MHz), fast, MPLL on, UPLL on
CPU0: D VIVT write-back cache
CPU0: I cache: 16384 bytes, associativity 64, 32 byte lines, 8 sets
CPU0: D cache: 16384 bytes, associativity 64, 32 byte lines, 8 sets
Built 1 zonelists.  Total pages: 16256
Kernel command line: noinitrd root=/dev/mtdblock3 init=/linuxrc console=ttySAC0,115200
irq: clearing subpending status 00000003
irq: clearing subpending status 00000002
PID hash table entries: 256 (order: 8, 1024 bytes)
timer tcon=00500000, tcnt a2c1, tcfg 00000200,00000000, usec 00001eb8
Console: colour dummy device 80x30
Dentry cache hash table entries: 8192 (order: 3, 32768 bytes)
Inode-cache hash table entries: 4096 (order: 2, 16384 bytes)
Memory: 64MB = 64MB total
Memory: 60976KB available (3264K code, 458K data, 140K init)
Mount-cache hash table entries: 512
CPU: Testing write buffer coherency: ok
NET: Registered protocol family 16
S3C2410 Power Management, (c) 2004 Simtec Electronics
S3C2440: Initialising architecture
S3C2440: IRQ Support
S3C2440: Clock Support, DVS off
S3C24XX DMA Driver, (c) 2003-2004,2006 Simtec Electronics
DMA channel 0 at c4800000, irq 33
DMA channel 1 at c4800040, irq 34
DMA channel 2 at c4800080, irq 35
DMA channel 3 at c48000c0, irq 36
SCSI subsystem initialized
usbcore: registered new interface driver usbfs
usbcore: registered new interface driver hub
usbcore: registered new device driver usb
NET: Registered protocol family 2
IP route cache hash table entries: 1024 (order: 0, 4096 bytes)
TCP established hash table entries: 2048 (order: 2, 16384 bytes)
TCP bind hash table entries: 2048 (order: 1, 8192 bytes)
TCP: Hash tables configured (established 2048 bind 2048)
TCP reno registered
NetWinder Floating Point Emulator V0.97 (double precision)
Registering GDB sysrq handler
JFFS2 version 2.2. (NAND) © 2001-2006 Red Hat, Inc.
yaffs Jul 14 2024 21:20:43 Installing. 
io scheduler noop registered
io scheduler anticipatory registered (default)
io scheduler deadline registered
io scheduler cfq registered
Console: switching to colour frame buffer device 60x34
fb0: s3c2410fb frame buffer device
lp: driver loaded but no devices found
ppdev: user-space parallel port driver
S3C2410 Watchdog Timer, (c) 2004 Simtec Electronics
Serial: 8250/16550 driver $Revision: 1.90 $ 4 ports, IRQ sharing enabled
s3c2440-uart.0: s3c2410_serial0 at MMIO map 0x50000000 mem 0xf0400000 (irq = 70) is a S3C2440
s3c2440-uart.1: s3c2410_serial1 at MMIO map 0x50004000 mem 0xf0404000 (irq = 73) is a S3C2440
s3c2440-uart.2: s3c2410_serial2 at MMIO map 0x50008000 mem 0xf0408000 (irq = 76) is a S3C2440
RAMDISK driver initialized: 16 RAM disks of 4096K size 1024 blocksize
loop: module loaded
line 400 <DM9KS> I/O: c486a000, VID: 90000a46 
line 408 <DM9KS> I/O: c486a000, VID: 90000a46 
<DM9KS> error version, chip_revision = 0x1a, chip_info = 0x3
id_val=0
S3C24XX NAND Driver, (c) 2004 Simtec Electronics
s3c2440-nand s3c2440-nand: Tacls=3, 30ns Twrph0=7 70ns, Twrph1=3 30ns
NAND device: Manufacturer ID: 0xec, Chip ID: 0xda (Samsung NAND 256MiB 3,3V 8-bit)
Scanning device for bad blocks
Bad eraseblock 574 at 0x047c0000
Bad eraseblock 1505 at 0x0bc20000
Creating 4 MTD partitions on "NAND 256MiB 3,3V 8-bit":
0x00000000-0x00040000 : "bootloader"
0x00040000-0x00060000 : "params"
0x00060000-0x00260000 : "kernel"
0x00260000-0x10000000 : "root"
s3c2410-ohci s3c2410-ohci: S3C24XX OHCI
s3c2410-ohci s3c2410-ohci: new USB bus registered, assigned bus number 1
s3c2410-ohci s3c2410-ohci: irq 42, io mem 0x49000000
usb usb1: configuration #1 chosen from 1 choice
hub 1-0:1.0: USB hub found
hub 1-0:1.0: 2 ports detected
Initializing USB Mass Storage driver...
usbcore: registered new interface driver usb-storage
USB Mass Storage support registered.
mice: PS/2 mouse device common for all mice
s3c2410 TouchScreen successfully loaded
input: s3c2410 TouchScreen as /class/input/input0
S3C24XX RTC, (c) 2004,2006 Simtec Electronics
s3c2440-i2c s3c2440-i2c: slave address 0x10
s3c2440-i2c s3c2440-i2c: bus frequency set to 390 KHz
s3c2440-i2c s3c2440-i2c: i2c-0: S3C I2C adapter
mapped channel 0 to 0
s3c2440-sdi s3c2440-sdi: powered down.
s3c2440-sdi s3c2440-sdi: initialisation done.
s3c2440-sdi s3c2440-sdi: running at 0kHz (requested: 0kHz).
s3c2440-sdi s3c2440-sdi: running at 196kHz (requested: 195kHz).
s3c2440-sdi s3c2440-sdi: running at 196kHz (requested: 195kHz).
s3c2440-sdi s3c2440-sdi: running at 196kHz (requested: 195kHz).
usbcore: registered new interface driver hiddev
s3c2440-sdi s3c2440-sdi: CMD[TIMEOUT] #2 op:UNKNOWN(8) arg:0x000001aa flags:0x0875 retries:0 Status:nothing to complete
usbcore: registered new interface driver usbhid
s3c2440-sdi s3c2440-sdi: CMD[TIMEOUT] #3 op:APP_CMD(55) arg:0x00000000 flags:0x0875 retries:0 Status:nothing to complete
drivers/hid/usbhid/hid-core.c: v2.6:USB HID core driver
Advanced Linux Sound Architecture Driver Version 1.0.14 (Thu May 31 09:03:25 2007 UTC).
s3c2440-sdi s3c2440-sdi: CMD[TIMEOUT] #4 op:APP_CMD(55) arg:0x00000000 flags:0x0875 retries:0 Status:nothing to complete
ASoC version 0.13.1
s3c2440-sdi s3c2440-sdi: CMD[TIMEOUT] #5 op:APP_CMD(55) arg:0x00000000 flags:0x0875 retries:0 Status:nothing to complete
s3c2410iis_probe...
s3c2440-sdi s3c2440-sdi: CMD[TIMEOUT] #6 op:APP_CMD(55) arg:0x00000000 flags:0x0875 retries:0 Status:nothing to complete
s3c2440-sdi s3c2440-sdi: CMD[TIMEOUT] #7 op:ALL_SEND_OCR(1) arg:0x00000000 flags:0x0861 retries:0 Status:nothing to complete
s3c2440-sdi s3c2440-sdi: powered down.
UDA1341 audio driver initialized
ALSA device list:
  No soundcards found.
TCP cubic registered
NET: Registered protocol family 1
drivers/rtc/hctosys.c: unable to open rtc device (rtc0)
UDF-fs: No VRS found
yaffs: dev is 32505859 name is "mtdblock3"
yaffs: passed flags ""
yaffs: Attempting MTD mount on 31.3, "mtdblock3"
yaffs: auto selecting yaffs2
block 556 is bad
block 1487 is bad
VFS: Mounted root (yaffs filesystem).
Freeing init memory: 140K
init started: BusyBox v1.7.0 (2024-07-17 21:34:42 CST)

Please press Enter to activate this console. 
```

3. 输入enter，进入shell目录

```sh
Please press Enter to activate this console.

starting pid 764, tty '/dev/console': '/bin/sh'
# ls
bin         etc         linuxrc     sbin
dev         lib         lost+found  usr
```

至此，最小根文件系统制作完成

## 第2章 完善根文件系统

### 2.1 存在的问题

最小根文件系统已经具备基础的功能了，比如ls命令。为什么还要完善？

比如我们想使用ps命令，但出现了下面的报错，提示打不开/proc目录。

```sh
# ps
  PID  Uid        VSZ Stat Command
ps: can't open '/proc': No such file or directory
```

要怎么做，才能执行ps命令？需要执行下面的2条命令

```sh
mkdir /proc

mount -t proc none /proc
```

然后就可以执行ps命令了

```sh
# mkdir proc
# mount -t proc none /proc
# ps
  PID  Uid        VSZ Stat Command
    1 0          3092 S   init     
    2 0               SW< [kthreadd]
    3 0               SWN [ksoftirqd/0]
    4 0               SW< [watchdog/0]
    5 0               SW< [events/0]
    6 0               SW< [khelper]
   55 0               SW< [kblockd/0]
   56 0               SW< [ksuspend_usbd]
   59 0               SW< [khubd]
   61 0               SW< [kseriod]
   73 0               SW  [pdflush]
   74 0               SW  [pdflush]
   75 0               SW< [kswapd0]
   76 0               SW< [aio/0]
  710 0               SW< [mtdblockd]
  745 0               SW< [kmmcd]
  764 0          3096 S   -sh 
  770 0          3096 R   ps 
```

### 2.2 在根文件系统中挂载proc目录

1. 先把最小根文件系统复制一份出来

```sh
sudo cp -r -d first_fs second_fs
```

2. 创建proc目录

```sh
sudo mkdir proc
```

3. 在配置文件中添加脚本

/etc/inittab文件中，新增一行指定脚本

```sh
console::askfirst:-/bin/sh
::sysinit:/etc/init.d/rcS   # 指定脚本文件
```

4. 在脚本程序中挂载文件系统

先创建脚本文件

```sh
sudo mkdir etc/init.d
ding@linux:~/nfs_root/second_fs$ sudo touch etc/init.d/rcS
```

`etc/init.d/rcS`脚本文件中添加如下内容，来挂载proc

```sh
mount -t proc none /proc
```

5. 给`etc/init.d/rcS`脚本添加可执行权限

```sh
sudo cat etc/init.d/rcS
```

把这个制作成镜像文件烧录进去后，启动过程如下。可以看到，现在能直接运行ps命令了

```sh
init started: BusyBox v1.7.0 (2024-07-17 21:34:42 CST)
starting pid 764, tty '': '/etc/init.d/rcS'

Please press Enter to activate this console. 
starting pid 766, tty '/dev/console': '/bin/sh'
# ps
  PID  Uid        VSZ Stat Command
    1 0          3092 S   init     
    2 0               SW< [kthreadd]
    3 0               SWN [ksoftirqd/0]
    4 0               SW< [watchdog/0]
    5 0               SW< [events/0]
    6 0               SW< [khelper]
   55 0               SW< [kblockd/0]
   56 0               SW< [ksuspend_usbd]
   59 0               SW< [khubd]
   61 0               SW< [kseriod]
   73 0               SW  [pdflush]
   74 0               SW  [pdflush]
   75 0               SW< [kswapd0]
   76 0               SW< [aio/0]
  710 0               SW< [mtdblockd]
  745 0               SW< [kmmcd]
  766 0          3096 S   -sh 
  767 0          3096 R   ps 
```

## 第3章 挂载全部的文件系统

在第2章中，通过在脚本程序rcS中执行`mount -t proc none /proc`命令，来挂载proc文件系统。除此之外，还有一种方式：`mount -a`命令。

`mount -a`命令，会去读取`/etc/fstab`这个文件，所以我们要设置好`/etc/fstab`文件内容。

1. 修改rcS脚本

```sh
# mount -t proc none /proc
mount -a
```

2. 创建/etc/fstab文件

```sh
ding@linux:~/nfs_root$ cd second_fs/
ding@linux:~/nfs_root/second_fs$ sudo touch etc/fstab
```

3. 设置/etc/fstab文件文件内容

```sh
# defive    mount-point    type    options    dump    fsck    order
proc        /proc          proc    defaults    0       0
```

完成。让我们再梳理一下完整的过程：

1. etc/inittab中做两个事情，打开shell+执行rcS脚本
2. rcS脚本中执行`mount -a`，读取fstab文件
3. fstab文件中，写好了我们要挂载的全部的文件系统

至此，启动过程就是自动挂载+启动shell

启动过程如下：

```sh
VFS: Mounted root (yaffs filesystem).
Freeing init memory: 140K
init started: BusyBox v1.7.0 (2024-07-17 21:34:42 CST)
starting pid 764, tty '': '/etc/init.d/rcS'

Please press Enter to activate this console. 
starting pid 766, tty '/dev/console': '/bin/sh'
# ps
  PID  Uid        VSZ Stat Command
    1 0          3092 S   init     
    2 0               SW< [kthreadd]
    3 0               SWN [ksoftirqd/0]
    4 0               SW< [watchdog/0]
    5 0               SW< [events/0]
    6 0               SW< [khelper]
   55 0               SW< [kblockd/0]
   56 0               SW< [ksuspend_usbd]
   59 0               SW< [khubd]
   61 0               SW< [kseriod]
   73 0               SW  [pdflush]
   74 0               SW  [pdflush]
   75 0               SW< [kswapd0]
   76 0               SW< [aio/0]
  710 0               SW< [mtdblockd]
  745 0               SW< [kmmcd]
  766 0          3096 S   -sh 
  767 0          3096 R   ps 
```

## 第4章 mdev自动创建/dev设备节点

当前根文件系统的dev目录下，只有这2个东西。实际的设备和驱动都在dev目录下，有成千上万个文件的话，每个都要自己创建太麻烦了。linux中有一种udev机制，可以自动创建dev目录下的设备节点。在busybox有mdev，他是udev的简化版本。

```sh
# ls /dev
console  null
```

busybox的mdev怎么使用？在mdev.txt的文档中有描述。下面是操作过程：

1. 进入根文件系统目录

```sh
ding@linux:~/nfs_root$ cd second_fs/
```

2. 创建sys目录

```sh
ding@linux:~/nfs_root/second_fs$ sudo mkdir sys
```

3. 修改fstab文件，现在需要挂载更多的文件系统

```sh
# defive    mount-point    type    options    dump    fsck    order
proc        /proc          proc    defaults    0       0
sysfs       /sys           sysfs   defaults    0       0
tmpfs       /dev           tmpfs   defaults    0       0
```

4. 修改rcS脚本，添加mdev相关信息

```sh
# mount -t proc none /proc
mount -a
mkdir /dev/pts
mount -t devpts devpts /dev/pts
echo /sbin/mdev > /proc/sys/kernel/hotplug
mdev -s
```

5. 完成。重新烧录文件系统看看效果

可以看到，mdev自动给我们创建了很多文件

```sh
# ls /dev/
console          ptyrf            tty17            ttyq7
dsp              ptys0            tty18            ttyq8
event0           ptys1            tty19            ttyq9
fb0              ptys2            tty2             ttyqa
full             ptys3            tty20            ttyqb
kmem             ptys4            tty21            ttyqc
kmsg             ptys5            tty22            ttyqd
loop0            ptys6            tty23            ttyqe
loop1            ptys7            tty24            ttyqf
loop2            ptys8            tty25            ttyr0
loop3            ptys9            tty26            ttyr1
loop4            ptysa            tty27            ttyr2
loop5            ptysb            tty28            ttyr3
loop6            ptysc            tty29            ttyr4
loop7            ptysd            tty3             ttyr5
mem              ptyse            tty30            ttyr6
mice             ptysf            tty31            ttyr7
mixer            ptyt0            tty32            ttyr8
mouse0           ptyt1            tty33            ttyr9
mtd0             ptyt2            tty34            ttyra
mtd0ro           ptyt3            tty35            ttyrb
mtd1             ptyt4            tty36            ttyrc
mtd1ro           ptyt5            tty37            ttyrd
mtd2             ptyt6            tty38            ttyre
mtd2ro           ptyt7            tty39            ttyrf
mtd3             ptyt8            tty4             ttys0
mtd3ro           ptyt9            tty40            ttys1
mtdblock0        ptyta            tty41            ttys2
mtdblock1        ptytb            tty42            ttys3
mtdblock2        ptytc            tty43            ttys4
mtdblock3        ptytd            tty44            ttys5
null             ptyte            tty45            ttys6
psaux            ptytf            tty46            ttys7
ptmx             ptyu0            tty47            ttys8
pts              ptyu1            tty48            ttys9
ptya0            ptyu2            tty49            ttysa
ptya1            ptyu3            tty5             ttysb
ptya2            ptyu4            tty50            ttysc
ptya3            ptyu5            tty51            ttysd
ptya4            ptyu6            tty52            ttyse
ptya5            ptyu7            tty53            ttysf
ptya6            ptyu8            tty54            ttyt0
ptya7            ptyu9            tty55            ttyt1
ptya8            ptyua            tty56            ttyt2
ptya9            ptyub            tty57            ttyt3
ptyaa            ptyuc            tty58            ttyt4
ptyab            ptyud            tty59            ttyt5
ptyac            ptyue            tty6             ttyt6
ptyad            ptyuf            tty60            ttyt7
ptyae            ptyv0            tty61            ttyt8
ptyaf            ptyv1            tty62            ttyt9
ptyb0            ptyv2            tty63            ttyta
ptyb1            ptyv3            tty7             ttytb
ptyb2            ptyv4            tty8             ttytc
ptyb3            ptyv5            tty9             ttytd
ptyb4            ptyv6            ttyS0            ttyte
ptyb5            ptyv7            ttyS1            ttytf
ptyb6            ptyv8            ttyS2            ttyu0
ptyb7            ptyv9            ttyS3            ttyu1
ptyb8            ptyva            ttya0            ttyu2
ptyb9            ptyvb            ttya1            ttyu3
ptyba            ptyvc            ttya2            ttyu4
ptybb            ptyvd            ttya3            ttyu5
ptybc            ptyve            ttya4            ttyu6
ptybd            ptyvf            ttya5            ttyu7
ptybe            ptyw0            ttya6            ttyu8
ptybf            ptyw1            ttya7            ttyu9
ptyc0            ptyw2            ttya8            ttyua
ptyc1            ptyw3            ttya9            ttyub
ptyc2            ptyw4            ttyaa            ttyuc
ptyc3            ptyw5            ttyab            ttyud
ptyc4            ptyw6            ttyac            ttyue
ptyc5            ptyw7            ttyad            ttyuf
ptyc6            ptyw8            ttyae            ttyv0
ptyc7            ptyw9            ttyaf            ttyv1
ptyc8            ptywa            ttyb0            ttyv2
ptyc9            ptywb            ttyb1            ttyv3
ptyca            ptywc            ttyb2            ttyv4
ptycb            ptywd            ttyb3            ttyv5
ptycc            ptywe            ttyb4            ttyv6
ptycd            ptywf            ttyb5            ttyv7
ptyce            ptyx0            ttyb6            ttyv8
ptycf            ptyx1            ttyb7            ttyv9
ptyd0            ptyx2            ttyb8            ttyva
ptyd1            ptyx3            ttyb9            ttyvb
ptyd2            ptyx4            ttyba            ttyvc
ptyd3            ptyx5            ttybb            ttyvd
ptyd4            ptyx6            ttybc            ttyve
ptyd5            ptyx7            ttybd            ttyvf
ptyd6            ptyx8            ttybe            ttyw0
ptyd7            ptyx9            ttybf            ttyw1
ptyd8            ptyxa            ttyc0            ttyw2
ptyd9            ptyxb            ttyc1            ttyw3
ptyda            ptyxc            ttyc2            ttyw4
ptydb            ptyxd            ttyc3            ttyw5
ptydc            ptyxe            ttyc4            ttyw6
ptydd            ptyxf            ttyc5            ttyw7
ptyde            ptyy0            ttyc6            ttyw8
ptydf            ptyy1            ttyc7            ttyw9
ptye0            ptyy2            ttyc8            ttywa
ptye1            ptyy3            ttyc9            ttywb
ptye2            ptyy4            ttyca            ttywc
ptye3            ptyy5            ttycb            ttywd
ptye4            ptyy6            ttycc            ttywe
ptye5            ptyy7            ttycd            ttywf
ptye6            ptyy8            ttyce            ttyx0
ptye7            ptyy9            ttycf            ttyx1
ptye8            ptyya            ttyd0            ttyx2
ptye9            ptyyb            ttyd1            ttyx3
ptyea            ptyyc            ttyd2            ttyx4
ptyeb            ptyyd            ttyd3            ttyx5
ptyec            ptyye            ttyd4            ttyx6
ptyed            ptyyf            ttyd5            ttyx7
ptyee            ptyz0            ttyd6            ttyx8
ptyef            ptyz1            ttyd7            ttyx9
ptyp0            ptyz2            ttyd8            ttyxa
ptyp1            ptyz3            ttyd9            ttyxb
ptyp2            ptyz4            ttyda            ttyxc
ptyp3            ptyz5            ttydb            ttyxd
ptyp4            ptyz6            ttydc            ttyxe
ptyp5            ptyz7            ttydd            ttyxf
ptyp6            ptyz8            ttyde            ttyy0
ptyp7            ptyz9            ttydf            ttyy1
ptyp8            ptyza            ttye0            ttyy2
ptyp9            ptyzb            ttye1            ttyy3
ptypa            ptyzc            ttye2            ttyy4
ptypb            ptyzd            ttye3            ttyy5
ptypc            ptyze            ttye4            ttyy6
ptypd            ptyzf            ttye5            ttyy7
ptype            ram0             ttye6            ttyy8
ptypf            ram1             ttye7            ttyy9
ptyq0            ram10            ttye8            ttyya
ptyq1            ram11            ttye9            ttyyb
ptyq2            ram12            ttyea            ttyyc
ptyq3            ram13            ttyeb            ttyyd
ptyq4            ram14            ttyec            ttyye
ptyq5            ram15            ttyed            ttyyf
ptyq6            ram2             ttyee            ttyz0
ptyq7            ram3             ttyef            ttyz1
ptyq8            ram4             ttyp0            ttyz2
ptyq9            ram5             ttyp1            ttyz3
ptyqa            ram6             ttyp2            ttyz4
ptyqb            ram7             ttyp3            ttyz5
ptyqc            ram8             ttyp4            ttyz6
ptyqd            ram9             ttyp5            ttyz7
ptyqe            random           ttyp6            ttyz8
ptyqf            root             ttyp7            ttyz9
ptyr0            s3c2410_serial0  ttyp8            ttyza
ptyr1            s3c2410_serial1  ttyp9            ttyzb
ptyr2            s3c2410_serial2  ttypa            ttyzc
ptyr3            timer            ttypb            ttyzd
ptyr4            ts0              ttypc            ttyze
ptyr5            tty              ttypd            ttyzf
ptyr6            tty0             ttype            urandom
ptyr7            tty1             ttypf            usbdev1.1_ep00
ptyr8            tty10            ttyq0            usbdev1.1_ep81
ptyr9            tty11            ttyq1            vcs
ptyra            tty12            ttyq2            vcsa
ptyrb            tty13            ttyq3            watchdog
ptyrc            tty14            ttyq4            zero
ptyrd            tty15            ttyq5
ptyre            tty16            ttyq6
```

到现在为止，这个最小根文件系统基本就比较完善了。

## 第5章 NFS网络文件系统

在之前的文件系统中，我们每次修改都要重新烧写，很麻烦。有没有办法不烧写？有的，这就是NFS网络文件系统。

所谓的网络文件系统，就是把文件系统放在服务器上，然后内核启动时直接去识别到服务器上的这个目录，把他当作我们的根文件系统。

要挂接NFS，需要什么前提条件呢？

1. 服务器“允许”那个目录可被别人挂接
2. 单板去挂接
   
### 5.1 配置NFS服务器

1. 指定允许挂载的目录

NFS服务器的配置文件为：`/etc/exports`。修改如下：

```sh
ding@linux:~/nfs_root$ sudo cat /etc/exports 
# /etc/exports: the access control list for filesystems which may be exported
#		to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/home/ding/nfs_root/nfs_fs	*(rw,sync,no_root_squash)
```

2. 重启FNS服务器
   
```sh
ding@linux:~/nfs_root$ sudo /etc/init.d/nfs-kernel-server restart
```

3. 在服务器上测试，看服务器自己能否挂接NFS

通常来说，到这一步了NFS应该就可以正常用了。我们也可以在服务器上测试一下，确认服务器自己可以挂接NFS。下面把NFS挂载到/mnt目录下

```sh
ding@linux:~$ sudo mount -t nfs 192.168.1.5:/home/ding/nfs_root/nfs_fs /mnt/

ding@linux:~$ ls /mnt/
bin  dev  etc  lib  linuxrc  proc  sbin  sys  usr
```

挂接成功！

### 5.2 单板手动挂载NFS
   
现在要在开发板上也挂接FNS，同样挂载在/mnt目录下。我们现在开始手动挂载

1. 创建/mnt目录

```sh
# mkdir /mnt
```

2. 挂载FNS
   
```sh
# mount -t nfs -o nolock 192.168.1.5:/home/ding/nfs_root/nfs_fs /mnt
```

注意，如果出现了`nfs: server 192.168.1.5 not responding, timed out`这种报错，可以按以下修改

```sh
# mount -t nfs -o tcp,nolock 192.168.1.5:/home/ding/nfs_root/nfs_fs /mnt
```

*NFS默认使用的是 UDP协议，在客户端把协议改成TCP解决*

[NFS超时报错解决方案](https://blog.csdn.net/changzehai/article/details/124495480)

3. 查看NFS
   
```sh
# ls mnt/
bin      etc      linuxrc  sbin     usr
dev      lib      proc     sys
```

4. 测试FNS。我们在服务器的根文件系统中创建一个hello.txt的文件，在单板上也同样能看到这个文件

服务器上的文件：

```sh
ding@linux:~/nfs_root/nfs_fs$ sudo vim hello.txt
ding@linux:~/nfs_root/nfs_fs$ ls
bin  dev  etc  hello.txt  lib  linuxrc  proc  sbin  sys  usr
```

单板上的文件：

```sh
# ls /mnt/
bin        etc        lib        proc       sys
dev        hello.txt  linuxrc    sbin       usr

# cat /mnt/hello.txt 
hello world
```

这种方法是启动之后手动挂载的。

### 5.3 从NFS启动

前面介绍了启动之后手动挂载NFS的方法。如果我们不想每次都敲命令，可以设置直接从NFS启动。

linux内核文档nfsroot.txt，介绍了怎么从NFS启动。

1. 在uboot中输入`print`，查看uboot的默认环境变量参数

```sh
bootargs=noinitrd root=/dev/nfs init=/linuxrc console=ttySAC0,115200
```

2. 在uboot中输入下面的命令，重设启动参数

set bootargs noinitrd root=/dev/nfs nfsroot=192.168.1.5:/home/ding/nfs_root/nfs_fs,v3,tcp ip=192.168.1.10:192.168.1.5:192.168.1.1:255.255.255.0::eth0:off console=ttySAC0,115200

3. 如果NFS服务器UDP一直报错，可以参考下面的文档

[Ubuntu 22.04 LTS 环境下 开发板 nfsroot配置要点](https://blog.csdn.net/weixin_42144057/article/details/129864379)

### 5.4 挂载NFS成功

下面是挂载NFS成功工作状态。可以看到，单板也识别到了虚拟机的hello.txt文档。

```sh
+---------------------------------------------+
| S3C2440A USB Downloader ver R0.03 2004 Jan  |
+---------------------------------------------+
USB: IN_ENDPOINT:1 OUT_ENDPOINT:3
FORMAT: <ADDR(DATA):4>+<SIZE(n+10):4>+<DATA:n>+<CS:2>
NOTE: Power off/on or press the reset button for 1 sec
      in order to get a valid USB device address.

Hit any key to stop autoboot:  0 
Booting Linux ...

NAND read: device 0 offset 0x60000, size 0x200000

Reading data from 0x25f800 -- 100% complete.
 2097152 bytes read: OK
## Booting image at 30007fc0 ...
   Image Name:   Linux-2.6.22.6-g6d79e08f
   Created:      2024-07-14  13:21:23 UTC
   Image Type:   ARM Linux Kernel Image (uncompressed)
   Data Size:    1848688 Bytes =  1.8 MB
   Load Address: 30008000
   Entry Point:  30008000
   Verifying Checksum ... OK
   XIP Kernel Image ... OK

Starting kernel ...

Uncompressing Linux...................................................................................................................... done, booting the kernel.
Linux version 2.6.22.6-g6d79e08f (ding@linux) (gcc version 3.4.5) #2 Sun Jul 14 21:21:21 CST 2024
CPU: ARM920T [41129200] revision 0 (ARMv4T), cr=c0007177
Machine: SMDK2440
Memory policy: ECC disabled, Data cache writeback
CPU S3C2440A (id 0x32440001)
S3C244X: core 400.000 MHz, memory 100.000 MHz, peripheral 50.000 MHz
S3C24XX Clocks, (c) 2004 Simtec Electronics
CLOCK: Slow mode (1.500 MHz), fast, MPLL on, UPLL on
CPU0: D VIVT write-back cache
CPU0: I cache: 16384 bytes, associativity 64, 32 byte lines, 8 sets
CPU0: D cache: 16384 bytes, associativity 64, 32 byte lines, 8 sets
Built 1 zonelists.  Total pages: 16256
Kernel command line: noinitrd root=/dev/nfs nfsroot=192.168.1.5:/home/ding/nfs_root/nfs_fs,v3,tcp ip=192.168.1.10:192.168.1.5:192.168.1.1:255.255.255.0::eth0:off console=ttySAC0,115200
irq: clearing subpending status 00000002
PID hash table entries: 256 (order: 8, 1024 bytes)
timer tcon=00500000, tcnt a2c1, tcfg 00000200,00000000, usec 00001eb8
Console: colour dummy device 80x30
Dentry cache hash table entries: 8192 (order: 3, 32768 bytes)
Inode-cache hash table entries: 4096 (order: 2, 16384 bytes)
Memory: 64MB = 64MB total
Memory: 60976KB available (3264K code, 458K data, 140K init)
Mount-cache hash table entries: 512
CPU: Testing write buffer coherency: ok
NET: Registered protocol family 16
S3C2410 Power Management, (c) 2004 Simtec Electronics
S3C2440: Initialising architecture
S3C2440: IRQ Support
S3C2440: Clock Support, DVS off
S3C24XX DMA Driver, (c) 2003-2004,2006 Simtec Electronics
DMA channel 0 at c4800000, irq 33
DMA channel 1 at c4800040, irq 34
DMA channel 2 at c4800080, irq 35
DMA channel 3 at c48000c0, irq 36
SCSI subsystem initialized
usbcore: registered new interface driver usbfs
usbcore: registered new interface driver hub
usbcore: registered new device driver usb
NET: Registered protocol family 2
IP route cache hash table entries: 1024 (order: 0, 4096 bytes)
TCP established hash table entries: 2048 (order: 2, 16384 bytes)
TCP bind hash table entries: 2048 (order: 1, 8192 bytes)
TCP: Hash tables configured (established 2048 bind 2048)
TCP reno registered
NetWinder Floating Point Emulator V0.97 (double precision)
Registering GDB sysrq handler
JFFS2 version 2.2. (NAND) © 2001-2006 Red Hat, Inc.
yaffs Jul 14 2024 21:20:43 Installing. 
io scheduler noop registered
io scheduler anticipatory registered (default)
io scheduler deadline registered
io scheduler cfq registered
Console: switching to colour frame buffer device 60x34
fb0: s3c2410fb frame buffer device
lp: driver loaded but no devices found
ppdev: user-space parallel port driver
S3C2410 Watchdog Timer, (c) 2004 Simtec Electronics
Serial: 8250/16550 driver $Revision: 1.90 $ 4 ports, IRQ sharing enabled
s3c2440-uart.0: s3c2410_serial0 at MMIO map 0x50000000 mem 0xf0400000 (irq = 70) is a S3C2440
s3c2440-uart.1: s3c2410_serial1 at MMIO map 0x50004000 mem 0xf0404000 (irq = 73) is a S3C2440
s3c2440-uart.2: s3c2410_serial2 at MMIO map 0x50008000 mem 0xf0408000 (irq = 76) is a S3C2440
RAMDISK driver initialized: 16 RAM disks of 4096K size 1024 blocksize
loop: module loaded
line 400 <DM9KS> I/O: c486a000, VID: 90000a46 
line 408 <DM9KS> I/O: c486a000, VID: 90000a46 
<DM9KS> error version, chip_revision = 0x1a, chip_info = 0x3
id_val=0
S3C24XX NAND Driver, (c) 2004 Simtec Electronics
s3c2440-nand s3c2440-nand: Tacls=3, 30ns Twrph0=7 70ns, Twrph1=3 30ns
NAND device: Manufacturer ID: 0xec, Chip ID: 0xda (Samsung NAND 256MiB 3,3V 8-bit)
Scanning device for bad blocks
Bad eraseblock 574 at 0x047c0000
Bad eraseblock 1505 at 0x0bc20000
Creating 4 MTD partitions on "NAND 256MiB 3,3V 8-bit":
0x00000000-0x00040000 : "bootloader"
0x00040000-0x00060000 : "params"
0x00060000-0x00260000 : "kernel"
0x00260000-0x10000000 : "root"
s3c2410-ohci s3c2410-ohci: S3C24XX OHCI
s3c2410-ohci s3c2410-ohci: new USB bus registered, assigned bus number 1
s3c2410-ohci s3c2410-ohci: irq 42, io mem 0x49000000
usb usb1: configuration #1 chosen from 1 choice
hub 1-0:1.0: USB hub found
hub 1-0:1.0: 2 ports detected
Initializing USB Mass Storage driver...
usbcore: registered new interface driver usb-storage
USB Mass Storage support registered.
mice: PS/2 mouse device common for all mice
s3c2410 TouchScreen successfully loaded
input: s3c2410 TouchScreen as /class/input/input0
S3C24XX RTC, (c) 2004,2006 Simtec Electronics
s3c2440-i2c s3c2440-i2c: slave address 0x10
s3c2440-i2c s3c2440-i2c: bus frequency set to 390 KHz
s3c2440-i2c s3c2440-i2c: i2c-0: S3C I2C adapter
mapped channel 0 to 0
s3c2440-sdi s3c2440-sdi: powered down.
s3c2440-sdi s3c2440-sdi: initialisation done.
s3c2440-sdi s3c2440-sdi: running at 0kHz (requested: 0kHz).
s3c2440-sdi s3c2440-sdi: running at 196kHz (requested: 195kHz).
s3c2440-sdi s3c2440-sdi: running at 196kHz (requested: 195kHz).
s3c2440-sdi s3c2440-sdi: running at 196kHz (requested: 195kHz).
usbcore: registered new interface driver hiddev
s3c2440-sdi s3c2440-sdi: CMD[TIMEOUT] #2 op:UNKNOWN(8) arg:0x000001aa flags:0x0875 retries:0 Status:nothing to complete
usbcore: registered new interface driver usbhid
s3c2440-sdi s3c2440-sdi: CMD[TIMEOUT] #3 op:APP_CMD(55) arg:0x00000000 flags:0x0875 retries:0 Status:nothing to complete
drivers/hid/usbhid/hid-core.c: v2.6:USB HID core driver
Advanced Linux Sound Architecture Driver Version 1.0.14 (Thu May 31 09:03:25 2007 UTC).
s3c2440-sdi s3c2440-sdi: CMD[TIMEOUT] #4 op:APP_CMD(55) arg:0x00000000 flags:0x0875 retries:0 Status:nothing to complete
ASoC version 0.13.1
s3c2440-sdi s3c2440-sdi: CMD[TIMEOUT] #5 op:APP_CMD(55) arg:0x00000000 flags:0x0875 retries:0 Status:nothing to complete
s3c2410iis_probe...
s3c2440-sdi s3c2440-sdi: CMD[TIMEOUT] #6 op:APP_CMD(55) arg:0x00000000 flags:0x0875 retries:0 Status:nothing to complete
s3c2440-sdi s3c2440-sdi: CMD[TIMEOUT] #7 op:ALL_SEND_OCR(1) arg:0x00000000 flags:0x0861 retries:0 Status:nothing to complete
s3c2440-sdi s3c2440-sdi: powered down.
UDA1341 audio driver initialized
ALSA device list:
  No soundcards found.
TCP cubic registered
NET: Registered protocol family 1
drivers/rtc/hctosys.c: unable to open rtc device (rtc0)
IP-Config: Complete:
      device=eth0, addr=192.168.1.10, mask=255.255.255.0, gw=192.168.1.1,
     host=192.168.1.10, domain=, nis-domain=(none),
     bootserver=192.168.1.5, rootserver=192.168.1.5, rootpath=
Looking up port of RPC 100003/3 on 192.168.1.5
Looking up port of RPC 100005/3 on 192.168.1.5
VFS: Mounted root (nfs filesystem).
Freeing init memory: 140K
init started: BusyBox v1.7.0 (2024-07-17 21:34:42 CST)
starting pid 765, tty '': '/etc/init.d/rcS'

Please press Enter to activate this console. 
starting pid 770, tty '/dev/console': '/bin/sh'
# 
# ls
bin        etc        lib        proc       sys
dev        hello.txt  linuxrc    sbin       usr
# 
```
