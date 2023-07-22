# การใช้งาน ZFS สำหรับเก็บข้อมูลใน LN Node
อ้างอิงจาก: [Create a ZFS pool to be used as a Raspiblitz data disk](https://github.com/openoms/bitcoin-tutorials/blob/master/zfs/create-raspiblitz-zfs-disk.md)

ZFS เป็นระบบ File System ที่ใช้ใน Linux เพื่อเก็บไฟล์ข้อมูลต่าง ๆ ซึ่งถ้าเปรียบเหมือน FAT32 บน Windows แต่ ZFS มีความเสถียรมากกว่า และมีฟังก์ชันในการ redundant ข้อมูลไฟล์ต่าง ๆ ได้ เพื่อป้องกันความเสียหายจาก Disk ได้

## เตรียมความพร้อมของอุปกรณ์ที่ต้องใช้
ก่อนเริ่มต้องอธิบายอุปกรณ์ที่จำเป็นต้องใช้เสียก่อน นอกจาก Node ที่ติดตั้ง Ubuntu เรียบร้อยแล้ว เราจำเป็นต้องมี Disk เพิ่มอีกจำนวน 2 ลูก (โดยไม่นับ NVMe Drive ที่อยู่ใน Nuc11 ที่ใช้สำหรับติดตั้ง OS) ซึ่งผมจะใช้เป็น USB Flash Drive จำนวน 2 ตัวและทั้งคู่ถูกเสียบด้วย USB Port ของเครื่อง Nuc11 หลังจากเสียบ USB เราสามารถเช็คอุปกรณ์ได้ดังนี้
~~~
$ lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0  63.3M  1 loop /snap/core20/1822
loop1                       7:1    0 111.9M  1 loop /snap/lxd/24322
loop2                       7:2    0  49.8M  1 loop /snap/snapd/18357
sda                         8:0    1  57.3G  0 disk
sdb                         8:16   1  14.4G  0 disk
├─sdb1                      8:17   1   1.8G  0 part
├─sdb2                      8:18   1   4.9M  0 part
├─sdb3                      8:19   1   300K  0 part
└─sdb4                      8:20   1  12.6G  0 part
nvme0n1                   259:0    0   1.8T  0 disk
├─nvme0n1p1               259:1    0     1G  0 part /boot/efi
├─nvme0n1p2               259:2    0     2G  0 part /boot
└─nvme0n1p3               259:3    0   1.8T  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0   100G  0 lvm  /

~~~
จากคำสั่งด้านบนสามารถอธิบายอุปกรณ์ต่าง ๆ ได้ดังนี้
 - nvme0n1 - เป็น OS Disk ห้ามไปยุ่งกับมัน ใช้สำหรับเก็บ OS และ Software ต่าง ๆ ทั้งระบบ
    - nvme0n1p1, nvme0n1p2, nvme0n1p3 - อุปกรณ์ทั้ง 3 เป็น partition ย่อยที่อยู่บน nvme0n1 ห้ามยุ่งเช่นกัน
 - sda - USB Drive ตัวแรก ขนาด 57.3GB ซึ่งเราใช้มันสร้าง zfs กัน
 - sdb - USB Drive ตัวที่สอง ขนาด 14.4GB สำหรับ mirror กับตัวแรก
   - sdb1, sdb2, sdb3, sdb4 - partition ที่อยู่บน sdb แสดงว่า USB Drive ตัวนี้มีข้อมูลอย่างอื่นค้างอยู่ เราจำเป็นต้องลบให้หมด

อีกคำสั่งที่สามารถเช็ค partition ต่าง ๆ ได้ คือ fdisk ดังนี้
~~~
$ sudo fdisk -l
...
Disk /dev/sda: 57.3 GiB, 61530439680 bytes, 120176640 sectors
Disk model: Ultra USB 3.0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdb: 14.44 GiB, 15502147584 bytes, 30277632 sectors
Disk model: DataTraveler 2.0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: C3B09D5C-3F01-457B-AC5E-206E8818BFA0

Device       Start      End  Sectors  Size Type
/dev/sdb1       64  3848587  3848524  1.8G Microsoft basic data
/dev/sdb2  3848588  3858655    10068  4.9M EFI System
/dev/sdb3  3858656  3859255      600  300K Microsoft basic data
/dev/sdb4  3862528 30277568 26415041 12.6G Linux filesystem

~~~
เมื่อเช็คจนแน่ใจแล้ว เราจะเริ่มลบข้อมูลใน sdb เสียก่อนดังนี้
 - เลือก disk ที่ต้องการ
~~~
Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

The device contains 'iso9660' signature and it will be removed by a write command. See fdisk(8) man page and --wipe option for more details.

Command (m for help):
~~~
 - ใช้คำสัง่ d เพื่อลบ partition ทั้งหมด
~~~
Command (m for help): d
Partition number (1-4, default 4): 4

Partition 4 has been deleted.

Command (m for help): d
Partition number (1-3, default 3): 3

Partition 3 has been deleted.

Command (m for help): d
Partition number (1,2, default 2): 2

Partition 2 has been deleted.

Command (m for help): d
Selected partition 1
Partition 1 has been deleted.

Command (m for help):
~~~
 - คำสั่ง w เพื่อยืนยันการลบ partition และออกจาก fdisk

หลังจากลบ partition เสร็จแล้วให้เช็คอีกครั้ง
~~~
$ sudo fdisk -l
...
Disk /dev/sda: 57.3 GiB, 61530439680 bytes, 120176640 sectors
Disk model: Ultra USB 3.0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdb: 14.44 GiB, 15502147584 bytes, 30277632 sectors
Disk model: DataTraveler 2.0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
~~~

แต่เนื่องจาก flash drive ทั้งสองมีขนาดต่างกัน เราจำเป็นต้องทำให้ device เรามีขนาดเท่ากันเสียก่อนจึงจะสร้าง zfs mirror pool บน device ทั้งสองได้ ผมจึงสร้าง partition ขนาด 14GB บน flash drive ทั้งสองเสียก่อน
 - เลือก disk ลูกแรกคือ /dev/sda
~~~
$ sudo fdisk /dev/sda

Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

The device contains 'iso9660' signature and it will be removed by a write command. See fdisk(8) man page and --wipe option for more details.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x0282a869.

Command (m for help):
~~~
 - การสร้าง partition ใหม่ ขนาด 14GB ต้องคำนวณก่อนว่าใช้กี่ sector โดย 1 sector มีขนาด 512 bytes ดังนั้น 14GB = 29360128 Sector แต่ต้องสร้างกำหนด sector เริ่มที่ 2048 ดังนั้นจะจบที่ 29362176 นำค่าทั้งสองไปใช้สร้าง partition
 - ใช้คำสั่ง n เพื่อสร้าง partition
~~~
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-120176639, default 2048): 2048
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-120176639, default 120176639): 29362176

