# CS240/245 — ครบทุกขั้นตอนใน notebook เดียว

ไฟล์หลัก: **Rice_Risk_CS240_Colab.ipynb**

## โครงสร้างโฟลเดอร์

```
Rice_Project/
├── Rice_Risk_CS240_Colab.ipynb      notebook หลัก (ไฟล์ที่ส่ง)
├── README_TH.md                     ไฟล์นี้
├── data/
│   ├── raw/Rice_Mycotoxin_Dashboard.xlsx   ข้อมูลต้นฉบับ (notebook โหลดจาก GitHub)
│   └── generated/                   ไฟล์ที่ notebook สร้างระหว่างทาง
│       ├── NASA_POWER.xlsx
│       └── Rice_Dirty.xlsx
├── outputs/                         ผลลัพธ์สำหรับรายงาน
│   ├── Rice_Project_Results.xlsx
│   ├── results_summary.txt
│   └── figures/  eda_simple.png, validation_simple.png, test_simple.png
├── docs/SLIDE_HANDOFF_TH.md         แผนสไลด์
└── archive/                         notebook รุ่นก่อนแก้ (ไม่ต้องส่ง)
```

ใน Colab ทุกไฟล์จะอยู่รวมกันใน `/content` ตามที่โค้ดเขียนไว้ หลังรันเสร็จให้ดาวน์โหลดแล้วนำมาวางตามโฟลเดอร์ข้างบน

## วิธีใช้บน Google Colab

1. เปิด notebook นี้ใน Colab แล้วเลือก Runtime → Run all
2. Notebook โหลด **Rice_Mycotoxin_Dashboard.xlsx** จาก `https://github.com/GearJP2/CS240` (`data/raw/`) เอง ไม่ต้องอัปโหลด
3. Notebook เรียก NASA POWER API 22 จุด สร้าง `NASA_POWER.xlsx` แล้วอ่านกลับมาทำ cleansing, merge และโมเดลต่อโดยอัตโนมัติ
4. ถึงหัวข้อ Demo ให้เปลี่ยนค่า `LOCATION` แล้วรัน cell นั้นใหม่
5. ผลตารางอยู่ใน `Rice_Project_Results.xlsx`; Excel อากาศและภาพกราฟดาวน์โหลดเพิ่มเติมได้จาก Files ด้านซ้าย

ไม่ต้องเปิด notebook อื่น และไม่ต้องเตรียมหรืออัปโหลด NASA_POWER.xlsx ล่วงหน้า ต้องมีอินเทอร์เน็ตเพื่อเรียก NASA API หากบริการมีปัญหา notebook จะหยุดพร้อม error ไม่แทนข้อมูลจริงด้วยข้อมูลปลอม

## ลำดับใน notebook

- Problem Framing: ทำนายผล Positive/Negative ของตัวอย่างใหม่เพื่อประกอบการพิจารณาพื้นที่
- Data Collection: อ่าน Excel ข้าว → เรียก NASA API → สร้าง Excel อากาศ → อ่านกลับ
- Data Preparation: mock วัน / inconsistent data → cleansing → สรุปประวัติปี 2021 → merge
- EDA: สถิติและกราฟ
- Classification: train/validation/test ตามเวลา → encoding/imputation/scaling → เปรียบเทียบ 3 โมเดลกับ baseline
- Demo: เปลี่ยนจังหวัดดู have risk/no risk ณ วันที่สมมติ 2023-01-01
- Interpretation: สรุปผลและข้อจำกัด พร้อมบันทึกไฟล์ประกอบรายงาน

`NASA_POWER.xlsx` เป็น **ผลลัพธ์ระหว่างขั้น Data Collection** มี 4 sheet:

| Sheet | เนื้อหา |
|---|---|
| Daily | อุณหภูมิ ความชื้น ฝนรายวันจาก API |
| Weather_30d | อุณหภูมิเฉลี่ย ความชื้นเฉลี่ย ฝนสะสม 30 วันก่อน Date ใช้ merge ด้วย Location + Date |
| Locations | พิกัดตัวแทนสมมติ 22 จังหวัด |
| Sources | URL / source / time standard |

