# Handoff สไลด์ใหม่ — เวอร์ชัน CS240 อ่านง่าย

ใช้เฉพาะผลจาก `Rice_Risk_CS240_Colab.ipynb` และ `outputs/Rice_Project_Results.xlsx` ในโปรเจกต์นี้ ไม่ใช้ handoff/กราฟ/ตัวเลขของรุ่น TimeSeriesSplit เดิม เพราะวิธีแบ่งข้อมูล โมเดล และ metric เปลี่ยนแล้ว

**เรื่องที่จะเล่า:** ภายใน notebook เดียว: Data Collection ดึง NASA API มาเป็น Excel → cleansing ข้อมูลข้าว → merge location/date → ใช้ประวัติปี 2021 ทำนายตัวอย่างปี 2022 → เปรียบเทียบ 3 โมเดล → พบว่าผลยังไม่พร้อมใช้จริง

ทำ 13 หน้า / ประมาณ 15 นาที ข้อความสั้นบนสไลด์ และใส่รายละเอียดใน notes ทุกหน้าผลลัพธ์ติดป้าย **Scenario: วันเก็บและพิกัดสมมติ**

| หน้า | เวลา | ข้อความหลักบนสไลด์ | Notes / ภาพที่ใช้ |
|---|---:|---|---|
| 1 | 0:45 | ต้นแบบเลือกพื้นที่เก็บตัวอย่างข้าวด้วยประวัติ Mycotoxin และ NASA POWER | ชื่อ รหัส หน้าที่สมาชิกทุกคน; ยังไม่มีรายชื่อให้เว้นช่อง |
| 2 | 1:00 | คาดการณ์ผล Positive/Negative ของตัวอย่างครั้งใหม่ | Input พื้นที่/วัน + ประวัติ + อากาศ; output have risk/no risk; ไม่ใช่ตัดสินทั้งจังหวัด |
| 3 | 1:00 | Data Collection: Excel ข้าว + NASA API → NASA_POWER.xlsx | ผัง Data Collection → NASA_POWER.xlsx → Data Preparation ใน notebook เดียว; แสดง 4 sheet: Daily, Weather_30d, Locations, Sources |
| 4 | 1:15 | จริง: ผลตรวจ/จังหวัด/NASA weather; สมมติ: วันเก็บ/วันทราบผล/จุดตัวแทน | Weather จริงที่จุดสมมติไม่ใช่ weather ของแปลงจริงที่ยืนยันแล้ว |
| 5 | 1:15 | Cleansing: จังหวัด 44 → 22 รูปแบบ, อำเภอ 75 → 39, Site_ID 171 → 140, วันที่ 2 รูปแบบ, พิกัดมี N/E | ภาพ Dashboard_Mock_Detailed_Raw.xlsx และชีต Cleansing; strip/title/replace/to_datetime/drop_duplicates; อำเภอ/ตำบลชื่อจริงแต่จับคู่แบบสุ่ม |
| 6 | 1:15 | Join ด้วย Location + Date; อุณหภูมิเฉลี่ย/ความชื้นเฉลี่ย/ฝนสะสมก่อนวันทำนาย 30 วัน | แสดงตัวอย่างแถวใน Weather_30d กับ Merged_2022 และคำสั่ง merge สั้น ๆ |
| 7 | 1:15 | ประวัติปี 2021 คงที่ → ทำนายปี 2022 | 171 แถวใน 8 จังหวัดเป็น history; model 267 แถว; 177 ไม่มี history; ห้ามใช้ค่าสารตัวอย่างใหม่เป็น input |
| 8 | 1:30 | Train 151 / Validation 50 / Test 51 / Purged 15 | แบ่งตาม 60/20/20 ของวันไม่ซ้ำ ไม่ใช่จำนวนแถว; ตัดผลที่ยังไม่ทราบก่อนช่วงถัดไป; fillna/scaler fit เฉพาะ train |
| 9 | 1:30 | Logistic Regression, Decision Tree, KNN + 2 baselines | ใช้ validation F1; ภาพ validation_simple.png; class_weight='balanced' กับ LR/DT; F1: LR 0.400, DT 0, KNN 0, Always Positive 0.305 → เลือก LR |
| 10 | 1:15 | Test: จับ Positive ได้ 3 จาก 5 | test_simple.png: TN=29 FP=17 FN=2 TP=3; recall=0.6, F1=0.240 > Always Positive 0.179 แต่ Positive มีแค่ 5 ตัวอย่าง; ยังเลือกพื้นที่จริงไม่ได้ |
| 11 | 1:00 | Demo: เปลี่ยน LOCATION แล้วรัน cell | วันที่สมมติ 2023-01-01; Lampang ได้ no risk, score 0.269, history=0; ไม่ใช่ความน่าจะเป็นที่สอบเทียบแล้ว และไม่รับรองปลอดภัย |
| 12 | 1:00 | Insights: ประวัติไม่ครบ คลาสไม่สมดุล และวัน mock ทำให้ความสัมพันธ์ยังไม่ยืนยัน | eda_simple.png; ไม่อ้างว่าเพิ่ม weather แล้วดีขึ้นหรือไม่มีประโยชน์ เพราะยังไม่มี ablation |
| 13 | 1:00 | ได้ pipeline ที่อธิบายได้ แต่ยังไม่ได้โมเดลพร้อมใช้ | ต่อไปเก็บวัน/พิกัด/ล็อต/วันทราบผลจริง ยืนยัน label แล้วประเมินใหม่; low/medium/high ยังไม่ทำ |

