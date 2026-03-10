# คู่มือวาด Petri Net ใน nd.app (TINA Toolbox)
## Aircraft Turnaround Process — P/T Petri Net Model

---

## ภาพรวมโมเดล

โมเดลนี้มี **3 เครื่องบิน (A1, A2, A3)** แต่ละลำมีขั้นตอนเดียวกัน และใช้ทรัพยากรร่วมกัน

**สรุปจำนวน node:**
- Places (วงกลม): **45 places** (13 ต่อเครื่องบิน + 6 shared resources)
- Transitions (สี่เหลี่ยม): **33 transitions** (11 ต่อเครื่องบิน)
- Arcs (ลูกศร): **126 arcs**

---

## ส่วนที่ 1 — พื้นฐานการใช้ nd.app

### 1.1 เปิดโปรแกรม

1. เปิด **nd.app** จาก Desktop
2. เลือก **File → New** เพื่อสร้าง net ใหม่
3. ใส่ชื่อ net: `Aircraft_Turnaround_3Aircraft`

### 1.2 วิธีสร้าง Place (วงกลม ○)

- **Double-click** บนพื้นที่ว่าง → เลือก **Place**
- หรือกด **Shortcut** สำหรับ Place แล้วคลิกบนพื้นที่
- **ตั้งชื่อ**: คลิกขวาที่ place → Properties → ใส่ชื่อ
- **ตั้ง Initial Marking** (จำนวน token): คลิกขวา → Properties → ใส่ตัวเลขใน Marking

### 1.3 วิธีสร้าง Transition (สี่เหลี่ยม □)

- **Double-click** บนพื้นที่ว่าง → เลือก **Transition**
- หรือกด shortcut สำหรับ Transition แล้วคลิก
- **ตั้งชื่อ**: คลิกขวา → Properties → ใส่ชื่อ
- สำหรับ P/T net ปกติ ไม่ต้องใส่ timing (หรือใส่ `[0,w[` ถ้าโปรแกรมถาม)

### 1.4 วิธีลาก Arc (ลูกศร →)

- คลิกที่ **ต้นทาง** (place หรือ transition)
- กด **Ctrl + drag** หรือใช้ arc tool แล้ว drag ไปยัง **ปลายทาง**
- Weight ปกติ = **1** (ไม่ต้องเปลี่ยน)

### 1.5 คำแนะนำการจัด Layout

- วาง **Shared Resources** ด้านขวามือ (แยกจากเส้นหลัก)
- วาง **เครื่องบินแต่ละลำ** เป็นแถวแนวนอน (A1 บน, A2 กลาง, A3 ล่าง)
- ส่วน Ground Handling (Fuel + Catering) วางแยกเป็น **2 แถวย่อย** ตรงกลาง

---

## ส่วนที่ 2 — วาด Shared Resources ก่อน (ด้านขวา)

วาง places เหล่านี้ไว้ที่มุมขวาของ canvas (จะมีลูกศรเชื่อมไปทุกเครื่องบิน)

| ชื่อ Place | Initial Marking | ความหมาย |
|---|---|---|
| `P_Runway` | **1** | Runway มี 1 เส้น |
| `P_Bay` | **1** | จอดได้ 1 Bay |
| `P_FuelTruck` | **1** | รถเติมน้ำมัน 1 คัน |
| `P_CateringTruck` | **1** | รถ Catering 1 คัน |
| `P_Apron` | **2** | Apron รับได้ 2 คัน พร้อมกัน |
| `P_GSEPermit` | **2** | ใบอนุญาต GSE 2 ใบ |

> **วิธีทำ:** สร้าง place 6 อัน → ตั้งชื่อตามตาราง → ตั้ง Marking ตามคอลัมน์ขวา

---

## ส่วนที่ 3 — วาดเครื่องบิน A1 (แถวบน)

### 3.1 สร้าง Places ของ A1

สร้างวงกลมทั้ง 13 อัน เรียงจากซ้ายไปขวา:

| ลำดับ | ชื่อ Place | Marking | ความหมาย |
|---|---|---|---|
| 1 | `P_A1_Ready` | **1** | เครื่องพร้อมบิน |
| 2 | `P_A1_OnRunway` | 0 | อยู่บน Runway |
| 3 | `P_A1_Docked` | 0 | จอดที่ Bay แล้ว |
| 4 | `P_A1_ReadyFuel` | 0 | พร้อมเติมน้ำมัน |
| 5 | `P_A1_ReadyCat` | 0 | พร้อม Catering |
| 6 | `P_A1_Fueling` | 0 | กำลังเติมน้ำมัน |
| 7 | `P_A1_Catering` | 0 | กำลัง Catering |
| 8 | `P_A1_FuelDone` | 0 | เติมน้ำมันเสร็จ |
| 9 | `P_A1_CatDone` | 0 | Catering เสร็จ |
| 10 | `P_A1_GHDone` | 0 | Ground Handling เสร็จ |
| 11 | `P_A1_Boarding` | 0 | กำลัง Boarding |
| 12 | `P_A1_Departing` | 0 | พร้อม Depart |
| 13 | `P_A1_Done` | 0 | Takeoff สำเร็จ |

