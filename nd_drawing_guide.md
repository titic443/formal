# คู่มือวาด Petri Net ใน nd.app (TINA Toolbox)
## Aircraft Turnaround Process — P/T Petri Net Model

---

## ภาพรวมโมเดล

โมเดลนี้มี **3 เครื่องบิน (A1, A2, A3)** แต่ละลำมีขั้นตอนเดียวกัน และใช้ **resource pool ร่วมกัน** คือ FuelTruck, CateringTruck และ MaintCrew

**โมเดลนี้เป็น Cyclic (วนซ้ำ):** เมื่อเครื่องบิน Takeoff แล้ว จะวนกลับมาที่ `P_Ax_Ready` ได้ใหม่ (ไม่มี terminal state) — เหมาะสำหรับ modeling เที่ยวบินหมุนเวียน

**แนวคิดสำคัญ — Resource Pool (Floating Resources):**

ทรัพยากรแต่ละชิ้นเป็น token ที่ "ลอย" อยู่ในระบบ เมื่อทีมงานเสร็จกับเครื่องบินลำหนึ่ง token จะ "คืน" กลับมา และเครื่องบินลำใดก็ตามที่รอใช้อยู่จะรับไปทำต่อได้ทันที

**ตัวอย่าง concurrent scenario:**
```
A1: กำลังเติมน้ำมัน (ใช้ P_FuelTruck)
A1: กำลัง Catering (ใช้ P_CateringTruck)    ← ทำพร้อมกันใน A1
A2: กำลัง Maintenance (ใช้ P_MaintCrew)     ← ทำพร้อมกับ A1

เมื่อ A2 Maintenance เสร็จ:
  → P_MaintCrew token คืน → A1 หรือ A3 ที่รออยู่รับไปทำได้ทันที

เมื่อ A1 เติมน้ำมันเสร็จ:
  → P_FuelTruck token คืน → A2 หรือ A3 ที่รออยู่รับไปเติมน้ำมันได้ทันที
```

**P_Bay = 3 tokens** — stand รับได้ 3 ลำพร้อมกัน (ทั้ง 3 ลำ dock พร้อมกันได้เลย) ครองตั้งแต่ Dock จนถึง Pushback
**P_Apron = 2 tokens** — service zone สำหรับ GSE รับได้ 2 ลำพร้อมกัน ลำที่ 3 ที่ dock แล้วต้องรอ Apron ว่างก่อนถึงเริ่ม ground service ได้ — ครองตั้งแต่ Fork (เริ่ม GSE) จนถึง Board (รถทุกคันออกแล้ว)

**สรุปจำนวน node:**
- Places (วงกลม): **51 places** (15 ต่อเครื่องบิน × 3 + 6 shared resources)
- Transitions (สี่เหลี่ยม): **39 transitions** (13 ต่อเครื่องบิน × 3)
- Arcs (ลูกศร): **132 arcs** (44 ต่อเครื่องบิน × 3)

> **เปลี่ยนจากเดิม:** ลบ `P_Ax_Done` ออก (ลด 3 places) และเปลี่ยน arc สุดท้ายจาก `T_Ax_Takeoff → P_Ax_Done` เป็น `T_Ax_Takeoff → P_Ax_Ready` (วนกลับ)

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

- วาง **Shared Resources** ด้านขวามือ (แยกออกจากเส้นหลัก)
- วาง **เครื่องบินแต่ละลำ** เป็นแถวแนวนอน (A1 บน, A2 กลาง, A3 ล่าง)
- ส่วน Ground Handling วางแยกเป็น **3 แถวย่อย** (Fuel / Catering / Maintenance) ตรงกลางของแต่ละลำ

---

## ส่วนที่ 2 — วาด Shared Resources ก่อน (ด้านขวา)

วาง places เหล่านี้ที่มุมขวาของ canvas โดยจะมีลูกศรเชื่อมไปยัง transitions ของ **ทุกเครื่องบิน**

