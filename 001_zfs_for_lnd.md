# การใช้งาน ZFS สำหรับเก็บข้อมูลใน LN Node
อ้างอิงจาก: [Create a ZFS pool to be used as a Raspiblitz data disk](https://github.com/openoms/bitcoin-tutorials/blob/master/zfs/create-raspiblitz-zfs-disk.md)

ZFS เป็นระบบ File System ที่ใช้ใน Linux เพื่อเก็บไฟล์ข้อมูลต่าง ๆ ซึ่งถ้าเปรียบเหมือน FAT32 บน Windows แต่ ZFS มีความเสถียรมากกว่า และมีฟังก์ชันในการ redundant ข้อมูลไฟล์ต่าง ๆ ได้ เพื่อป้องกันความเสียหายจาก Disk ได้
    ก่อนเริ่มต้องอธิบายอุปกรณ์ที่จำเป็นต้องใช้เสียก่อน นอกจาก Node ที่ติดตั้ง Ubuntu เรียบร้อยแล้ว เราจำเป็นต้องมี Disk เพิ่มอีกจำนวน 2 ลูก (โดยไม่นับ NVMe Drive ที่อยู่ใน Nuc11 ที่ใช้สำหรับติดตั้ง OS) ซึ่งผมจะใช้เป็น USB Flash Drive  และทั้งคู่ถูกเสียบด้วย USB Port ของเครื่อง Nuc11 หลังจากเสียบ USB เราสามารถเช็คอุปกรณ์ได้ดังนี้