> **หมายเหตุ Layout:** Places 1–3, 10–13 วางในแถวเดียวกัน (แนวนอน)
> Places 4, 6, 8 วางแถวบน (Fuel lane) / Places 5, 7, 9 วางแถวล่าง (Catering lane)

### 3.2 สร้าง Transitions ของ A1

สร้างสี่เหลี่ยมทั้ง 11 อัน วางระหว่าง places:

| ลำดับ | ชื่อ Transition | ความหมาย |
|---|---|---|
| 1 | `T_A1_Land` | ลงจอด |
| 2 | `T_A1_Dock` | แล่นเข้า Bay |
| 3 | `T_A1_Fork` | **AND-fork** (แยกเป็น 2 งาน) |
| 4 | `T_A1_StartFuel` | เริ่มเติมน้ำมัน |
| 5 | `T_A1_EndFuel` | เสร็จเติมน้ำมัน |
| 6 | `T_A1_StartCat` | เริ่ม Catering |
| 7 | `T_A1_EndCat` | เสร็จ Catering |
| 8 | `T_A1_Join` | **AND-join** (รวม 2 งาน) |
| 9 | `T_A1_Board` | Boarding ผู้โดยสาร |
| 10 | `T_A1_Pushback` | ดัน/ออกจาก Bay |
| 11 | `T_A1_Takeoff` | Takeoff |

### 3.3 ลาก Arcs ของ A1 (เส้นหลัก)

ลากลูกศรตามลำดับต่อไปนี้:

**ขั้นตอน Landing → Docking:**
```
P_A1_Ready     → T_A1_Land
P_Runway       → T_A1_Land      ← จาก Shared Resource
T_A1_Land      → P_A1_OnRunway
P_A1_OnRunway  → T_A1_Dock
P_Bay          → T_A1_Dock      ← จาก Shared Resource
T_A1_Dock      → P_A1_Docked
T_A1_Dock      → P_Runway       → คืน Runway
```

**ขั้นตอน AND-Fork:**
```
P_A1_Docked    → T_A1_Fork
T_A1_Fork      → P_A1_ReadyFuel
T_A1_Fork      → P_A1_ReadyCat
```

**ขั้นตอน Fueling (แถวบน):**
```
P_A1_ReadyFuel  → T_A1_StartFuel
P_FuelTruck     → T_A1_StartFuel   ← Shared Resource
P_Apron         → T_A1_StartFuel   ← Shared Resource
P_GSEPermit     → T_A1_StartFuel   ← Shared Resource
T_A1_StartFuel  → P_A1_Fueling
P_A1_Fueling    → T_A1_EndFuel
T_A1_EndFuel    → P_A1_FuelDone
T_A1_EndFuel    → P_FuelTruck      → คืน FuelTruck
T_A1_EndFuel    → P_Apron          → คืน Apron
T_A1_EndFuel    → P_GSEPermit      → คืน GSEPermit
```

**ขั้นตอน Catering (แถวล่าง):**
```
P_A1_ReadyCat    → T_A1_StartCat
P_CateringTruck  → T_A1_StartCat   ← Shared Resource
P_Apron          → T_A1_StartCat   ← Shared Resource
P_GSEPermit      → T_A1_StartCat   ← Shared Resource
T_A1_StartCat    → P_A1_Catering
P_A1_Catering    → T_A1_EndCat
T_A1_EndCat      → P_A1_CatDone
T_A1_EndCat      → P_CateringTruck → คืน CateringTruck
T_A1_EndCat      → P_Apron         → คืน Apron
T_A1_EndCat      → P_GSEPermit     → คืน GSEPermit
```

**ขั้นตอน AND-Join → Departure:**
```
P_A1_FuelDone   → T_A1_Join
P_A1_CatDone    → T_A1_Join
T_A1_Join       → P_A1_GHDone
P_A1_GHDone     → T_A1_Board
T_A1_Board      → P_A1_Boarding
P_A1_Boarding   → T_A1_Pushback
T_A1_Pushback   → P_A1_Departing
T_A1_Pushback   → P_Bay            → คืน Bay
P_A1_Departing  → T_A1_Takeoff
P_Runway        → T_A1_Takeoff     ← Shared Resource
T_A1_Takeoff    → P_A1_Done
T_A1_Takeoff    → P_Runway         → คืน Runway
```

---

## ส่วนที่ 4 — วาดเครื่องบิน A2 (แถวกลาง)

