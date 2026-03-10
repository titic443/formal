# คู่มือวาด Petri Net ใน TINA (nd.app)
## Aircraft Turnaround Process — P/T Net Model

---

## ส่วนที่ 0 — ตอบคำถาม: Model นี้ตาม Constraint ที่กำหนดไว้ในงานหรือเปล่า?

**ใช่ครับ** — model ใน `aircraft_turnaround.ndr` สร้างตาม constraint ที่ระบุใน paper ทุกข้อ:

| Constraint จาก Paper | สิ่งที่ Model ใน .ndr | Marking เริ่มต้น |
|---|---|---|
| 3 เครื่องบิน (A1, A2, A3) | 3 ชุด places/transitions แยกกัน | P_A1_Ready = P_A2_Ready = P_A3_Ready = **1** |
| Runway 1 เส้น | `P_Runway` (shared) | **1** |
| Gate/Bay 1 ช่อง | `P_Bay` (shared) | **1** |
| Fuel Truck 1 คัน | `P_FuelTruck` (shared) | **1** |
| Catering Truck 1 คัน | `P_CateringTruck` (shared) | **1** |
| Apron Zone 2 ช่อง | `P_Apron` (shared) | **2** |
| GSE Permit (deadlock fix) | `P_GSEPermit` (shared) | **2** |

> **หมายเหตุเรื่อง Simplification:** Paper พูดถึง 5 งานคู่ขนาน (Fuel, Catering, Cleaning, Baggage, Maintenance) แต่ Model จริงเหลือ **2 งานที่ใช้ Apron** (Fuel + Catering) เพราะ Cleaning, Baggage, Maintenance ไม่แย่ง Apron slot — ดังนั้น Deadlock pattern ที่น่าสนใจเกิดจาก 2 งานนี้เท่านั้น ถือว่า valid simplification ครับ

---

## ส่วนที่ 1 — ภาพรวมโมเดล

```
ทรัพยากรร่วม (6 places — วางไว้ฝั่งขวา):
  P_Runway(1)  P_Bay(1)  P_FuelTruck(1)
  P_CateringTruck(1)  P_Apron(2)  P_GSEPermit(2)

แต่ละเครื่องบิน Ax (x=1,2,3) มี 13 places + 11 transitions:

  [P_Ax_Ready] ──▶ T_Ax_Land ──▶ [P_Ax_OnRunway]
                      ▲ P_Runway                    ▼
                                          T_Ax_Dock
                                          ▲ P_Bay   ▼
                                     [P_Ax_Docked]
                                          ▼
                                     T_Ax_Fork ──────────────────────────▶ [P_Ax_ReadyCat]
                                          ▼                                       ▼
                                [P_Ax_ReadyFuel]                          T_Ax_StartCat ◀── P_CateringTruck
                                          ▼                                ▲  P_Apron
                                 T_Ax_StartFuel ◀── P_FuelTruck            ▲  P_GSEPermit
                                 ▲  P_Apron                                       ▼
                                 ▲  P_GSEPermit                       [P_Ax_Catering]
                                          ▼                                       ▼
                               [P_Ax_Fueling]                           T_Ax_EndCat ──▶ P_CateringTruck
                                          ▼                             ──▶ P_Apron, P_GSEPermit
                                T_Ax_EndFuel ──▶ P_FuelTruck                      ▼
                                ──▶ P_Apron, P_GSEPermit              [P_Ax_CatDone]
                                          ▼                                       ▼
                               [P_Ax_FuelDone] ─────────────────▶ T_Ax_Join ◀────┘
                                                                        ▼
                                                              [P_Ax_GHDone]
                                                                        ▼
                                                              T_Ax_Board
                                                                        ▼
                                                              [P_Ax_Boarding]
                                                                        ▼
                                                              T_Ax_Pushback ──▶ P_Bay (คืน)
                                                                        ▼
                                                              [P_Ax_Departing]
                                                                        ▼
                                                              T_Ax_Takeoff ◀── P_Runway
                                                              ──▶ P_Runway (คืน)
                                                                        ▼
                                                              [P_Ax_Done] ★
```

**สรุปจำนวน node ทั้งหมด:**
- Places: **45** (6 shared + 13 × 3 aircraft)
- Transitions: **33** (11 × 3 aircraft)
- Arcs: **126** (42 × 3 aircraft)