Created a new partition 1 of type 'Linux' and of size 14 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

$
~~~
 - ทำเช่นเดียวกันกับ /dev/sdb
~~~
$ sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition number (1-248, default 1): 1
First sector (64-30277568, default 2048): 2048
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-30277568, default 30277568): 29362176

Created a new partition 1 of type 'Linux filesystem' and of size 14 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

$
~~~
 - เช็ค partition ด้วยคำสัง fdisk
~~~
$ sudo fdisk -l
...
Disk /dev/sda: 57.3 GiB, 61530439680 bytes, 120176640 sectors
Disk model: Ultra USB 3.0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x0282a869

Device     Boot Start      End  Sectors Size Id Type
/dev/sda1        2048 29362176 29360129  14G 83 Linux


Disk /dev/sdb: 14.44 GiB, 15502147584 bytes, 30277632 sectors
Disk model: DataTraveler 2.0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: C3B09D5C-3F01-457B-AC5E-206E8818BFA0

Device     Start      End  Sectors Size Type
/dev/sdb1   2048 29362176 29360129  14G Linux filesystem
$
~~~
เราจะได้ partition /dev/sda1 และ /dev/sdb1 ขนาดเท่ากันที่ 14GB ซึ่งพร้อมที่จะสร้าง zfs mirror pool กันได้เลย

## การติดตั้ง ZFS และสร้าง ZPOOL ลงบน Ubuntu
  
 - เริ่มแรกติดตั้ง zfs บน ubuntu เสียก่อน
~~~
$ sudo apt install zfsutils-linux

# Check zpool by following command
$ zpool status
~~~

 - สร้าง encryption key จากค่าสุ่มและบันทึกไว้ที่ /root/.zpoolraw.key ขนาด 32 bytes และแก้ไข permission
