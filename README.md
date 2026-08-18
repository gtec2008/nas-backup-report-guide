# คู่มืออ่านรายงานระบบสำรองข้อมูล

หน้านี้ใช้อธิบายความหมายของสถานะในรายงาน Backup ประจำวัน

## Backup Metrics

### ⏱️ Duration

ระยะเวลารวมตั้งแต่ SQL Backup → NAS → Cleanup

- `s` = วินาที
- `m` = นาที
- `h` = ชั่วโมง

### ✅ SYNCED X→X

ผลการคัดลอกไฟล์จาก Local ไป NAS ซึ่งอ่านจาก Robocopy Log

- `28→28` = สำเร็จครบ 28 ไฟล์
- `28→27` = สำเร็จ 27 จาก 28 ไฟล์

### ✅ VERIFIED

ตรวจสอบไฟล์ `.bak` ด้วย SQL Server `RESTORE VERIFYONLY`

- `✅ VERIFIED` = ไฟล์ผ่านการตรวจสอบ
- `❌ VERIFY FAILED` = ไฟล์ไม่ผ่านการตรวจสอบ

### 📈 ขนาดไฟล์

เปรียบเทียบขนาด FULL Backup กับรอบเวลาเดียวกันของวันก่อน

- `📈` = ขนาดเพิ่มขึ้น
- `📉` = ขนาดลดลง

## Offsite Backblaze B2

ตรวจสอบจาก Completion Marker บน Backblaze B2

- `SUCCESS` = อัปโหลดเสร็จสมบูรณ์ภายในเวลาที่กำหนด
- `DELAYED` = Completion Marker เกิน 1 วัน

การตรวจ Completion Marker ช่วยป้องกันการรายงานสำเร็จ ทั้งที่การอัปโหลดเสร็จเพียงบางส่วน

## NAS Health

อ่านสถานะ NAS และ DX517 ผ่าน SNMPv3

- `SYSTEM NORMAL` = ระบบ NAS ปกติ
- `STORAGE POOL NORMAL` = Storage Pool ปกติ
- `HDD NAS X/Y` = HDD ภายใน NAS ที่ปกติ/ทั้งหมด
- `HDD DX517 X/Y` = HDD ใน Expansion Unit ที่ปกติ/ทั้งหมด
- `HEALTHY (UNALLOCATED)` = HDD ปกติ แต่ยังไม่ได้เพิ่มเข้า Storage Pool หรือ Volume
- `SSD CACHE X/Y` = SSD Cache ที่ปกติ/ทั้งหมด
- `FAN NORMAL` = พัดลมปกติ
- `POWER NORMAL` = แหล่งจ่ายไฟปกติ

### อุณหภูมิ

แสดงอุณหภูมิแยกเป็น:

- NAS
- HDD ภายใน NAS
- HDD ใน DX517
- SSD Cache

เกณฑ์แจ้งเตือน:

- NAS ตั้งแต่ 60°C
- HDD หรือ SSD ตั้งแต่ 55°C

### MONITOR ERROR

ระบบอ่านข้อมูล SNMPv3 ไม่สำเร็จหลัง Retry ครบทุกครั้ง

สิ่งที่ควรตรวจสอบ:

- SNMPv3
- Network
- ภาระการทำงานของ NAS
- PowerShell SNMP Module
- Data Scrubbing หรือ Backup ที่กำลังทำงาน

### POWER FAILED

NAS ตรวจพบความผิดปกติของแหล่งจ่ายไฟ ไม่ได้หมายความว่า NAS ดับทันที

ควรตรวจสอบอะแดปเตอร์ สายไฟ ปลั๊กไฟ และ UPS

## Storage Capacity

พื้นที่ของไดรฟ์ C: และ NAS Volume แสดงเป็น:

```text


Free / Total และเปอร์เซ็นต์พื้นที่ว่าง

ใช้หน่วย GiB:

1 GiB = 1024³ bytes
จำนวนไฟล์ Backup
FULL = 28 ไฟล์ .bak รวม master, msdb และ model
DIFF = 25 ไฟล์ .bak


Version History
Version 2.1.1 — Offsite B2 Secure Environment
Version 2.4.0 — NAS Health Monitoring via SNMPv3
Version 2.5.0 — DX517 Monitoring and SNMPv3 Reliability Improvements