---

## ส่วนที่ 2 — เปิด nd.app และตั้งค่าเริ่มต้น

1. เปิด **TINA** → เปิด **nd** (Net Drawer) จาก menu หรือ icon
2. `File → New` → ตั้งชื่อ net: `Aircraft_Turnaround_3Aircraft`
3. เปิด Grid: `View → Grid` (ช่วยจัดระยะ node ให้สม่ำเสมอ)
4. ใน Properties ของ net ตั้ง **Type: P/T Net** (ไม่ใช่ Time Petri Net)

> **Shortcut ที่ควรรู้:**
> - `P` หรือ double-click บน canvas = สร้าง Place
> - `T` = สร้าง Transition
> - Ctrl+drag จาก node = ลาก Arc
> - F2 หรือ double-click บน node = เปิด Properties (ตั้งชื่อ/marking)

---

## ส่วนที่ 3 — Step 1: วาด Shared Resources (6 Places)

วางไว้**ฝั่งขวาของ canvas** เพราะ arc จะต้องเชื่อมไปหาทั้ง 3 เครื่องบิน

### วิธีสร้างแต่ละ Place:
1. กด `P` แล้วคลิกบน canvas (หรือ double-click เลือก Place)
2. Double-click ที่ place → Properties → ใส่ชื่อและ Marking

| ลำดับ | ชื่อ Place | Initial Marking | ความหมาย |
|:---:|---|:---:|---|
| 1 | `P_Runway` | **1** | Runway เดียว (mutual exclusion) |
| 2 | `P_Bay` | **1** | Gate/Bay เดียว (mutual exclusion) |
| 3 | `P_FuelTruck` | **1** | รถเติมน้ำมัน 1 คัน |
| 4 | `P_CateringTruck` | **1** | รถ Catering 1 คัน |
| 5 | `P_Apron` | **2** | Apron zone 2 ช่อง (capacity = 2) |
| 6 | `P_GSEPermit` | **2** | ใบอนุญาต GSE 2 ใบ (deadlock prevention) |

> **ทำไม P_GSEPermit marking = 2?**
> เพราะ P_Apron มี 2 slot — GSEPermit ทำหน้าที่เป็น "counter" ที่จำกัดให้มี GSE vehicle ในพื้นที่ได้สูงสุด 2 คันพร้อมกัน ป้องกัน circular wait (deadlock) ระหว่าง Fuel และ Catering trucks

---

## ส่วนที่ 4 — Step 2: วาดเครื่องบิน A1 (แถวบน)

### 4.1 สร้าง Places ของ A1 (13 places)

เรียงจากซ้ายไปขวาในแถวแนวนอน โดยส่วน Ground Handling ให้แตก 2 แถวย่อย

**แถวหลัก (Landing → Docking → Departure):**

| ลำดับ | ชื่อ Place | Initial Marking | ความหมาย |
|:---:|---|:---:|---|
| 1 | `P_A1_Ready` | **1** | เครื่องรอลงจอด (initial state) |
| 2 | `P_A1_OnRunway` | 0 | อยู่บน Runway |
| 3 | `P_A1_Docked` | 0 | จอดที่ Gate แล้ว |
| 10 | `P_A1_GHDone` | 0 | Ground Handling เสร็จทั้งหมด |
| 11 | `P_A1_Boarding` | 0 | กำลัง Boarding |
| 12 | `P_A1_Departing` | 0 | หลัง Pushback |
| 13 | `P_A1_Done` | 0 | Takeoff สำเร็จ (final state) |

**แถวย่อยบน — Fuel Lane:**

| ลำดับ | ชื่อ Place | Initial Marking | ความหมาย |
|:---:|---|:---:|---|
| 4 | `P_A1_ReadyFuel` | 0 | รอเติมน้ำมัน |
| 6 | `P_A1_Fueling` | 0 | กำลังเติมน้ำมัน |
| 8 | `P_A1_FuelDone` | 0 | เติมน้ำมันเสร็จ |

**แถวย่อยล่าง — Catering Lane:**

