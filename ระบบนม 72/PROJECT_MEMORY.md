# PROJECT_MEMORY.md
## ระบบบริหารจัดการอาหารเสริมนม + ระบบเช็คดื่มนม
โรงเรียนบ้านแม่สลิดหลวงวิทยา

เอกสารนี้เขียนขึ้นเพื่อให้ AI ตัวใหม่ (หรือนักพัฒนาคนใหม่) "ทำงานต่อได้ทันที" โดย**ไม่ต้องเปิดอ่านไฟล์ index.html (5000+ บรรทัด) และ teacher.html (1080+ บรรทัด) ทั้งหมด** อ่านเอกสารนี้ให้ครบก่อนแก้ไขโค้ดทุกครั้ง

---

## 1. ระบบที่สร้างเสร็จแล้ว

ทั้งระบบเป็น **Single-file HTML + JS + CSS** (ไม่มี build step, ไม่มี backend server) จำนวน 2 ไฟล์หลัก:

### 1.1 `index.html` — ระบบหลัก (สำหรับผู้ดูแลระบบ/ผู้รับผิดชอบโครงการอาหารเสริมนม)
ระบบบริหารจัดการนมของโรงเรียนทั้งหมด แบ่งเป็น 2 โมดูลใหญ่:

**โมดูล A: บริหารจัดการนม (อบต. → โรงเรียน → ห้องเรียน)**
- รับนมจาก อบต. (`page-receive`, `page-receive-list`)
- นำเข้าข้อมูลนักเรียนจาก Excel (`page-import-students`)
- รายงานนักเรียน (`page-students-report`)
- จ่ายนมให้ห้องเรียน (`page-distribute`, `page-distribute-list`)
- นมค้างรายสัปดาห์ — กรณีนักเรียนขาดเรียน (`page-stored-milk`)
- จ่ายนมช่วงปิดเทอม (`page-vacation-dist`)
- จ่ายนมย้อนหลัง / บันทึกหนี้นม (`page-backdate-dist`)
- รายงานสรุปการบริหารนม (`page-report`)
- สต็อกนมคงเหลือ (`page-stock`)
- ตั้งค่าระบบ + Firebase (`page-settings`)

**โมดูล B: เช็คดื่มนมรายวัน (Milk Check / "mc-*")**  ✅ ทำงานสมบูรณ์
- ภาพรวมการดื่มนม (`page-mc-home`) — สถิติรวม + สถานะรายห้องวันนี้ + **ตารางสถานะการเช็คของครูแต่ละห้อง (ใหม่)**
- เช็คดื่มนมรายวัน (`page-mc-check`)
- ประวัติการเช็ค (`page-mc-history`)
- สรุปรายงานการดื่มนม — รายวัน/สัปดาห์/15วัน/เดือน/ภาคเรียน (`page-mc-report`)
- พิมพ์รายงาน A4 — รายวัน/สัปดาห์/15วัน/เดือน/ภาคเรียน/ปี, ระดับห้อง/ชั้น/โรงเรียน (`page-mc-print`)

**ระบบ Login**: หน้า login overlay เลือก "ผู้ดูแลระบบ" (รหัสจาก `db.settings.adminPassword`, default `1234`) หรือ "ครูประจำชั้น" (รหัสจาก `db.settings.teacherPassword`, default `1234`) — ถ้า login ด้วยรหัสครูจะ auto-navigate ไปหน้าเช็คดื่มนมของห้องตัวเอง

### 1.2 `teacher.html` — แอปสำหรับครูประจำชั้น (ออนไลน์ ใช้ Firebase) ✅ ทำงานสมบูรณ์
แอปแยกอิสระ ไม่ใช้ localStorage ของ index.html เลย — **อ่าน/เขียนผ่าน Firebase Realtime Database REST API โดยตรง** เพื่อใช้ฐานข้อมูลร่วมกับ index.html

หน้าที่มีให้ครู:
- ภาพรวมการดื่มนม (`pg-ov`)
- เช็คดื่มนมรายวัน (`pg-chk`) — เหมือน index.html ทุกอย่าง (ตาราง ✓/✗, หมายเหตุ, รูปถ่ายสูงสุด 5 รูป, ลายเซ็นครู)
- ประวัติการเช็ค (`pg-hist`)
- สรุปรายงาน — รายวัน/สัปดาห์/เดือน/ภาคเรียน (`pg-sum`)
- พิมพ์รายงาน A4 (`pg-prt`)
- นมค้างรายสัปดาห์ (`pg-abs`)
- จ่ายนมย้อนหลัง (`pg-ret`)
- จ่ายนมช่วงปิดเทอม (`pg-vac`)
- รายงานนักเรียน (`pg-std`)
- ตั้งค่า/Firebase (`pg-cfg`)

Login: ใส่ Firebase URL (มี **ค่า default ตั้งไว้แล้ว** ให้ตรงกับ index.html) → กด "โหลดห้องเรียน" → เลือกห้องของตัวเอง → ใส่รหัสผ่านครู (`teacherPassword` หรือ `adminPassword`) → เข้าระบบ

### 1.3 งานที่เพิ่งทำเสร็จในรอบนี้ (สรุป diff ล่าสุด)
1. **index.html — Sidebar เลื่อนได้**: เปลี่ยน `#sidebar { min-height:100vh }` → `height:100vh; max-height:100vh;` ทำให้ `.nav-section` (ซึ่งมี `flex:1; overflow-y:auto;` อยู่แล้ว) สามารถ scroll ได้จริงเมื่อเมนูยาวเกินจอ
2. **index.html — มุมมองผู้ดูแลดูข้อมูลครูแต่ละห้อง**: เพิ่ม card ใหม่ใน `page-mc-home` ชื่อ "👩‍🏫 สถานะการเช็คดื่มนมของครูประจำชั้น (รายห้อง)" แสดงตารางทุกห้อง: สถานะวันนี้ / % สัปดาห์นี้ / % เดือนนี้ / เช็คล่าสุดวันที่ / ปุ่ม "ดูรายละเอียด" (เปิด modal ประวัติทั้งหมดของห้องนั้น + ลายเซ็น/รูปถ่ายล่าสุด) และปุ่ม "เช็ค" (ไปหน้าเช็คของห้องนั้นทันที)
3. **index.html — ปุ่ม "☁️ ซิงก์ข้อมูลครูล่าสุด"** ใน `page-mc-home` + auto-sync (`mcSyncFromCloud`) ที่ดึงเฉพาะ node `milkApp/mcAttendance` จาก Firebase มา merge เข้า `mcDB.attendance` (cloud ชนะถ้า `savedAt` ใหม่กว่า) ทุกครั้งที่เข้าหน้า mc-home / mc-history / mc-report
4. **index.html — บันทึกแบบ shared DB ทันที**: `mcSaveAttendance()` หลังบันทึก local แล้ว เรียก `fbPatchAttendance(key, rec)` → `PUT milkApp/mcAttendance/{clsId}_{date}.json` ขึ้น Firebase ทันที (ไม่ทับข้อมูลอื่นทั้ง DB)
5. **teacher.html — ใช้ฐานข้อมูลเดียวกันโดยไม่ต้องตั้งค่า**: เพิ่ม `const DEFAULT_FB_URL` (ตรงกับ `db.settings.firebaseUrl` ของ index.html) เป็นค่าเริ่มต้นของ `cfgUrl`, ทำให้ **Database Secret เป็นค่าไม่บังคับ** (ถ้า Firebase Rules เป็น public read/write ก็ใช้งานได้โดยไม่ต้องกรอก), แก้ `fbUrl()` ไม่ใส่ `?auth=` ถ้า key ว่าง

### 1.4 🆕 รอบนี้ — แก้ปัญหา localStorage quota เต็ม + Firebase ใช้โควตาเกิน (ดูหัวข้อ 2.5 สำหรับรายละเอียดเต็ม)

สาเหตุเดิม: รูปถ่าย/ลายเซ็นถูกเก็บเป็น base64 **เต็มความละเอียด** (ไม่มีการย่อ/บีบอัด) ฝังตรงใน record แล้ว `mcSave()` เขียน `mcDB` **ทั้งก้อน** ลง `localStorage` ทุกครั้งที่บันทึก พร้อมทั้ง auto-push **ทั้ง DB** ขึ้น Firebase ทุก 30 วินาที (หรือ debounce 2 วินาทีหลัง `mcSave()`) — ทำให้ localStorage เต็ม (error `exceeded the quota` ที่ key `milkCheckDB_v2`) และ Firebase ใช้โควตาฟรี (Spark plan) เกินจนถูกระงับการใช้งาน

แก้ไข 3 จุด (ทำครบทั้ง 3 ข้อในรอบนี้):
1. **บีบอัดรูปก่อนเก็บ** — ทุกจุดที่อัปโหลดรูปในทั้ง 2 ไฟล์ ผ่าน canvas ย่อขนาด + แปลงเป็น JPEG คุณภาพ 0.7 ก่อนเก็บ (`compressImageFile()` — เพิ่มใหม่ทั้ง 2 ไฟล์ เป็นฟังก์ชันคู่กัน ต้องใช้ค่า `PHOTO_MAX_DIM`/`PHOTO_QUALITY` เดียวกัน)
2. **เปลี่ยนจาก full-DB push เป็น patch เฉพาะ record** — ลบ debounce push (`setTimeout(fbPushData, 2000)`) ออกจาก `mcSave()` ทั้งหมด, `fbPushData()` เปลี่ยนจาก `PUT` เป็น `PATCH` และไม่ส่ง `mcAttendance` อีกต่อไป (กันบั๊ก 8.1 เดิมไปด้วยในตัว), auto-sync เปลี่ยนจาก push ทั้ง DB ทุก 30s → `mcSyncFromCloud` (ดึงอย่างเดียว) ทุก 3 นาที, `saveAtt()` ของ teacher.html (ใช้ `fbSet` เขียน record เดียวอยู่แล้ว) ไม่ต้องแก้
3. **IndexedDB เก็บรูป/ลายเซ็นจริงแทน localStorage** (เฉพาะ index.html — ดูหัวข้อ 2.5.3 ว่าทำไม teacher.html ไม่ต้องทำ) — `mcDB.attendance` ในหน่วยความจำที่แอปใช้งานจริงยังมี `photos`/`signature` เป็น base64 จริงเหมือนเดิมทุกประการ (ไม่กระทบ record shape เลย) แต่ตอนเขียนลง `localStorage` จะถูกแทนด้วย IndexedDB reference string สั้นๆ (`mcSavePersist()`) แล้ว "คลาย" กลับเป็นของจริงตอนโหลดแอป (`mcHydrateAllMedia()`)

ผลลัพธ์: ขนาดข้อมูลต่อการเช็ค 1 ครั้ง (5 รูป + ลายเซ็น) ลดจากหลัก 10-40 MB เหลือประมาณ 0.5-1.5 MB ก่อนเก็บ และยังถูกแยกเก็บรูปจริงไป IndexedDB อีกชั้นสำหรับ localStorage ของผู้ดูแล — ทำให้พอดีกับโควตา localStorage (~5-10MB/origin) และโควตาฟรี Firebase Spark (1GB storage, 10GB/เดือน download) ได้ในการใช้งานจริง

### 1.5 🆕 รอบนี้ — แก้ teacher.html โหลดเข้าระบบช้า + เพิ่ม 3 สรุปรายงาน + พิมพ์ A4

**(a) แก้ปัญหาโหลดเข้าระบบช้า (ค้างที่ "กำลังเข้าสู่ระบบ...")** — ดูรายละเอียดเต็มที่หัวข้อ 8.8 ด้านล่าง สรุปสั้นๆ: เดิม login 1 ครั้งโหลดทั้งก้อน `milkApp` (รวม `mcAttendance` ที่มีรูป/ลายเซ็นทุกห้องทุกวัน) ผ่าน `getApp()` ถึง 3 รอบซ้อนกัน (`loadRooms()`→`doLogin()`→`enterApp()`/`loadFB()`) แก้ด้วยการแคชผลลัพธ์จาก `loadRooms()` ไว้ใน `_cachedApp` (อายุแคช 2 นาที) แล้วให้ `doLogin()` ใช้ของที่แคชไว้ถ้ายังใหม่พอ จากนั้นส่งต่อให้ `enterApp()` ผ่านฟังก์ชันใหม่ `applyFD()` โดยไม่ต้องโหลดซ้ำรอบที่ 3 — ลดจาก 3 ครั้งเหลือ 1 ครั้งต่อการ login ปกติ (เคส session ที่บันทึกไว้แล้ว ซึ่งเดิมโหลด 1 ครั้งอยู่แล้ว ไม่กระทบ) (หมายเหตุ: `startSync()` poll 60 วินาทีเป็นค่าที่ปรับไว้แล้วจากรอบก่อนหน้า ไม่ได้แก้เพิ่มในรอบนี้)

**(b) เพิ่มสรุปรายงาน + พิมพ์ A4 ใหม่ 3 รายงาน** — ทุกรายงานเป็น "การ์ดใหม่" เพิ่มต่อจากการ์ดประวัติเดิมในหน้านั้นๆ (ไม่กระทบโครงสร้าง Firebase หรือฟอร์มบันทึกข้อมูลเดิมเลย เป็นการอ่านข้อมูลที่มีอยู่แล้วมาสรุป/จัดพิมพ์เท่านั้น):
1. **สรุปรายงานนมค้างรายสัปดาห์** (หน้า `pg-abs`) — เลือกเดือน (`#absSumMon`) แล้วสรุปทุกสัปดาห์ของเดือนนั้นจาก `FD.absentMilk` (กรอง `roomId` ตรงห้องตัวเอง, สัปดาห์ใดก็ตามที่ `weekStart`/`weekEnd` อยู่ในเดือนนั้น) เป็นตาราง A4 พร้อมยอดรวมนักเรียน/กล่องนม
2. **สรุปรายงานจ่ายนมย้อนหลัง** (หน้า `pg-ret`) — เลือกปีการศึกษา+ภาคเรียน (`#rtSumYr`/`#rtSumTm`) สรุปจาก `FD.retroMilk` เป็นตาราง A4 พร้อมยอดรวมวัน/กล่อง และแยก "ชำระแล้ว/ค้าง" ตาม `status`
3. **สรุปรายงานจ่ายนมช่วงปิดเทอม** (หน้า `pg-vac`) — เลือกปีการศึกษา+ภาคเรียน (`#vcSumYr`/`#vcSumTm`) สรุปจาก `FD.vacationMilk` เป็นตาราง A4 พร้อมยอดรวมวัน/นักเรียน/กล่อง และจำนวนลายเซ็นผู้ปกครองที่เซ็นรับแล้ว (`signatures`)

ทั้ง 3 รายงานใช้ฟังก์ชันรวม `openA4Print(innerHtml)` ตัวใหม่ (เพิ่มต่อจาก `printA4()` เดิม) เปิดหน้าต่างพิมพ์ A4 แบบเดียวกับที่ `printRetDoc()`/`printBlankMonthSheet()` ใช้อยู่ก่อนแล้ว (ไม่ได้แก้ของเดิม เพิ่มฟังก์ชันใหม่แยกเพื่อลดความเสี่ยง) ตัว preview ในแอปใช้ class `.a4-preview`/`.a4-table`/`.total-row` ที่มีอยู่แล้วในสไตล์ชีตหลัก ส่วนการ์ดตัวกรอง/ปุ่มพิมพ์ใส่ class `np` (ซ่อนตอนพิมพ์ ตามรูปแบบเดิมของหน้า `pg-prt`) ทุกหน้าที่เพิ่มสรุปนี้ ปรับ `goP()`'s `acts` ให้โหลดข้อมูล (`loadAbs`/`loadRetH`/`loadVacH`) เสร็จก่อนแล้วค่อยเรียก render สรุป (`renderAbsSum`/`renderRtSum`/`renderVcSum`) ผ่าน `.then()` กันไม่ให้สรุปขึ้นก่อนข้อมูลโหลดเสร็จ

**ไม่มีการเปลี่ยนโครงสร้าง Firebase ใดๆ ในรอบนี้** (อ่าน `absentMilk`/`retroMilk`/`vacationMilk` ตามที่มีอยู่แล้วเท่านั้น) ดังนั้นหัวข้อ 10 ("ห้ามเปลี่ยนโครงสร้างใดบ้าง") ไม่มีอะไรต้องอัปเดต — บั๊ก 8.2 (ที่ index.html ยังไม่แสดงข้อมูล 3 node นี้) ยังเป็นบั๊กเดิมที่ไม่ได้แก้ในรอบนี้ (อยู่นอกขอบเขตคำขอ ซึ่งระบุเฉพาะ teacher.html)

### 1.6 🆕🆕 รอบล่าสุด — Stock Model v2 (สต็อกย่อยรายห้อง) + แก้ Double-Deduction + Cross-file Bridge Sync
**นี่คือการเปลี่ยนแปลงที่ใหญ่ที่สุดเท่าที่เคยทำกับระบบนี้** เปลี่ยนจาก "สต็อกกลางตัวเดียวที่ทุกอย่างหักรวมกัน" (เสี่ยงหักซ้ำซ้อน) ไปเป็น "สต็อกกลาง (คงไว้) + สต็อกย่อยรายห้อง (ใหม่)" พร้อมเชื่อม `smDB/vdDB/bdDB` ของ index.html เข้ากับ `absentMilk/retroMilk/vacationMilk` ของ teacher.html ที่เคยแยกกันสนิท (ปิดบั๊ก 8.2) **อ่านรายละเอียดเต็มที่หัวข้อ 11 ก่อนแก้โค้ดส่วน stock/นมค้าง/ย้อนหลัง/ปิดเทอมทุกครั้ง**

ที่มา: ผู้ใช้รายงานว่าตัวเลขสต็อกในหน้าต่างๆ ไม่สอดคล้องกัน → วิเคราะห์ data flow ทั้งระบบ → พบว่า "จ่ายนมให้ห้องเรียน" (index.html) กับ "เช็คดื่มนมรายวัน" (teacher.html) ต่างหักจาก `milkApp/stock` ตัวเดียวกันโดยไม่รู้จักกัน (double-deduction จริง) และ "เช็คดื่มนมรายวัน" ฝั่งผู้ดูแลกับฝั่งครูให้ผลต่าง stock ต่างกัน (ครูหัก ผู้ดูแลไม่หัก) → แก้โดยเปลี่ยนสถาปัตยกรรมเป็นสต็อกรายห้อง (รายละเอียดหัวข้อ 11)



---

## 2. Firebase Structure ทั้งหมด

ใช้ **Firebase Realtime Database** (REST API ผ่าน `fetch`, ไม่ใช้ Firebase SDK) ค่า default:
```
firebaseUrl = "https://realtime-database-9fc52-default-rtdb.asia-southeast1.firebasedatabase.app/"
```
เก็บที่ `db.settings.firebaseUrl` (index.html) และ `DEFAULT_FB_URL` (teacher.html) — **ต้องตรงกันเสมอ**

