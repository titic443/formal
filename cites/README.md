# Citation Papers — Formal Verification of Aircraft Turnaround Process

**Main paper:** "Formal Verification of Aircraft Turnaround Process Using Petri Net for Deadlock Detection and Liveness Analysis"
**Conference:** IEEE JCSSE 2026
**Author:** Thiti Changphu, Chulalongkorn University

---

## Papers Confirmed (6 papers evaluated / 5 recommended)

| # | Paper | ปี | Method | สถานะ |
|---|-------|-----|--------|--------|
| CITE-1 | Critical Paths Simulation — Turnaround | 2018 | Simulation | ใช้ |
| CITE-2 | Ground Handling PERT-COST | 2025 | PERT-COST | ใช้ |
| CITE-3 | Deadlock Avoidance Petri Net — Manufacturing | 2005 | Petri Net | ใช้ (backup) |
| CITE-4 | Liveness + Deadlock Control — Manufacturing | 2020 | Petri Net | ใช้ (ดีที่สุดกลุ่มนี้) |
| CITE-5 | Deadlock/Livelock via TINA — Petri Net | 2024 | Petri Net + TINA | ใช้ (ดีที่สุดทั้งหมด) |
| CITE-6 | GSE Management — Turnaround Performance | 2021 | Simulation | ใช้ |

---

## รายละเอียด Paper แต่ละใบ

---

### [CITE-1] Using Simulation to Estimate Critical Paths and Survival Functions in Aircraft Turnaround Processes

| Field | Info |
|-------|------|
| File | `Using_simulation_to_estimate_critical_paths_and_survival_functions_in_aircraft_turnaround_processes.pdf` |
| Year | 2018 |
| Method | Simulation, Critical Path Analysis |
| Domain | Aircraft Turnaround |

**เนื้อหาหลัก:**
- วิเคราะห์ turnaround process ด้วย simulation
- หา critical path และ survival functions ของกระบวนการ
- ศึกษา concurrent operations ใน ground handling

**ใช้ cite ตรงไหน:** Section I / Section II-A — ใช้ contrast กับ formal verification

**วิธี cite:**
> Prior work has used simulation to analyze critical paths in turnaround processes [X], however simulation cannot provide formal guarantees about deadlock-freedom or liveness.

---

### [CITE-2] Modelling the Sustainable Development of the Ground Handling Process Using the PERT-COST Method

| Field | Info |
|-------|------|
| File | `sustainability-17-11278.pdf` |
| DOI | 10.3390/su172411278 |
| Authors | Kierzkowski A., Ryczyński J., Kisiel T., Mardeusz E., Prentkovskis O. |
| Journal | Sustainability (MDPI) |
| Year | 2025 |
| Method | PERT-COST |
| Domain | Aircraft Ground Handling |

**เนื้อหาหลัก:**
- วิเคราะห์ turnaround efficiency ด้วย PERT-COST
- เก็บข้อมูลจริงจาก ground operations หา duration และ variability
- ระบุ bottleneck (baggage handling, refueling)
- เปรียบเทียบ scenario: remote vs gate, day vs night, high vs low fuel uplift

**ใช้ cite ตรงไหน:** Section I / Section II-A — ใช้ contrast

**วิธี cite:**
> Recent studies have identified ground handling bottlenecks using PERT-COST methods [X], but these approaches do not formally guarantee deadlock-freedom or verify safety properties.

---

### [CITE-3] Deadlock Avoidance Petri Net Controller for Manufacturing Systems with Multiple Resource Service

| Field | Info |
|-------|------|
| File | `xx/Deadlock_Avoidance_Petri_Net_Controller_for_Manufacturing_Systems_with_Multiple_Resource_Service.pdf` |
| DOI | 10.1109/ROBOT.2005.1570855 |
| Authors | Keli Xing, Xiajie Jin, Yu Feng |
| Conference | IEEE ICRA |
| Year | 2005 |
| Method | Petri Net, Deadlock Avoidance |
| Domain | Manufacturing Systems |

**เนื้อหาหลัก:**
- ออกแบบ Petri Net Controller สำหรับป้องกัน deadlock
- จัดการ multiple shared resources พร้อมกัน
- เสนอ optimal control policy ให้ระบบ deadlock-free

**ใช้ cite ตรงไหน:** Section II-C / Section V-C — support deadlock resolution technique

**หมายเหตุ:** เก่า (2005) ควรใช้คู่กับ CITE-4 ซึ่งใหม่กว่าและครอบคลุมกว่า

---

### [CITE-4] Liveness Analysis and Deadlock Control for Automated Manufacturing Systems With Multiple Resource Requirements