ทำซ้ำเหมือน A1 ทุกอย่าง **แต่เปลี่ยน A1 → A2 ในชื่อทั้งหมด** และวางไว้ใต้ A1

### Places ของ A2 (Marking เหมือน A1):
`P_A2_Ready(1)`, `P_A2_OnRunway(0)`, `P_A2_Docked(0)`, `P_A2_ReadyFuel(0)`, `P_A2_ReadyCat(0)`, `P_A2_Fueling(0)`, `P_A2_Catering(0)`, `P_A2_FuelDone(0)`, `P_A2_CatDone(0)`, `P_A2_GHDone(0)`, `P_A2_Boarding(0)`, `P_A2_Departing(0)`, `P_A2_Done(0)`

### Transitions ของ A2:
`T_A2_Land`, `T_A2_Dock`, `T_A2_Fork`, `T_A2_StartFuel`, `T_A2_EndFuel`, `T_A2_StartCat`, `T_A2_EndCat`, `T_A2_Join`, `T_A2_Board`, `T_A2_Pushback`, `T_A2_Takeoff`

### Arcs ของ A2:
ลากเหมือน A1 ทุก arc แต่เปลี่ยน A1→A2 ในชื่อ
Arc ที่เชื่อม Shared Resources ให้เชื่อมจาก **places เดิม** (P_Runway, P_Bay, P_FuelTruck, P_CateringTruck, P_Apron, P_GSEPermit) ไปยัง transitions ของ A2 โดยตรง

---

## ส่วนที่ 5 — วาดเครื่องบิน A3 (แถวล่าง)

ทำซ้ำเหมือน A1 และ A2 **แต่เปลี่ยน A1 → A3** และวางไว้ใต้ A2

### Places ของ A3:
`P_A3_Ready(1)`, `P_A3_OnRunway(0)`, `P_A3_Docked(0)`, `P_A3_ReadyFuel(0)`, `P_A3_ReadyCat(0)`, `P_A3_Fueling(0)`, `P_A3_Catering(0)`, `P_A3_FuelDone(0)`, `P_A3_CatDone(0)`, `P_A3_GHDone(0)`, `P_A3_Boarding(0)`, `P_A3_Departing(0)`, `P_A3_Done(0)`

### Transitions ของ A3:
`T_A3_Land`, `T_A3_Dock`, `T_A3_Fork`, `T_A3_StartFuel`, `T_A3_EndFuel`, `T_A3_StartCat`, `T_A3_EndCat`, `T_A3_Join`, `T_A3_Board`, `T_A3_Pushback`, `T_A3_Takeoff`

### Arcs ของ A3:
เหมือน A1/A2 ทุกอย่าง เพียงเปลี่ยน A3 ในชื่อ

---

## ส่วนที่ 6 — ตรวจสอบการเชื่อม Shared Resources

Shared Resource แต่ละตัวต้องมีลูกศรเชื่อมถึง **transitions ของทั้ง 3 เครื่องบิน** ตรวจดังนี้:

### P_Runway (1 token)
- Input จาก: `T_A1_Land`, `T_A2_Land`, `T_A3_Land` *(ใช้ก่อน Landing)*
- Input จาก: `T_A1_Takeoff`, `T_A2_Takeoff`, `T_A3_Takeoff` *(ใช้ก่อน Takeoff)*
- Output ไปยัง: `T_A1_Land`, `T_A2_Land`, `T_A3_Land`
- Output ไปยัง: `T_A1_Takeoff`, `T_A2_Takeoff`, `T_A3_Takeoff`

> คำอธิบาย: Runway ถูกใช้ตอน Landing **และ** ตอน Takeoff แต่ละครั้ง จะคืนทันทีหลังใช้

### P_Bay (1 token)
- Input จาก: `T_A1_Dock`, `T_A2_Dock`, `T_A3_Dock`
- Output ไปยัง: `T_A1_Dock`, `T_A2_Dock`, `T_A3_Dock`
- คืนโดย: `T_A1_Pushback`, `T_A2_Pushback`, `T_A3_Pushback`

### P_FuelTruck (1 token)
- Input จาก: `T_A1_StartFuel`, `T_A2_StartFuel`, `T_A3_StartFuel`
- Output ไปยัง: `T_A1_StartFuel`, `T_A2_StartFuel`, `T_A3_StartFuel`
- คืนโดย: `T_A1_EndFuel`, `T_A2_EndFuel`, `T_A3_EndFuel`

### P_CateringTruck (1 token)
- Input จาก: `T_A1_StartCat`, `T_A2_StartCat`, `T_A3_StartCat`
- Output ไปยัง: `T_A1_StartCat`, `T_A2_StartCat`, `T_A3_StartCat`
- คืนโดย: `T_A1_EndCat`, `T_A2_EndCat`, `T_A3_EndCat`

