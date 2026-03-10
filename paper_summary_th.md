# สรุป Paper: การตรวจสอบเชิงรูปนัยของกระบวนการ Aircraft Turnaround ด้วย Petri Net

**ชื่อเรื่อง:** Formal Verification of Aircraft Turnaround Process Using Petri Net for Deadlock Detection and Liveness Analysis

**ผู้เขียน:** Titi Changpoo, Nuengwong Tuaycharoen, Wiwat Vatanawood
**สถาบัน:** ภาควิชาวิศวกรรมคอมพิวเตอร์ จุฬาลงกรณ์มหาวิทยาลัย

---

## 1. ที่มาและปัญหา

กระบวนการ **Aircraft Turnaround** คือชุดของการดำเนินงานที่ต้องทำกับเครื่องบินนับตั้งแต่ลงจอด จนถึงออกเดินทางเที่ยวบินถัดไป กระบวนการนี้ส่งผลโดยตรงต่อ **On-Time Performance (OTP)** ของสายการบิน และเกี่ยวข้องกับหลายหน่วยงาน ได้แก่ AOT, AEROTHAI และทีม Ground Handling ของแต่ละสายการบิน

ความท้าทายหลักของกระบวนการนี้คือ **Ground Handling Phase** ซึ่งมีทีมงานหลายทีมทำงานพร้อมกัน เช่น การเติมน้ำมัน, การส่งอาหาร (Catering), การทำความสะอาดห้องโดยสาร, การขนสัมภาระ และการตรวจซ่อมบำรุง โดยทุกทีมต้องแย่งใช้ทรัพยากรร่วมกัน เช่น พื้นที่ Apron และอุปกรณ์ Ground Support (GSE)

ปัญหาที่อาจเกิดขึ้นจากการทำงานพร้อมกันบนทรัพยากรที่มีจำกัด คือ **Deadlock** — สถานการณ์ที่ทุกกระบวนการต่างรอทรัพยากรจากกัน ส่งผลให้ระบบหยุดชะงักโดยสมบูรณ์ วิธีการที่ใช้อยู่เดิม เช่น Simulation และ PERT ไม่สามารถพิสูจน์ได้อย่างเป็นทางการว่าระบบปลอดจาก Deadlock ในทุกกรณีที่เป็นไปได้

---

## 2. วิธีการ (Methodology)

งานวิจัยนี้ใช้ **Place/Transition (P/T) Petri Net** เพื่อสร้างแบบจำลองกระบวนการ Aircraft Turnaround และใช้เครื่องมือ **TINA (TIme petri Net Analyzer)** ในการวิเคราะห์ผ่าน 2 วิธีหลัก:

- **State Space Analysis** — สร้าง Reachability Graph เพื่อค้นหา Dead Marking (Deadlock)
- **LTL Model Checking** — ตรวจสอบ Safety และ Liveness properties ด้วย Linear Temporal Logic

### สิ่งที่ Petri Net ใช้แทน

| องค์ประกอบ | ความหมายในโมเดล |
|---|---|
| **Place (วงกลม)** | สถานะของระบบ เช่น เครื่องบินอยู่บน Runway |
| **Transition (สี่เหลี่ยม)** | เหตุการณ์ที่เกิดขึ้น เช่น การลงจอด |
| **Arc (ลูกศร)** | การไหลของ Token ระหว่าง Place และ Transition |
| **Token (จุด)** | ทรัพยากรหรือสถานะที่มีอยู่ ณ ขณะนั้น |

---

## 3. โมเดล Petri Net ที่สร้างขึ้น

### สถานการณ์จำลอง
- **เครื่องบิน 3 ลำ (A1, A2, A3)** ใช้ทรัพยากรร่วมกัน
- **Runway 1 เส้น**, **Gate/Bay 1 ช่อง**, **รถเติมน้ำมัน 1 คัน**, **รถ Catering 1 คัน**
- **Apron Zone 2 ช่อง** (จำกัดจำนวน GSE ที่ทำงานพร้อมกัน)

### จำนวน Node ในโมเดล
- **Places:** 45 (13 สถานะต่อเครื่องบิน × 3 + 6 Shared Resources)
- **Transitions:** 33 (11 เหตุการณ์ต่อเครื่องบิน × 3)
- **Arcs:** 126

### ขั้นตอน 13 สถานะของแต่ละเครื่องบิน
Landing → Taxi-in → Docking → Debarkation → **Refueling ‖ Catering** (ขนานกัน) → Weight & Balance → Boarding → Pushback → Takeoff

### AND-Fork/AND-Join Pattern
หลังจาก Docking เครื่องบินจะเข้าสู่ **AND-Fork** เพื่อเริ่มงาน Ground Handling หลายอย่างพร้อมกัน (Refueling และ Catering) และรอให้ทุกงานเสร็จก่อนผ่าน **AND-Join** เพื่อไปสู่ขั้นตอน Boarding

---

## 4. คุณสมบัติที่ตรวจสอบ

