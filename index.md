# คู่มืออ่านรายงานระบบสำรองข้อมูล

หน้านี้อธิบายความหมายของตัวชี้วัดและสถานะต่าง ๆ ในรายงาน Backup ประจำวัน

---

## 📊 Backup Metrics

### ⏱️ Duration

ระยะเวลารวมของกระบวนการ:

```text
SQL Backup → NAS → Cleanup
```

หน่วยเวลา:

* `s` = วินาที
* `m` = นาที
* `h` = ชั่วโมง

### ✅ SYNCED X→X

ผลการคัดลอกไฟล์จาก Local ไปยัง NAS โดยอ่านจาก Robocopy Log

ตัวอย่าง:

* `28→28` = คัดลอกสำเร็จครบ 28 ไฟล์
* `28→27` = คัดลอกสำเร็จ 27 จากทั้งหมด 28 ไฟล์

### ✅ VERIFIED

ตรวจสอบความสมบูรณ์ของไฟล์ `.bak` ด้วยคำสั่ง SQL Server:

```sql
RESTORE VERIFYONLY
```

สถานะที่แสดง:

* `✅ VERIFIED` = ไฟล์ผ่านการตรวจสอบ
* `❌ VERIFY FAILED` = ไฟล์ไม่ผ่านการตรวจสอบ

> `RESTORE VERIFYONLY` ตรวจสอบว่า SQL Server สามารถอ่านโครงสร้าง Backup ได้ แต่ไม่สามารถทดแทนการทดลอง Restore ฐานข้อมูลจริงได้

### 📈 แนวโน้มขนาดไฟล์

เปรียบเทียบขนาด FULL Backup กับรอบเวลาเดียวกันของวันก่อน

* `📈 +X MB` = ขนาด Backup เพิ่มขึ้น
* `📉 -X MB` = ขนาด Backup ลดลง

---

## ☁️ Offsite Backblaze B2

ตรวจสอบความสำเร็จจาก Completion Marker บน Backblaze B2

* `SUCCESS` = Completion Marker ได้รับการอัปเดตภายในเวลาที่กำหนด
* `DELAYED` = Completion Marker มีอายุเกิน 1 วัน

การตรวจ Completion Marker ช่วยลดปัญหา False Success ซึ่งอาจเกิดจากการอัปโหลดข้อมูลเพียงบางส่วนแล้วกระบวนการหยุดทำงาน

---

## 🩺 NAS Health

อ่านสถานะของ NAS และ Expansion Unit DX517 ผ่าน SNMPv3

### สถานะระบบ

#### SYSTEM

* `NORMAL` = ระบบ NAS ทำงานปกติ
* `FAILED` = ระบบ NAS ตรวจพบความผิดปกติ

#### STORAGE POOL

สถานะที่อาจแสดง ได้แก่:

* `NORMAL` = Storage Pool ทำงานปกติ
* `REPAIRING` = กำลังซ่อมแซม Storage Pool
* `SYNCING` = กำลังซิงโครไนซ์ข้อมูล
* `DEGRADED` = Storage Pool สูญเสียความทนทานบางส่วน
* `CRASHED` = Storage Pool เกิดความเสียหายร้ายแรง

> Synology MIB แสดงสถานะ Storage Pool โดยไม่ระบุว่าเป็น RAID หรือ SHR ประเภทใด

### สถานะอุปกรณ์จัดเก็บข้อมูล

#### HDD NAS X/Y NORMAL

จำนวน HDD ภายใน NAS ที่ทำงานปกติ เทียบกับจำนวน HDD ภายในทั้งหมด

ตัวอย่าง:

```text
HDD NAS 4/4 NORMAL
```

หมายถึง HDD ภายใน NAS ปกติครบทั้ง 4 ลูก

#### HDD DX517 X/Y HEALTHY

จำนวน HDD ภายใน Expansion Unit DX517 ที่มีสุขภาพปกติ เทียบกับจำนวนทั้งหมด

ตัวอย่าง:

```text
HDD DX517 2/2 HEALTHY (UNALLOCATED)
```

หมายถึง HDD ใน DX517 มีสุขภาพปกติครบทั้ง 2 ลูก แต่ยังไม่ได้ถูกจัดสรรเข้า Storage Pool หรือ Volume

#### HEALTHY (UNALLOCATED)

หมายถึง HDD มีสุขภาพปกติ แต่ยังไม่ได้เพิ่มเข้า:

* Storage Pool
* Volume
* Hot Spare

สถานะนี้ไม่ถือเป็น Disk Failure และไม่ทำให้สถานะระบบเป็น `CRITICAL`

#### SSD CACHE X/Y NORMAL

จำนวน SSD Cache ที่ทำงานปกติ เทียบกับจำนวนทั้งหมด