| ชื่อ Place | Initial Marking | ความหมาย |
|---|---|---|
| `P_Runway` | **1** | Runway 1 เส้น — ใช้ตอน Landing และ Takeoff |
| `P_Bay` | **3** | Stand/Gate 3 ช่อง — ครองตั้งแต่ Dock จนถึง Pushback (ทั้ง 3 ลำ dock พร้อมกันได้) |
| `P_Apron` | **2** | Service zone บน Apron 2 zone — ครองตั้งแต่ **Fork** (เริ่ม GSE) จนถึง Board (รถทุกคันออกแล้ว) |
| `P_FuelTruck` | **1** | รถเติมน้ำมัน 1 คัน — **pool ร่วม** ลอยระหว่าง A1/A2/A3 |
| `P_CateringTruck` | **1** | รถ Catering 1 คัน — **pool ร่วม** ลอยระหว่าง A1/A2/A3 |
| `P_MaintCrew` | **1** | ทีม Maintenance 1 ทีม — **pool ร่วม** ลอยระหว่าง A1/A2/A3 |

> **วิธีทำ:** สร้าง place 6 อัน → ตั้งชื่อตามตาราง → ตั้ง Marking ตามคอลัมน์ขวา

> **หมายเหตุ P_Bay:** consume ที่ `T_Ax_Dock` — คืนที่ `T_Ax_Pushback` (stand ว่างเมื่อเครื่องถูกดันออก) — **3 tokens** ทุกลำ dock พร้อมกันได้
>
> **หมายเหตุ P_Apron:** consume ที่ `T_Ax_Fork` — คืนที่ `T_Ax_Board` (service zone ว่างเมื่อ boarding เริ่ม = รถทุกคันออกจาก apron แล้ว) — **2 tokens** ลำที่ 3 ที่ dock แล้วต้องรอ Apron ว่างก่อนถึงเริ่ม GSE ได้

---

## ส่วนที่ 3 — วาดเครื่องบิน A1 (แถวบน)

### 3.1 สร้าง Places ของ A1

สร้างวงกลมทั้ง 16 อัน เรียงจากซ้ายไปขวา:

| ลำดับ | ชื่อ Place | Marking | ความหมาย |
|---|---|---|---|
| 1 | `P_A1_Ready` | **1** | เครื่องพร้อมบิน (และ **จุดวนกลับ** หลัง Takeoff) |
| 2 | `P_A1_OnRunway` | 0 | อยู่บน Runway |
| 3 | `P_A1_Docked` | 0 | จอดที่ Bay แล้ว |
| 4 | `P_A1_ReadyFuel` | 0 | รอ FuelTruck |
| 5 | `P_A1_ReadyCat` | 0 | รอ CateringTruck |
| 6 | `P_A1_ReadyMaint` | 0 | รอ MaintCrew |
| 7 | `P_A1_Fueling` | 0 | กำลังเติมน้ำมัน |
| 8 | `P_A1_Catering` | 0 | กำลัง Catering |
| 9 | `P_A1_Maintaining` | 0 | กำลัง Maintenance |
| 10 | `P_A1_FuelDone` | 0 | เติมน้ำมันเสร็จ → FuelTruck คืนสู่ pool |
| 11 | `P_A1_CatDone` | 0 | Catering เสร็จ → CateringTruck คืนสู่ pool |
| 12 | `P_A1_MaintDone` | 0 | Maintenance เสร็จ → MaintCrew คืนสู่ pool |
| 13 | `P_A1_GHDone` | 0 | Ground Handling ครบทั้ง 3 งานแล้ว |
| 14 | `P_A1_Boarding` | 0 | กำลัง Boarding |
| 15 | `P_A1_Departing` | 0 | พร้อม Depart |

> **ไม่มี `P_A1_Done` แล้ว** — เครื่องบิน Takeoff แล้ววนกลับ `P_A1_Ready` โดยตรง