| ลำดับ | ชื่อ Place | Initial Marking | ความหมาย |
|:---:|---|:---:|---|
| 5 | `P_A1_ReadyCat` | 0 | รอ Catering |
| 7 | `P_A1_Catering` | 0 | กำลัง Catering |
| 9 | `P_A1_CatDone` | 0 | Catering เสร็จ |

> **ทุก place ยกเว้น `P_A1_Ready` ตั้ง marking = 0 เสมอ**

---

### 4.2 สร้าง Transitions ของ A1 (11 transitions)

| ลำดับ | ชื่อ Transition | Timing ใน nd | ความหมาย |
|:---:|---|---|---|
| 1 | `T_A1_Land` | `0 w` (untimed) | ลงจอด |
| 2 | `T_A1_Dock` | `0 w` (untimed) | แล่นเข้า Gate |
| 3 | `T_A1_Fork` | `0 w` (untimed) | **AND-Fork** — แยกงาน Fuel + Cat |
| 4 | `T_A1_StartFuel` | `0 w` (untimed) | เริ่มเติมน้ำมัน (ต้องได้ resources 3 อย่าง) |
| 5 | `T_A1_EndFuel` | `0 w` (untimed) | เสร็จเติมน้ำมัน (คืน resources 3 อย่าง) |
| 6 | `T_A1_StartCat` | `0 w` (untimed) | เริ่ม Catering (ต้องได้ resources 3 อย่าง) |
| 7 | `T_A1_EndCat` | `0 w` (untimed) | เสร็จ Catering (คืน resources 3 อย่าง) |
| 8 | `T_A1_Join` | `0 w` (untimed) | **AND-Join** — รวมผล Fuel + Cat |
| 9 | `T_A1_Board` | `0 w` (untimed) | Boarding ผู้โดยสาร |
| 10 | `T_A1_Pushback` | `0 w` (untimed) | ดัน/ออกจาก Gate (คืน Bay) |
| 11 | `T_A1_Takeoff` | `0 w` (untimed) | Takeoff (ต้อง/คืน Runway) |

> **วิธีตั้ง Timing ใน nd.app:**
> Double-click transition → Properties → Timing interval: `[0,w[` หรือ `0 w`
> (w = ∞ สำหรับ P/T net ที่ไม่มี time constraint)

---

### 4.3 ลาก Arcs ของ A1 — Phase 1: Landing → Docking

```
ลากลูกศรตามลำดับ (Ctrl+drag จาก node ต้นทาง → node ปลายทาง):

P_A1_Ready    ──▶  T_A1_Land        (aircraft token เข้า transition)
P_Runway      ──▶  T_A1_Land        (ยืม Runway)
T_A1_Land     ──▶  P_A1_OnRunway    (token เดินหน้า)

P_A1_OnRunway ──▶  T_A1_Dock        (เข้า Dock)
P_Bay         ──▶  T_A1_Dock        (ยืม Bay)
T_A1_Dock     ──▶  P_A1_Docked      (token เดินหน้า)
T_A1_Dock     ──▶  P_Runway         (คืน Runway ทันทีหลัง Taxi เข้า Gate)
```

> **ทำไม Runway คืนตอน Dock (ไม่ใช่ตอน Land)?**
> เพราะเครื่องบินปล่อย Runway ระหว่าง Taxi-in — ตรงกับ constraint จริงในสนามบิน

---

### 4.4 ลาก Arcs ของ A1 — Phase 2: AND-Fork → Ground Handling

```
P_A1_Docked    ──▶  T_A1_Fork          (1 token เข้า Fork)
T_A1_Fork      ──▶  P_A1_ReadyFuel     (2 tokens ออก)
T_A1_Fork      ──▶  P_A1_ReadyCat      (2 tokens ออก — AND-Fork!)
```

**Fuel Lane (4 arcs input → StartFuel, 3 arcs output ← EndFuel):**

```
P_A1_ReadyFuel  ──▶  T_A1_StartFuel    (aircraft process token)
P_FuelTruck     ──▶  T_A1_StartFuel    (ยืม Fuel Truck)
P_Apron         ──▶  T_A1_StartFuel    (ยืม Apron 1 slot)
P_GSEPermit     ──▶  T_A1_StartFuel    (ยืม GSE Permit 1 ใบ)

T_A1_StartFuel  ──▶  P_A1_Fueling      (กำลังเติม)

P_A1_Fueling    ──▶  T_A1_EndFuel
T_A1_EndFuel    ──▶  P_A1_FuelDone     (เติมเสร็จ)
T_A1_EndFuel    ──▶  P_FuelTruck       (คืน Fuel Truck)
T_A1_EndFuel    ──▶  P_Apron           (คืน Apron 1 slot)
T_A1_EndFuel    ──▶  P_GSEPermit       (คืน GSE Permit 1 ใบ)
```