| Field | Info |
|-------|------|
| File | `xx/Liveness_Analysis_and_Deadlock_Control_for_Automated_Manufacturing_Systems_With_Multiple_Resource_Requirements.pdf` |
| DOI | 10.1109/tsmc.2017.2767902 |
| Authors | Feng Yanxiang, Xing Keyi, Zhou MengChu, Liu Huixia |
| Journal | IEEE Transactions on Systems, Man, and Cybernetics: Systems |
| Year | 2020 |
| Method | Petri Net, Liveness + Deadlock Control |
| Domain | Manufacturing Systems |

**เนื้อหาหลัก:**
- วิเคราะห์ Liveness และควบคุม Deadlock ด้วย Petri Net
- ครอบคลุม multiple resource requirements
- ตีพิมพ์ใน IEEE Trans. SMC — journal tier สูง

**ใช้ cite ตรงไหน:** Section II-C / Section IV / Section V — support ทั้ง liveness และ deadlock

**วิธี cite:**
> Liveness analysis and deadlock control for systems with multiple resource requirements have been studied extensively using Petri Net formalisms [X].

---

### [CITE-5] On Deadlock/Livelock Studies Based on Reachability Graph of Petri Nets by Using TINA

| Field | Info |
|-------|------|
| File | `xx/On_Deadlock_Livelock_Studies_Based_on_Reachability_Graph_of_Petri_Nets_by_Using_TINA.pdf` |
| DOI | 10.1109/ACCESS.2024.3461168 |
| Authors | Uzam M., Liu D., Berthomieu B. (ผู้สร้าง TINA), Gelen G., Zhang Z., Mostafa A., Li Z. |
| Journal | IEEE Access |
| Year | 2024 |
| Method | Petri Net + TINA, Reachability Graph |
| Domain | Flexible Manufacturing Systems |

**เนื้อหาหลัก:**
- ใช้ TINA วิเคราะห์ Deadlock และ Livelock ผ่าน Reachability Graph
- Berthomieu (ผู้สร้าง TINA) เป็น co-author
- ครอบคลุม liveness enforcing supervisor (LES)

**ใช้ cite ตรงไหน:** Section II-C / Section V — ตรงที่สุดกับ methodology ของ paper

**วิธี cite:**
> The TINA toolbox supports deadlock and livelock detection via reachability graph analysis of Petri Nets [X], which forms the basis of our verification approach.

**หมายเหตุ:** paper นี้ดีที่สุดในทั้งหมด — ใช้ TINA เหมือนกันเป๊ะ และใหม่มาก (2024)

---

### [CITE-6] Enhanced Operational Management of Airport Ground Support Equipment for Better Aircraft Turnaround Performance

| Field | Info |
|-------|------|
| File | `xx/Enhanced_Operational_Management_Of_Airport_Ground_Support_Equipment_For_Better_Aircraft_Turnaround_Performance.pdf` |
| DOI | 10.1109/WSC52266.2021.9715320 |
| Authors | Saggar S., Tomasella M., Cattaneo G., Matta A. |
| Conference | Winter Simulation Conference (WSC) |
| Year | 2021 |
| Method | Simulation |
| Domain | Airport GSE / Aircraft Turnaround |

**เนื้อหาหลัก:**
- จัดการ GSE ให้มีประสิทธิภาพเพื่อ improve turnaround performance
- ครอบคลุม operations หลายด้านของ apron management
- ใช้ simulation-based approach

**ใช้ cite ตรงไหน:** Section I / Section II-A — เสริม GSE resource contention domain

**วิธี cite:**
> Simulation-based approaches have been applied to optimize GSE management for turnaround performance [X], however these methods cannot formally guarantee deadlock-freedom under all resource contention scenarios.

---

## Papers Rejected

| Paper | ปี | เหตุผล |
|-------|-----|--------|
| Petri Net Emergency Control — Shipping | 2012 | Domain ไม่ตรง, ไม่เน้น deadlock (ลบแล้ว) |
| Concurrent Algorithms in SPIN | 2016 | ใช้ SPIN/Promela ไม่ใช่ TINA/Petri Net |
| Simulation Logistic Ground Handling (Kierzkowski) | 2015 | ผู้แต่งซ้ำ CITE-2, venue ไม่ตรง |

---

## สรุปภาพรวม

| Section ใน Paper | Cite |
|-----------------|------|
| Section I — Introduction | CITE-1, CITE-2, CITE-6 |
| Section II-A — Formal Verification in Aviation | CITE-1, CITE-2 |
| Section II-C — Deadlock Analysis Petri Net | CITE-3, CITE-4, CITE-5 |
| Section IV — Properties to Verify | CITE-4, CITE-5 |
| Section V — Verification Results | CITE-5 |

---

## Notes

- ทุก aviation paper (CITE-1, 2, 6) ใช้ **contrast**: simulation/PERT ไม่สามารถ formal guarantee ได้
- CITE-5 สำคัญที่สุด — ใช้ TINA เหมือนกันและใหม่มาก (2024)
- CITE-3 เก่า (2005) พิจารณาตัดออกถ้า page limit ตึง โดยใช้ CITE-4 แทน