> **หมายเหตุ Layout:**
> - Places 1–3, 13–16 อยู่แถวแนวนอนหลัก
> - Places 4, 7, 10 อยู่แถว **Fuel lane** (บน)
> - Places 5, 8, 11 อยู่แถว **Catering lane** (กลาง)
> - Places 6, 9, 12 อยู่แถว **Maintenance lane** (ล่าง)

### 3.2 สร้าง Transitions ของ A1

สร้างสี่เหลี่ยมทั้ง 13 อัน:

| ลำดับ | ชื่อ Transition | ความหมาย |
|---|---|---|
| 1 | `T_A1_Land` | ลงจอด — ใช้ P_Runway |
| 2 | `T_A1_Dock` | เข้า Bay — ใช้ **P_Bay** เท่านั้น (จอด stand) |
| 3 | `T_A1_Fork` | **AND-fork** — จอง **P_Apron** (GSE slot) แล้วแตกเป็น 3 งานพร้อมกัน |
| 4 | `T_A1_StartFuel` | รับ FuelTruck จาก pool แล้วเริ่มเติม |
| 5 | `T_A1_EndFuel` | เติมเสร็จ — **คืน FuelTruck กลับ pool** |
| 6 | `T_A1_StartCat` | รับ CateringTruck จาก pool แล้วเริ่ม |
| 7 | `T_A1_EndCat` | Catering เสร็จ — **คืน CateringTruck กลับ pool** |
| 8 | `T_A1_StartMaint` | รับ MaintCrew จาก pool แล้วเริ่ม |
| 9 | `T_A1_EndMaint` | Maintenance เสร็จ — **คืน MaintCrew กลับ pool** |
| 10 | `T_A1_Join` | **AND-join** — รอครบ 3 งาน (FuelDone + CatDone + MaintDone) |
| 11 | `T_A1_Board` | Boarding ผู้โดยสาร |
| 12 | `T_A1_Pushback` | ออกจาก Bay — **คืน P_Bay กลับ** (stand ว่าง) |
| 13 | `T_A1_Takeoff` | Takeoff — ใช้ P_Runway |

### 3.3 ลาก Arcs ของ A1

**ขั้นตอน Landing → Docking:**
```
P_A1_Ready     → T_A1_Land
P_Runway       → T_A1_Land         ← ใช้ Runway (คืนหลัง Dock)
T_A1_Land      → P_A1_OnRunway
P_A1_OnRunway  → T_A1_Dock
P_Bay          → T_A1_Dock         ← ใช้ Bay slot (3 slots — ทุกลำ dock พร้อมกันได้)
T_A1_Dock      → P_A1_Docked
T_A1_Dock      → P_Runway          → คืน Runway ให้ลำอื่น
```

**ขั้นตอน AND-Fork (1 → 3 งานพร้อมกัน):**
```
P_A1_Docked    → T_A1_Fork
P_Apron        → T_A1_Fork         ← จอง Apron slot ก่อนเริ่ม GSE (2 slots เท่านั้น!)
T_A1_Fork      → P_A1_ReadyFuel    ─┐
T_A1_Fork      → P_A1_ReadyCat      ├── ทั้ง 3 งานเริ่มพร้อมกันทันที
T_A1_Fork      → P_A1_ReadyMaint   ─┘
```

**ขั้นตอน Fuel lane:**
```
P_A1_ReadyFuel  → T_A1_StartFuel
P_FuelTruck     → T_A1_StartFuel    ← รับจาก pool (ถ้า pool ว่าง fire ได้เลย)
T_A1_StartFuel  → P_A1_Fueling
P_A1_Fueling    → T_A1_EndFuel
T_A1_EndFuel    → P_A1_FuelDone
T_A1_EndFuel    → P_FuelTruck       → คืน FuelTruck กลับ pool ทันที
                                        (A2 หรือ A3 ที่รออยู่รับได้เลย)
```