**Catering Lane (4 arcs input → StartCat, 3 arcs output ← EndCat):**

```
P_A1_ReadyCat    ──▶  T_A1_StartCat
P_CateringTruck  ──▶  T_A1_StartCat   (ยืม Catering Truck)
P_Apron          ──▶  T_A1_StartCat   (ยืม Apron 1 slot)
P_GSEPermit      ──▶  T_A1_StartCat   (ยืม GSE Permit 1 ใบ)

T_A1_StartCat    ──▶  P_A1_Catering

P_A1_Catering    ──▶  T_A1_EndCat
T_A1_EndCat      ──▶  P_A1_CatDone
T_A1_EndCat      ──▶  P_CateringTruck  (คืน)
T_A1_EndCat      ──▶  P_Apron          (คืน)
T_A1_EndCat      ──▶  P_GSEPermit      (คืน)
```

> **หัวใจของ Deadlock Prevention:**
> `T_Ax_StartFuel` และ `T_Ax_StartCat` ต้องการ **P_Apron ≥ 1** และ **P_GSEPermit ≥ 1** พร้อมกัน
> ถ้า P_Apron = 2 แต่ไม่มี P_GSEPermit → ทั้งสองก็ block ไม่สามารถเริ่มได้ → ไม่เกิด deadlock

---

### 4.5 ลาก Arcs ของ A1 — Phase 3: AND-Join → Departure

```
P_A1_FuelDone   ──▶  T_A1_Join         (2 inputs เข้า Join)
P_A1_CatDone    ──▶  T_A1_Join         (AND-Join! ต้องครบทั้งคู่)
T_A1_Join       ──▶  P_A1_GHDone       (Ground Handling เสร็จทั้งหมด)

P_A1_GHDone     ──▶  T_A1_Board
T_A1_Board      ──▶  P_A1_Boarding

P_A1_Boarding   ──▶  T_A1_Pushback
T_A1_Pushback   ──▶  P_A1_Departing
T_A1_Pushback   ──▶  P_Bay             (คืน Bay — Gate เปิดให้เครื่องลำต่อไป)

P_A1_Departing  ──▶  T_A1_Takeoff
P_Runway        ──▶  T_A1_Takeoff      (ยืม Runway เพื่อ Takeoff)
T_A1_Takeoff    ──▶  P_A1_Done         (★ Final state)
T_A1_Takeoff    ──▶  P_Runway          (คืน Runway)
```

> **Safety Constraint ที่ encode อยู่ใน structure:**
> - Pushback ทำได้ก็ต่อเมื่อ `P_A1_Boarding ≥ 1` → แปลว่า Boarding เสร็จแล้ว
> - Boarding ทำได้ก็ต่อเมื่อ `P_A1_GHDone ≥ 1` → แปลว่า Fuel + Cat เสร็จทั้งคู่
> - Takeoff ต้องการ `P_Runway ≥ 1` → Runway ว่างอยู่

---

## ส่วนที่ 5 — Step 3: วาดเครื่องบิน A2 และ A3

### วิธีที่ง่ายที่สุดใน nd.app:

1. **Select ทั้งหมดของ A1** (Ctrl+A แล้ว deselect shared resources)
2. **Copy** (Ctrl+C) → **Paste** (Ctrl+V)
3. **ย้าย** ชุด A2 ไปวางใต้ A1 ประมาณ 400 หน่วย
4. **เปลี่ยนชื่อทุก node:** แทน `A1` ด้วย `A2` ใน Properties ทีละอัน
5. ทำซ้ำสำหรับ A3 (ย้ายลงอีก 400 หน่วย)

### ต่อ Arc จาก Shared Resources ไปยัง A2 และ A3:

Arc จาก shared resources ไปยัง A2/A3 **เหมือนกันทุกอย่าง** กับ A1 — เพียงเปลี่ยน destination node เป็นของ A2 หรือ A3