~~~
$ sudo su -
$ dd if=/dev/urandom of=/root/.zpoolraw.key bs=32 count=1
$ chmod 400 /root/.zpoolraw.key
~~~

 - สร้าง zpool
~~~
# list physical disks
$ lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0                       7:0    0  63.3M  1 loop /snap/core20/1822
loop1                       7:1    0 111.9M  1 loop /snap/lxd/24322
loop2                       7:2    0  49.8M  1 loop /snap/snapd/18357
loop3                       7:3    0  53.3M  1 loop /snap/snapd/19457
loop4                       7:4    0  63.4M  1 loop /snap/core20/1974
sda                         8:0    1  57.3G  0 disk
└─sda1                      8:1    1    14G  0 part
sdb                         8:16   1  14.4G  0 disk
└─sdb1                      8:17   1    14G  0 part
nvme0n1                   259:0    0   1.8T  0 disk
├─nvme0n1p1               259:1    0     1G  0 part /boot/efi
├─nvme0n1p2               259:2    0     2G  0 part /boot
└─nvme0n1p3               259:3    0   1.8T  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0   100G  0 lvm  /

# get the IDs
$ ls -la /dev/disk/by-partuuid/
total 0
drwxr-xr-x 2 root root 140 Jul 22 05:49 .
drwxr-xr-x 7 root root 140 Jul 22 05:46 ..
lrwxrwxrwx 1 root root  10 Jul 22 05:46 0282a869-01 -> ../../sda1
lrwxrwxrwx 1 root root  15 Jul 22 01:57 7797a70d-9ee2-4b02-94ac-c3eff79c3919 -> ../../nvme0n1p2
lrwxrwxrwx 1 root root  15 Jul 22 01:57 9e101d8c-135d-44ce-a42c-bae7f5ac7dd3 -> ../../nvme0n1p1
lrwxrwxrwx 1 root root  10 Jul 22 05:49 c6ba2aa9-b24e-6943-8e7b-0994780cd8ef -> ../../sdb1
lrwxrwxrwx 1 root root  15 Jul 22 01:57 eeea96f7-fd6c-4bcf-8484-1d2cabd3392e -> ../../nvme0n1p3
~~~
จากด้านบน sda1=/dev/disk/by-partuuid/0282a869-01 และ sdb1=/dev/disk/by-partuuid/c6ba2aa9-b24e-6943-8e7b-0994780cd8ef นำชื่ออุปกรณ์ไปกำหนดค่าตัวแปรดังนี้
~~~
$ DISK1="/dev/disk/by-partuuid/0282a869-01"
$ DISK2="/dev/disk/by-partuuid/c6ba2aa9-b24e-6943-8e7b-0994780cd8ef"
~~~
สาเหตุที่ต้องใช้ชื่ออุปกรณ์เป็น ID แทนที่จะเป็น sda1 หรือ sdb1 เพราะชื่อ sda1 สามารถเปลี่ยนแปลงได้เมื่อมีออุปกรณ์ใหม่เพิ่มในระบบ หรือมีการ reboot เครื่อง ทำให้มีปัญหาในการใช้งานได้ แต่ ID จะคงเดิมเสมอ แม้จะเปลี่ยน USB Port ก็ตาม จึงเหมาะสมใช้งานเป็น ID มากกว่า

สำหนดค่าตัวแปร POOL_NAME เป็นชื่อ zpool ที่ต้องการ
~~~
$ POOL_NAME="lndpool"
~~~
คำสั่งสร้าง zpool
~~~
$ zpool create \
-o cachefile=/etc/zfs/zpool.cache \
-o ashift=12 -d \
-o feature@async_destroy=enabled \
-o feature@bookmarks=enabled \
-o feature@bookmark_v2=enabled \
-o feature@embedded_data=enabled \
-o feature@empty_bpobj=enabled \
-o feature@enabled_txg=enabled \
-o feature@encryption=enabled \
-o feature@extensible_dataset=enabled \
-o feature@filesystem_limits=enabled \
-o feature@hole_birth=enabled \
-o feature@large_blocks=enabled \
-o feature@livelist=enabled \
-o feature@lz4_compress=enabled \
-o feature@spacemap_histogram=enabled \
-o feature@zpool_checkpoint=enabled \
-O acltype=posixacl -O canmount=off -O compression=lz4 \
-O devices=off -O normalization=formD -O relatime=on -O xattr=sa \
-O encryption=on \
-O keyformat=raw -O keylocation=file:///root/.zpoolraw.key \
$POOL_NAME \
mirror $DISK1 $DISK2
~~~

