# Check EDI Sinotrans

เครื่องมือตรวจสอบเอกสารขาเข้าเรือก่อนจัดทำใบขนสินค้าขาเข้า (Thailand ocean import) —
เทียบ **MANIFEST** (รายงานจากสายเรือ, .xls/.xlsx หรือ PDF "CARGO MANIFEST") กับ **ENTER**
(จดหมายแก้ MANIFEST ของ SINOTRANS, .pdf) ทีละ B/L ย่อย แล้วสร้างรายงาน `EDI.xlsx` แบบไฮไลต์สี
บอกจุดที่ต้องแก้ก่อนยื่นใบขน

> **ข้อจำกัด:** หน้าเว็บรองรับเฉพาะ **ENTER ฟอร์มของ Sinotrans**
> ถ้าเจอฟอร์มของเจ้าอื่นหน้าเว็บจะแจ้งว่าไม่ใช่ฟอร์มของ Sinotrans · ไฟล์ CNTRS ไม่นำมาเทียบ

## Check EDI Sinotrans บนเว็บ (ให้เพื่อนใช้ได้ ไม่ต้องติดตั้ง)

https://rattanao.github.io/Sinotrans/

1. ลากไฟล์ **MANIFEST** (.xls/.xlsx หรือ PDF) และ **ENTER ของ Sinotrans** มาวางพร้อมกัน
   (ตั้งชื่ออะไรก็ได้ — PDF ที่เป็น CARGO MANIFEST จะถูกจัดเป็น MANIFEST ให้เอง, PDF อื่นเป็น ENTER;
   ENTER หลายไฟล์ใส่พร้อมกันได้; ไฟล์ที่ชื่อมี `CNTRS` จะถูกข้าม)
2. ใส่ SHED NO. ที่แจ้ง (ถ้ามี) แล้วกด **ตรวจสอบ**
3. ดูผลบนหน้าเว็บ (ตารางสีเหมือน EDI.xlsx, ค้นหา/กรอง "แสดงเฉพาะที่ไม่ตรง") หรือกด **ดาวน์โหลด EDI.xlsx**

ไฟล์ถูกประมวลผลในเบราว์เซอร์ของผู้ใช้เท่านั้น ไม่มีการอัปโหลดไปที่ใด

โค้ดหน้าเว็บอยู่ที่ `docs/` (`index.html` + `core.js` — พอร์ตจาก `build_edi.py` layout L,
ฐานเดียวกับ [Check EDI LX Pantos](https://rattanao.github.io/LX-Pantos/) และ
[Check EDI Union World](https://rattanao.github.io/Union-world/))

## ฟอร์ม ENTER ที่รองรับ

หน้าละ 1 B/L (มีหน้าว่างคั่นระหว่าง B/L ได้):

```
TO :    HEUNG A LINE CO.,LTD.                 DATE  : 29.09.2026
FEEDER VESSEL   : SAWASDEE XXXX V.2609S
MB/L NO. :    HASLC01260000123A
PORT: LAEM CHABANG
WE WOULD LIKE TO REQUEST YOU TO AMEND YOUR MANIFEST OF THE ABOVE B/L AND
ISSUED D/O AS FOLLOWING:
  CNEE :  ABC TRADING CO.,LTD.
MARKS & NOS.          QUANTITY         DESCRIPTION OF GOODS
RB 123456789          6  PALLETS       WHEEL-SPEED SENSOR
CONTR NO.     HALU1234567 / 40' HQ     G.W.     :     670.00  KGS
VOLUME   :
STATUS   :    CY/CY                    CBM      :     6.00  CBM
```

- MARKS / QUANTITY / DESCRIPTION แยกตามตำแหน่งคอลัมน์ (x < 165 / 165–265 / ≥ 265) ลงไปถึงแถว `CONTR NO.`
- เลข B/L ที่พิมพ์ตัวนำซ้ำ (`HHASLC…`) ระบบตัดตัวอักษรนำให้เองจนตรงรูปแบบเลข B/L
- ตรา `CS IMPORT` ท้ายหน้า (FreeText) ไม่นับเป็นคำสั่งลูกค้า
- STATUS: `CY`/`CY/CY`/`ลากตู้` = CY, `LCL`/`ขน` = LCL, `CFS`/`เปิดตู้` = LCL/CFS
- ฟอร์มนี้ไม่มียอดรวมทั้งเอกสาร — แถว TOTAL เทียบผลรวม ENTER กับ MANIFEST (และ GRAND TOTAL ของ MANIFEST)

## สิ่งที่ตรวจ

CONSIGNEE · TAX ID vs NOTIFY · CONTAINER · STATUS · จำนวน/ชนิดหีบห่อ · G.W. · MEAS. · MARKS · DESCRIPTION ·
REEFER · DG · TRANSIT · SHED NO. · ประเทศปลายทาง · CARGO MOVEMENT — สีแดง ⚠ = ไม่ตรง, ส้ม = ต้องยืนยัน,
เหลือง = ไม่มี ENTER เทียบ, เขียว = ผ่าน · B/L ที่มีใน ENTER แต่ไม่มีใน MANIFEST = วิกฤต

## ใช้แบบ Python (ในเครื่อง)

```
pip install -r requirements.txt
python build_edi.py --manifest input/MANIFEST.xls --enter input/ENTER.pdf --outdir output [--shed 2813]
```

## ตรวจความตรงกันของหน้าเว็บกับ Python

2026-10-27: งาน SAWASDEE SPICA 2609S (26 B/L) ตรงกัน 756/756 ช่อง (ค่าและสี) ทั้งแบบไม่ใส่ SHED,
`--shed 2813` และ `--shed 0141` · หน้าเว็บปฏิเสธ ENTER ของเจ้าอื่นทั้ง 18 ฟอร์มที่มีตัวอย่าง

ไม่มีข้อมูลลูกค้าจริงใน repo นี้ (`.gitignore` กัน `input/`, `output/`, `*.xls*`, `*.pdf`)

Project by Ms. Rattana Mudlae — Position Import DOC.