## โค้ดระดับ CS240

อิง 05_3_Practice_DataCleansing.ipynb, MachineLearning_Supervised-Classification_Example-1.ipynb และ CS240_Lab_Practice_04_Iris_Comprehensive_Exam.ipynb

ใช้ `read_excel`, `replace`, `drop_duplicates`, `groupby`, `merge`, `get_dummies`, `fillna`, `StandardScaler`, `fit`, `predict` ไม่มี Pipeline/ColumnTransformer หรือระบบสร้างประวัติใหม่ทุก fold ใช้ประวัติปี 2021 คงที่ทำนายปี 2022 และเลือกโมเดลด้วย validation F1

## สมมติฐานและผลล่าสุด

- ค่าสาร/label/จังหวัดมาจาก Excel เดิม; วันเก็บ/วันทราบผล +7 วัน/จุดพิกัดตัวแทนเป็นสมมติ
- NASA เป็นอากาศจริง ณ จุด/เวลาที่ร้องขอ ไม่ใช่อากาศของแปลงจริงที่ยืนยันแล้ว
- Scenario 440 ตัวอย่าง; dirty 445 แถว/41 รูปแบบจังหวัด → clean 440 แถว/22 จังหวัด
- ประวัติปี 2021 มี 171 ตัวอย่างใน 8 จังหวัด; ชุดโมเดลปี 2022 มี 268 ตัวอย่าง โดย 178 ไม่มีประวัติจังหวัดในปี 2021
- Train 153 / Validation 52 / Test 48 / Purged 15
- ใช้ `class_weight='balanced'` กับ LR/DT (ถ้าใช้ default ทั้งสามโมเดลทาย Negative หมด F1=0)
- Validation F1: LR 0.273, DT 0.235, KNN 0; Always Positive 0.267 → เลือก Logistic Regression
- Test LR: accuracy 0.583, recall 0.444 (จับ Positive ได้ 4 จาก 9), F1 0.286 ต่ำกว่า Always Positive (0.316) จึงยังไม่พร้อมใช้เลือกพื้นที่ภาคสนาม
- `no risk` ไม่รับรองความปลอดภัย; คะแนนยังไม่ calibrated และยังไม่แบ่ง low/medium/high

ผลรุ่นนี้เป็น F1 จาก validation ครั้งเดียว ไม่ใช่ CV F2 รุ่นเก่า ห้ามนำตัวเลขมาปะปนกัน

## การตรวจ

รัน notebook รวมจากโฟลเดอร์ทดสอบใหม่ที่มีเพียง notebook กับ Excel ข้าวต้นฉบับ โดยใช้ JSON ตอบกลับ NASA จริงที่บันทึกไว้ 22 จุดแทน network request สดในรอบทดสอบ Notebook สร้าง NASA_POWER.xlsx ภายในขั้น Data Collection เอง แล้วอ่าน Excel และทำโมเดลต่อครบ Assertions และ notebook schema ผ่าน ผล cleansing/validation/test ตรงกับเวอร์ชันแยกไฟล์

ยังไม่ได้ทดสอบหน้า Colab จริงหรือ network request สดของ notebook รวม

## ก่อนส่งงาน

เติมชื่อ รหัส หน้าที่สมาชิกทุกคนในบรรทัดแรก เปลี่ยนชื่อ notebook นี้เป็น `Sectionxxxxxx_Gyy_ชื่อกลุ่ม.ipynb` แล้วทำ PDF ชื่อเดียวกันตาม instruction โดยใส่ภาพผลรันจริงและอธิบายส่วน mock ให้ชัด รวม **ipynb หนึ่งไฟล์ + PDF** เป็น ZIP ส่งงาน

ใช้ `docs/SLIDE_HANDOFF_TH.md` สำหรับทำสไลด์ ยังไม่ได้สร้าง PDF หรือกรอกรายชื่อแทนกลุ่ม ไฟล์ 01_NASA_to_Excel.ipynb และ 02_Rice_Risk_CS240.ipynb ใน workspace เป็นรุ่นก่อน ไม่ต้องใช้หรือส่งสองไฟล์นั้นแทน notebook รวม