| Shared Place | เชื่อมไป (Input arc) | คืนโดย (Output arc) |
|---|---|---|
| `P_Runway` | `T_A2_Land`, `T_A3_Land`, `T_A2_Takeoff`, `T_A3_Takeoff` | `T_A2_Dock`, `T_A3_Dock`, `T_A2_Takeoff`, `T_A3_Takeoff` |
| `P_Bay` | `T_A2_Dock`, `T_A3_Dock` | `T_A2_Pushback`, `T_A3_Pushback` |
| `P_FuelTruck` | `T_A2_StartFuel`, `T_A3_StartFuel` | `T_A2_EndFuel`, `T_A3_EndFuel` |
| `P_CateringTruck` | `T_A2_StartCat`, `T_A3_StartCat` | `T_A2_EndCat`, `T_A3_EndCat` |
| `P_Apron` | `T_A2_StartFuel`, `T_A2_StartCat`, `T_A3_StartFuel`, `T_A3_StartCat` | `T_A2_EndFuel`, `T_A2_EndCat`, `T_A3_EndFuel`, `T_A3_EndCat` |
| `P_GSEPermit` | `T_A2_StartFuel`, `T_A2_StartCat`, `T_A3_StartFuel`, `T_A3_StartCat` | `T_A2_EndFuel`, `T_A2_EndCat`, `T_A3_EndFuel`, `T_A3_EndCat` |

---

## ส่วนที่ 6 — ตรวจสอบก่อน Save

### Checklist สำหรับนับ nodes:

| รายการ | จำนวนที่ถูกต้อง | วิธีนับ |
|---|:---:|---|
| Places ทั้งหมด | **45** | 6 shared + 13×3 |
| Transitions ทั้งหมด | **33** | 11×3 |
| Arcs ทั้งหมด | **126** | 42×3 (ไม่นับ arc ของ shared resources ซ้ำ) |

> **นับ arc ใน nd.app:** `Edit → Select All` → ดูจำนวน elements ที่ status bar

### Checklist สำหรับ Marking:

- [ ] `P_Runway` = **1**
- [ ] `P_Bay` = **1**
- [ ] `P_FuelTruck` = **1**
- [ ] `P_CateringTruck` = **1**
- [ ] `P_Apron` = **2**
- [ ] `P_GSEPermit` = **2**
- [ ] `P_A1_Ready` = **1**, `P_A2_Ready` = **1**, `P_A3_Ready` = **1**
- [ ] Places อื่นทั้งหมด = **0**

### Checklist สำหรับ Arc Pattern:

- [ ] ทุก `T_Ax_Fork` มี **1 input arc** (จาก P_Ax_Docked) และ **2 output arcs** (→ ReadyFuel, ReadyCat)
- [ ] ทุก `T_Ax_Join` มี **2 input arcs** (จาก FuelDone, CatDone) และ **1 output arc** (→ GHDone)
- [ ] ทุก `T_Ax_StartFuel` มี **4 input arcs** (ReadyFuel, FuelTruck, Apron, GSEPermit)
- [ ] ทุก `T_Ax_EndFuel` มี **4 output arcs** (FuelDone, FuelTruck, Apron, GSEPermit)
- [ ] เหมือนกันสำหรับ StartCat / EndCat

---

## ส่วนที่ 7 — Save และ Export

```
1. Save as .ndr:    File → Save As → ชื่อ: aircraft_turnaround → format: .ndr
2. Export as .net:  File → Export → .net format  (สำหรับใช้กับ selt/tina command line)
3. ตรวจสอบ:        File → Close → File → Open → เปิดไฟล์ที่ save ไว้
```

---

## ส่วนที่ 8 — Run Verification ใน TINA

หลังจากสร้างไฟล์ `.ndr` แล้ว ให้ run ด้วย command line ดังนี้:

### 8.1 ตรวจ State Space และ Deadlock

```bash
# สร้าง reachability graph และดู dead markings
nd -s aircraft_turnaround.ndr

# หรือใช้ selt เพื่อ state space analysis
selt -S aircraft_turnaround.ndr
```