**ขั้นตอน Catering lane:**
```
P_A1_ReadyCat    → T_A1_StartCat
P_CateringTruck  → T_A1_StartCat    ← รับจาก pool
T_A1_StartCat    → P_A1_Catering
P_A1_Catering    → T_A1_EndCat
T_A1_EndCat      → P_A1_CatDone
T_A1_EndCat      → P_CateringTruck  → คืน CateringTruck กลับ pool ทันที
```

**ขั้นตอน Maintenance lane:**
```
P_A1_ReadyMaint  → T_A1_StartMaint
P_MaintCrew      → T_A1_StartMaint  ← รับจาก pool
T_A1_StartMaint  → P_A1_Maintaining
P_A1_Maintaining → T_A1_EndMaint
T_A1_EndMaint    → P_A1_MaintDone
T_A1_EndMaint    → P_MaintCrew      → คืน MaintCrew กลับ pool ทันที
                                        (A2 หรือ A3 ที่รออยู่รับได้เลย)
```

**ขั้นตอน AND-Join (3 → 1) → Departure:**
```
P_A1_FuelDone   → T_A1_Join  ─┐
P_A1_CatDone    → T_A1_Join   ├── ต้องมี token ครบทั้ง 3 ถึง fire ได้
P_A1_MaintDone  → T_A1_Join  ─┘
T_A1_Join       → P_A1_GHDone
P_A1_GHDone     → T_A1_Board
T_A1_Board      → P_A1_Boarding
T_A1_Board      → P_Apron           → คืน Apron slot ทันที (service zone ว่าง รถออกแล้ว)
P_A1_Boarding   → T_A1_Pushback
T_A1_Pushback   → P_A1_Departing
T_A1_Pushback   → P_Bay             → คืน Bay slot (stand ว่าง)
P_A1_Departing  → T_A1_Takeoff
P_Runway        → T_A1_Takeoff      ← ใช้ Runway
T_A1_Takeoff    → P_A1_Ready        ← *** วนกลับ (CYCLIC) *** ไม่ใช่ P_A1_Done
T_A1_Takeoff    → P_Runway          → คืน Runway
```

> **Cyclic arc:** `T_A1_Takeoff → P_A1_Ready` ทำให้เครื่องบิน "กลับมา" ใหม่ได้ไม่จำกัดรอบ
> ใน nd.app ให้ลาก arc วนยาวผ่านด้านล่างของ canvas เพื่อไม่ให้ทับ arc อื่น

---

## ส่วนที่ 4 — วาดเครื่องบิน A2 (แถวกลาง)

ทำซ้ำเหมือน A1 **แต่เปลี่ยน A1 → A2** และวางไว้ใต้ A1

### Places ของ A2 (Marking เหมือน A1):
`P_A2_Ready(1)`, `P_A2_OnRunway(0)`, `P_A2_Docked(0)`,
`P_A2_ReadyFuel(0)`, `P_A2_ReadyCat(0)`, `P_A2_ReadyMaint(0)`,
`P_A2_Fueling(0)`, `P_A2_Catering(0)`, `P_A2_Maintaining(0)`,
`P_A2_FuelDone(0)`, `P_A2_CatDone(0)`, `P_A2_MaintDone(0)`,
`P_A2_GHDone(0)`, `P_A2_Boarding(0)`, `P_A2_Departing(0)`

> **ไม่มี `P_A2_Done`** — arc สุดท้ายคือ `T_A2_Takeoff → P_A2_Ready` (cyclic)

### Transitions ของ A2:
`T_A2_Land`, `T_A2_Dock`, `T_A2_Fork`,
`T_A2_StartFuel`, `T_A2_EndFuel`,
`T_A2_StartCat`, `T_A2_EndCat`,
`T_A2_StartMaint`, `T_A2_EndMaint`,
`T_A2_Join`, `T_A2_Board`, `T_A2_Pushback`, `T_A2_Takeoff`

