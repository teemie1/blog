# การใช้งาน ZFS สำหรับเก็บข้อมูลใน LN Node
อ้างอิงจาก: [Create a ZFS pool to be used as a Raspiblitz data disk](https://github.com/openoms/bitcoin-tutorials/blob/master/zfs/create-raspiblitz-zfs-disk.md)

ZFS เป็นระบบ File System ที่ใช้ใน Linux เพื่อเก็บไฟล์ข้อมูลต่าง ๆ ซึ่งถ้าเปรียบเหมือน FAT32 บน Windows แต่ ZFS มีความเสถียรมากกว่า และมีฟังก์ชันในการ redundant ข้อมูลไฟล์ต่าง ๆ ได้ เพื่อป้องกันความเสียหายจาก Disk ได้

ก่อนเริ่มต้องอธิบายอุปกรณ์ที่จำเป็นต้องใช้เสียก่อน นอกจาก Node ที่ติดตั้ง Ubuntu เรียบร้อยแล้ว เราจำเป็นต้องมี Disk เพิ่มอีกจำนวน 2 ลูก (โดยไม่นับ NVMe Drive ที่อยู่ใน Nuc11 ที่ใช้สำหรับติดตั้ง OS) ซึ่งผมจะใช้เป็น USB Flash Drive  และทั้งคู่ถูกเสียบด้วย USB Port ของเครื่อง Nuc11 หลังจากเสียบ USB เราสามารถเช็คอุปกรณ์ได้ดังนี้
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