**Database Secret** (`db.settings.firebaseKey`) เป็นค่า "ไม่บังคับ" — ถ้า Firebase Rules ตั้งเป็น
```json
{ "rules": { ".read": true, ".write": true } }
```
ก็ไม่ต้องใส่ secret เลย (URL request จะไม่มี `?auth=`)

### โครงสร้าง node บน Firebase (รากคือ `milkApp`)
```
milkApp/
├── settings/            ← จาก db.settings ทั้งหมด (school, year, perCrate, warnLevel,
│                            adminPassword, teacherPassword, firebaseUrl, firebaseKey)
├── stock                ← number, สต็อกกลางของโรงเรียน (กล่อง) — 🆕 ตั้งแต่ Stock Model v2 (หัวข้อ 11)
│                            กระทบจาก "รับนมจาก อบต." (+) และ "จ่ายนมให้ห้องเรียน"/"จ่ายปิดเทอม" (-) เท่านั้น
│                            🆕 ไม่ถูกหักจาก "เช็คดื่มนมรายวัน" อีกต่อไป (ย้ายไปหัก roomStock ของห้องนั้นแทน)
├── roomStock/           ← 🆕 ใหม่ทั้งหมด (Stock Model v2) object, key = roomId → number (กล่อง)
│                            สต็อกย่อยของห้องนั้น — เพิ่มจาก "จ่ายนมให้ห้องเรียน", ลดจาก "เช็คดื่มนมรายวัน"/
│                            "นมค้างรายสัปดาห์" — ดูหัวข้อ 11 (เป็นแหล่งความจริงเดียว/authoritative source
│                            สำหรับการดำเนินงานระดับห้องเรียนทุกชนิด — ห้ามใช้ milkApp/stock แทน)
├── stockLog/            ← 🆕 ใหม่ (Stock Model v2) push by ทั้ง 2 ไฟล์ — audit ledger ของการเข้า/ออก roomStock
│                            แต่ละ record: { type:'IN'|'OUT', category, roomId, roomName, date, qty,
│                            balanceAfter, note, savedAt } — `category` ที่ใช้จริงตอนนี้: 'PENDING' (นมค้าง)
│                            ส่วนเช็คดื่มนมรายวัน/คืนสต็อกจากการลบประวัติ ยังไม่ได้แท็ก category (ใช้ `note`
│                            บรรยายแทน) — 🆕 ใช้ ledger เดียวนี้สำหรับทุกธุรกรรม ห้ามสร้าง node ใหม่ซ้ำซ้อน
├── receives/            ← array ของ receive records (รับนมจาก อบต.)
├── rooms/                ← array ของห้องเรียน (ใช้ร่วมกันทุกระบบ — ดูหัวข้อ 3.1) 🆕 มี field `stock` เพิ่ม
├── distributes/         ← array ของ distribute records (จ่ายนมให้ห้องเรียน) 🆕 มี `roomStockBefore`/
│                            `roomStockAfter` เพิ่ม (ดูหัวข้อ 3.5)
├── mcAttendance/        ← object, key = "{roomId}_{YYYY-MM-DD}" → attendance record
│                            (ดูหัวข้อ 3.2) — **นี่คือ node ที่ teacher.html อ่าน/เขียนหลัก**
│                            🆕 photos/signature ที่ node นี้เป็น **base64 จริงเสมอ ไม่ใช่ ref**
│                            (ดูหัวข้อ 2.5.3 — ref string ของ IndexedDB ใช้ในเครื่องผู้ดูแลเท่านั้น)
├── absentMilk/          ← นมค้างที่บันทึกผ่าน pg-abs (ครู) **หรือ** หน้านมค้างของ index.html (ผู้ดูแล)
│                            🆕 record ที่ index.html push มา มี `source:'admin'` + `txnId` เพิ่ม (ดูหัวข้อ 11.3)
├── retroMilk/           ← จ่ายนมย้อนหลังที่บันทึกผ่าน pg-ret (ครู) **หรือ** หน้าย้อนหลังของ index.html (ผู้ดูแล)
│                            🆕 record ที่ index.html push มา มี `source:'admin'` + `txnId` เพิ่ม (ดูหัวข้อ 11.3)
├── vacationMilk/        ← จ่ายนมปิดเทอมที่บันทึกผ่าน pg-vac (ครู) **หรือ** หน้าปิดเทอมของ index.html (ผู้ดูแล)
│                            🆕 record ที่ index.html push มา มี `source:'admin'` + `txnId` เพิ่ม (ดูหัวข้อ 11.3)
├── updatedAt            ← ISO timestamp อัปเดตล่าสุด (จาก fbPushData)
└── updatedBy            ← ชื่อโรงเรียน (จาก fbPushData)
```

### วิธี sync ของ index.html (ผู้ดูแลระบบ) — 🆕 เปลี่ยนใหม่ทั้งหมดในรอบนี้ (ดูหัวข้อ 2.5.2)
- **Push เฉพาะ field ของผู้ดูแล** (`fbPushData()`): 🆕 เปลี่ยนจาก `PUT` → **`PATCH milkApp.json`** ด้วย `{ ...db, updatedAt, updatedBy }` — **ไม่ส่ง `mcAttendance` อีกต่อไป** (กันบั๊ก 8.1 เดิมไปด้วยในตัว เพราะ `PATCH` ที่ไม่มี field นี้จะไม่แก้ไข node `mcAttendance` บน cloud เลย)
  - เรียกตอน: กด "อัปโหลดสู่ Cloud (ทั้งหมด)" ใน หน้าตั้งค่าเท่านั้น (manual) — **ไม่มี auto-push อัตโนมัติอีกต่อไป**
  - ใช้สำหรับ sync ข้อมูล **ห้องเรียน/สต็อก/การตั้งค่า/รับ-จ่ายนม** เท่านั้น ไม่เกี่ยวกับการเช็คดื่มนมรายวัน
- **Pull ทั้ง DB** (`fbPullData()`): `GET milkApp.json` แล้ว confirm() ก่อนทับ `db` และ `mcDB.attendance` ทั้งหมด — ใช้แบบ manual เท่านั้น (ไม่เปลี่ยนจากเดิม)
- **ซิงก์เฉพาะ mcAttendance** (`mcSyncFromCloud(showStatus)`): `GET milkApp/mcAttendance.json` แล้ว merge เข้า `mcDB.attendance` แบบ record-by-record โดยเทียบ `savedAt` (cloud ใหม่กว่า → ใช้ cloud) — เรียกอัตโนมัติทุกครั้งที่เข้าหน้า mc-home / mc-history / mc-report **และ 🆕 ทุก 3 นาทีถ้าเปิด Auto-Sync** (เปลี่ยนจาก push ทั้ง DB ทุก 30 วินาทีในเวอร์ชันเดิม)
- **Patch รายการเดียว** (`fbPatchAttendance(key, rec)`): `PUT milkApp/mcAttendance/{key}.json` ด้วย record เดียว (base64 จริง ไม่ใช่ ref) — เรียกหลัง `mcSaveAttendance()` ทุกครั้ง 🆕 ถ้าล้มเหลว (เน็ตหลุด ฯลฯ) จะเข้าคิว retry อัตโนมัติ (ดูหัวข้อ 2.5.4)
- 🆕 **`fbToggleAutoSync()`**: เปลี่ยนจาก "auto-push ทั้ง DB ทุก 30s" → **"auto-pull `mcSyncFromCloud` ทุก 3 นาที + retry คิวที่ค้าง"** ปุ่ม "อัปโหลด/ดึงข้อมูล (ทั้งหมด)" ในหน้าตั้งค่ายังอยู่ แต่เป็น manual action สำหรับข้อมูลที่ไม่ใช่การเช็คดื่มนมเท่านั้น

### วิธี sync ของ teacher.html (ครูประจำชั้น)
- โหลดทั้ง `milkApp` ผ่าน `getApp()` → `fbGet('milkApp')` ตอน login และทุก 🆕 **60 วินาที** (`startSync()` — เดิม 30 วินาที ขยายเพื่อลด bandwidth)
- บันทึกเช็คดื่มนม: `fbSet('milkApp/mcAttendance/{roomId}_{date}', rec)` = `PUT` (ตรงกับ node เดียวกับ `fbPatchAttendance` ของ index.html — **เขียน node เดียวกัน ปลอดภัยทั้งสองทาง**) 🆕 รูปถูกบีบอัดก่อนเก็บใน `rec` แล้ว (ดู 2.5.1) ไม่ต้องเปลี่ยนอะไรที่ `saveAtt()` เอง
- บันทึกนมค้าง/ย้อนหลัง/ปิดเทอม: `fbPush('milkApp/absentMilk' | 'retroMilk' | 'vacationMilk', data)` = `POST` (Firebase auto-generate key) 🆕 รูปในฟอร์มเหล่านี้ก็ถูกบีบอัดก่อนเก็บแล้วเช่นกัน
- 🆕 (แก้แล้ว — เดิมบั๊ก 8.2) **index.html อ่าน `absentMilk`/`retroMilk`/`vacationMilk` แล้ว** ผ่าน `bridgeDB` (ดึงทุกครั้งที่ `mcSyncFromCloud()` ทำงาน) ใช้เช็คซ้ำซ้อนข้ามไฟล์ + แสดงในรายงานของผู้ดูแล — ดูหัวข้อ 11.3 สำหรับรายละเอียดเต็ม (ยังเป็นข้อมูลคนละ "บ้าน" กับ `smDB`/`vdDB`/`bdDB` เดิม แค่อ่านเชื่อมกันแล้ว ไม่ได้ unify schema เป็นชุดเดียว)
- 🆕 ทุก request Firebase (`fbGet/fbSet/fbPatch/fbPush`) มี timeout 10 วินาทีผ่าน `fbFetchWithTimeout()` กันหน้าจอค้างถ้าเน็ตหลุด

### 2.5 🆕 รายละเอียดการแก้ปัญหา Storage/Quota รอบนี้ (สำคัญมาก — อ่านก่อนแก้ไขโค้ดส่วนรูปภาพ/sync)

#### 2.5.1 บีบอัดรูปภาพ — `compressImageFile(file, maxDim, quality)`
ฟังก์ชันคู่กัน เหมือนกันทุกตัวอักษร อยู่ใน **ทั้ง 2 ไฟล์**:
```js
const PHOTO_MAX_DIM = 1000, PHOTO_QUALITY = 0.7;
function compressImageFile(file, maxDim=PHOTO_MAX_DIM, quality=PHOTO_QUALITY) {
  // FileReader → Image → วาดลง <canvas> ที่ย่อขนาดแล้ว → canvas.toDataURL('image/jpeg', quality)
  // ถ้า canvas ใช้ไม่ได้ หรือไฟล์บีบอัดแล้วใหญ่กว่าเดิม → fallback ใช้ dataURL ดิบจาก FileReader (ไม่พัง)
}
```
**ทุกจุดอัปโหลดรูปต้องเรียกผ่านฟังก์ชันนี้** — ปัจจุบันใช้ที่:
- index.html: `handlePhotos()` (ฟอร์มทั่วไป: รับนม/จ่ายนม/นมค้าง/ปิดเทอม/ย้อนหลัง), `mcHandlePhotos()` (เช็คดื่มนม)
- teacher.html: `handlePhotos()` (เช็คดื่มนมหลัก), `absHandlePhotos()`, `rtHandlePhotos()`, `vcHandlePhotos()`

⚠️ ถ้าเพิ่มจุดอัปโหลดรูปใหม่ในอนาคต **ต้องเรียก `compressImageFile()` ก่อนเก็บเสมอ** ห้ามใช้ `FileReader.readAsDataURL()` ตรงๆ อีก (เป็นสาเหตุหลักของปัญหา quota เดิม)

#### 2.5.2 Patch-based sync แทน full-DB push
ดูหัวข้อ "วิธี sync ของ index.html" ด้านบนสำหรับรายละเอียดเต็ม สรุปสั้น: **เขียน (write) ใช้ patch เฉพาะ record/field ที่เปลี่ยน เสมอ ห้ามกลับไปใช้ full-DB `PUT`/auto-push ทุก N วินาทีอีก** เพราะเป็นต้นเหตุให้ Firebase ใช้โควตาเกินและมีความเสี่ยงทับข้อมูลของอีกฝั่ง (บั๊ก 8.1 เดิม)

#### 2.5.3 IndexedDB media offload (เฉพาะ index.html)
**แนวคิดหลัก**: `mcDB.attendance` ในหน่วยความจำ (ตัวแปร `mcDB` ที่แอป index.html ใช้งานจริงระหว่างรัน) และข้อมูลที่ส่งขึ้น Firebase ผ่าน `fbPatchAttendance` **มี `photos`/`signature` เป็น base64 จริงเสมอ ไม่เปลี่ยนรูปแบบจากเดิมแม้แต่นิดเดียว** — เพื่อให้ทุกฟังก์ชันที่อ่าน `r.photos`/`r.signature` (มีหลายสิบจุดทั้งหน้าประวัติ/รายงาน/พิมพ์ A4) **ไม่ต้องแก้ไขอะไรเลย** และเพื่อให้ teacher.html (ซึ่งไม่รู้จัก IndexedDB ของเครื่องผู้ดูแล) อ่านข้อมูลจาก Firebase ได้ปกติทุกประการ

"IndexedDB reference string" (รูปแบบ `"idb:" + attKey` เช่น `"idb:ป.1_2026-06-13"`) ใช้ **เฉพาะตอนเขียนค่าลง `localStorage` ของเครื่องผู้ดูแลเองเท่านั้น** ไม่เคยถูกส่งขึ้น Firebase หรือข้ามไปยัง teacher.html:

| ฟังก์ชัน | หน้าที่ |
|---|---|
| `mcIdbOpen()/mcIdbPut()/mcIdbGet()/mcIdbDelete()` | wrapper พื้นฐานของ IndexedDB (database `milkCheckMediaDB`, object store `media`) — `mcIdbPut` คืนค่า `true/false` บอกผลลัพธ์จริง (🆕 สำคัญ — ดูด้านล่าง) |
| `mcIdbRefFor(attKey)` | คำนวณ ref string จาก attendance key |
| `mcSavePersist()` (🆕 เปลี่ยนเป็น `async`) | เขียน `mcDB.attendance` ลง `localStorage` แบบ "เบา" — ลองออฟโหลด media ดิบไป IndexedDB ก่อน **ถ้าสำเร็จจริง** (เช็คจาก return value ของ `mcIdbPut`) จึงใช้ ref แทน **ถ้าล้มเหลว** (เช่น browser ปิดกั้น IndexedDB ในโหมดส่วนตัว) จะเก็บ base64 จริงไว้ใน localStorage ตามเดิม ไม่ใช้ ref ที่ไม่มีข้อมูลรองรับ |
| `mcHydrateMediaForRead(attKey, rec)` / `mcHydrateAllMedia()` | คลาย ref string กลับเป็น base64 จริงจาก IndexedDB — เรียกตอน `mcLoad()` (ตอนเปิดแอป) เท่านั้น |

**กฎสำคัญที่ต้องรักษาไว้ถ้าแก้ไขส่วนนี้ในอนาคต**:
1. ห้ามส่ง ref string ขึ้น Firebase เด็ดขาด — `mcSaveAttendance()` ต้องส่ง `attRec` ตัวเต็ม (base64 จริง) ให้ `fbPatchAttendance` เสมอ ไม่ใช่ค่าที่ผ่าน `mcSavePersist`/offload แล้ว
2. `mcIdbPut` ต้อง**ยืนยันผลสำเร็จจริง**ก่อนที่ `mcSavePersist` จะตัดสินใจทิ้ง base64 ออกจาก localStorage (ของเดิมที่เคยพลาด: เขียน ref ทันทีแบบ optimistic โดยไม่รอผล ทำให้ถ้า IndexedDB ใช้ไม่ได้ รูปจะหายถาวรเพราะ ref ไม่มีข้อมูลจริงรองรับ)
3. จุดอ่าน `mcDB.attendance[...]` ที่ render เป็น `<img>` ทุกจุด (`mcLoadGrid`, `mcShowTeacherDetail`, print preview ฯลฯ) มี **defensive filter** กรอง ref string (`startsWith('idb:')`) ออกก่อนแสดงผลเสมอ — ป้องกันกรณี hydration ยังไม่เสร็จแล้วผู้ใช้กดเร็วเกินไป (ref string หลุดเข้าไปในสถานะที่กำลังแก้ไข แล้วถูกบันทึกซ้ำขึ้น Firebase กลายเป็นข้อมูลเสีย)
4. **เหตุผลที่ teacher.html ไม่ต้องทำ IndexedDB**: teacher.html ไม่เคยเก็บ `mcAttendance` ใน localStorage เลย (`FD` เป็นแค่ cache ในหน่วยความจำที่หายไปตอนปิดแท็บ, `localStorage` ของ teacher.html มีแค่ `tc_cfg`/`tc_sess` ขนาดเล็กมาก) จึงไม่มีความเสี่ยง localStorage เต็มจากฝั่งนี้ — error เดิมที่เจอ (key `milkCheckDB_v2`) เป็นของ index.html เท่านั้น

#### 2.5.4 Retry queue สำหรับ `fbPatchAttendance` ที่ล้มเหลว
เดิม (ก่อนรอบนี้) ถ้า `fbPatchAttendance` ล้มเหลว จะมี `fbPushData()` แบบ debounce 2 วินาทีช่วยกู้ข้อมูลอยู่เสมอ (เพราะ push ทั้ง DB ซ้ำอยู่ดี) — เมื่อลบ auto-push fallback นั้นออกในรอบนี้ (หัวข้อ 2.5.2) ต้องเพิ่มกลไกใหม่แทน ไม่ให้ข้อมูลที่ "หน้าจอบอกว่าบันทึกสำเร็จ" หายไปจาก Cloud จริงๆ ถ้าเน็ตหลุดตอนกดบันทึก:
- `localStorage` key `milkCheckPendingPatches_v1` — array ของ attendance key ที่ patch ล้มเหลวล่าสุด (`mcGetRetryQueue/mcSetRetryQueue/mcAddToRetryQueue/mcRemoveFromRetryQueue`)
- `mcRetryPendingPatches()` — วนลองส่งทุก key ที่ค้างอยู่ซ้ำ (ใช้ `mcDB.attendance[key]` ปัจจุบันเป็นข้อมูลที่จะส่ง ถ้า key ถูกลบไปแล้วจะเอาออกจากคิวเฉยๆ)
- เรียกที่ 3 จุด: ตอนเปิดแอป (`mcLoad()` เสร็จ), event `window.online`, และในตัว auto-sync interval (ทุก 3 นาที) — ครอบคลุมทั้งกรณี browser แจ้ง online event และกรณีที่ไม่แจ้ง (อุปกรณ์บางรุ่น)
- `fbPatchAttendance` เองก็เพิ่ม/ลบตัวเองจากคิวอัตโนมัติ (สำเร็จ → ลบออก, ล้มเหลว/timeout → เพิ่มเข้าคิว)