ตัวอย่าง:

```text
SSD CACHE 2/2 NORMAL
```

หมายถึง SSD Cache ปกติครบทั้ง 2 ลูก

---

## 🌡️ อุณหภูมิ

รายงานแสดงอุณหภูมิแยกตามกลุ่มอุปกรณ์:

* NAS
* HDD ภายใน NAS
* HDD ใน DX517
* SSD Cache

ตัวอย่าง:

```text
TEMP: NAS 49°C
| HDD NAS 46°C/45°C/40°C/42°C
| HDD DX517 38°C/38°C
| SSD CACHE 45°C/46°C
```

### เกณฑ์แจ้งเตือน

| อุปกรณ์   |         ปกติ | เริ่มแจ้งเตือน |
| --------- | -----------: | -------------: |
| NAS       | ต่ำกว่า 60°C |   ตั้งแต่ 60°C |
| HDD       | ต่ำกว่า 55°C |   ตั้งแต่ 55°C |
| SSD Cache | ต่ำกว่า 55°C |   ตั้งแต่ 55°C |

---

## 🌀 FAN

* `FAN NORMAL` = พัดลมทำงานปกติ
* `FAN FAILED` = ระบบตรวจพบความผิดปกติของพัดลม

เมื่อพบ `FAN FAILED` ควรตรวจสอบพัดลม ช่องระบายอากาศ และฝุ่นภายใน NAS โดยเร็ว

---

## ⚡ POWER

* `POWER NORMAL` = แหล่งจ่ายไฟทำงานปกติ
* `POWER FAILED` = NAS ตรวจพบความผิดปกติของแหล่งจ่ายไฟ

### POWER FAILED หมายถึงอะไร

`POWER FAILED` ไม่ได้หมายความว่า NAS ดับทันที

NAS อาจยังทำงานได้ในกรณีต่อไปนี้:

* อะแดปเตอร์หรือแหล่งจ่ายไฟเริ่มผิดปกติ
* สายไฟหรือขั้วต่อมีปัญหา
* NAS ยังได้รับไฟจาก UPS
* ระบบตรวจพบแรงดันไฟผิดปกติ

หากไฟขาดจน NAS ดับ ระบบจะไม่สามารถอ่าน SNMP ได้ และรายงานอาจแสดง:

```text
MONITOR ERROR
```

หรือ:

```text
NAS OFFLINE
```

เมื่อพบ `POWER FAILED` ควรตรวจสอบ:

1. อะแดปเตอร์ของ NAS
2. สายไฟและขั้วต่อ
3. ปลั๊กไฟ
4. UPS

---

## ⚠️ MONITOR ERROR

`MONITOR ERROR` หมายถึงระบบไม่สามารถอ่านข้อมูลผ่าน SNMPv3 ได้ หลังจาก Retry ครบทุกครั้งแล้ว

สาเหตุที่เป็นไปได้:

* การตั้งค่า SNMPv3 ไม่ถูกต้อง
* Network ระหว่างเครื่อง Monitor และ NAS มีปัญหา
* NAS มีภาระการทำงานสูง
* NAS กำลัง Backup หรือ Data Scrubbing
* PowerShell SNMP Module มีปัญหา
* NAS Offline หรือไม่ตอบสนอง

> `MONITOR ERROR` หมายถึงระบบ Monitor อ่านสถานะไม่ได้ ไม่ได้ยืนยันว่า NAS, Storage Pool หรือ HDD เสีย

---

## 💾 Storage Capacity

พื้นที่ว่างของไดรฟ์ C: และ NAS Volume แสดงในรูปแบบ:

```text
Free / Total และเปอร์เซ็นต์พื้นที่ว่าง
```

ตัวอย่าง:

```text
NAS Volume 1: Free 10059 GiB / 21430.08 GiB (47%)
```

ระบบใช้หน่วย GiB:

```text
1 GiB = 1024³ bytes
```

---

## 🗄️ จำนวนไฟล์ Backup

### FULL Backup

```text
28 ไฟล์ .bak
```

รวมฐานข้อมูลระบบ:

* `master`
* `msdb`
* `model`

### DIFF Backup

```text
25 ไฟล์ .bak
```

จำนวนไฟล์ FULL และ DIFF ปัจจุบันกำหนดไว้ในโปรแกรมแบบ Hardcode

---

## 📝 Version History

* **Version 2.1.1 (06/2026)**
  Offsite B2 Secure Environment

* **Version 2.4.0 (25/07/2569)**
  Add NAS Health and Status Monitoring via SNMPv3

* **Version 2.5.0 (18/08/2569)**
  Add DX517 Disk and Temperature Monitoring
  Improve SNMPv3 Timeout and Retry