## ตัวเลขที่ห้ามสับสน

| ข้อมูล | ค่า |
|---|---:|
| Excel ทั้งหมด / ข้าวทั้งหมด | 507 / 446 |
| แยกปีนอก 2021–2022 / scenario | 6 / 440 |
| Dirty / Clean | 440 / 440 (ไฟล์ไม่มีแถวซ้ำ) |
| รูปแบบชื่อจังหวัด dirty / clean | 44 / 22 |
| ประวัติปี 2021 | 171 |
| ปี 2022 ต้นมกราคมที่ตัดออก | 2 |
| ข้อมูลโมเดลปี 2022 | 267 |
| ในชุดโมเดลที่ไม่มี history ของจังหวัด | 177 |
| Train / Validation / Test / Purged | 151 / 50 / 51 / 15 |
| Validation เริ่ม / Test เริ่ม (สมมติ) | 2022-08-01 / 2022-10-13 |
| Test Positive / Negative | 5 / 46 |
| Test LR accuracy / recall / F1 | 0.627 / 0.6 / 0.240 |
| Always Positive: validation F1 / test F1 | 0.305 / 0.179 |

**ไม่ได้ทำ Cross-validation ในรุ่นนี้** อย่าเรียกผล validation ว่า CV และอย่านำ F2 ของรุ่นก่อนมาเทียบกับ F1 รุ่นนี้ตรง ๆ

## สคริปต์อธิบายโค้ดแบบสั้น

> เราทำทุกอย่างใน notebook เดียว เริ่มจาก Data Collection วนเรียก NASA API แล้วเก็บเป็น Excel เพื่อเปิดตรวจข้อมูลได้ จากนั้นอ่านชีตอากาศมาใช้ต่อใน Data Preparation ด้วย pandas ที่เรียน เช่น replace, groupby และ merge จากนั้นใช้ get_dummies และ StandardScaler แล้ว fit/predict โมเดลสามตัว ส่วนที่ต้องเพิ่มความระวังคือใช้ประวัติเก่าเท่านั้น และแบ่งตามเวลา เพราะโจทย์คือทำนายตัวอย่างครั้งใหม่

## แนวตอบคำถาม

- **ทำไมใช้ปี 2021 เป็นประวัติคงที่?** ลดโค้ดให้เข้าใจง่ายและกันการใช้คำตอบปี 2022 แต่แลกกับประวัติที่ไม่อัปเดตและไม่ครบจังหวัด
- **ทำไมไม่ใช้ train_test_split แบบ Iris?** Iris ไม่ได้ถามเรื่องอนาคต แต่โจทย์นี้ต้องทำนายครั้งใหม่ จึงต้องแยกเวลาที่ข้อมูลทราบแล้วกับยังไม่ทราบ
- **ทำไมใช้ class_weight='balanced'?** Positive มีราว 15% ถ้าใช้ default ทุกโมเดลทาย Negative หมด (F1=0) ค่านี้ให้น้ำหนักคลาส Positive มากขึ้น ตัดสินจาก validation เท่านั้น ไม่ได้ดู test
- **Baseline ดีกว่าทำไมยังมี demo?** Demo แสดงการทำงานของโค้ดเท่านั้น ไม่ใช่ข้อเสนอให้ใช้งานจริง
- **NASA ละเอียดแล้วทำไมยังต้อง mock?** ข้อมูลอากาศละเอียดไม่ได้บอกวัน/พิกัดตัวอย่างที่ Excel ไม่มี
- **ค่าสาร >0 ทุกตัวคือ Positive หรือไม่?** ใช้ label เดิม พบ 2 แถวไม่ตรงกฎนี้ จึงต้องถามเจ้าของข้อมูลก่อนสร้าง label ใหม่
- **เพิ่ม weather แล้วดีขึ้นไหม?** ยังตอบไม่ได้ ต้องเปรียบเทียบ features บน split เดียวกัน; รุ่นก่อนกับรุ่นนี้เทียบตรง ๆ ไม่ได้

## Checklist ส่งต่อ

เติมสมาชิกครบทุกคน จัดผู้พูด 8–9 คนให้ทุกคนมีส่วนร่วมและซ้อม 15 นาที ใช้ภาพจากการรันจริง ไม่สร้าง screenshot Colab ปลอม Export PDF ชื่อเดียวกับ notebook ตาม instruction รายวิชา แนบคำอธิบาย mock ทุกหน้าที่มีผลโมเดล และใส่แหล่งข้อมูลท้ายหน้า Data Collection:

- Excel Rice_Mycotoxin_Dashboard.xlsx — เติมเจ้าของข้อมูลจริงจากทีม
- [NASA POWER Daily API](https://power.larc.nasa.gov/docs/services/api/temporal/daily/)
- Rice_Risk_CS240_Colab.ipynb และ Rice_Project_Results.xlsx ในชุดนี้

ยังไม่ได้สร้าง PowerPoint/PDF ส่งงาน หรือทดสอบหน้าจอ Colab จริง


การส่ง: เปลี่ยนชื่อ notebook รวมเป็น Sectionxxxxxx_Gyy_ชื่อกลุ่ม.ipynb แล้วส่งคู่กับ PDF ชื่อเดียวกัน ไม่ต้องส่ง notebook แยก 01/02 แทนไฟล์รวม
