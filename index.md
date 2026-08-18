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

## 🩺 NAS Health

สุขภาพของ **NAS และ DX517** อ่านผ่าน **SNMPv3**

### สถานะระบบ

- **SYSTEM**
  - `NORMAL` = ระบบทำงานปกติ
  - `FAILED` = ระบบตรวจพบความผิดปกติ

- **STORAGE POOL**
  - สถานะที่อาจแสดง ได้แก่ `NORMAL`, `REPAIRING`, `SYNCING`, `DEGRADED` และ `CRASHED`
  - MIB แสดงสถานะของ Storage Pool โดยไม่ระบุประเภท RAID หรือ SHR

### สถานะอุปกรณ์จัดเก็บข้อมูล

- **HDD NAS X/Y NORMAL**
  - จำนวน HDD ภายใน NAS ที่ปกติ เทียบกับจำนวนทั้งหมด

- **HDD DX517 X/Y HEALTHY**
  - จำนวน HDD ภายใน Expansion Unit DX517 ที่ปกติ เทียบกับจำนวนทั้งหมด

- **HEALTHY (UNALLOCATED)**
  - HDD ทำงานปกติ แต่ยังไม่ได้เพิ่มเข้า Storage Pool หรือ Volume
  - ไม่ถือเป็น Disk Failure
  - ไม่ทำให้สถานะระบบเป็น `CRITICAL`

- **SSD CACHE X/Y NORMAL**
  - จำนวน SSD Cache ที่ปกติ เทียบกับจำนวนทั้งหมด

### 🌡️ อุณหภูมิ

แสดงอุณหภูมิแยกตามกลุ่มอุปกรณ์:

- NAS
- HDD ภายใน NAS
- HDD ใน DX517
- SSD Cache

เกณฑ์แจ้งเตือน:

| อุปกรณ์ | แจ้งเตือนเมื่อ |
|---|---:|
| NAS | ตั้งแต่ 60°C |
| HDD | ตั้งแต่ 55°C |
| SSD Cache | ตั้งแต่ 55°C |

### พัดลมและแหล่งจ่ายไฟ

- **FAN**
  - `NORMAL` = พัดลมทำงานปกติ
  - `FAILED` = ระบบตรวจพบความผิดปกติของพัดลม

- **POWER**
  - `NORMAL` = แหล่งจ่ายไฟปกติ
  - `FAILED` = ระบบตรวจพบความผิดปกติของแหล่งจ่ายไฟ

### MONITOR ERROR

`MONITOR ERROR` หมายถึง ระบบไม่สามารถอ่านข้อมูลผ่าน SNMPv3 ได้ แม้จะ Retry ครบทุกครั้งแล้ว

ควรตรวจสอบ:

- การตั้งค่า SNMPv3
- Network ระหว่างเครื่อง Monitor และ NAS
- ภาระการทำงานของ NAS หรือ `NAS Load`
- PowerShell SNMP Module
- งาน Backup หรือ Data Scrubbing ที่กำลังทำงาน

> **หมายเหตุ:** `MONITOR ERROR` หมายถึงระบบ Monitor อ่านสถานะไม่ได้  
> ไม่ได้ยืนยันว่า NAS, Storage Pool หรือ HDD เสีย

### ⚠️ POWER FAILED

`POWER FAILED` หมายถึง NAS ตรวจพบความผิดปกติของแหล่งจ่ายไฟ แต่ไม่ได้หมายความว่าไฟดับทันที

- NAS อาจยังทำงานได้ หากอะแดปเตอร์หรือแหล่งจ่ายไฟเริ่มผิดปกติ
- NAS อาจยังได้รับไฟจาก UPS
- หากไฟขาดจน NAS ดับ ระบบจะอ่าน SNMP ไม่ได้
- กรณี NAS ดับ รายงานจะเป็น `MONITOR ERROR` หรือ `NAS OFFLINE` ไม่ใช่ `POWER FAILED`

เมื่อพบ `POWER FAILED` ควรรีบตรวจสอบ:

1. อะแดปเตอร์ของ NAS
2. สายไฟและขั้วต่อ
3. ปลั๊กไฟ
4. UPS

พื้นที่ C: และ NAS Volume 1 : Free / Total ใช้หน่วย GiB
(1 GiB = 1024^3 bytes) และแสดงเปอร์เซ็นต์พื้นที่ว่าง

full = 28 bak (Hardcode) รวม master, msdb และ model
diff = 25 bak (Hardcode)

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



Free / Total และเปอร์เซ็นต์พื้นที่ว่าง

ใช้หน่วย GiB:

1 GiB = 1024³ bytes
จำนวนไฟล์ Backup
FULL = 28 ไฟล์ .bak รวม master, msdb และ model
DIFF = 25 ไฟล์ .bak


### Version History

Version : 2.1.1 (06/2026) - Offsite B2 Secure Env
Version : 2.4.0 (25/07/2569) - Add NAS Health and Status Monitoring via SNMPv3
Version : 2.5.0 (18/08/2569) - Add DX517 Disk and Temperature Monitoring | Improve SNMPv3 Timeout and Retry