### P_Apron (2 tokens)
- Input จาก: `T_A1_StartFuel`, `T_A1_StartCat`, `T_A2_StartFuel`, `T_A2_StartCat`, `T_A3_StartFuel`, `T_A3_StartCat`
- Output ไปยัง: transitions เดิมทั้ง 6
- คืนโดย: `T_Ax_EndFuel`, `T_Ax_EndCat` ทุกตัว

### P_GSEPermit (2 tokens)
- เหมือน P_Apron ทุกประการ

---

## ส่วนที่ 7 — ตรวจสอบก่อน Save

### รายการตรวจ:
- [ ] Places ทั้งหมด: **45** (นับ: 6 shared + 13×3 aircraft)
- [ ] Transitions ทั้งหมด: **33** (11×3 aircraft)
- [ ] Arcs ทั้งหมด: **126**
- [ ] `P_A1_Ready`, `P_A2_Ready`, `P_A3_Ready` มี marking = **1**
- [ ] `P_Runway` มี marking = **1**
- [ ] `P_Bay` มี marking = **1**
- [ ] `P_FuelTruck` มี marking = **1**
- [ ] `P_CateringTruck` มี marking = **1**
- [ ] `P_Apron` มี marking = **2**
- [ ] `P_GSEPermit` มี marking = **2**
- [ ] Place อื่นๆ ทั้งหมด marking = **0**
- [ ] ทุก AND-Fork (`T_Ax_Fork`) มี **2 output arcs** ออกไป
- [ ] ทุก AND-Join (`T_Ax_Join`) มี **2 input arcs** เข้ามา

### วิธีนับ arc ใน nd.app:
ไปที่ **Edit → Select All** แล้วดูจำนวน elements ที่เลือกทั้งหมด
หรือดูที่ status bar ด้านล่าง

---

## ส่วนที่ 8 — Save และ Export

1. **Save**: `File → Save As` → เลือก format `.ndr` → ตั้งชื่อ `aircraft_turnaround`
2. **Verify**: ลอง `File → Close` แล้ว `File → Open` ไฟล์ที่ save ไว้
3. **Export เป็น .net** (สำหรับใช้กับ TINA เพื่อ verification):
   `File → Export → .net format`

---

## ส่วนที่ 9 — Diagram แสดง Layout โครงสร้าง

```
[P_A1_Ready]─→[T_A1_Land]─→[P_A1_OnRunway]─→[T_A1_Dock]─→[P_A1_Docked]
                  ↑↓ P_Runway                     ↑↓ P_Bay       │
                                                               [T_A1_Fork]
                                                              ↙           ↘
                                           [P_A1_ReadyFuel]   [P_A1_ReadyCat]
                                                  │                  │
                                           [T_A1_StartFuel]   [T_A1_StartCat]
                                           ↑ P_FuelTruck      ↑ P_CateringTruck
                                           ↑ P_Apron          ↑ P_Apron
                                           ↑ P_GSEPermit      ↑ P_GSEPermit
                                                  │                  │
                                           [P_A1_Fueling]     [P_A1_Catering]
                                                  │                  │
                                           [T_A1_EndFuel]     [T_A1_EndCat]
                                                  │                  │
                                           [P_A1_FuelDone]    [P_A1_CatDone]
                                                  ↘           ↙
                                               [T_A1_Join]
                                                    │
                                            [P_A1_GHDone]─→[T_A1_Board]─→[P_A1_Boarding]
                                                                               │
                                                                          [T_A1_Pushback]─→ P_Bay คืน
                                                                               │
                                                                          [P_A1_Departing]─→[T_A1_Takeoff]─→[P_A1_Done]
                                                                                               ↑↓ P_Runway
```

*(วาดซ้ำ pattern นี้สำหรับ A2 และ A3 ในแถวถัดลงไป)*

---

## ส่วนที่ 10 — เคล็ดลับ

- **ใช้ Grid**: เปิด View → Grid เพื่อจัดตำแหน่งได้สม่ำเสมอ
- **Copy-Paste**: วาด A1 เสร็จแล้ว Select ทั้งหมด (ยกเว้น Shared Resources) → Copy → Paste → เปลี่ยนชื่อ A1 เป็น A2 ทีละอัน
- **Zoom out**: ใช้ Ctrl+- เพื่อดูภาพรวม
- **คืน resource (arc วนกลับ)**: arc จาก transition กลับไป shared resource place ต้องทำทุกครั้งที่มีการใช้ทรัพยากร

---

*คู่มือนี้อ้างอิงโมเดล `aircraft_turnaround.ndr` ที่สร้างสำหรับ paper: "Formal Verification of Aircraft Turnaround Process Using Petri Net"*