### Arcs ของ A2:
ลากเหมือน A1 ทุก arc เปลี่ยน A1 → A2 ในชื่อ
Shared resource places (P_Runway, P_Bay, P_Apron, P_FuelTruck, P_CateringTruck, P_MaintCrew) เชื่อมไปยัง transitions ของ A2 **โดยตรง** (place เดิม — ไม่สร้างใหม่)

---

## ส่วนที่ 5 — วาดเครื่องบิน A3 (แถวล่าง)

ทำซ้ำเหมือน A1 และ A2 **แต่เปลี่ยน A1 → A3** และวางไว้ใต้ A2

### Places ของ A3:
`P_A3_Ready(1)`, `P_A3_OnRunway(0)`, `P_A3_Docked(0)`,
`P_A3_ReadyFuel(0)`, `P_A3_ReadyCat(0)`, `P_A3_ReadyMaint(0)`,
`P_A3_Fueling(0)`, `P_A3_Catering(0)`, `P_A3_Maintaining(0)`,
`P_A3_FuelDone(0)`, `P_A3_CatDone(0)`, `P_A3_MaintDone(0)`,
`P_A3_GHDone(0)`, `P_A3_Boarding(0)`, `P_A3_Departing(0)`

> **ไม่มี `P_A3_Done`** — arc สุดท้ายคือ `T_A3_Takeoff → P_A3_Ready` (cyclic)

### Transitions ของ A3:
`T_A3_Land`, `T_A3_Dock`, `T_A3_Fork`,
`T_A3_StartFuel`, `T_A3_EndFuel`,
`T_A3_StartCat`, `T_A3_EndCat`,
`T_A3_StartMaint`, `T_A3_EndMaint`,
`T_A3_Join`, `T_A3_Board`, `T_A3_Pushback`, `T_A3_Takeoff`

### Arcs ของ A3:
เหมือน A1/A2 ทุกอย่าง เปลี่ยนชื่อเป็น A3

---

## ส่วนที่ 6 — ตรวจสอบการเชื่อม Shared Resources

Shared Resource แต่ละตัวต้องมีลูกศรเชื่อมถึง transitions ของ **ทั้ง 3 เครื่องบิน** ตรวจดังนี้:

### P_Runway (1 token)
- Input arc จาก: `T_A1_Land`, `T_A2_Land`, `T_A3_Land`, `T_A1_Takeoff`, `T_A2_Takeoff`, `T_A3_Takeoff`
- Output arc ไปยัง: transitions เดิมทั้ง 6
- คืนโดย: `T_A1_Dock` (หลัง land), `T_A2_Dock`, `T_A3_Dock`, `T_A1_Takeoff`, `T_A2_Takeoff`, `T_A3_Takeoff`

### P_Bay (3 tokens) — Physical Stand
- Output arc ไปยัง: `T_A1_Dock`, `T_A2_Dock`, `T_A3_Dock`
- Input arc จาก: `T_A1_Pushback`, `T_A2_Pushback`, `T_A3_Pushback`

> Stand ถูกครองตั้งแต่เครื่องเข้าจอดจนถึงถูก tug ดันออก — **3 tokens = 3 stand** ทุกลำ dock พร้อมกันได้เลย

### P_Apron (2 tokens) — Ground Service Zone
- Output arc ไปยัง: `T_A1_Fork`, `T_A2_Fork`, `T_A3_Fork`
- Input arc จาก: `T_A1_Board`, `T_A2_Board`, `T_A3_Board`

> Service zone ถูกครองตั้งแต่ **Fork** (เริ่ม GSE) จนถึง Boarding เริ่ม (รถทุกคันออกแล้ว)
> ลำที่ 3 ที่ dock แล้วต้องรอ Apron ว่างก่อน (จาก T_Board ของลำอื่น) ถึงจะเริ่ม GSE ได้

### P_FuelTruck (1 token) — Floating Resource Pool
- Input arc จาก: `T_A1_StartFuel`, `T_A2_StartFuel`, `T_A3_StartFuel`
- Output arc ไปยัง: transitions เดิมทั้ง 3
- คืนโดย: `T_A1_EndFuel`, `T_A2_EndFuel`, `T_A3_EndFuel`