#### 2.5.5 Timeout ทุก fetch ไปยัง Firebase
เพิ่ม `fbFetchWithTimeout(url, opts, ms=10000)` (index.html) และ `fbFetchWithTimeout(url, opts, ms=10000)` (teacher.html — implementation แยกกัน แต่ signature เหมือนกัน) ครอบทุกการเรียก Firebase REST ที่เคยเป็น raw `fetch()` ตรงๆ ป้องกันหน้าจอค้างถ้าเน็ตหลุด/ Firebase ตอบช้ามาก — ใช้ `AbortController` + `setTimeout(() => controller.abort(), ms)` ค่า default 10 วินาที (ปรับเป็น 15-20 วินาทีสำหรับ call ที่ payload ใหญ่กว่าปกติ เช่น full-DB push/pull)

---

## 3. ตารางข้อมูลทั้งหมด

### 3.1 `db.rooms[]` — ห้องเรียน (ใช้ร่วมกันทุกระบบ ทั้ง milk-check และระบบจ่ายนม)
```js
{
  id: string,          // genId()
  name: string,        // เช่น "ป.1", "ป.2/1"
  level: string,       // ระดับชั้น (อาจว่าง)
  teacher: string,     // ชื่อครูประจำชั้น
  count: number,       // จำนวนนักเรียน (กรณีไม่มี students[])
  stock: number,       // 🆕 Stock Model v2 (หัวข้อ 11) — สต็อกย่อยของห้องนี้ (กล่อง) ค่าจริงที่ใช้งานคือ
                        // milkApp/roomStock/{id} บน Firebase (ดู `pushRoomStock()`) ค่านี้ในเครื่อง
                        // ผู้ดูแลเป็น local mirror — ติดลบได้ถ้าห้องได้รับจัดสรรไม่พอ (ไม่ clamp ที่ 0)
  students: [
    {
      // จาก import Excel — key เป็นภาษาไทย ขึ้นกับไฟล์ที่นำเข้า อาจมี/ไม่มีบาง key
      'เลขที่': string,
      'รหัส' หรือ 'รหัสประจำตัว': string,
      'เพศ': string,        // "ชาย" / "หญิง" หรือค่าดิบอื่นๆ
      'ชื่อ': string,
      'นามสกุล': string,
      // หรือ 'ชื่อ-นามสกุล': string (ถ้าไม่ได้ split)
      // ฟิลด์อื่นจากไฟล์ Excel เก็บไว้ทั้งหมด (เผื่อใช้)
    }
  ]
}
```
**`mcGetClasses()`** (ทั้ง index.html และ teacher.html มีฟังก์ชันคู่กัน `parseRooms`/`parseStudents`) แปลง `db.rooms` → `classes[]` รูปแบบ normalized:
```js
{ id, name, teacher, students: [{ id, num, name, gender }] }
```
- `id` = `s['รหัส']` หรือ `s['รหัสประจำตัว']` หรือ `roomId + '_s' + index`
- `name` = `s['ชื่อ'] + ' ' + s['นามสกุล']` หรือ `s['ชื่อ-นามสกุล']`
- `gender` = normalize เป็น `'ชาย'` / `'หญิง'` / `''`

### 3.2 `mcDB.attendance` — บันทึกเช็คดื่มนม (key = `{roomId}_{YYYY-MM-DD}`)
**นี่คือตารางหลักของระบบเช็คดื่มนม ใช้ร่วมกันทั้ง index.html (`mcDB.attendance`, localStorage key `milkCheckDB_v2`) และ teacher.html (อ่านจาก `FD.mcAttendance`)**
```js
mcDB.attendance["ป.1_2026-06-13"] = {
  clsId: "ป.1",
  date: "2026-06-13",
  year: "2569",
  term: "1",
  teacher: "นางสมศรี ...",
  data:  { "<studentId>": "present" | "absent", ... },
  notes: { "<studentId>": "หมายเหตุ", ... },   // เฉพาะคนที่มีหมายเหตุ
  photos: ["data:image/...base64...", ...],     // สูงสุด 5 รูป — 🆕 ผ่าน compressImageFile() แล้ว
  signature: "data:image/png;base64,...",       // ลายเซ็นครู (canvas toDataURL)
  savedAt: "2026-06-13T08:30:00.000Z"            // ISO — ใช้เทียบเวลาตอน sync
}
```
> นักเรียนที่ไม่มีใน `data` = ยังไม่ได้กดสถานะ (ไม่ใช่ present หรือ absent)

> 🆕 **สำคัญ**: shape ข้างบนคือ shape ที่ใช้ใน **หน่วยความจำ (`mcDB` ที่แอป index.html รันอยู่จริง)** และที่ส่งขึ้น **Firebase** เสมอ — `photos`/`signature` เป็น base64 จริง 100% ไม่เปลี่ยนแปลง ส่วนที่เขียนจริงลง **`localStorage`** ของเครื่องผู้ดูแล (`mcSavePersist()`) อาจแทนค่า `photos`/`signature` ด้วย IndexedDB reference string (เช่น `"idb:ป.1_2026-06-13"`) ถ้าออฟโหลดสำเร็จ — เป็นรายละเอียดภายในของ index.html เท่านั้น ดูหัวข้อ 2.5.3

### 3.3 `db.settings`
```js
{
  school: string,            // ชื่อโรงเรียน
  year: string,               // ปีการศึกษา เช่น "2568"
  perCrate: number,            // จำนวนกล่องนม/หีบ (default 36)
  warnLevel: number,           // ระดับเตือนสต็อกต่ำ
  adminPassword: string,       // default "1234"
  teacherPassword: string,     // default "1234"
  firebaseUrl: string,
  firebaseKey: string           // ไม่บังคับ
}
```

### 3.4 `db.receives[]` — รับนมจาก อบต.
```js
{ id, date, crates, extra, total, year, note, photos: [], sigReceiver, sigSender, createdAt }
```

### 3.5 `db.distributes[]` — จ่ายนมให้ห้องเรียน
```js
{ id, date, roomId, roomName, students, days, total, crates, boxes,
  stockBefore, stockAfter,           // สต็อกกลางของโรงเรียน ก่อน/หลัง (ไม่เปลี่ยน)
  roomStockBefore, roomStockAfter,   // 🆕 Stock Model v2 — สต็อกย่อยของห้องนี้ ก่อน/หลัง
  year, note, photos: [], sigTeacher, sigSender, createdAt }
```

### 3.6 `smDB` (localStorage `storedMilkDB_v1`) — นมค้างรายสัปดาห์ (index.html เท่านั้น)
```js
{
  stored: [],
  dispensed: [
    { id, classId, className, studentId, studentName, absentDate, weekStart, weekEnd,
      dispenseDate, boxes, sig, receiverName, photos: [], createdAt }
  ]
}
```
🆕 ตั้งแต่ Stock Model v2: `saveStoredMilkDispense()` หักจาก `room.stock`/`milkApp/roomStock/{roomId}` (ไม่ใช่ `db.stock`/`milkApp/stock` กลางอีกต่อไป) **และ** push record รูปแบบเดียวกับ `absentMilk` ของ teacher.html ขึ้น `milkApp/absentMilk` ด้วย (แท็ก `source:'admin'`, `txnId`) — ดูหัวข้อ 11.3

### 3.7 `vdDB` (localStorage `vacationDistDB_v1`) — จ่ายนมปิดเทอม (index.html เท่านั้น)
```js
{ records: [ { id, type:'vacation', year, term, date, classId, className, students, days, total,
  note, photos: [], signatures: [{studentId, studentName, sig, receiverName}], createdAt,
  txnId,                 // 🆕 unique transaction id (กันบันทึกซ้ำข้ามไฟล์)
  vacationMilkId? } ] }  // 🆕 push-id บน milkApp/vacationMilk ถ้า bridge sync สำเร็จ (ดูหัวข้อ 11.3)
```
⚠️ **ไม่กระทบ `room.stock`** เลย (ทั้งฝั่ง index.html และ teacher.html) — หักจาก `db.stock`/`milkApp/stock` กลางตรงๆ เหมือนเดิม เพราะเป็นการส่งมอบนมตรงให้ครอบครัวช่วงปิดเทอม ไม่ผ่านขั้นตอนเช็คดื่มนมรายวันที่ใช้ room.stock เลย (ไม่มีความเสี่ยง double-deduction กับ room.stock)

### 3.8 `bdDB` (localStorage `backdateDistDB_v1` — ตรวจชื่อจริงใน `bdLoad()`) — จ่ายนมย้อนหลัง/หนี้นม (index.html เท่านั้น)
```js
{ records: [ { id, type:'backdate', year, term, date, fromDate, toDate, classId, className,
  students, days, total, note, photos: [], status: 'debt'|'paid',
  signatures: [{studentId, studentName, sig, receiverName}], createdAt, paidAt?,
  txnId,               // 🆕 unique transaction id (กันบันทึกซ้ำข้ามไฟล์)
  retroMilkId? } ] }   // 🆕 push-id บน milkApp/retroMilk ถ้า bridge sync สำเร็จ — ใช้ patch สถานะ
                        // "ชำระแล้ว" ให้ตรงกันทั้ง 2 ฝั่งตอน markBdPaid() (ดูหัวข้อ 11.3)
```
⚠️ **ไม่กระทบ stock ใดๆ ทั้งกลางและรายห้อง** โดยตั้งใจ (เป็นหนี้ที่ยังไม่มีนมจริงจาก อบต. ส่งมา) — ห้ามเปลี่ยนพฤติกรรมนี้โดยไม่คุยกับโรงเรียนก่อน เพราะกระทบความหมายทางบัญชี

### 3.9 🆕 `bridgeDB` (in-memory เท่านั้น, index.html — ไม่ persist ลง localStorage)
```js
{
  absentMilk: {},     // mirror ของ milkApp/absentMilk ทั้งก้อน (object keyed by Firebase push-id)
  retroMilk: {},      // mirror ของ milkApp/retroMilk
  vacationMilk: {}    // mirror ของ milkApp/vacationMilk
}
```
รีเฟรชทุกครั้งที่ `mcSyncFromCloud()` ทำงาน (auto ทุก 3 นาที + ตอนเข้าหน้า mc-*/เรียก `recalcRoomStock()`) ใช้เป็น "หน้าต่าง" ให้ index.html มองเห็นสิ่งที่ครูบันทึกผ่าน teacher.html แบบ read-cache เร็วๆ โดยไม่ต้อง fetch ใหม่ทุกจุดที่ต้องใช้ — **เป็น object เก็บด้วย Firebase push-id เป็น key เสมอ (ไม่ใช่ array)** เพราะ Firebase REST คืนข้อมูลแบบนี้อยู่แล้ว และ push-id ที่ได้มาคือ "unique transaction id" สำหรับกันบันทึกซ้ำ/ผูก status update ข้ามไฟล์ในตัว — ถ้าแปลงเป็น array จะเสีย id นี้ไปและทำให้ patch สถานะย้อนหลัง (เช่น `markRetroMilkPaid`) ทำไม่ได้ ดูหัวข้อ 11.3 สำหรับรายละเอียดเต็มของกลไกนี้

> ⚠️ `smDB`, `vdDB`, `bdDB` ยังเป็น **localStorage ของ index.html เป็นหลัก** (ไม่ได้ "ย้าย" ไปอยู่บน Firebase) แค่ **เพิ่มการ push ขึ้น node เดียวกับที่ teacher.html ใช้ด้วย** (bridge แบบ one-way write + cross-check แบบ two-way read ผ่าน `bridgeDB`) — ไม่ใช่การ unify schema เป็นชุดเดียวกัน 100%

---

## 4. Function สำคัญทั้งหมด

### index.html — Core / Navigation
- `loadDB()` / `saveDB()` — localStorage key `milk_school_db` 🆕 `saveDB()` คืนค่า `true`/`false` ตามผลลัพธ์ มี try/catch + `alert()` ถ้า quota เต็ม (ดูบั๊ก 8.0) — ทุก caller ต้องเช็คผลลัพธ์ก่อนถือว่าบันทึกสำเร็จ 🆕 `loadDB()` ใช้ `withDbDefaults()` เติม field ที่ขาดไปก่อนเสมอ (ดูบั๊ก 8.0a)
- 🆕 `defaultSettings()` — คืนค่า default ของ `db.settings` (ใช้ร่วมกันใน `withDbDefaults()` และ `db` เริ่มต้น)
- 🆕 `withDbDefaults(parsed)` — เติม field ที่จำเป็นแต่ขาดไปของ `db` (`settings/stock/receives/rooms/distributes`) ก่อนนำมาทับด้วยข้อมูลจริง ใช้ทุกครั้งที่แทนที่ `db` ทั้งก้อนจากแหล่งข้อมูลภายนอก (`loadDB`, `fbPullData`, `importData`) — ดูบั๊ก 8.0a
- `genId()` — สร้าง unique id
- `showPage(id)` — สลับหน้า, อัปเดต `pageTitles`, เรียก render function ของหน้านั้น
- `init()` — เริ่มระบบ: loadDB, login system, signature pads, smLoad/vdLoad/bdLoad, renderDashboard
- `toggleSidebar()` / `closeSidebar()` — เมนูมือถือ
- 🆕 `smLoad()` / `smSave()` — localStorage key `storedMilkDB_v1` (`smDB`) — เขียนทั้งก้อนทุกครั้ง 🆕 `smSave()` มี try/catch + คืนค่า `true`/`false` แล้ว (ดูบั๊ก 8.0) แต่ยังไม่มีชั้น IndexedDB แยกรูปเหมือน `mcSave`/`mcSavePersist` (ดูหมายเหตุท้ายหัวข้อนี้)
- 🆕 `vdLoad()` / `vdSave()` — localStorage key `vacationDistDB_v1` (`vdDB`) — เขียนทั้งก้อนทุกครั้ง 🆕 `vdSave()` มี try/catch + คืนค่า `true`/`false` แล้ว เช่นเดียวกัน
- 🆕 `bdLoad()` / `bdSave()` — localStorage key (ตรวจจริงผ่าน `BD_KEY` ใน `bdLoad()`, ดู `backdateDistDB_v1`) (`bdDB`) — เขียนทั้งก้อนทุกครั้ง 🆕 `bdSave()` มี try/catch + คืนค่า `true`/`false` แล้ว เช่นเดียวกัน

> ⚠️ **หมายเหตุสำคัญจาก storage audit รอบนี้**: `smSave/vdSave/bdSave` ยัง **ไม่ได้** ผ่านการแก้ปัญหา quota แบบเดียวกับ `mcSave`/`mcSavePersist` (ไม่มีการบีบอัดรูปแยก IndexedDB) เพราะ 3 module นี้ (นมค้าง/ปิดเทอม/ย้อนหลัง) ใช้งานน้อยกว่าการเช็คดื่มนมรายวันมาก (ไม่ใช่ daily-recurring ทุกห้องทุกวัน) และไม่ใช่ต้นเหตุของปัญหาที่รายงานมา แต่ `handlePhotos()` ที่ 3 module นี้ใช้ร่วม **ได้รับการบีบอัดแล้ว** (ดูหัวข้อ 2.5.1) ดังนั้นขนาดข้อมูลลดลงไปมากแล้วในทางปฏิบัติ เหลือแค่ยังไม่มีชั้น IndexedDB แยกเหมือน mc — ถ้าต้องการความครอบคลุม 100% ในอนาคต ควรพิจารณาขยาย pattern เดียวกันมาที่ 3 module นี้ด้วย (เพิ่มเป็น backlog ข้อใหม่ได้ถ้าต้องการ)

### index.html — Milk Check ("mc*")
- `mcLoad()` / `mcSave()` — localStorage key `milkCheckDB_v2` 🆕 `mcLoad()` เรียก `mcHydrateAllMedia()` คลาย ref กลับเป็น base64 หลังโหลด, `mcSave()` เรียก `mcSavePersist()` (async) แทนการ auto-push (ลบออกแล้ว)
- `mcGetClasses()` — แปลง `db.rooms` → classes (ดู 3.1)
- `mcToday()`, `mcThaiDate(d)`, `mcThaiMonthYear(m)`, `mcGetWeekRange(d)` — date helpers (สัปดาห์ = จ-ศ)
- `mcPopulateClassSelects(ids)`, `mcPopulateYearSelects()` — เติม dropdown
- `mcRenderHome()` — stat cards + การเช็คล่าสุด + สถานะรายห้องวันนี้
- `mcQuickCheck(clsId)` — ไปหน้าเช็คของห้องนั้นทันที
- `mcRenderTeacherStatus()` — render ตารางสถานะครูทุกห้อง (today/week%/month%/last date) ใน `#mc-teacher-status-wrap`
- `mcShowTeacherDetail(clsId)` — เปิด modal `#modal-mc-teacher-detail` แสดงประวัติทั้งหมด + ลายเซ็น/รูปถ่ายล่าสุดของห้อง 🆕 กรอง ref string ออกก่อนแสดงผล (defensive)
- `mcInitCheck()`, `mcLoadGrid()` 🆕 กรอง ref string ออกจาก `mcCurrentPhotos` ก่อนโหลดเข้าสถานะแก้ไข, `mcToggle(sid,status)`, `mcMarkAll(status)`, `mcClearAll()`
- `mcSaveAttendance()` — 🆕 เป็น `async`: บันทึก record (base64 จริง) ลง `mcDB.attendance`, `await mcSavePersist()`, `fbPatchAttendance(key, attRec)` (ส่ง record เต็ม ไม่ใช่ light/ref), แสดง toast, กลับหน้า mc-home
- `mcHandlePhotos/mcRenderPhotoGrid/mcRemovePhoto` — รูปถ่าย (สูงสุด 5) 🆕 `mcHandlePhotos` เรียก `compressImageFile()` ก่อนเก็บ
- `mcInitSignature()/mcClearSig()` — canvas ลายเซ็น (`#mc-sig-canvas`)
- `mcRenderHistory()`, `mcClearHistFilters()`
- `mcShowTab/mcBuildSummaryTable` — ใช้ทำตารางสรุปทุกแท็บรายงาน
- `mcRenderBiweeklyReport/mcRenderDailyReport/mcRenderWeeklyReport/mcRenderMonthlyReport/mcRenderTermReport`
- `mcUpdatePrintPreview/mcBuildGradePrintHtml/mcBuildSchoolPrintHtml/mcPrintA4/_mcDoPrint` — สร้าง A4 preview + เปิดหน้าต่างพิมพ์ 🆕 กรอง ref string ออกจากรูป/ลายเซ็นก่อนแสดงในรายงาน (defensive)