### Safety Properties (สิ่งที่ต้องไม่เกิดขึ้น)

| Property | เงื่อนไข | สูตร LTL |
|---|---|---|
| S1 | ห้ามเครื่องบิน 2 ลำใช้ Runway พร้อมกัน | `G(¬(Runway_A1 ∧ Runway_A2))` |
| S2 | ห้ามเครื่องบิน 2 ลำจอด Gate พร้อมกัน | `G(¬(Bay_A1 ∧ Bay_A2))` |
| S3 | ห้าม Pushback ขณะกำลังเติมน้ำมัน | `G(Pushback_Ai → ¬Refueling_Ai)` |
| S4 | Boarding ได้ต่อเมื่อ Ground Handling เสร็จทุกอย่าง | `G(Boarding_Ai → RefuelDone ∧ CatDone ∧ ...)` |
| S5 | จำนวน GSE ใน Apron ต้องไม่เกิน 2 คัน | `G(Apron_Occupied ≤ 2)` |
| S6 | ห้ามเติมน้ำมันขณะผู้โดยสาร Boarding | `G(Boarding_Ai → ¬Refueling_Ai)` |

### Liveness Properties (สิ่งที่ต้องเกิดขึ้นในที่สุด)

| Property | เงื่อนไข | สูตร LTL |
|---|---|---|
| L1 | เครื่องบินทุกลำที่ลงจอดต้อง Takeoff ได้ | `G(Landing_Ai → F Takeoff_Ai)` |
| L2 | ทุก Ground Handling ที่เริ่มต้องเสร็จสิ้น | `G(StartRefuel_Ai → F RefuelDone_Ai)` |
| L3 | ไม่มีกระบวนการใดรอทรัพยากรตลอดไป (No Starvation) | `G(Waiting_Ai → F Refueling_Ai)` |

---

## 5. ผลการวิเคราะห์

### Deadlock ที่พบในโมเดลเริ่มต้น

โมเดลเริ่มต้นมีศักยภาพที่จะเกิด **Deadlock** จาก Circular Wait บน Apron Zone:

1. เครื่องบิน A1 ครอบครอง Apron Zone 1 (รถเติมน้ำมัน) และรอ Zone 2 สำหรับ Catering
2. เครื่องบิน A2 ครอบครอง Apron Zone 2 (รถ Catering) และรอ Zone 1
3. ผลลัพธ์: ทั้งสองรอกันอยู่ ไม่มีใครเดินหน้าได้

### การแก้ไข Deadlock

เพิ่ม **P_GSEPermit** — Control Place ที่มี Token เริ่มต้น 2 ตัว ทุก Ground Handling Transition ต้องขอ GSEPermit ก่อนเข้า Apron Zone และคืนเมื่อเสร็จงาน วิธีนี้จำกัดจำนวน GSE ที่ทำงานพร้อมกันไม่เกิน 2 คัน ตัดวงจรของ Circular Wait ได้โดยสมบูรณ์

### ผลการ Verify หลังแก้ไข

| หมวด | ผลลัพธ์ |
|---|---|
| Dead Markings (Deadlock) | **0** — ระบบปลอด Deadlock |
| Safety Properties S1–S6 | **ผ่านทั้งหมด** |
| Liveness Properties L1–L3 | **ผ่านทั้งหมด** |

---

## 6. บทสรุป

งานวิจัยนี้แสดงให้เห็นว่า **Formal Verification ด้วย Petri Net** สามารถพิสูจน์ความถูกต้องของกระบวนการ Aircraft Turnaround ได้อย่างเป็นรูปนัย โดย:

- **ค้นพบ Deadlock** ที่ซ่อนอยู่ในโมเดลเริ่มต้น ซึ่งการ Simulation ปกติอาจตรวจไม่พบ
- **แก้ไขและพิสูจน์** ว่าโมเดลที่แก้แล้วปลอดจาก Deadlock ในทุก Execution Path ที่เป็นไปได้
- **ตรวจสอบ Safety และ Liveness** ครบ 9 properties ผ่านทั้งหมด
- แนวทางนี้สามารถนำไปใช้เป็น **ส่วนเสริม SOP** เพื่อยืนยันความปลอดภัยของ Ground Operations ก่อนนำไปใช้จริง

### ข้อจำกัดและงานในอนาคต
- โมเดลปัจจุบันใช้ Untimed Petri Net (ไม่มีมิติเวลา) — ขยายได้ด้วย **Time Petri Net**
- ยังเป็นสนามบินขนาดเล็ก (3 เครื่องบิน, 1 Gate) — สามารถขยายด้วย **Compositional Verification**
- ยังไม่รวม Stochastic Events เช่น อุปกรณ์เสียหรือสภาพอากาศ

---

*Paper นี้เป็นส่วนหนึ่งของงานวิจัยระดับปริญญาตรี ภาควิชาวิศวกรรมคอมพิวเตอร์ จุฬาลงกรณ์มหาวิทยาลัย ส่ง JCSSE 2026*