> 1 token ลอยระหว่างทุกลำ — ใครว่างรับใครได้ เพราะ transition ไหน fire ได้ก็รับ token ไปเลย

### P_CateringTruck (1 token) — Floating Resource Pool
- Input arc จาก: `T_A1_StartCat`, `T_A2_StartCat`, `T_A3_StartCat`
- Output arc ไปยัง: transitions เดิมทั้ง 3
- คืนโดย: `T_A1_EndCat`, `T_A2_EndCat`, `T_A3_EndCat`

### P_MaintCrew (1 token) — Floating Resource Pool
- Input arc จาก: `T_A1_StartMaint`, `T_A2_StartMaint`, `T_A3_StartMaint`
- Output arc ไปยัง: transitions เดิมทั้ง 3
- คืนโดย: `T_A1_EndMaint`, `T_A2_EndMaint`, `T_A3_EndMaint`

> **ตัวอย่าง:** A2 Maintenance เสร็จ → `T_A2_EndMaint` fire → P_MaintCrew ได้ token คืน → ถ้า A1 ยัง ReadyMaint อยู่ → `T_A1_StartMaint` fire ได้ทันที

---

## ส่วนที่ 7 — ตรวจสอบก่อน Save

### รายการตรวจ:
- [ ] Places ทั้งหมด: **51** (6 shared + 15×3 aircraft) ← ไม่มี P_Ax_Done แล้ว
- [ ] Transitions ทั้งหมด: **39** (13×3 aircraft)
- [ ] Arcs ทั้งหมด: **132** (44×3 aircraft) ← จำนวนเท่าเดิม แต่ arc สุดท้ายเปลี่ยนปลายทาง
- [ ] `P_A1_Ready`, `P_A2_Ready`, `P_A3_Ready` มี marking = **1**
- [ ] `P_Runway` มี marking = **1**
- [ ] `P_Bay` มี marking = **2**
- [ ] `P_Apron` มี marking = **2**
- [ ] `P_FuelTruck` มี marking = **1**
- [ ] `P_CateringTruck` มี marking = **1**
- [ ] `P_MaintCrew` มี marking = **1**
- [ ] Place อื่นๆ ทั้งหมด marking = **0**
- [ ] ทุก AND-Fork (`T_Ax_Fork`) มี **3 output arcs** (ReadyFuel, ReadyCat, ReadyMaint)
- [ ] ทุก AND-Join (`T_Ax_Join`) มี **3 input arcs** (FuelDone, CatDone, MaintDone)
- [ ] `T_Ax_Dock` มี input arc จาก **P_Bay** เท่านั้น (ไม่มี P_Apron แล้ว)
- [ ] `T_Ax_Fork` มี input arc จาก **P_Apron** (consume ก่อนเริ่ม GSE)
- [ ] `T_Ax_Board` มี output arc ไป **P_Apron** (คืน service zone)
- [ ] `T_Ax_Pushback` มี output arc ไป **P_Bay** (คืน stand)
- [ ] P_FuelTruck เชื่อมกับ **StartFuel ทั้ง 3 ลำ** และ **EndFuel ทั้ง 3 ลำ**
- [ ] P_CateringTruck เชื่อมกับ **StartCat ทั้ง 3 ลำ** และ **EndCat ทั้ง 3 ลำ**
- [ ] P_MaintCrew เชื่อมกับ **StartMaint ทั้ง 3 ลำ** และ **EndMaint ทั้ง 3 ลำ**
- [ ] **[CYCLIC]** ทุก `T_Ax_Takeoff` มี output arc วนกลับไป **`P_Ax_Ready`** (ไม่ใช่ P_Ax_Done!)
- [ ] **[ไม่มี]** P_A1_Done, P_A2_Done, P_A3_Done — ไม่ควรมี node เหล่านี้ในโมเดล

