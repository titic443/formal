# LTL Model Checking — Aircraft Turnaround Petri Net

โมเดล: `aircraft_turnaround.ndr`
เครื่องมือ: TINA Toolbox (selt)
รัน: `selt -ltl "<formula>" aircraft_turnaround.ndr`

---

## โมเดลสรุป (ณ ตอนนี้)

| Resource | Tokens | Consume ที่ | Return ที่ |
|---|---|---|---|
| P_Runway | 1 | T_Land, T_Takeoff | T_Dock, T_Takeoff |
| P_Bay | 3 | T_Dock | T_Pushback |
| P_Apron | 2 | **T_Fork** | T_Board |
| P_FuelTruck | 1 | T_StartFuel | T_EndFuel |
| P_CateringTruck | 1 | T_StartCat | T_EndCat |
| P_MaintCrew | 1 | T_StartMaint | T_EndMaint |

> P_Bay = 3 → ทั้ง 3 ลำ dock พร้อมกันได้
> P_Apron = 2 → GSE ทำได้พร้อมกันแค่ 2 ลำ ลำที่ 3 รอที่ P_Docked จนมี Apron ว่าง

---

## Safety Properties (4 ข้อ)

สิ่งที่ต้องเป็นจริงตลอดเวลา — ถ้า verify ผ่าน = ระบบไม่มีวันละเมิด

### S1 — Runway Mutual Exclusion
ไม่มีสองลำบน runway พร้อมกันเด็ดขาด

```
[] ~(a1_onRunway /\ a2_onRunway)
[] ~(a1_onRunway /\ a3_onRunway)
[] ~(a2_onRunway /\ a3_onRunway)
```

> ตรวจว่า P_Runway = 1 token ทำหน้าที่ mutex ได้จริง

---

### S2 — Apron Capacity Constraint
ไม่มีสามลำเริ่ม GSE พร้อมกัน (Apron จำกัด 2 slot)

```
[] ~(a1_readyFuel /\ a2_readyFuel /\ a3_readyFuel)
```

> a1_readyFuel มี token ได้ก็ต่อเมื่อ T_Fork fire แล้ว (= ใช้ Apron token ไปแล้ว)
> ถ้า property นี้ hold = P_Apron บังคับ constraint ได้จริง

---

### S3 — FuelTruck Mutual Exclusion
รถเติมน้ำมันใช้ได้ทีละลำเท่านั้น

```
[] ~(a1_fueling /\ a2_fueling)
[] ~(a1_fueling /\ a3_fueling)
[] ~(a2_fueling /\ a3_fueling)
```

> ตรวจ shared resource pool ทำงานถูกต้อง — 1 token ลอยได้ทีละลำเท่านั้น

---

### S4 — No Boarding During Active GSE
เครื่องที่กำลัง boarding จะต้องไม่มี ground service ค้างอยู่

```
[] (a1_boarding -> ~(a1_catering \/ a1_fueling \/ a1_maintaining))
[] (a2_boarding -> ~(a2_catering \/ a2_fueling \/ a2_maintaining))
[] (a3_boarding -> ~(a3_catering \/ a3_fueling \/ a3_maintaining))
```

> ตรวจว่า AND-Join บังคับให้ครบทั้ง 3 งานก่อน T_Board fire จริง

---

## Liveness Properties (4 ข้อ)

สิ่งที่ต้องเกิดขึ้นในที่สุด — ถ้า verify ผ่าน = ระบบไม่ติดค้างตลอดกาล

### L1 — Cyclic Progress (No Starvation)
ทุกลำที่ลงจอดจะกลับมา ready ได้เสมอ — ไม่มีลำไหนค้างตลอดกาล

```
[] (a1_onRunway -> <> a1_ready)
[] (a2_onRunway -> <> a2_ready)
[] (a3_onRunway -> <> a3_ready)
```

> property สำคัญที่สุดของ cyclic model — ถ้า fail = มี deadlock หรือ starvation

---

### L2 — GSE Completion Guarantee
เครื่องที่เริ่มรอเติมน้ำมันจะได้เติมในที่สุด — FuelTruck ไม่ขาดหายจากระบบ

```
[] (a1_readyFuel -> <> a1_fuelDone)
[] (a2_readyFuel -> <> a2_fuelDone)
[] (a3_readyFuel -> <> a3_fuelDone)
```

> ตรวจว่า resource pool ไม่ทำให้ทีมใดทีมหนึ่ง deadlock

---

### L3 — Resource Return (No Token Loss)
FuelTruck ที่ถูกใช้จะกลับ pool เสมอ — token ไม่หายออกจากระบบ

```
[] (a1_fueling -> <> fuel)
[] (a2_fueling -> <> fuel)
[] (a3_fueling -> <> fuel)
```

> ถ้า fail = มี return arc หายไปในโมเดล (ตรวจหา bug ได้ตรงๆ)

---

### L4 — Boarding Reachability
เครื่องที่ dock แล้วจะ boarding ได้เสมอ — ไม่ติดค้างรอ Apron ตลอดกาล

```
[] (a1_docked -> <> a1_boarding)
[] (a2_docked -> <> a2_boarding)
[] (a3_docked -> <> a3_boarding)
```

> ตรวจว่า Apron constraint (2 slot) ไม่ทำให้ลำที่ 3 stuck ตลอดกาล
> ถ้า hold = ระบบ fair — ทุกลำได้ GSE ในที่สุดแน่นอน

---

## หมายเหตุก่อนรัน

1. **ตั้งชื่อ place ทุกตัว** ใน nd.app ให้ครบ (a2_xxx, a3_xxx) — อย่าปล่อยให้เป็น p73, p86
2. **แก้ spelling** ในโมเดล: `appron` → `apron`, `maintainance` → `maintenance`
3. **ต้องเป็น bounded net** ก่อน — รัน state space analysis ให้ผ่านก่อนแล้วค่อยรัน LTL
4. สำหรับ Timed Petri Net (TPN) property เชิงเวลา ใช้ **TCTL** ใน selt แทน LTL ธรรมดา

---

*อ้างอิง paper: "Formal Verification of Aircraft Turnaround Process Using Petri Net"*