### index.html — 🆕 IndexedDB media offload (เฉพาะ index.html — ดูหัวข้อ 2.5.3 สำหรับรายละเอียดเต็ม)
- `mcIdbOpen()` — เปิด/สร้าง IndexedDB `milkCheckMediaDB` (object store `media`)
- `mcIdbPut(key, value)` — เขียนค่าลง IndexedDB, **คืน `true`/`false` ตามผลจริง** (ไม่ throw)
- `mcIdbGet(key)` — อ่านค่าจาก IndexedDB, คืน `undefined` ถ้าไม่พบ/ผิดพลาด
- `mcIdbDelete(key)` — ลบค่าออกจาก IndexedDB
- `mcIdbRefFor(attKey)` — คำนวณ ref string (`"idb:" + attKey`)
- `mcHydrateMediaForRead(attKey, rec)` / `mcHydrateAllMedia()` — คลาย ref กลับเป็น base64 จริง (เรียกตอน `mcLoad()`)
- `mcSavePersist()` — 🆕 `async` เขียน `mcDB.attendance` ลง localStorage แบบเบา (ref ถ้าออฟโหลดสำเร็จจริง, base64 จริงถ้าไม่สำเร็จ) — ดูกฎสำคัญในหัวข้อ 2.5.3

### index.html — 🆕 Cloud Sync (shared DB)
- `mcSyncFromCloud(showStatus)` — `GET milkApp/mcAttendance.json` → merge เข้า `mcDB.attendance` (เทียบ `savedAt`) 🆕 `await mcSavePersist()` หลัง merge, ใช้ `fbFetchWithTimeout` (15s)
- `fbPatchAttendance(key, rec)` — `PUT milkApp/mcAttendance/{key}.json` (เขียนทันทีหลังบันทึก) 🆕 ใช้ `fbFetchWithTimeout` (10s), เข้า/ออกคิว retry อัตโนมัติตามผลลัพธ์ (ดู 2.5.4)
- 🆕 `mcGetRetryQueue()/mcSetRetryQueue()/mcAddToRetryQueue()/mcRemoveFromRetryQueue()` — จัดการคิว retry ใน localStorage key `milkCheckPendingPatches_v1`
- 🆕 `mcRetryPendingPatches()` — วนลองส่งทุก key ที่ค้างอยู่ในคิวซ้ำ (เรียกตอนเปิดแอป/online event/ทุก 3 นาทีใน auto-sync)

### index.html — Firebase (เดิม) 🆕 เปลี่ยนพฤติกรรม
- `fbGetConfig()` — return `{ url, key }` จาก `db.settings`
- `fbFetchWithTimeout(url, opts, ms=10000)` — 🆕 wrapper รอบ `fetch()` ด้วย `AbortController` timeout
- `fbPushData()` — 🆕 เปลี่ยนจาก `PUT` ทั้ง DB → `PATCH milkApp.json` ด้วย `{ ...db, updatedAt, updatedBy }` **ไม่ส่ง `mcAttendance`** — manual เท่านั้น (ปุ่ม "อัปโหลดสู่ Cloud (ทั้งหมด)")
- `fbPullData()` — pull ทั้ง DB (มี confirm dialog) — ไม่เปลี่ยนพฤติกรรม แค่เพิ่ม timeout
- `fbToggleAutoSync()` — 🆕 เปลี่ยนจาก "auto push ทุก 30s" → "auto `mcSyncFromCloud` + `mcRetryPendingPatches` ทุก 3 นาที"

### index.html — 🆕 Stock Model v2 (Room-level Stock + Bridge Sync) — รายละเอียดเต็มที่หัวข้อ 11
- 🆕 `pushRoomStock(roomId, value)` — `PUT milkApp/roomStock/{roomId}.json` แบบ best-effort (ไม่ throw)
- 🆕 `fbPush(path, data)` / `fbPatch(path, data)` — generic helper เทียบเท่า `fbPush()` ของ teacher.html (เดิม index.html มีแค่ `fbPushData()` เฉพาะ full-db) คืน push-id (ใช้เป็น unique transaction id)
- 🆕 `recalcRoomStock(showReport)` — คำนวณ `room.stock` ทุกห้องใหม่จาก source เสมอ (idempotent): `รวมจ่ายนมให้ห้องเรียนของห้อง − รวมเช็คดื่มนมจริงของห้อง − รวมนมค้างที่เบิกแล้วของห้อง (ทั้ง smDB.dispensed และ bridgeDB.absentMilk แบบ dedup)` — sync `mcSyncFromCloud()` ก่อนคำนวณเสมอ, push ผลขึ้น Cloud ทุกห้อง, แสดงรายงานห้องที่ติดลบ (ขาดดุล) ที่ `#room-stock-report` — รันอัตโนมัติครั้งแรกใน `init()` ถ้า `!db.roomStockMigrated`, กดซ้ำเองได้ที่หน้าตั้งค่า
- 🆕 `getStoredMilkPending()` — เพิ่ม cross-check กับ `bridgeDB.absentMilk` (ไม่ใช่แค่ `smDB.dispensed`) กันแสดงเป็น "ยังไม่จ่าย" ซ้ำถ้าครูจ่ายไปแล้วผ่าน teacher.html
- 🆕 `saveStoredMilkDispense()` — เปลี่ยนเป็น `async`, หักจาก `room.stock` (กลุ่มตาม classId), push `milkApp/roomStock/{roomId}` + `milkApp/stockLog` (category `'PENDING'`) + `milkApp/absentMilk` (รูปแบบเดียวกับ teacher.html, `source:'admin'`, `txnId`) แล้วอัปเดต `bridgeDB.absentMilk` ทันที
- 🆕 `saveDistribute()`/`deleteDist()` — เพิ่ม/ลด `room.stock` แบบสมมาตรกับ `db.stock` กลาง, push `milkApp/roomStock/{roomId}` ทันที (ไม่รอ manual upload)
- 🆕 `saveBackdateDist()` — เปลี่ยนเป็น `async`, push record รูปแบบเดียวกับ `retroMilk` ของ teacher.html ขึ้น `milkApp/retroMilk` ด้วย เก็บ push-id ไว้ที่ `rec.retroMilkId`; `markBdPaid()` patch สถานะ `paid` ไปที่ `milkApp/retroMilk/{retroMilkId}` ด้วย best-effort; เพิ่ม `markRetroMilkPaid(firebaseId)` สำหรับ mark รายการที่ครูบันทึกเอง (ไม่มีคู่ใน `bdDB`); `calcBackdateDist()` เตือนถ้าทับซ้อนกับ `bridgeDB.retroMilk`
- 🆕 `saveVacationDist()` — เปลี่ยนเป็น `async`, push record รูปแบบเดียวกับ `vacationMilk` ของ teacher.html ด้วย (ไม่กระทบ stock ใดๆ เพิ่ม); `renderVacationReport()`/`renderBackdateReport()` รวมรายการที่ครูบันทึก (จาก `bridgeDB`) เข้ามาแสดงด้วยเป็นแถว read-only พร้อม badge "👩‍🏫 ครูประจำชั้น"
- 🆕 `mcSaveAttendance()` — เพิ่ม diff-logic หักสต็อกย่อยของห้อง (เหมือน `saveAtt()` ของ teacher.html ทุกประการ) แก้ asymmetry เดิมที่เช็คโดยผู้ดูแลไม่หักสต็อกอะไรเลย
- 🆕 `mcSyncFromCloud()` — เพิ่มดึง `milkApp/stock` (อัปเดต `db.stock`) และ `milkApp/absentMilk`/`retroMilk`/`vacationMilk` (อัปเดต `bridgeDB`) ทุกครั้งที่ sync นอกจาก `mcAttendance` เดิม
- 🆕 `fbPushData()` — ตัด `stock` ออกจาก payload ด้วย (เพิ่มจาก `mcAttendance` เดิมที่ตัดไปแล้ว) กัน overwrite ค่าที่ครูหักไปแล้วบน Cloud


- `getLoginConfig()`, `populateLoginClassSelect()`, `checkLoginSession()`, `setUserUI()`
- `doLogin()`, `doLogout()`, `applyTeacherMode(classId)`, `initLoginSystem()`
- Session เก็บใน `sessionStorage` key `milkApp_loginSession`

### index.html — โมดูลจ่ายนม (ไม่เกี่ยวกับงานรอบนี้ แต่ใช้ `mcGetClasses()` ร่วม)
- รับนม: `calcReceive/saveReceive/renderReceiveList/deleteReceive/viewReceive/printReceiveDoc`
- นำเข้านักเรียน: `parseSchoolExcel/findHeaderRow/previewExcel/showSheetPreview/confirmImport/cancelImport/addManualRoom`
- รายงานนักเรียน: `renderStudentsReport/deleteRoom/viewRoom/printRoomDoc/printStudentsAll`
- จ่ายนม: `initDistForm/onRoomSelect/calcDistribute/saveDistribute/renderDistList/viewDist/printDistDoc`
- นมค้าง: `getStoredMilkPending/renderStoredMilkPage/openSmSigModal/confirmSmSig/saveStoredMilkDispense/renderSmHistory/printStoredMilkReport`
- ปิดเทอม: `initVacationDistForm/calcVacDist/saveVacationDist/renderVacationReport/printVacSingleReport`
- ย้อนหลัง: `initBackdateDistForm/calcBackdateDist/saveBackdateDist/markBdPaid/renderBackdateReport/printBdSingleReport`
- รวม/สต็อก/dashboard/settings: `renderReport/renderStock/renderDashboard/loadSettings/saveSettings/updateStockBadge`
- ทั่วไป: `exportData/exportForDrive/importData/clearAllData/printDocument/openModal/closeModal/showToast/formatDate/isBlankSig/initSigPad/clearSig/getSigData/handlePhotos/renderPhotoPreviews/removePhoto` 🆕 `handlePhotos` เรียก `compressImageFile()` ก่อนเก็บ (ดู 2.5.1) — ใช้ร่วมกับ `compressImageFile()` ที่นิยามไว้ใกล้กัน

