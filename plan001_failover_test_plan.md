## Failover Test Plan for Core Lightning with PostgreSQL Replication
ระบบใหม่มี 2 nodes ทำเป็น Core Lightning ใช้งานกับ PostgreSQL Database ที่มี Replication ข้อมูลกันอยู่ตลอด
 - วันศุกร์ ทดสอบ fail over ระหว่าง node1 และ node2 ในขณะที่ยังไม่มี channel
 - วันเสาร์ เปิด channel กับ ⚡LnwLightning⚡(022618bb95c7cd788a4f2d54e638d73212ffce867e1714734b2b65bac5274635ad@159.65.137.62:9735) ขนาด 3000000 sat หลังจากนั้นให้โอนเงิน 10000 sat
 - วันอาทิตย์ หยุด cln และ postgresql บน node1 และเริ่มการทำงานบน node2, ตรวจเช็คสถานะของ channel และโอนเงิน 20000 sat
 - วันจันทร์ ไม่ว่าง นัดกับ Jan เพื่อทำ Nginx Reverse Proxy สำหรับ Start9 ของ BOB Space
 - วันอังคาร หยุด cln และ postgresql บน node2 และย้ายกลับมาทำงานบน node1, ตรวจสอบสถานะ