คำสั่งสำหรับ check zpool
~~~
$ zpool status
  pool: lndpool
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
        The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
        the pool may no longer be accessible by software that does not support
        the features. See zpool-features(7) for details.
config:

        NAME                                      STATE     READ WRITE CKSUM
        lndpool                                   ONLINE       0     0     0
          mirror-0                                ONLINE       0     0     0
            0282a869-01                           ONLINE       0     0     0
            c6ba2aa9-b24e-6943-8e7b-0994780cd8ef  ONLINE       0     0     0

errors: No known data errors
$ zpool list
NAME      SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
lndpool  13.5G   724K  13.5G        -         -     0%     0%  1.00x    ONLINE  -
~~~

 - Mount to /data/lnd
~~~
$ sudo -i
# create a dataset named data_lnd (so it can be mounted as /data/lnd)
$ POOL_NAME="lndpool"
$ zfs create $POOL_NAME/data_lnd

# mount a ZFS dataset to /data/lnd
$ zfs set mountpoint=/data/lnd $POOL_NAME/data_lnd
$ zfs load-key -a
$ zfs mount -la


# check
$ zfs list
NAME               USED  AVAIL     REFER  MOUNTPOINT
lndpool            788K  13.1G      184K  /lndpool
lndpool/data_lnd   184K  13.1G      184K  /data/lnd
$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              769M  1.7M  767M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   98G  7.0G   86G   8% /
tmpfs                              3.8G     0  3.8G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/nvme0n1p2                     2.0G  131M  1.7G   8% /boot
/dev/nvme0n1p1                     1.1G  6.1M  1.1G   1% /boot/efi
tmpfs                              769M  4.0K  769M   1% /run/user/1000
lndpool/data_lnd                    14G  256K   14G   1% /data/lnd


# automount with cron
$ cronjob="@reboot  /sbin/zfs load-key -a; /sbin/zfs mount -la"
(
crontab -u root -l
echo "$cronjob"
) | crontab -u root -


# list the active crontab for root
$ crontab -u root -l

~~~
ทดสอบ automount ของ zfs ที่เพิ่งสร้างขึ้นด้วยการ reboot เครื่อง
~~~
$ reboot
# หลังจากนั้น รอให้เครื่อง reboot จนเสร็จแล้วเช็ค filesystem
$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              769M  1.6M  767M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   98G  7.0G   86G   8% /
tmpfs                              3.8G     0  3.8G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/nvme0n1p2                     2.0G  131M  1.7G   8% /boot
/dev/nvme0n1p1                     1.1G  6.1M  1.1G   1% /boot/efi
lndpool/data_lnd                    14G  256K   14G   1% /data/lnd
tmpfs                              769M  4.0K  769M   1% /run/user/1000
~~~ 

 - Backup Encryption Key

เนื่องจากการะบวนการสร้าง zpool มีการใช้การเข้ารหัสเพื่อป้องกันไม่ให้ผู้ไม่ประสงค์ดีถอด disk ไปใช้ได้ แต่ก็จำเป็นต้องสำรองรหัสดังกล่าวไว้ เพื่อใช้กู้ข้อมูลจาก disk บนเครื่องอื่นได้ จึงควรเก็บรหัสไว้อย่างดีนะครับ (รหัสที่แสดงด้านล่างไม่ได้ถูกใช้งานจริง เราไม่ควรแสดงรหัสที่ใช้งานจริงให้ผู้อื่นเห็นเด็ดขาด)
   
~~~
# backup the key
$ sudo -i
$ xxd /root/.zpoolraw.key
00000000: ad7d f5dc 7c6e 6a09 58ec 1107 919a a5d6 .}..|nj)X.......
00000010: 9e19 1d26 89f0 e433 ae6b 1101 9259 9d61 ...&...3.k...Y.a


# recover the key from text
# https://lightning.readthedocs.io/BACKUP.html#hsm-secret
$ cat >.zpoolraw_hex.txt <<HEX
00: 30cc f221 94e1 7f01 cd54 d68c a1ba f124
10: e1f3 1d45 d904 823c 77b7 1e18 fd93 1676
HEX
$ xxd -r .zpoolraw_hex.txt >/root/.zpoolraw.key
$ chmod 0400 .zpoolraw.key
$ srm .zpoolraw_hex.txt
~~~