### teacher.html — Core
- `fbUrl(p)`, `fbGet/fbSet/fbPatch/fbPush` — REST helpers (🆕 `fbUrl` ไม่ใส่ `?auth=` ถ้า `DB_KEY` ว่าง, 🆕 ทุกตัวเรียกผ่าน `fbFetchWithTimeout()` ป้องกันค้าง)
- 🆕 `fbFetchWithTimeout(url, opts, ms=10000)` — wrapper `fetch()` + `AbortController` timeout (signature เดียวกับฝั่ง index.html)
- 🆕 `compressImageFile(file, maxDim, quality)` — เหมือนกันทุกตัวอักษรกับ index.html (ดู 2.5.1) ใช้ค่า `PHOTO_MAX_DIM=1000, PHOTO_QUALITY=0.7` เดียวกัน
- `parseRooms(app)/parseStudents(r)` — เหมือน `mcGetClasses()` ของ index.html
- `getAttendance(app)` — `app.mcAttendance || app.attendance || {}`
- `attKey(clsId,date)` — `clsId + '_' + date`
- `init()` — prefill `DEFAULT_FB_URL` ถ้าไม่มี config เดิม
- `loadRooms()`, `doLogin()`, `enterApp()`, `setLbls()`, `loadFB()`, `startSync()` poll **60 วินาที** (เดิม 30s — ลด bandwidth, ปรับไว้จากรอบก่อนหน้า) 🆕 `applyFD(app)` (แยกออกจาก `loadFB()` — ใช้ตั้ง `FD`/`STUDS` จากข้อมูลที่โหลดมาแล้วโดยไม่ต้องยิง fetch ซ้ำ), 🆕 `_cachedApp/_cachedAppAt/_cachedUrl/_cachedKey` (ตัวแปร cache ผลลัพธ์ `getApp()` จาก `loadRooms()` ให้ `doLogin()` ใช้ต่อ อายุแคช 120000ms — ดูหัวข้อ 8.8 เรื่องแก้ login ช้า)
- `goP(name)` — navigation ระหว่าง `pg-*`
- เช็คดื่มนม: `loadGrid/toggle/markAll/clearAll/saveAtt` (เขียน `milkApp/mcAttendance/{key}` ด้วย `fbSet`) — ไม่ต้องแก้ไขเพิ่ม เพราะรูปถูกบีบอัดตั้งแต่ตอนอัปโหลดแล้ว
- รูป/ลายเซ็น: `handlePhotos/renderPhotoGrid/removePhoto/initSig/initAllSigs/clearSig/clrSig2/getSig` 🆕 `handlePhotos` เรียก `compressImageFile()`
- ประวัติ/สรุป/พิมพ์: `renderHist/showTab/buildSumTable/renderSum/initPrt/updatePreview/printA4`, `editHistRecord(date)` (ไปหน้าเช็ค+เติมวันที่เดิม), 🆕 `deleteHistRecord(key,date)` (ลบ record ด้วย `fbSet('milkApp/mcAttendance/{key}', null)` — Firebase จะลบ key นั้นออกจาก JSON ทั้งหมด ไม่ใช่ตั้งเป็น `null` ค้างไว้ — index.html's `mcSyncFromCloud()` ตรวจจับและลบ `mcDB.attendance[key]` + IndexedDB entry ที่เกี่ยวข้องให้อัตโนมัติในรอบ sync ถัดไป ดู 2.5.3/2.5.4), 🆕 `openA4Print(innerHtml)` (ฟังก์ชันรวม เปิดหน้าต่างพิมพ์ A4 ใช้ร่วมกันโดย `printAbsSum/printRtSum/printVcSum` — ไม่ได้แก้ `printA4()`/`printRetDoc()` เดิม)
- นมค้าง/ย้อนหลัง/ปิดเทอม: `loadAbs/saveAbs/loadAbsH`, `calcRt/saveRt/loadRetH`, `updVac/saveVac/loadVacH` 🆕 `absHandlePhotos/rtHandlePhotos/vcHandlePhotos` (3 photo handler ของ 3 module นี้) เรียก `compressImageFile()` เช่นกัน
- 🆕 สรุปรายงาน + พิมพ์ A4 (เพิ่มรอบนี้ — ดูหัวข้อ 1.5b): `absSumRecs(m)/buildAbsSumA4(ent,m)/renderAbsSum()/printAbsSum()` (หน้า `pg-abs`), `rtSumRecs(yr,tm)/buildRtSumA4(ent,yr,tm)/renderRtSum()/printRtSum()` (หน้า `pg-ret`), `vcSumRecs(yr,tm)/buildVcSumA4(ent,yr,tm)/renderVcSum()/printVcSum()` (หน้า `pg-vac`)
- นักเรียน/ตั้งค่า/ออกจากระบบ: `loadStd`, `fillCfg/saveCfg/forceReload`, `doLogout`
- ทั่วไป: `toast`, `setDot`, date helpers (`todStr/thD/thMY/wkRng/dBetween/curY`)

---

## 5. ตัวแปร Global

### index.html
- `db` — object หลักทั้งหมด (`{ settings, stock, receives, rooms, distributes, updatedAt }`), localStorage `milk_school_db`
- `mcDB = { attendance: {} }` — localStorage key คือค่าของ const `MC_KEY = 'milkCheckDB_v2'`
- `smDB = { stored: [], dispensed: [] }` — localStorage key คือค่าของ const `SM_KEY = 'storedMilkDB_v1'`
- `vdDB = { records: [] }` — localStorage key คือค่าของ const `VD_KEY = 'vacationDistDB_v1'`
- `bdDB = { records: [] }` — localStorage key คือค่าของ const `BD_KEY = 'backdateDistDB_v1'`
- `sigPads = {}` — เก็บ context ของ canvas ลายเซ็นต่างๆ
- `mcCurrentPhotos = []`, `mcSigDrawing`, `mcSigCtx` — state หน้าเช็คดื่มนม
- `pageTitles` — map page id → ชื่อหัวข้อ
- `fbAutoSyncInterval` — interval handle ของ auto-sync 🆕 ตอนนี้ทำหน้าที่เก็บ interval ของ `mcSyncFromCloud`+`mcRetryPendingPatches` ทุก 3 นาที (ไม่ใช่ full-DB push ทุก 30s อีกแล้ว)
- `LOGIN_SESSION_KEY = 'milkApp_loginSession'` — sessionStorage
- `photoStore` — global object เก็บรูป preview ของฟอร์มต่างๆ (key = previewId)
- `pendingImport` — array รอ confirm จากการนำเข้า Excel
- 🆕 `PHOTO_MAX_DIM = 1000`, `PHOTO_QUALITY = 0.7` — ค่า default ของ `compressImageFile()` ต้องตรงกับค่าเดียวกันใน teacher.html เสมอ
- 🆕 `MC_IDB_NAME = 'milkCheckMediaDB'`, `MC_IDB_STORE = 'media'` — ชื่อ IndexedDB database/object store
- 🆕 `_mcIdbInstance` — cache การเชื่อมต่อ IndexedDB ที่เปิดแล้ว (`mcIdbOpen()` เปิดครั้งเดียว ใช้ซ้ำ)
- 🆕 `MC_IDB_REF_PREFIX = 'idb:'` — prefix ของ reference string ที่ใช้แทนรูป/ลายเซ็นใน localStorage
- 🆕 `MC_RETRY_QUEUE_KEY = 'milkCheckPendingPatches_v1'` — localStorage key ของคิว retry สำหรับ `fbPatchAttendance` ที่ล้มเหลว

### teacher.html
- `DEFAULT_FB_URL` 🆕 — ค่า Firebase URL เริ่มต้น (ต้องตรงกับ `db.settings.firebaseUrl`)
- `DB_URL`, `DB_KEY` — Firebase config ปัจจุบัน (เก็บใน localStorage key `tc_cfg`)
- `S` — session object `{ room, roomName, teacher, schoolName, isAdmin }` (localStorage `tc_sess`)
- `STUDS` — array นักเรียนของห้องที่ login
- `FD` — cache ของ `milkApp` ทั้งก้อนจาก Firebase
- `TIMER` — interval สำหรับ `startSync()`
- `curPhotos`, `sigCtxMap` — state รูป/ลายเซ็นของหน้าเช็คดื่มนมหลัก (`pg-chk`)
- 🆕 `absPhotos`, `rtPhotos`, `vcPhotos` — state รูปถ่ายแยกของ 3 module (นมค้าง/ย้อนหลัง/ปิดเทอม) คู่กับ `absHandlePhotos/rtHandlePhotos/vcHandlePhotos`
- `LS` — wrapper สำหรับ localStorage (`tc_*` prefix)
- 🆕 `PHOTO_MAX_DIM=1000, PHOTO_QUALITY=0.7` — ต้องตรงกับค่าเดียวกันใน index.html เสมอ (ดู 2.5.1)

---

## 6. UI Pages ทั้งหมด

### index.html (`id="page-*"`, สลับโดย `showPage(id)`)
| id | ชื่อ (pageTitles) | หมวด |
|---|---|---|
| dashboard | ภาพรวมระบบ | หน้าหลัก |
| receive | รับนมจาก อบต. | รับนม |
| receive-list | รายการรับนมทั้งหมด | รับนม |
| import-students | นำเข้าข้อมูลนักเรียน | นักเรียน |
| students-report | รายงานนักเรียน | นักเรียน |
| distribute | จ่ายนมให้ห้องเรียน | จ่ายนม |
| distribute-list | รายการจ่ายนมทั้งหมด | จ่ายนม |
| stored-milk | นมค้างรายสัปดาห์ | จ่ายนม |
| vacation-dist | จ่ายนมช่วงปิดเทอม | จ่ายนม |
| backdate-dist | จ่ายนมย้อนหลัง | จ่ายนม |
| report | สรุปการบริหารจัดการนม | รายงาน |
| stock | สต็อกนมคงเหลือ | รายงาน |
| **mc-home** | ภาพรวมการดื่มนม | เช็คดื่มนม — 🆕 มีตารางสถานะครูทุกห้อง + ปุ่มซิงก์คลาวด์ |
| mc-check | เช็คดื่มนมรายวัน | เช็คดื่มนม |
| mc-history | ประวัติการเช็ค | เช็คดื่มนม — 🆕 sync cloud ก่อนแสดง |
| mc-report | สรุปรายงาน (5 แท็บ: รายวัน/สัปดาห์/15วัน/เดือน/ภาคเรียน) | เช็คดื่มนม — 🆕 sync cloud ก่อนแสดง |
| mc-print | พิมพ์รายงาน A4 | เช็คดื่มนม |
| settings | ตั้งค่าระบบ | ตั้งค่า |

Modals: `modal-receive-detail`, `modal-dist-detail`, `modal-room-detail`, `modal-sm-sig`, `modal-vd-sig`, 🆕 `modal-mc-teacher-detail`

### teacher.html (`id="pg-*"`, สลับโดย `goP(name)`)
| id | ชื่อ |
|---|---|
| ov | ภาพรวมการดื่มนม |
| chk | เช็คดื่มนมรายวัน |
| hist | ประวัติการเช็ค |
| sum | สรุปรายงาน (4 แท็บ: รายวัน/สัปดาห์/เดือน/ภาคเรียน — **ไม่มีแท็บ 15 วัน** ต่างจาก index.html) |
| prt | พิมพ์รายงาน A4 (4 ประเภท — **ไม่มี biweekly/year** ต่างจาก index.html) |
| abs | นมค้างรายสัปดาห์ |
| ret | จ่ายนมย้อนหลัง |
| vac | จ่ายนมช่วงปิดเทอม |
| std | รายงานนักเรียน |
| cfg | ตั้งค่า/Firebase |

---

## 7. สิ่งที่ต้องทำต่อ (Backlog / แนวทางต่อยอด)

1. **realtime sync ที่แท้จริง**: ปัจจุบัน index.html sync แบบ poll (ทุก 3 นาทีถ้าเปิด auto-sync) / manual; teacher.html poll ทุก 🆕 60s — ถ้าต้องการ realtime จริงควรเปลี่ยนมาใช้ Firebase JS SDK + `onValue()` listener แทน REST polling
2. ~~แก้ปัญหา fbPushData ทับข้อมูลทั้ง DB~~ — 🆕 **แก้แล้วในรอบนี้** (ดูหัวข้อ 2.5.2, บั๊ก 8.1 ปิดแล้ว) `fbPushData()` ใช้ `PATCH` และไม่ส่ง `mcAttendance` อีกต่อไป
3. ~~เชื่อม absentMilk/retroMilk/vacationMilk จาก teacher.html เข้ากับ index.html~~ — 🆕 **ทำไปแล้วบางส่วนในรอบ Stock Model v2** (ดูหัวข้อ 11.3, ปิดบั๊ก 8.2 บางส่วน): index.html อ่านเข้า `bridgeDB` แล้ว ใช้เช็คซ้ำซ้อน + แสดงรวมในรายงาน — **ยังไม่ได้ unify schema เป็นชุดเดียวกัน 100%** (`smDB`/`vdDB`/`bdDB` ในเครื่องผู้ดูแล ยังเป็นคนละก้อนกับ `absentMilk`/`retroMilk`/`vacationMilk` บน Firebase แค่ "เชื่อม" กันด้วยการ push ขึ้นทั้ง 2 ที่ + cross-check เท่านั้น) ถ้าต้องการ unify จริง 100% ต้องคุยเรื่อง schema migration เพิ่ม
4. **teacher.html — เพิ่มแท็บ "15 วัน" ในหน้าสรุป/พิมพ์** ให้ตรงกับ index.html (item #4 ของผู้ใช้)
5. **teacher.html — ปีการศึกษา dropdown แบบ dynamic**: ปัจจุบัน hardcode `2567/2568/2569`, ควรคำนวณจาก `app.settings.year ± 1` เหมือน `mcPopulateYearSelects()` ของ index.html
6. **mc-teacher-status ใน index.html**: ปัจจุบันคำนวณ % จาก `Object.values(r.data||{}).length` (= จำนวนคนที่ "กดสถานะแล้ว") เป็นตัวหารของ % รายวัน/สัปดาห์/เดือน ไม่ใช่จำนวนนักเรียนทั้งหมดในห้อง — ถ้าครูเช็คไม่ครบทุกคน % อาจดูสูงเกินจริง ควรพิจารณาเปลี่ยนตัวหารเป็น `total` ของห้องคูณจำนวนวันที่เช็ค
7. **เพิ่ม indicator แสดงว่าข้อมูลกำลังซิงก์ / ซิงก์ล่าสุดเมื่อไร** ใน index.html (teacher.html มี `#syncDot` อยู่แล้ว)
8. **Validation รหัสผ่าน**: เปลี่ยนจาก plaintext compare เป็น hash หากต้องใช้งานจริงอย่างปลอดภัย (ปัจจุบันเก็บ `adminPassword`/`teacherPassword` เป็น plaintext ใน `db.settings` ซึ่งถูก push ขึ้น Firebase แบบ public-readable ได้)
9. 🆕 **`getApp()` ของ teacher.html ยังโหลดทั้ง `milkApp` ทุกครั้ง** (ไม่ได้แยกดึงเฉพาะ field ที่จำเป็น) — ปัจจุบันยอมรับได้เพราะเป็น read ไม่ใช่ write และ payload หลักที่ใหญ่ (รูปภาพ) ถูกบีบอัดแล้ว แต่ถ้าจำนวนห้อง/นักเรียนโตมากในอนาคต ควรพิจารณาแยก fetch เฉพาะ `rooms`/`mcAttendance` ที่จำเป็นต่อหน้าที่เปิดอยู่
10. 🆕 **`fbPushData()`/`fbPullData()` (full-DB) ยังไม่มี retry queue เหมือน `fbPatchAttendance`** — ถ้าผู้ดูแลกด "อัปโหลด/ดึงข้อมูล (ทั้งหมด)" แล้วเน็ตหลุดพอดี จะแค่เห็น toast error ต้องกดใหม่เอง (ยอมรับได้เพราะเป็น manual action ที่ผู้ใช้กดเอง ไม่ใช่ background auto-save เหมือน `fbPatchAttendance`)
11. 🆕 **`mcRetryPendingPatches()` ส่งทีละรายการแบบ sequential (`for...of` + `await`)** — ถ้ามีหลายสิบรายการค้างคิวพร้อมกัน (เช่น ปิดเครื่องไปหลายวัน) จะใช้เวลานานกว่าการส่งแบบ parallel ยอมรับได้ในขนาดการใช้งานปัจจุบัน (ไม่กี่ห้องเรียน) แต่ถ้าโรงเรียนใหญ่มากควรพิจารณาเปลี่ยนเป็น `Promise.all` แบบจำกัด concurrency
12. 🆕 **ขยาย pattern บีบอัด+IndexedDB ไปยัง `smDB`/`vdDB`/`bdDB`** — ปัจจุบัน `smSave/vdSave/bdSave` (นมค้าง/ปิดเทอม/ย้อนหลัง) ยังเขียนทั้งก้อนลง localStorage ตรงๆ โดยไม่มีชั้น IndexedDB แยกรูปเหมือน `mcSave`/`mcSavePersist` แม้ว่ารูปจะถูกบีบอัดแล้ว (ใช้ `handlePhotos()` ร่วมกัน) ความเสี่ยง quota ต่ำกว่า mc มากเพราะใช้งานไม่บ่อยเท่า แต่ถ้าต้องการความครอบคลุม 100% ควรทำให้สอดคล้องกัน
13. 🆕 **index.html ไม่มี UI ลบ attendance record โดยตรง** — มีแค่ teacher.html (`deleteHistRecord`) ที่ลบได้ (ผ่าน `fbSet('milkApp/mcAttendance/{key}', null)`) ฝั่งผู้ดูแลจะเห็นรายการหายไปเฉพาะตอน `mcSyncFromCloud()` รอบถัดไป (ไม่ realtime ทันที ต้องรอ auto-sync ทุก 3 นาที หรือเข้าหน้า mc-home/mc-history/mc-report เพื่อ trigger manual sync) — พฤติกรรมนี้ถูกต้องและปลอดภัย (ไม่ใช่บั๊ก) เพียงแต่ยังไม่มีปุ่มลบฝั่งผู้ดูแลเอง ถ้าต้องการเพิ่ม ควรเรียก `fbSet`-equivalent (PUT ค่า `null`) แล้วลบ `mcDB.attendance[key]` + `mcIdbDelete(mcIdbRefFor(key))` ในเครื่องตัวเองทันที

---

## 8. Bugs ที่ยังไม่แก้

### 8.0a ✅ แก้แล้ว (ยืนยันสาเหตุจริงด้วยการทดสอบ) — `loadDB()`/`fbPullData()`/`importData()` ไม่เติม field ที่ขาดไปก่อนแทนที่ `db` ทั้งก้อน → `Cannot read properties of undefined (reading 'push')`
**อาการที่ผู้ใช้รายงาน**: กดปุ่ม "บันทึก" ในหน้า "จ่ายนมให้ห้องเรียน" แล้วเด้ง error ชัดเจน (หลังเพิ่ม try/catch ในบั๊ก 8.0 แล้ว): `เกิดข้อผิดพลาดในการบันทึก: Cannot read properties of undefined (reading 'push')`

**สาเหตุที่แท้จริง (ยืนยันแล้วด้วยการรันโค้ดจริงจำลองสถานการณ์)**: `saveDistribute()` เรียก `db.distributes.push(rec)` โดยสมมติว่า `db.distributes` เป็น array เสมอ แต่ `loadDB()` เดิมทำ `db = JSON.parse(saved)` แบบ **แทนที่ทั้งก้อนตรงๆ ไม่มีการเติม default field ที่ขาดไป** ถ้าข้อมูลที่เก็บไว้จริงใน `localStorage` (key `milk_school_db`) ไม่มี key `"distributes"` อยู่เลย (เป็นไปได้จากหลายสาเหตุ: ข้อมูลเก่าจากเวอร์ชันก่อนมีฟีเจอร์นี้, นำเข้าไฟล์ JSON ที่ export มาจากเวอร์ชันเก่า, หรือดึงข้อมูลจาก Firebase ที่ครั้งหนึ่งเคยถูก patch แบบไม่ครบทุก field) → `db.distributes` จะเป็น `undefined` → `.push()` throw ทันที

จุดที่มีความเสี่ยงเดียวกัน (แทนที่ `db` ทั้งก้อนจากแหล่งข้อมูลภายนอกโดยไม่เติม default): `loadDB()` (จาก localStorage), `fbPullData()` (จาก Firebase, `db = mainData`), `importData()` (จากไฟล์ JSON ที่ผู้ใช้นำเข้า, `db = imported`) — ทั้ง 3 จุดมีความเสี่ยงเดียวกันหมด ไม่ใช่แค่ `loadDB()` จุดเดียว

**การแก้ไข**: เพิ่มฟังก์ชันกลาง `withDbDefaults(parsed)` ที่เติมค่า default ของทุก field ที่จำเป็น (`settings`, `stock`, `receives`, `rooms`, `distributes`) ก่อนนำมา spread ทับด้วยข้อมูลจริงที่ parse ได้ (`{ ...defaults, ...parsed }` — field ที่ `parsed` มีอยู่แล้วจะใช้ค่าจริง ส่วนที่ขาดไปจะได้ค่า default) ใช้ฟังก์ชันนี้แทนการ assign ตรงๆ ทั้ง 3 จุด (`loadDB`, `fbPullData`, `importData`) นอกจากนี้เพิ่ม **defensive guard ตรงจุดใช้งานจริง** เป็นเซฟตี้เน็ตอีกชั้น (ราคาถูก ไม่กระทบ performance):
```js
if (!db.distributes || !Array.isArray(db.distributes)) { db.distributes = []; }
```
ใส่ไว้ก่อน `.push()` ทุกจุดที่ดันเข้า `db.distributes` ตรงๆ (`saveDistribute()`, vacation-distribute save) และก่อน `.sort()/.reduce()` ใน `renderDistList()` (จุดอ่านก็เสี่ยง error แบบเดียวกัน ถ้า `db.distributes` เป็น undefined หน้านี้จะพังตอน render ด้วย ไม่ใช่แค่ตอนบันทึก)

**ยืนยันด้วยการทดสอบจริง**: จำลองสถานการณ์ด้วย headless browser โดย seed `localStorage` ด้วย `db` ที่ไม่มี key `distributes` เลย (ตรงกับสมมติฐาน root cause) แล้วรันไฟล์เดิม (ก่อนแก้) — **reproduce error ได้ตรงทุกคำ**: `Cannot read properties of undefined (reading 'push')` รันไฟล์ที่แก้แล้วด้วยสถานการณ์เดียวกัน — **บันทึกสำเร็จ ไม่มี error, สต็อกหักถูกต้อง, ข้อมูลถูกบันทึกจริง** ยืนยันว่า root cause ตรงตามที่วิเคราะห์ และการแก้ไขแก้ปัญหาได้จริง 100%

⚠️ **กฎใหม่สำหรับโค้ดในอนาคต**: ทุกจุดที่ "แทนที่ `db` ทั้งก้อน" จากแหล่งข้อมูลภายนอก (localStorage, Firebase, ไฟล์ที่ผู้ใช้นำเข้า) **ต้องผ่าน `withDbDefaults()` เสมอ** ห้าม assign ตรงๆ (`db = parsed`) อีก — ดูหัวข้อ 9 ข้อ 17 (เพิ่มใหม่)

### 8.0 ✅ แก้แล้ว — `saveDB()`/`smSave()`/`vdSave()`/`bdSave()` เขียน localStorage โดยไม่มี try/catch (สาเหตุของ "กดบันทึกแล้วไม่บันทึก")
**อาการที่ผู้ใช้รายงาน**: กดปุ่ม "บันทึก" ในหน้า "จ่ายนมให้ห้องเรียน" (และหน้าอื่นที่ใช้ pattern เดียวกัน เช่น รับนม/นมค้าง/ปิดเทอม/ย้อนหลัง/ตั้งค่า/นำเข้า Excel) แล้วไม่มีอะไรเกิดขึ้น ไม่มี toast, ไม่มี alert, ไม่เปลี่ยนหน้า ดูเหมือนปุ่มไม่ทำงาน

**สาเหตุที่แท้จริง**: `saveDB()` (และ `smSave`/`vdSave`/`bdSave` ที่มี pattern เดียวกัน) เดิมเขียนแบบ
```js
function saveDB() { db.updatedAt = new Date().toISOString(); localStorage.setItem(DB_KEY, JSON.stringify(db)); }
```
**ไม่มี try/catch ล้อม `localStorage.setItem()` เลย** — ถ้า `db` (ซึ่งรวม `db.receives`/`db.distributes` ที่มีรูปถ่ายฝังอยู่) ใกล้/เกิน localStorage quota ของเบราว์เซอร์ การเรียก `setItem()` จะ throw `QuotaExceededError` แบบไม่มีใครจับ — เพราะฟังก์ชันที่เรียก `saveDB()` (เช่น `saveDistribute()`, `saveReceive()`) ไม่มี try/catch ห่ออยู่ด้วย ผลคือ **โค้ดทั้งหมดที่อยู่หลังจุดเรียก `saveDB()` ในฟังก์ชันนั้นจะไม่ถูกรันเลย** (ไม่มี toast, ไม่มี alert, ไม่เปลี่ยนหน้า) — ตรงกับอาการ "กดบันทึกแล้วไม่บันทึก" ทุกประการ ที่ร้ายคือ record ที่ `push()` เข้า array ในหน่วยความจำไปแล้วก่อนเรียก `saveDB()` จะยังอยู่ใน `db` (เช่น `db.distributes`) แต่ไม่ถูกเขียนลง localStorage จริง — ถ้าผู้ใช้ไม่ refresh หน้า อาจดูเหมือนข้อมูลค้างอยู่ (เพราะอยู่ในหน่วยความจำ) แต่พอ refresh จะหายไปทันที

**การแก้ไข**: เปลี่ยน `saveDB()`/`smSave()`/`vdSave()`/`bdSave()` ทั้ง 4 ให้มี try/catch จริง และคืนค่า `true`/`false` ตามผลลัพธ์ พร้อม `alert()` แจ้งผู้ใช้ชัดเจนถ้าล้มเหลว (ไม่ใช่ปล่อยให้ throw เงียบอีกต่อไป) จากนั้นไล่แก้ **ทุกจุดที่เรียกฟังก์ชันเหล่านี้** (13 จุดใน `saveDB()`, รวม `smSave`/`vdSave`/`bdSave` อีก 5 จุด) ให้เช็คผลลัพธ์และ **rollback การเปลี่ยนแปลงในหน่วยความจำ** (เช่น `db.distributes.pop()`, คืนค่า `db.stock` เดิม) ถ้าบันทึกล้มเหลว — ครอบคลุมทุกฟังก์ชัน: `saveDistribute`, `saveReceive`, `deleteReceive`, `deleteRoom`, `deleteDist`, `confirmImport` (นำเข้า Excel), `addManualRoom`, `saveSettings`, `fbPullData`, `importData`, `clearAllData`, `saveStoredMilkDispense`, vacation-distribute save, backdate-distribute save + `markBdPaid`

**ความเชื่อมโยงกับงานแก้ quota รอบก่อน (หัวข้อ 2.5)**: นี่คือบั๊กแบบเดียวกันกับที่เคยพบและแก้ใน `mcSavePersist()` (milk-check) แต่เกิดที่ localStorage key **คนละตัว** (`milk_school_db` ของ `db` หลัก ไม่ใช่ `milkCheckDB_v2` ของ `mcDB`) — ยืนยันว่ารูปแบบบั๊กนี้ (`localStorage.setItem` ไม่มี try/catch) เป็นปัญหาเชิงโครงสร้างที่เคยมีอยู่หลายจุดในไฟล์ ไม่ใช่แค่จุดเดียว ตอนนี้แก้ครบทุกจุดที่เขียน localStorage ในระบบแล้ว (`saveDB`, `smSave`, `vdSave`, `bdSave`, `mcSavePersist`, `mcSetRetryQueue`)

⚠️ **กฎใหม่สำหรับโค้ดในอนาคต**: ทุกฟังก์ชันที่เขียน `localStorage.setItem()` **ต้องมี try/catch และคืนค่า true/false เสมอ** ห้ามเขียนแบบ fire-and-forget ที่ไม่มีการจับ error อีก — ดูหัวข้อ 9 ข้อ 16 (เพิ่มใหม่)

### 8.1 ✅ แก้แล้ว — ความเสี่ยง full-DB push ทับข้อมูล mcAttendance ของครู
~~`fbPushData()` (เรียกผ่าน auto-sync 30s หรือ debounce 2s หลัง `mcSave()` ของผู้ดูแล) จะ `PUT milkApp.json` ด้วยค่า `mcDB.attendance` ฝั่ง local ของผู้ดูแล ณ ขณะนั้น~~ — 🆕 **แก้แล้วในรอบนี้**: ลบ debounce push ออกจาก `mcSave()` ทั้งหมด, เปลี่ยน `fbPushData()` จาก `PUT` → `PATCH` และไม่ส่ง `mcAttendance` อีกต่อไป (ดูหัวข้อ 2.5.2) ตอนนี้ `mcAttendance` แก้ไขผ่าน `fbPatchAttendance()` (เขียนเฉพาะ record) เท่านั้น ไม่มีทางที่ผู้ดูแลจะทับข้อมูลครูทั้ง node แบบเดิมได้อีก

### 8.2 🆕 แก้แล้วบางส่วน — absentMilk/retroMilk/vacationMilk จาก teacher.html ไม่ปรากฏใน index.html
~~teacher.html หน้า "นมค้าง"/"จ่ายนมย้อนหลัง"/"จ่ายนมปิดเทอม" push ข้อมูลขึ้น `milkApp/absentMilk`, `milkApp/retroMilk`, `milkApp/vacationMilk` (Firebase) แต่ index.html ใช้ `smDB`/`vdDB`/`bdDB` ใน localStorage คนละชุด และไม่เคยอ่าน 3 node ข้างต้นจาก Firebase เลย~~ — 🆕 **แก้แล้วบางส่วนในรอบ Stock Model v2** (หัวข้อ 11.3): เพิ่ม `bridgeDB` (ดึงทั้ง 3 node ทุกครั้งที่ `mcSyncFromCloud()`/`loadBridgeDB()` ทำงาน) ใช้:
1. `getStoredMilkPending()` เช็คซ้ำกับ `bridgeDB.absentMilk` ก่อนแสดง "ยังไม่จ่าย" (กันจ่ายซ้ำ)
2. `saveStoredMilkDispense()`/`saveBackdateDist()`/`saveVacationDist()` push record ที่ตัวเองสร้างขึ้น `milkApp/absentMilk`/`retroMilk`/`vacationMilk` ด้วย (`source:'admin'`, `txnId`)
3. `renderBackdateReport()`/`renderVacationReport()` รวมรายการที่ครูบันทึก (จาก `bridgeDB`) มาแสดงด้วย พร้อม badge ระบุที่มา
4. `calcBackdateDist()` เตือนถ้าทับซ้อนช่วงวันที่กับ `bridgeDB.retroMilk`

**ยังไม่แก้/ข้อจำกัดที่เหลือ**: `smDB`/`vdDB`/`bdDB` ในเครื่องผู้ดูแล **ยังเป็นคนละชุดข้อมูล** กับ `absentMilk`/`retroMilk`/`vacationMilk` บน Firebase (แค่ "bridge" แบบ push-ทั้งคู่ + cross-check เท่านั้น ไม่ใช่ unify schema เดียวกัน 100%) — ถ้าครูแก้ไข/ลบรายการที่บันทึกไว้ผ่าน teacher.html (เช่น `pg-abs` มีปุ่มลบ) ฝั่ง index.html's `smDB`/`bdDB`/`vdDB` (ถ้า admin เคยสร้างคู่ขนานไว้) จะไม่รู้ด้วย เพราะยังไม่มีกลไก sync "ลบ" ข้ามไฟล์ (มีแค่ sync ทิศทาง "สร้างใหม่/อัปเดตสถานะ" เท่านั้น)

### 8.9 🆕 ✅ แก้แล้ว — Double-deduction ระหว่าง "จ่ายนมให้ห้องเรียน" กับ "เช็คดื่มนมรายวัน" + asymmetry ผู้ดูแล/ครู
รายละเอียดเต็มที่หัวข้อ 11.1-11.2 สรุปสั้นๆ: เดิม `saveDistribute()` (index.html) และ `saveAtt()` (teacher.html) ต่างหักจาก `milkApp/stock` ตัวเดียวกันโดยไม่รู้จักกัน (หักซ้ำซ้อนจริงถ้าโรงเรียนใช้ทั้ง 2 ฟีเจอร์) และ `mcSaveAttendance()` (index.html, เช็คดื่มนมฝั่งผู้ดูแล) ไม่หักสต็อกอะไรเลย ขณะที่ `saveAtt()` (teacher.html, ฟีเจอร์เดียวกัน) หัก — แก้โดยเปลี่ยนเป็นสถาปัตยกรรมสต็อกรายห้อง (`room.stock`/`milkApp/roomStock`) ดูหัวข้อ 11 สำหรับรายละเอียดทั้งหมด

### 8.10 🆕 (ยังไม่แก้ — ต้องตรวจข้อมูลจริงเพิ่ม) ส่วนต่าง `db.stock` กับ ledger จริง ~2,889 กล่อง ที่พบในข้อมูลตัวอย่างของผู้ใช้
ก่อนเริ่ม Stock Model v2 พบว่า `db.stock` ของโรงเรียน (ตามภาพที่แนบมา) สูงกว่าผลรวม "รับทั้งหมด − จ่ายทั้งหมด" จาก ledger จริงอยู่ ~2,889 กล่อง ซึ่งไม่สามารถอธิบายได้จากบั๊กใดที่เจอ (ทุกบั๊กที่เจอทำให้ stock **ต่ำกว่า** ความจริง ไม่ใช่สูงกว่า) — ยังไม่ทราบสาเหตุที่แน่ชัด 100% (อาจมาจากการนำเข้า JSON ที่ตั้งค่า stock ตรงๆ, การ pull จาก Cloud ที่มีข้อมูลทดสอบ/ก่อนเปลี่ยนระบบ, หรือ receive/distribute record ที่ถูกลบในอดีตแบบ rollback ไม่สมบูรณ์) **ต้องขอ export JSON จริงจากโรงเรียนมาตรวจเพิ่มจึงจะสรุปได้** — `recalcRoomStock()` ของ Stock Model v2 คำนวณ `room.stock` ใหม่จาก ledger ของแต่ละห้องเสมอ (ไม่อิงค่าเดิมที่อาจเพี้ยน) แต่ `db.stock`/`milkApp/stock` (สต็อกกลาง) เองยังเป็นค่าสะสมแบบเดิม ไม่ได้ recalculate รากของปัญหานี้

### 8.11 🆕 (ข้อจำกัดสำคัญ — ต้องทดสอบกับระบบจริงก่อนใช้งาน) Stock Model v2 ยังไม่ผ่านการทดสอบกับ Firebase/ข้อมูลจริงของโรงเรียน
การพัฒนา Stock Model v2 + bridgeDB ทั้งหมด (หัวข้อ 11) ตรวจสอบด้วย `node --check` (syntax ผ่านทุก script block) และจำลอง logic ด้วยข้อมูลสมมติใน Node.js (รวม 11 เคสทดสอบ — ดูหัวข้อ 11.5) **ไม่ได้รันกับ Firebase Realtime Database จริงหรือข้อมูล localStorage จริงของโรงเรียนเลย** (ไม่มีสิทธิ์เข้าถึง) — ก่อนใช้งานจริง ผู้ดูแลระบบควร: (1) อัปโหลดไฟล์ไปแทนที่ของเดิม (2) กดปุ่ม "คำนวณสต็อกย่อยรายห้องใหม่จากประวัติ" ที่หน้าตั้งค่า ดูรายงานห้องที่ขาดดุล (3) ทดสอบเช็คดื่มนม 1 ห้อง 1 วันจริง เทียบเลขทั้ง 2 ไฟล์ให้ตรงกันก่อนใช้งานเต็มรูปแบบ

### 8.3 `debugGender()` เป็นฟังก์ชัน debug ที่หลงเหลือ
ฟังก์ชัน `debugGender()` ใช้ `alert()` แสดงค่าเพศดิบทั้งหมดในระบบ — ไม่มีปุ่มเรียกใช้ในหน้าใดแล้ว (legacy debug tool) ปลอดภัยที่จะลบทิ้งได้

### 8.4 teacher.html `chkYr`/`sumYr`/`prYr`/`rtYr`/`vcYr` ปีการศึกษา hardcode
ค่า dropdown ปีการศึกษาใน teacher.html เป็น `2569/2568/2567` ตายตัว ไม่ผูกกับ `app.settings.year` เหมือน index.html — ถ้าเปลี่ยนปีการศึกษาในระบบหลัก ต้องไปแก้ teacher.html ด้วยมือ

### 8.5 `showTab` ใน teacher.html มี mapping แปลกๆ
```js
function showTab(id,btn){... const map={t_daily:'daily',t_weekly:'weekly',t_monthly:'monthly',t_term:'term'};renderSum(map[id.replace('-','_')]||'daily');}
```
`id` ที่ส่งมาจริงคือ `t-daily` (มี `-`) → `replace('-','_')` ได้ `t_daily` ตรงกับ map ได้พอดี — ใช้งานได้ แต่เขียนแบบ fragile ถ้าเปลี่ยนชื่อ id ในอนาคตต้องแก้ map คู่กันด้วย

### 8.6 🆕 (Known limitation, ไม่ใช่บั๊ก) `mcSavePersist()` ยังเขียน localStorage ตรงๆ ไม่มี chunking
ถ้าจำนวน record สะสมในประวัติทั้งหมด (ไม่ใช่แค่รูป) โตมากจนแม้แต่ข้อมูล "เบา" (ref string ล้วน ไม่มี base64) ยังเกิน localStorage quota — `mcSavePersist()` จะ `alert()` เตือนผู้ใช้ (ดูหัวข้อ 2.5) แต่ไม่มีกลไก auto-archive ข้อมูลเก่าออกจาก `mcDB.attendance` โดยอัตโนมัติ ในทางปฏิบัติไม่ควรเกิดขึ้นในอายุการใช้งานปกติของระบบ (ข้อมูล metadata ต่อ record เล็กมากเมื่อไม่มีรูป/ลายเซ็นฝังอยู่) แต่ถ้าสะสมหลายปีการศึกษาในเครื่องเดียวควรพิจารณาเพิ่มฟีเจอร์ export+clear ข้อมูลเก่า

### 8.7 🆕 (Known limitation) `mcRetryPendingPatches()` ไม่มี exponential backoff
ถ้า Firebase ปิดใช้งานจริงเพราะเกินโควตา (ตามรูปที่แนบในการสนทนาก่อนหน้า — "You have gone over your usage limits") `mcRetryPendingPatches()` ที่ถูกเรียกทุก 3 นาทีจาก auto-sync จะพยายามซ้ำตลอด (และยังนับเป็น request ใช้โควตาเพิ่มแม้จะ fail ก็ตาม) ไม่มีการหยุดพักชั่วคราว (backoff) เมื่อเจอความล้มเหลวต่อเนื่องหลายครั้ง — ความเสี่ยงต่ำเพราะ request ที่ fail เร็ว (เช่น HTTP 4xx) ไม่ได้ใช้ bandwidth มาก แต่ในทางทฤษฎีควรเพิ่ม backoff ถ้าพบว่าเป็นปัญหาจริงในการใช้งาน

### 8.8 ✅ แก้แล้ว — teacher.html ค้างนาน "กำลังเข้าสู่ระบบ..." ตอน login
**อาการ:** หน้าจอ login ค้างที่ข้อความ "กำลังเข้าสู่ระบบ..." (มี loading spinner) เป็นเวลานานก่อนเข้าระบบสำเร็จ ตามภาพหน้าจอที่ผู้ใช้แนบมา

**สาเหตุที่แท้จริง (ยืนยันด้วยการอ่านโค้ด):** ขั้นตอน login ปกติ 1 ครั้ง เรียก `getApp()` (ซึ่งโหลด `milkApp` node **ทั้งก้อน** รวม `mcAttendance` ที่มีรูป/ลายเซ็นสะสมของทุกห้องทุกวันที่เช็คมา) ซ้ำกันถึง **3 รอบ**:
1. `loadRooms()` (ตอนกดปุ่ม "โหลดห้องเรียน") → `getApp()` ครั้งที่ 1
2. `doLogin()` (ตอนกดปุ่มเข้าสู่ระบบ — จุดที่ขึ้นข้อความ "กำลังเข้าสู่ระบบ...") → `getApp()` ครั้งที่ 2 ← **จุดที่ภาพหน้าจอจับได้ตรงนี้พอดี**
3. `enterApp()`→`loadFB()` → `getApp()` ครั้งที่ 3

ยิ่งข้อมูล `mcAttendance` สะสมมากขึ้นตามอายุการใช้งานของโรงเรียน (รูป/ลายเซ็น base64) เวลาที่เสียไปกับการโหลดซ้ำ 3 รอบก็ยิ่งมากขึ้นตามไปด้วย

**วิธีแก้ (ไม่กระทบ schema/พฤติกรรมอื่นใดเลย):** เพิ่มตัวแปร cache ระดับ global (`_cachedApp`/`_cachedAppAt`/`_cachedUrl`/`_cachedKey`) เก็บผลลัพธ์จาก `getApp()` ที่โหลดใน `loadRooms()` ไว้ (อายุแคช 2 นาที ผูกกับ URL+Key เดิม) แล้วให้ `doLogin()` ใช้ของที่แคชไว้แทนการโหลดซ้ำถ้ายังใหม่พอ จากนั้นแยกฟังก์ชัน `loadFB()` เดิมออกเป็น `applyFD(app)` (ส่วน "นำข้อมูลไปใช้ตั้ง `FD`/`STUDS`") + `loadFB()` แบบเดิมที่เรียก `getApp()` แล้วค่อย `applyFD()` ต่อ — `doLogin()` หลัง login ผ่านจะเรียก `applyFD(app)` ทันทีด้วยข้อมูลที่มีอยู่แล้ว แล้ว `enterApp()` เปลี่ยนจาก `await loadFB();` (ไม่มีเงื่อนไข) เป็น `if(!FD)await loadFB();` (โหลดซ้ำเฉพาะกรณีที่ยังไม่มี FD เช่น เคส auto-login จาก session ที่บันทึกไว้ ซึ่งเดิมก็โหลดอยู่แล้ว 1 ครั้งพอดี ไม่กระทบ)

**ผลลัพธ์:** เคส login ปกติ (กรอกรหัสผ่านเข้าระบบ) ลดจาก 3 ครั้ง → 1 ครั้งต่อการ login (ลดลง ~66%) เคส auto-login จาก session ที่บันทึกไว้ (ผ่าน `init()`) ไม่กระทบเลย ยังคงโหลด 1 ครั้งเหมือนเดิม

---

## 9. Coding Rules (ต้องทำตามเพื่อความสอดคล้อง)

1. **ภาษา**: UI ทั้งหมดเป็นภาษาไทย รวม comment ในโค้ดส่วนใหญ่ก็เป็นไทย — ให้คงรูปแบบนี้
2. **ห้ามใช้ framework/build step**: ทุกอย่างอยู่ใน 1 ไฟล์ `.html` (inline `<style>` + `<script>`) ห้าม import module แยกไฟล์ (ยกเว้น Google Fonts / xlsx CDN ที่มีอยู่แล้ว)
3. **Key ของ attendance ต้องเป็น `"{roomId}_{YYYY-MM-DD}"` เสมอ** — ทั้ง `mcDB.attendance`, `milkApp/mcAttendance` บน Firebase, และฟังก์ชัน `attKey()`/การ filter ด้วย `k.startsWith(roomId+'_')` / `k.split('_').pop()` ขึ้นกับ key รูปแบบนี้ — **roomId ต้องไม่มี `_` ปนอยู่** ไม่เช่นนั้น `k.split('_')` จะพังหมด (ปัจจุบัน room id มาจาก `genId()` ซึ่งไม่มี `_` จึงปลอดภัย)
4. **โครงสร้าง record การเช็ค ต้องมี field ตรงกันทั้ง index.html และ teacher.html**: `{ clsId, date, year, term, teacher, data, notes, photos, signature, savedAt }` — ถ้าจะเพิ่ม field ใหม่ ต้องเพิ่มทั้ง 2 ไฟล์พร้อมกัน และอัปเดต `mcBuildSummaryTable`/`buildSumTable` ทั้งสองฝั่งถ้าต้องแสดงผล 🆕 **`photos`/`signature` ใน record ต้องเป็น base64 จริงเสมอทุกครั้งที่ส่งขึ้น Firebase หรืออยู่ในหน่วยความจำ** — ไม่ใช่ IndexedDB ref string (ดูข้อ 12)
5. **Firebase**: ทุก fetch ไปยัง Firebase REST ต้อง handle error ด้วย try/catch และไม่ throw ขึ้นไปทำลาย UI — ใช้ `showToast`/`toast` แจ้งผล ไม่ใช้ `alert()` สำหรับ background sync 🆕 ยกเว้นกรณี critical storage failure ที่เสี่ยงข้อมูลหายถาวร (ดูข้อ 14)
6. **`fbGetConfig()`/`DB_URL`+`DB_KEY`**: การสร้าง URL ต้อง `replace(/\/+$/,'')` ก่อนต่อ path เสมอ (กัน URL ที่ผู้ใช้ใส่ trailing slash มาแล้ว) และต่อ `?auth=` **เฉพาะกรณีมี key** (ดู `fbUrl()` ใน teacher.html / pattern ใน `mcSyncFromCloud`)
7. **Signature canvas**: ตรวจว่ามีลายเซ็นจริงด้วย `getImageData(...).data.some(d=>d!==0)` ก่อน `toDataURL()` — ถ้า canvas เปล่าให้เก็บเป็น `''` ไม่ใช่ dataURL เปล่า (ใช้ใน `isBlankSig` ด้วย)
8. **รูปภาพ**: เก็บเป็น base64 dataURL ตรงใน record (ไม่ใช้ external storage) — จำกัดสูงสุด 5 รูปต่อการเช็ค 1 ครั้ง (`mcCurrentPhotos`/`curPhotos`) 🆕 **ต้องผ่าน `compressImageFile()` ก่อนเก็บเสมอ ทุกจุดอัปโหลด** — ห้ามใช้ `FileReader.readAsDataURL()` ตรงๆ อีก (ดูหัวข้อ 2.5.1)
9. **วันที่**: ใช้ ISO `YYYY-MM-DD` ภายในระบบเสมอ, แปลงเป็นไทย (พ.ศ.) เฉพาะตอนแสดงผลด้วย `mcThaiDate`/`thD` (`+543` ปี)
10. **สัปดาห์ = จันทร์-ศุกร์** (`mcGetWeekRange`/`wkRng` คำนวณ Mon-Fri เท่านั้น ไม่รวมเสาร์-อาทิตย์)
11. **ก่อนแก้ไข ให้ตรวจ syntax ด้วย `node --check`** กับเนื้อหาในแต่ละ `<script>` block (ทั้งสองไฟล์รวมกันมี 3 script blocks: index.html มี 2, teacher.html มี 1)
12. 🆕 **IndexedDB reference string (`"idb:" + key`) เป็นรายละเอียดภายในของ `localStorage` ของ index.html เท่านั้น ห้ามให้หลุดออกไปไหน**: ห้ามส่งขึ้น Firebase, ห้ามส่งให้ teacher.html, ห้ามปล่อยให้ค้างอยู่ใน state ที่กำลังแก้ไข (`mcCurrentPhotos`) — ทุกจุดที่อ่าน record ที่อาจมี ref (`mcLoadGrid`, `mcShowTeacherDetail`, print preview) ต้องมี defensive filter กรอง `startsWith('idb:')` ออกก่อนใช้งานเสมอ (ดูหัวข้อ 2.5.3 ข้อ 3)
13. 🆕 **ทุกการเขียนลง IndexedDB (`mcIdbPut`) ต้องเช็คผลลัพธ์ก่อนตัดสินใจทิ้งข้อมูลจริงออกจาก localStorage** — ห้ามเขียน ref แบบ optimistic โดยไม่รอผล (ดูหัวข้อ 2.5.3 ข้อ 2) ถ้า `mcIdbPut` คืน `false` ต้องเก็บข้อมูลจริง (base64) ไว้ใน localStorage แทน
14. 🆕 **ทุก fetch ไปยัง Firebase ต้องมี timeout** ผ่าน `fbFetchWithTimeout()` (มีอยู่แล้วทั้ง 2 ไฟล์ — ใช้ค่า default 10 วินาที, ปรับเพิ่มเป็น 15-20 วินาทีสำหรับ payload ใหญ่กว่าปกติ เช่น full-DB push/pull) ห้ามใช้ raw `fetch()` ตรงๆ ไปยัง Firebase อีก เพื่อป้องกัน UI ค้างถ้าเน็ตหลุด
15. 🆕 **การเขียนที่สำคัญ (เช่น `fbPatchAttendance`) ต้องมีกลไก retry ถ้าล้มเหลว** ไม่ใช่แค่ catch แล้วเงียบ — ใช้ pattern คิว retry ใน localStorage (`mcGetRetryQueue`/`mcAddToRetryQueue`/`mcRemoveFromRetryQueue`/`mcRetryPendingPatches`) เป็นตัวอย่างอ้างอิงถ้าต้องเพิ่มกลไกแบบเดียวกันให้ฟังก์ชันอื่นในอนาคต
16. 🆕 **ทุกฟังก์ชันที่เขียน `localStorage.setItem()` ต้องมี try/catch และคืนค่า `true`/`false` ตามผลลัพธ์เสมอ** (ดูบั๊ก 8.0 — `saveDB`/`smSave`/`vdSave`/`bdSave`/`mcSavePersist` ทั้งหมดทำตามรูปแบบนี้แล้ว) ห้ามเขียน `localStorage.setItem(...)` ตรงๆ แบบไม่มีการจับ error อีก เพราะถ้า quota เต็มจะ throw แบบไม่มีใครจับ ทำให้ฟังก์ชันที่เรียกมันหยุดกลางคันแบบไม่มี error ใดๆ ให้เห็น (อาการ "กดบันทึกแล้วไม่บันทึก") — และทุกจุดที่เรียกฟังก์ชันบันทึกเหล่านี้ **ต้องเช็คค่าที่คืนมาและ rollback การเปลี่ยนแปลงในหน่วยความจำ** ถ้าบันทึกล้มเหลว ไม่ให้สถานะในหน่วยความจำ (เช่น array ที่ push ไปแล้ว) เพี้ยนไปจากสิ่งที่ถูกเขียนลงดิสก์จริง
17. 🆕 **ทุกจุดที่ "แทนที่ `db` ทั้งก้อน" จากแหล่งข้อมูลภายนอก (localStorage/Firebase/ไฟล์นำเข้า) ต้องผ่าน `withDbDefaults()` เสมอ** (ดูบั๊ก 8.0a) ห้ามเขียน `db = parsed` หรือ `db = mainData` ตรงๆ อีก เพราะข้อมูลจากภายนอกอาจขาด field ที่จำเป็นไป (เช่น `distributes` ไม่มี เพราะมาจากเวอร์ชันเก่า/ไม่สมบูรณ์) ทำให้ `.push()`/`.sort()`/`.reduce()` ที่จุดอื่นในระบบ throw `Cannot read properties of undefined` แทน — ถ้าเพิ่ม field ใหม่ใน `db` ในอนาคต ต้องเพิ่มค่า default ของ field นั้นใน `withDbDefaults()`/`defaultSettings()` ด้วยเสมอ ไม่ใช่แค่ในตัวแปร `db` เริ่มต้นตัวเดียว
18. 🆕 **`room.stock`/`milkApp/roomStock/{roomId}` เป็นแหล่งความจริงเดียว (single source of truth) สำหรับการดำเนินงานระดับห้องเรียนทุกชนิด** (เช็คดื่มนมรายวัน, นมค้างรายสัปดาห์) — **ห้ามใช้ `db.stock`/`milkApp/stock` (สต็อกกลาง) แทนในการคำนวณ/ตรวจสอบระดับห้องอีก** ส่วน `db.stock` กลางยังใช้ถูกต้องสำหรับ: รับนมจาก อบต. (+), จ่ายนมให้ห้องเรียน (-), จ่ายปิดเทอม (-) เท่านั้น — ดูหัวข้อ 11 ก่อนแก้โค้ดส่วนนี้เสมอ
19. 🆕 **ห้ามสร้าง ledger/audit-trail ใหม่ซ้ำซ้อนกับ `milkApp/stockLog` ที่มีอยู่แล้ว** — ทุกธุรกรรมที่กระทบ `room.stock` ต้องบันทึกผ่าน node เดียวนี้ (แท็ก `category` ถ้าจำเป็นต้องแยกประเภท) การสร้าง ledger คู่ขนาน (เช่น `stockTransactions` แยกอีกชุด) จะกลับไปเป็นปัญหาแบบเดียวกับที่ทำให้ stock เพี้ยนในตอนแรก (หลายระบบบันทึกเรื่องเดียวกันแบบไม่รู้จักกัน — ดูหัวข้อ 11.1) ก่อนเพิ่ม field/ประเภทธุรกรรมใหม่ ให้ขยาย schema ของ `stockLog` เดิมก่อนเสมอ
20. 🆕 **`bridgeDB` (`absentMilk`/`retroMilk`/`vacationMilk` cache) เก็บเป็น object คีย์ด้วย Firebase push-id เสมอ ห้ามแปลงเป็น array** — push-id คือ unique transaction id ที่ใช้กันบันทึกซ้ำและผูก patch สถานะข้ามไฟล์ (เช่น `retroMilkId`) ถ้าแปลงเป็น array จะเสีย id นี้ไปและกันบันทึกซ้ำไม่ได้อีก (ดูหัวข้อ 11.3, 3.9)

---

## 10. ห้ามเปลี่ยนโครงสร้างใดบ้าง (Breaking-change risk — ห้ามแก้โดยไม่จำเป็น)

1. **รูปแบบ key ของ `mcDB.attendance` / `milkApp/mcAttendance`**: `"{roomId}_{date}"` — มีโค้ดหลายสิบที่ทำ `k.split('_')`, `k.startsWith(id+'_')`, `k.slice(roomId.length+1)`, `k.endsWith('_'+date)` ทั้ง 2 ไฟล์ ถ้าเปลี่ยน format ต้องแก้ทุกจุด (ความเสี่ยงสูงมาก) 🆕 ยังเป็น key เดียวกันที่ใช้คำนวณ IndexedDB ref ผ่าน `mcIdbRefFor(attKey)` ด้วย ถ้าเปลี่ยน format จะกระทบ ref ที่เคยเขียนไว้แล้วในเครื่องผู้ดูแลทุกเครื่อง (หาไม่เจอ เพราะคำนวณ ref ใหม่จาก key รูปแบบใหม่ไม่ตรงกับที่เคยเก็บไว้)
2. **`db.rooms[].id`** — ใช้เป็น roomId ทุกที่ (mcAttendance key, distribute records, login session) ห้ามเปลี่ยนวิธี generate (ปัจจุบัน `genId()`) หรือเปลี่ยนค่าที่มีอยู่ของห้องเดิม เพราะจะทำให้ key เช็คดื่มนมเก่าหา class ไม่เจอ
3. **field names ของ student object** (`'เลขที่'`, `'รหัส'`/`'รหัสประจำตัว'`, `'เพศ'`, `'ชื่อ'`, `'นามสกุล'`, `'ชื่อ-นามสกุล'`) — มาจากการ map คอลัมน์ Excel ตรงๆ ห้ามเปลี่ยนชื่อ key โดยไม่แก้ `parseSchoolExcel` (index.html) และ `parseStudents` (ทั้ง 2 ไฟล์) พร้อมกัน
4. **path Firebase `milkApp/mcAttendance/{key}`** — เป็น node เดียวที่ทั้ง index.html (`fbPatchAttendance`, `mcSyncFromCloud`) และ teacher.html (`saveAtt` ผ่าน `fbSet`) อ่าน/เขียนร่วมกัน ห้ามเปลี่ยน path นี้โดยไม่แก้พร้อมกันทั้ง 2 ไฟล์
5. **`db.settings.firebaseUrl`** ↔ **`DEFAULT_FB_URL`** ต้องตรงกันเสมอ — ถ้าเปลี่ยนโปรเจค Firebase ต้องอัปเดตทั้ง 2 จุด
6. **`record` shape ของการเช็คดื่มนม** (`clsId, date, year, term, teacher, data, notes, photos, signature, savedAt`) — `mcBuildSummaryTable`, `buildSumTable`, หน้าพิมพ์ A4 ทั้งหมด, `mcRenderTeacherStatus`/`mcShowTeacherDetail` อ้างอิง field เหล่านี้ตรงๆ 🆕 `photos`/`signature` **ต้องเป็น base64 จริงทุกครั้งที่ส่งขึ้น Firebase** — ห้ามส่ง IndexedDB ref string ขึ้นไปเด็ดขาด (ดูข้อ 9 ด้านล่าง)
7. **localStorage keys**: `milk_school_db` (db), `milkCheckDB_v2` (mcDB), `storedMilkDB_v1` (smDB), `vacationDistDB_v1` (vdDB), `tc_cfg`/`tc_sess` (teacher.html), 🆕 `milkCheckPendingPatches_v1` (คิว retry) — ห้ามเปลี่ยนโดยไม่มี migration เพราะข้อมูลเดิมของโรงเรียนจะหายไปจากมุมมอง
8. **`#sidebar` ต้องเป็น `height: 100vh; max-height: 100vh; display:flex; flex-direction:column;`** และ `.nav-section` ต้องคง `flex:1; overflow-y:auto;` ไว้ — เป็นกลไกที่ทำให้เมนูซ้าย scroll ได้ ถ้าแก้ CSS sidebar ใหม่ ต้องรักษา layout นี้
9. **ชื่อ id ของหน้า/element ที่ผูกกับ `onclick`** เช่น `showPage('mc-home')`, `goP('chk')`, `id="page-mc-home"`, `id="pg-chk"` — ห้าม rename โดยไม่ไล่แก้ `onclick`/`getElementById` ที่อ้างถึงทั้งหมด (ใช้ grep ก่อนเปลี่ยนเสมอ)
10. 🆕 **IndexedDB database/store name (`milkCheckMediaDB`/`media`) และ ref prefix (`"idb:"`)** — ถ้าเปลี่ยนชื่อ ข้อมูลรูป/ลายเซ็นที่เคย offload ไว้แล้วในเครื่องผู้ดูแลแต่ละคนจะหาไม่เจอ (กลายเป็น "ไม่พบใน IndexedDB" → fallback คืนค่าเดิมที่มี ref string ค้างอยู่ ซึ่งจะถูก defensive filter กรองออกจนเหมือนรูปหายไปจากมุมมอง แม้ตัวข้อมูลจริงยังอยู่ใน IndexedDB เก่าก็ตาม) ถ้าจำเป็นต้องเปลี่ยนชื่อ ต้องเขียน migration ย้ายข้อมูลจาก store เก่าไป store ใหม่ก่อน
11. 🆕 **ค่า `PHOTO_MAX_DIM`/`PHOTO_QUALITY` ต้องตรงกันทั้ง 2 ไฟล์เสมอ** (ดูข้อ 9.4 และ section 5) — ไม่บังคับทางเทคนิค (แต่ละไฟล์ใช้ค่าของตัวเองอิสระ ไม่กระทบกัน) แต่ถ้าไม่ตรงกันจะทำให้ขนาด/คุณภาพรูปจากผู้ดูแลกับครูต่างกันโดยไม่มีเหตุผล ควรหลีกเลี่ยง
12. 🆕 **path Firebase `milkApp/roomStock/{roomId}` และ `milkApp/stockLog`** — เป็น node ที่ทั้ง index.html (`pushRoomStock`, `mcSaveAttendance`, `saveDistribute`, `saveStoredMilkDispense`) และ teacher.html (`saveAtt`, `deleteHistRecord`) อ่าน/เขียนร่วมกัน ห้ามเปลี่ยน path โดยไม่แก้พร้อมกันทั้ง 2 ไฟล์ (ดูหัวข้อ 11)
13. 🆕 **field `txnId`/`source`/`retroMilkId`/`vacationMilkId`** บน record ของ `absentMilk`/`retroMilk`/`vacationMilk` — ใช้เป็นกลไกกันบันทึกซ้ำข้ามไฟล์ (ดูหัวข้อ 11.3) ห้ามลบ/เปลี่ยนชื่อ field เหล่านี้โดยไม่แก้ logic cross-check ใน `getStoredMilkPending()`/`calcBackdateDist()`/`renderBackdateReport()`/`renderVacationReport()` พร้อมกัน

---

## 11. 🆕🆕 Stock Model v2 — สต็อกย่อยรายห้อง (Room Stock) + แก้ Double-Deduction + Cross-file Bridge Sync

> อ่านหัวข้อนี้ทั้งหมดก่อนแก้โค้ดส่วน stock/นมค้าง/ย้อนหลัง/ปิดเทอม ในทั้ง 2 ไฟล์เสมอ — เป็นการเปลี่ยนสถาปัตยกรรมที่ใหญ่ที่สุดของระบบนี้

### 11.1 ปัญหาที่พบ (root cause)
ผู้ใช้รายงานว่าตัวเลขสต็อกในหน้าต่างๆ ไม่ตรงกัน → วิเคราะห์ data flow ทั้งระบบ (ทุกจุดที่อ่าน/เขียน `stock`) พบบั๊ก/conflict สำคัญดังนี้:

1. **Double-deduction**: `saveDistribute()` (index.html, "จ่ายนมให้ห้องเรียน" — จ่ายล่วงหน้าเป็นชุดตาม `นักเรียน × วัน`) และ `saveAtt()` (teacher.html, "เช็คดื่มนมรายวัน" — หักตามจริงทุกวัน) **ต่างหักจาก `milkApp/stock` ตัวเดียวกัน** โดยไม่รู้จักกันเลย — ถ้าโรงเรียนใช้ทั้ง 2 ฟีเจอร์ (กรณีจริงของผู้ใช้) กล่องนมจริงๆ 1 กล่องถูกหักออกจากสต็อกกลาง **สองครั้ง**
2. **Asymmetry**: `mcSaveAttendance()` (index.html, เช็คดื่มนมฝั่งผู้ดูแล) **ไม่หักสต็อกเลย** ขณะที่ `saveAtt()` (teacher.html, ฟีเจอร์เดียวกัน) หัก — ผลลัพธ์ของการเช็คขึ้นกับ "ใครกดบันทึก" ไม่ใช่ "ข้อมูลอะไรถูกบันทึก"
3. `deleteHistRecord()` (teacher.html) ลบประวัติเช็คแต่ไม่คืนสต็อกที่เคยหักไป → สต็อกหายถาวรทุกครั้งที่ลบ
4. Auto-sync (`mcSyncFromCloud`, ทุก 3 นาที) ไม่ดึงค่า `milkApp/stock` เข้ามาที่ local `db.stock` เลย → ผู้ดูแลเห็นค่าที่ไม่ตรงกับความเป็นจริงตลอดเวลา
5. `fbPushData()` ส่ง `stock` ไปด้วยตอน "อัปโหลดสู่ Cloud" → เขียนทับการหักของครูบน Cloud โดยไม่ตั้งใจ
6. `saveStoredMilkDispense()` (นมค้างรายสัปดาห์) หักสต็อกแต่ไม่บันทึกลง `db.distributes` → มองไม่เห็นใน report
7. `smDB`/`vdDB`/`bdDB` (index.html, localStorage) กับ `absentMilk`/`retroMilk`/`vacationMilk` (teacher.html, Firebase) เป็นข้อมูลคนละชุดที่ไม่เชื่อมกัน (บั๊ก 8.2 เดิม) — เสี่ยงบันทึก/จ่ายนมค้าง-ย้อนหลัง-ปิดเทอมซ้ำซ้อนข้ามไฟล์โดยไม่รู้ตัว

### 11.2 การตัดสินใจสถาปัตยกรรม (อนุมัติแล้วโดยผู้ใช้ — แนวทาง B)
มี 2 ทางเลือกที่เสนอ: (A) ใช้ "เช็คดื่มนมรายวัน" เป็นจุดหักสต็อกกลางจุดเดียว หรือ (B) คง "จ่ายนมให้ห้องเรียน" เป็นจุดหักสต็อกกลาง แล้วสร้าง **"สต็อกย่อยรายห้อง" (room stock)** ใหม่ให้เช็คดื่มนมรายวันหักจากสต็อกย่อยนั้นแทน — **ผู้ใช้เลือกแนวทาง B**

**สูตรที่ใช้จริงตอนนี้:**
```
db.stock (สต็อกกลาง)     = รับนมจาก อบต. (+) − จ่ายนมให้ห้องเรียน (−) − จ่ายปิดเทอม (−)
room.stock (สต็อกห้อง)   = จ่ายนมให้ห้องเรียนของห้องนี้ (+) − เช็คดื่มนมจริงของห้องนี้ (−) − นมค้างที่เบิกแล้วของห้องนี้ (−)
```
⚠️ **"จ่ายนมย้อนหลัง"/"หนี้นม" และ "จ่ายปิดเทอม" (ฝั่งครู, `vacationMilk`) ไม่กระทบ `room.stock` เลยโดยตั้งใจ**:
- "ย้อนหลัง" เป็นหนี้ที่ยังไม่มีนมจริงจาก อบต. ส่งมา (ไม่มีของให้หัก)
- "ปิดเทอม" ฝั่งครู (`saveVac()`) เป็นแค่บันทึกคู่กับ `vdDB`/`saveVacationDist()` ของผู้ดูแลที่หักสต็อก**กลาง**ไปแล้วตรงๆ (ไม่ผ่านห้อง เพราะเป็นการส่งมอบตรงให้ครอบครัวช่วงปิดเทอม ไม่ใช่การสะสมไว้ที่ห้องเพื่อแจกรายวัน) — รวมเข้าสูตร `room.stock` อีกจะกลายเป็นหักซ้ำซ้อนแบบเดียวกับปัญหาที่กำลังแก้

`room.stock` **ไม่ clamp ที่ 0** — ห้องที่ได้รับจัดสรรไม่พอจะเห็นค่าติดลบจริง (เพื่อให้ผู้ดูแลรู้ว่าต้องจ่ายเพิ่ม ไม่ใช่ซ่อนปัญหาด้วยการปัดเป็น 0 แบบโค้ดเดิม)

### 11.3 Cross-file Bridge Sync (`bridgeDB`) — เชื่อม smDB/vdDB/bdDB กับ absentMilk/retroMilk/vacationMilk
แก้บั๊ก 8.2 บางส่วน (ดูหัวข้อ 8.2 สำหรับสถานะล่าสุด) ด้วยกลไก "bridge": index.html เขียนขึ้น node เดียวกับที่ teacher.html ใช้ **และ** อ่านกลับเข้า cache ในหน่วยความจำชื่อ `bridgeDB` (ดูหัวข้อ 3.9 สำหรับ schema เต็ม)

**Firebase paths ที่ใช้:**
- `milkApp/roomStock/{roomId}` — number, สต็อกย่อยของห้อง (อ่าน/เขียนโดยทั้ง 2 ไฟล์)
- `milkApp/stockLog` — push-only, audit ledger เดียวของทุกธุรกรรมที่กระทบ `room.stock` (ดูหัวข้อ 2 สำหรับ schema)
- `milkApp/absentMilk`, `milkApp/retroMilk`, `milkApp/vacationMilk` — เดิมมีแค่ teacher.html push ตอนนี้ index.html push ด้วย (แท็ก `source:'admin'`, `txnId`)

**Duplicate protection logic** (ป้องกันบันทึก/จ่ายซ้ำข้ามไฟล์):
- `getStoredMilkPending()` (index.html) — รวม key `roomId_studentId_date` จาก `bridgeDB.absentMilk` (ที่ครูบันทึกผ่าน `students:{studentId:{days:[...]}}`) เข้ากับ `smDB.dispensed` ของตัวเอง ก่อนสรุปว่านักเรียนคนไหน "ยังไม่ได้รับนมค้าง" — ถ้าครูจ่ายไปแล้ว จะไม่โผล่มาให้ผู้ดูแลจ่ายซ้ำ
- `calcBackdateDist()` (index.html) — เช็ค `bridgeDB.retroMilk` ว่ามีห้อง+ช่วงวันที่ทับซ้อนหรือไม่ ถ้ามีจะ**เตือน** (ไม่บล็อก — เผื่อตั้งใจบันทึกเพิ่มจริง)
- ฝั่งครู (teacher.html) **ไม่ต้องแก้เพิ่ม** — `loadAbs()`/`loadVacH()`/`loadRetH()` เดิมอ่านจาก node เดียวกันนี้อยู่แล้ว พอ index.html push record รูปแบบเดียวกันเข้าไป ครูก็เห็น "จ่ายแล้ว" ทันทีโดยอัตโนมัติ ไม่ต้องแก้ teacher.html เลย

**กลไก unique transaction id**: ใช้ **Firebase push-id** (ค่าที่ `fbPush()` คืนมา) เป็น unique id อยู่แล้ว ไม่ได้สร้างระบบ id แยก — เก็บ id นี้ไว้ที่ `rec.retroMilkId`/`rec.vacationMilkId` (ใน `bdDB`/`vdDB` ของ index.html) เพื่อให้ `markBdPaid()` patch สถานะ "ชำระแล้ว" กลับไปที่ `milkApp/retroMilk/{id}` ได้ตรงตัว (สำหรับรายการที่ครูสร้างเอง ไม่มีคู่ใน `bdDB` ใช้ `markRetroMilkPaid(firebaseId)` แทน)

**ทำไมไม่สร้าง `milkApp/stockTransactions` แยกใหม่ (ตามที่มีการขอในบางรอบของการสนทนา)**: เพราะจะกลับไปเป็นปัญหาแบบเดียวกับข้อ 11.1.1 (ระบบหลายชุดบันทึกเรื่องเดียวกันแบบไม่ประสานกัน) — เลือกขยาย `milkApp/stockLog` ที่มีอยู่แล้วด้วย field `category` แทน (ดูหัวข้อ 9 ข้อ 19)

**ทำไม `room.stock` ไม่รวม retro/vacation ในสูตรหัก (ตามที่มีการขอในบางรอบของการสนทนา)**: เพราะทั้งคู่ไม่ใช่การหักสต็อกจริงโดยออกแบบของระบบเดิม (ดูเหตุผลเต็มในหัวข้อ 11.2) — การรวมเข้าสูตรจะทำให้ตัวเลขผิดจากความเป็นจริง (ติดลบเกินจริง สำหรับ retro ที่ยังไม่มีนมจริง หรือหักซ้ำสำหรับ vacation ที่หักสต็อกกลางไปแล้ว)

### 11.4 ฟังก์ชันที่แก้ไข/เพิ่มใหม่ทั้งหมด
ดูรายชื่อเต็มในหัวข้อ 4 (subsection "🆕 Stock Model v2") — สรุปไฟล์ที่แก้:
- **index.html**: `withDbDefaults`, `confirmImport`, `addManualRoom`, `pushRoomStock` (ใหม่), `fbPush`/`fbPatch` (ใหม่), `fbPushData`, `mcSyncFromCloud`, `loadBridgeDB`/`syncBridgeDB` (ใหม่), `mcSaveAttendance`, `deleteRoom`, `recalcRoomStock` (ใหม่), `saveDistribute`, `deleteDist`, `getStoredMilkPending`, `saveStoredMilkDispense`, `calcBackdateDist`, `saveBackdateDist`, `markBdPaid`, `markRetroMilkPaid` (ใหม่), `renderBackdateReport`, `saveVacationDist`, `renderVacationReport`, `init` (เพิ่ม auto-migration trigger)
- **teacher.html**: `saveAtt`, `deleteHistRecord`, `renderStk`, `renderHist` (ส่วน `histGlobalInfo`), `updVac` (เทียบกับ `roomStock` แทน `stock` กลาง) + ข้อความ HTML หน้า `pg-stk`

### 11.5 การทดสอบที่ทำแล้ว (และที่ยังไม่ได้ทำ — สำคัญมาก)
✅ **ทำแล้ว**: `node --check` ผ่านทุก script block ทั้ง 2 ไฟล์ทุกครั้งหลังแก้ไข, จำลอง logic ด้วยข้อมูลสมมติใน Node.js (ไม่ใช่รันโค้ดจริงในเบราว์เซอร์/Firebase) รวม 11 เคส ครอบคลุม: ไม่หักซ้ำซ้อนระหว่างจ่ายนมห้องเรียน/เช็คดื่มนม, diff-logic ตอนแก้ไขข้อมูลย้อนหลัง, ลบประวัติคืนสต็อกพอดี (symmetric), ลบรายการจ่ายนมคืนสต็อกพอดี, `recalcRoomStock()` idempotent, ห้องขาดดุลแสดงค่าติดลบไม่ถูกซ่อน, dedup ระหว่าง `smDB`/`bridgeDB.absentMilk` ตอนเหตุการณ์เดียวกันถูกบันทึก 2 ฝั่ง — **ผ่านทั้งหมด**

❌ **ยังไม่ทำ (ไม่มีสิทธิ์เข้าถึง)**: รันกับ Firebase Realtime Database จริงของโรงเรียน, รันกับข้อมูล localStorage จริงของโรงเรียน, เปิดแอปจริงในเบราว์เซอร์ทดสอบ UI/UX จุดที่เพิ่มใหม่ (เช่น ปุ่ม "คำนวณสต็อกย่อยรายห้องใหม่" หน้าตั้งค่า, badge "👩‍🏫 ครูประจำชั้น" ในรายงาน) — ดูบั๊ก 8.11 สำหรับ checklist ที่ผู้ดูแลควรทำก่อนใช้งานจริง

### 11.7 🆕 รอบ Audit ล่าสุด — Verified Behavior Audit + ปิด Gap เพิ่ม
มีการขอให้ตรวจสอบ behavior จริงของทุกฟังก์ชันที่เกี่ยวกับ stock (ไม่ใช่เชื่อ design notes) ผลตรวจที่ยืนยันจากโค้ดจริง:

**ฟังก์ชันที่ขอให้ตรวจ แต่ "ไม่มีอยู่จริงในโค้ด"** (สำคัญ — ห้ามสมมติว่ามี):
- `deleteStoredMilkDispense()` — ไม่มี (ปิดแล้วในรอบนี้ ดูด้านล่าง)
- `deleteRt()` — ไม่มี ทั้ง 2 ไฟล์ไม่มีปุ่มลบสำหรับ `retroMilk`/`bdDB`
- `deleteVacationDist()` — ไม่มี ทั้ง 2 ไฟล์ไม่มีปุ่มลบสำหรับ `vacationMilk`/`vdDB`

**ยืนยันอีกครั้ง (จากโค้ดจริง ไม่ใช่สมมติ): Retro และ Vacation ไม่ควรกระทบ `room.stock`**
- `saveRt()`/`saveBackdateDist()` ไม่เขียน stock field ใดๆ เลย (หนี้นม ยังไม่มีนมจริง)
- `saveVacationDist()` หัก **Main Stock โดยตรง** (`db.stock -= total`) ไม่ผ่าน room — ถ้ารวมเข้าสูตรหัก room.stock อีกจะกลายเป็นหักซ้ำซ้อน (ครั้งที่ 1 จาก Main, ครั้งที่ 2 จาก Room) — **สูตร `room.stock` คงเดิม: `Distributes − Attendance − Pending` เท่านั้น ไม่รวม Retro/Vacation**

**Gap ที่พบและปิดแล้วในรอบนี้:**
1. `mcSaveAttendance()` (index.html) หัก `room.stock` แต่ไม่เคยเขียน `milkApp/stockLog` เลย (ไม่สมมาตรกับ `saveAtt()` ของ teacher.html ที่เขียน) → เพิ่มให้ครบ (`category:'ATTENDANCE'`)
2. `saveAtt()`/`deleteHistRecord()` (teacher.html) เขียน `stockLog` แต่ไม่มี field `category` → เพิ่ม `'ATTENDANCE'`/`'ROLLBACK'`
3. **ไม่มี rollback path สำหรับ "นมค้างรายสัปดาห์" เลย** (จ่ายแล้วยกเลิกไม่ได้) → เพิ่ม `deleteStoredMilkDispense(id)` ใหม่ทั้งหมด: คืน `room.stock`, เขียน `stockLog` (`category:'ROLLBACK'`), ลบออกจาก `smDB.dispensed`, และ patch ลบเฉพาะวันของนักเรียนคนนั้นออกจาก `milkApp/absentMilk` ที่ผูกไว้ (ไม่กระทบนักเรียนคนอื่นใน batch เดียวกัน — ใช้ field ใหม่ `bridgeId` บน `smDB.dispensed` entry แต่ละตัว เพื่อระบุ record บน Firebase ที่ต้องแก้แบบเจาะจง) — มีปุ่ม "🗑️ ยกเลิก" ในตาราง "ประวัติการจ่ายนมค้าง" แล้ว
4. เพิ่ม `validateMainStock()`, `validateRoomStock(roomId)`, `validateAllRoomStock()` — read-only, เทียบค่าที่เก็บไว้กับค่าที่คำนวณสดจาก ledger รายงานส่วนต่างผ่าน `console.warn`/`console.table` **ไม่แก้ไขค่าใดๆ** (ต่างจาก `recalcRoomStock()` ที่ "แก้" ค่าจริง) — เรียกจาก browser console เพื่อ debug เป็นระยะ (ยังไม่ผูกปุ่ม UI ในรอบนี้)

**สิ่งที่ตั้งใจ "ไม่แก้" ตามคำขอบางรอบ (พร้อมเหตุผล — สำคัญ ต้องอ่านก่อนแก้ซ้ำ):**
1. **ไม่สร้าง `milkApp/stockTransactions` แยกใหม่** — ขยาย `milkApp/stockLog` เดิมด้วย field `category` แทน (เหตุผลเดียวกับหัวข้อ 11.3) — ลอจิกการตรวจสอบ "ถ้า stockLog ครอบคลุมความต้องการของ ledger ได้แล้ว ห้ามสร้างใหม่ซ้ำซ้อน" ยืนยันแล้วว่า "ครอบคลุมได้" หลังเพิ่ม `category`
2. **ไม่รวม Retro/Vacation เข้าสูตรหัก `room.stock`** — ยืนยันจากโค้ดจริงแล้วว่าทั้งคู่ไม่กระทบ room.stock จริง (ดูด้านบน) การรวมเข้าไปจะทำให้ตัวเลขผิดความจริง
3. **ไม่ย้าย Vacation ออกจาก `db.distributes`** (มีคำขอให้ "Remove all logic that writes Vacation into distributes") — ตรวจแล้วพบว่า record ที่ `saveVacationDist()` push เข้า `db.distributes` **ไม่มี field `roomId`** (มีแค่ `roomName` เป็น string แสดงผล) ทำให้ `recalcRoomStock()`/`validateRoomStock()` ที่ filter ด้วย `d.roomId === room.id` **ไม่นับรวมรายการนี้อยู่แล้วโดยอัตโนมัติ** (ไม่มี double-count กับ room stock จริง) — การย้ายออกจริงจะกระทบ `renderReport()` หน้า "สรุปการบริหารจัดการนม" ที่ใช้ `db.distributes` รวมเป็น `totDist` สำหรับคำนวณ "นมคงเหลือในสต็อกปัจจุบัน" (Phase 2 fix ที่ทดสอบแล้ว) ต้องแก้ 2 จุดพร้อมกันเพื่อไม่ให้ตัวเลขกลางเพี้ยนอีก — **ประเมินว่าความเสี่ยง regression สูงกว่าประโยชน์ที่ได้ (แค่ความสะอาดของโครงสร้างข้อมูล ไม่ใช่บั๊กที่กระทบผลลัพธ์จริง) จึงไม่ทำในรอบนี้** ถ้าต้องการทำจริงในอนาคต ต้องแก้ `renderReport()`'s totDist ให้รวม `vdDB.records` เข้าไปด้วยพร้อมกัน

### 11.8 🆕 ส่วนที่ยังไม่ได้ทำในรอบนี้ (ขอบเขตใหญ่ ต้องแยกทำต่างหาก)
มีคำขอเพิ่มเติมเรื่อง **performance ของ teacher.html** (login <2s, parallel loading, session cache, Firebase query optimization) และ **document module** (metadata-first loading, lazy load, IndexedDB index, pagination, Firebase Storage migration สำหรับไฟล์ใหญ่) — **ยังไม่ได้ลงมือทำในรอบนี้** เพราะเป็นขอบเขตงานคนละก้อนจาก stock/ledger (ความเสี่ยง regression ต่อ boot sequence ของ teacher.html สูง ถ้าทำแบบเร่งรัดในรอบเดียวกับงาน stock) — ควรหยิบเป็นงานแยกรอบถัดไป โดยอ่านโค้ดจริงของ `docLoad()`/`docDB`/หน้า `page-documents` (index.html, บรรทัด ~5859 เป็นต้นไป) และ teacher.html's boot sequence (`init()`/`loadFB()`) ให้ครบก่อนแก้ เช่นเดียวกับวิธีที่ทำกับ stock ในหัวข้อนี้ทั้งหมด — ห้ามเขียนทับด้วย speculative profiler/Firebase Storage migration โดยไม่ตรวจโค้ดจริงก่อน

### 11.9 งานคงเหลือ/ข้อจำกัดที่รู้แล้ว (สะสมจากทุกรอบ)
1. ส่วนต่าง `db.stock` กับ ledger จริง ~2,889 กล่อง ที่พบในข้อมูลตัวอย่างก่อนเริ่มงานรอบนี้ — ยังไม่ทราบสาเหตุ 100% (บั๊ก 8.10)
2. `smDB`/`vdDB`/`bdDB` ↔ `absentMilk`/`retroMilk`/`vacationMilk` ยังไม่ unify schema เป็นชุดเดียวกัน 100% (แค่ bridge แบบ push-ทั้งคู่ + cross-check) — ยังไม่มีกลไก sync "การลบ" ข้ามไฟล์ (บั๊ก 8.2)
3. ยังไม่มี UI แสดง `room.stock` ของแต่ละห้องในหน้า "จ่ายนมให้ห้องเรียน" ของ index.html ตอนเลือกห้อง (เป็น nice-to-have ที่ไม่ได้ทำในรอบนี้ เพื่อจำกัดความเสี่ยงแก้ HTML ที่ไม่ได้ตรวจสอบโครงสร้างเต็ม)
4. คำเตือนทับซ้อนของ `calcBackdateDist()` เป็นแค่ warning ไม่ใช่ hard block — ตั้งใจให้เป็นแบบนี้ (กันกรณี false positive บล็อกการบันทึกที่ตั้งใจจริง) แต่ไม่ได้ทำแบบเดียวกันสำหรับ vacation (ไม่มี warning UI เพราะไม่กระทบ stock ใดๆ ความเสี่ยงต่ำกว่า)
5. 🆕 Section 12-16 ของคำขอ (performance, profiler, test simulation ที่อิงสูตรผิด, backup/restore rebuild) ยังไม่ได้ทำ — ดู 11.8

---

*จบเอกสาร — หากต้องทำงานต่อ ให้เริ่มจากหัวข้อ 7 (สิ่งที่ต้องทำต่อ) และหัวข้อ 8 (บั๊ก) ก่อน หากต้องแก้ไขส่วนรูปภาพ/sync/storage ให้อ่านหัวข้อ 2.5 และหัวข้อ 9 ข้อ 8, 12-15 ก่อนเริ่มแก้โค้ดเสมอ ถ้าต้องแก้ไขส่วนสต็อก/นมค้าง/ย้อนหลัง/ปิดเทอม ให้อ่านหัวข้อ 11 ทั้งหมดก่อนเริ่มเสมอ*