### วิธีนับ arc ใน nd.app:
ไปที่ **Edit → Select All** แล้วดูจำนวน elements ที่เลือกทั้งหมด

---

## ส่วนที่ 8 — Save และ Export

1. **Save**: `File → Save As` → เลือก format `.ndr` → ตั้งชื่อ `aircraft_turnaround`
2. **Verify**: ลอง `File → Close` แล้ว `File → Open` ไฟล์ที่ save ไว้
3. **Export เป็น .net** (สำหรับใช้กับ TINA เพื่อ verification):
   `File → Export → .net format`

---

## ส่วนที่ 9 — Diagram แสดง Layout โครงสร้าง (1 เครื่องบิน)

```
[P_A1_Ready]─→[T_A1_Land]─→[P_A1_OnRunway]─→[T_A1_Dock]─→[P_A1_Docked]
                  ↑↓                           ↑ P_Bay          │
                P_Runway                   (3 tokens)       [T_A1_Fork]←─ P_Apron
                (คืนที่                                    ↙    ↓    ↘    (2 tokens
                T_A1_Dock)                ReadyFuel  ReadyCat  ReadyMaint  คืนที่ Board)
                                               │         │         │
                                         StartFuel  StartCat  StartMaint
                                          ↑ FuelTruck ↑ CatTruck ↑ MaintCrew
                                          (pool)      (pool)      (pool)
                                               │         │         │
                                           Fueling  Catering  Maintaining
                                               │         │         │
                                          EndFuel   EndCat   EndMaint
                                          → คืน     → คืน    → คืน
                                          FuelTruck CatTruck MaintCrew
                                               │         │         │
                                          FuelDone  CatDone  MaintDone
                                               ↘        ↓        ↙
                                                  [T_A1_Join]
                                                       │
                                    [P_A1_GHDone]─→[T_A1_Board]─→[P_A1_Boarding]
                                                                         │
                                                                  [T_A1_Pushback]
                                                                  → คืน P_Bay
                                                                         │
                                                                  [P_A1_Departing]
                                                                         │
                                                                  [T_A1_Takeoff]
                                                                    ↑↓ P_Runway  │
                                                                                 │ (cyclic arc ลาก
                                                                                 │  ผ่านด้านล่าง)
                                                                                 ↓
◄──────────────────────────────────────────────────────────────────────────[P_A1_Ready]
(วนกลับ)
```

*(วาดซ้ำ pattern นี้สำหรับ A2 และ A3 ในแถวถัดลงไป — ทุก StartFuel/StartCat/StartMaint เชื่อมกับ pool place เดียวกัน และทุก T_Ax_Takeoff วนกลับ P_Ax_Ready)*

---

## ส่วนที่ 10 — เคล็ดลับ

- **ใช้ Grid**: เปิด View → Grid เพื่อจัดตำแหน่งได้สม่ำเสมอ
- **Copy-Paste**: วาด A1 เสร็จแล้ว Select ทั้งหมด (ยกเว้น Shared Resources) → Copy → Paste → เปลี่ยนชื่อ A1 เป็น A2 ทีละอัน
- **Zoom out**: ใช้ Ctrl+- เพื่อดูภาพรวม
- **คืน resource (arc วนกลับ)**: arc จาก EndFuel/EndCat/EndMaint กลับไป FuelTruck/CateringTruck/MaintCrew ต้องทำให้ครบทุกลำ
- **ทดสอบ deadlock**: ถ้า A1, A2, A3 ต่างรอ MaintCrew พร้อมกัน โมเดลจะให้ 1 ลำทำก่อน ลำอื่นรอ — นี่คือ behavior ที่ถูกต้องของการแชร์ resource แบบ pool

---

*คู่มือนี้อ้างอิงโมเดล `aircraft_turnaround.ndr` ที่สร้างสำหรับ paper: "Formal Verification of Aircraft Turnaround Process Using Petri Net"*