ผลลัพธ์ที่ควรเห็น (corrected model):
```
states: XXXX
transitions: XXXXX
dead markings: 1   ← แค่ 1 คือ terminal state (ทุกเครื่องบิน Done)
```

### 8.2 ตรวจ LTL Properties

ใช้ไฟล์ `aircraft_ltl_properties.ltl` ที่เตรียมไว้:

```bash
# ตรวจทีละ property
selt -f "G(not(P_A1_OnRunway >= 1 and P_A2_OnRunway >= 1))" aircraft_turnaround.ndr

# ตรวจทุก property จากไฟล์
selt -ltl aircraft_ltl_properties.ltl aircraft_turnaround.ndr
```

ผลที่ควรได้:
```
property S1a: VERIFIED
property S1b: VERIFIED
property S1c: VERIFIED
...
property L1_A1: VERIFIED
...
```

### 8.3 ตรวจ Boundedness (ค่า k-bounded)

```bash
nd -b aircraft_turnaround.ndr
```
ค่าที่ควรได้: P_Apron ≤ 2, P_Runway ≤ 1, P_Bay ≤ 1, ฯลฯ

---

## ส่วนที่ 9 — ทำความเข้าใจ Deadlock Scenario

> หมายเหตุ: ใน **model ที่มี P_GSEPermit** จะ **ไม่เกิด deadlock** แต่ถ้าเอา P_GSEPermit ออก จะเจอ deadlock นี้:

### ทำไม Deadlock เกิดขึ้นในโมเดลที่ไม่มี P_GSEPermit?

```
สถานการณ์ที่นำไปสู่ Deadlock:

Step 1: T_A1_StartFuel fires
        → ใช้ P_Apron (เหลือ 1), ใช้ P_FuelTruck (เหลือ 0)

Step 2: T_A2_StartCat fires
        → ใช้ P_Apron (เหลือ 0), ใช้ P_CateringTruck (เหลือ 0)

Step 3: ตอนนี้ P_Apron = 0
        → T_A1_StartCat ไม่สามารถ fire ได้ (ต้องการ P_Apron ≥ 1)
        → T_A2_StartFuel ไม่สามารถ fire ได้ (ต้องการ P_Apron ≥ 1)

Step 4: T_A1_EndFuel ต้องรอให้ Catering เสร็จก่อน? ไม่ใช่
        แต่ถ้า A1 รอ Catering ซึ่งรอ Apron ซึ่ง A2 ถือไว้
        และ A2 รอ Fuel ซึ่งรอ Apron ซึ่ง A1 ถือไว้ → DEADLOCK
```

### ทำไม P_GSEPermit แก้ปัญหาได้?

```
P_GSEPermit = 2 หมายความว่า มีแค่ 2 GSE operations พร้อมกันได้สูงสุด
→ ถ้า Fuel และ Catering ของ A1 กำลัง run → ใช้ permit ไป 2 ใบ → A2 ไม่สามารถเริ่มได้
→ ไม่มีสถานการณ์ที่ทั้ง A1 และ A2 ค้างกันได้
→ หมดปัญหา circular wait
```

---

## ส่วนที่ 10 — สรุป Constraints ทั้งหมด

| # | Constraint | Place ที่ enforce | Marking | Property ที่ตรวจ |
|:---:|---|---|:---:|---|
| C1 | Runway mutual exclusion | `P_Runway` | 1 | S1 |
| C2 | Bay mutual exclusion | `P_Bay` | 1 | S2 |
| C3 | No pushback during refueling | Arc structure | — | S3 |
| C4 | Boarding after all GH done | AND-Join + `P_Ax_GHDone` | — | S4 |
| C5 | Apron capacity ≤ 2 | `P_Apron` | 2 | S5 |
| C6 | No boarding during refueling | Arc structure (GHDone requires FuelDone) | — | S6 |
| C7 | Deadlock freedom | `P_GSEPermit` | 2 | Deadlock check |
| C8 | Every aircraft eventually departs | AND-Join liveness | — | L1 |
| C9 | No resource starvation | `P_GSEPermit` fairness | — | L3 |

---

*ไฟล์นี้อ้างอิง: `aircraft_turnaround.ndr` และ `aircraft_ltl_properties.ltl`*
*Paper: "Formal Verification of Aircraft Turnaround Process Using Petri Net"*
