# Genetic Algorithm for Logistics Optimization

โปรเจกต์นี้ศึกษาและประยุกต์ใช้ **Genetic Algorithm (GA)** สำหรับแก้ปัญหาด้าน **Logistics** เช่น การจัดเส้นทางขนส่ง การมอบหมายงานให้ยานพาหนะ และการวางแผนการกระจายสินค้า

## Genetic Algorithm คืออะไร

Genetic Algorithm เป็นอัลกอริทึมการค้นหาคำตอบเชิงวิวัฒนาการ (Evolutionary Algorithm) ที่ได้แรงบันดาลใจจากการคัดเลือกโดยธรรมชาติ (Natural Selection) หลักการคือสร้าง "ประชากร" ของคำตอบที่เป็นไปได้ แล้วค่อย ๆ ปรับปรุงคำตอบผ่านการคัดเลือก ผสมพันธุ์ และกลายพันธุ์ ซ้ำไปเรื่อย ๆ จนได้คำตอบที่ดีที่สุด (หรือใกล้เคียงที่สุด)

เหมาะกับปัญหา Logistics เพราะปัญหาเหล่านี้ส่วนใหญ่เป็น **NP-hard** (เช่น TSP, VRP) ที่ไม่สามารถหาคำตอบที่ดีที่สุดแบบตรง ๆ ได้ในเวลาที่ยอมรับได้เมื่อขนาดปัญหาใหญ่ขึ้น

### NP-hard คืออะไร

**NP-hard ไม่ใช่ชื่ออัลกอริทึม** (ไม่เหมือน DFS หรือ Dijkstra ที่เป็น "วิธีแก้") แต่เป็น **Complexity Class** — ป้ายกำกับที่ใช้จัดประเภท*ตัวปัญหา*ว่ายากแค่ไหนในเชิงทฤษฎีการคำนวณ

| | คืออะไร | ตัวอย่าง |
|---|---|---|
| **Algorithm** (วิธีแก้) | ขั้นตอนวิธีที่ใช้*แก้*ปัญหา รันได้จริง | DFS, BFS, Dijkstra, Genetic Algorithm |
| **Complexity Class** (ระดับความยาก) | คุณสมบัติของ*ตัวปัญหา* ไม่ใช่วิธีแก้ | P, NP, NP-complete, NP-hard |

ปัญหาที่เป็น NP-hard คือปัญหาที่ยังไม่มีอัลกอริทึมใดหาคำตอบที่ดีที่สุดได้ในเวลา polynomial — เวลาคำนวณจะ "ระเบิด" เมื่อขนาดปัญหาโตขึ้น เช่น TSP แบบ brute force ต้องไล่เช็ค n! เส้นทาง:

| จำนวนลูกค้า | จำนวนเส้นทางที่เป็นไปได้ (n!) |
|---|---|
| 5 | 120 |
| 10 | 3.6 ล้าน |
| 20 | 2.4 × 10¹⁸ (คิดเป็นร้อยปี) |
| 50 | มากกว่าจำนวนอะตอมในจักรวาล |

เพราะหาคำตอบเป๊ะไม่ไหว ในทางปฏิบัติจึงยอมรับคำตอบ "ดีเกือบที่สุด" (near-optimal) แล้วใช้ Metaheuristic อย่าง **Genetic Algorithm** ค่อย ๆ วิวัฒนาการคำตอบให้ดีขึ้นภายในเวลาที่กำหนดแทน

## ปัญหา Logistics ที่ GA แก้ได้

| ปัญหา | คำอธิบาย |
|---|---|
| **TSP** (Travelling Salesman Problem) | หาเส้นทางที่สั้นที่สุดในการเดินทางผ่านทุกจุดส่งของ แล้วกลับจุดเริ่มต้น |
| **VRP** (Vehicle Routing Problem) | จัดเส้นทางให้รถหลายคันส่งของให้ลูกค้าหลายราย โดยรวมระยะทาง/ต้นทุนต่ำสุด |
| **CVRP** (Capacitated VRP) | VRP ที่รถแต่ละคันมีข้อจำกัดด้านความจุ (น้ำหนัก/ปริมาตร) |
| **VRPTW** (VRP with Time Windows) | VRP ที่ลูกค้าแต่ละรายมีช่วงเวลารับของที่กำหนด |
| **Warehouse / Facility Location** | เลือกตำแหน่งคลังสินค้าเพื่อให้ต้นทุนกระจายสินค้าโดยรวมต่ำสุด |
| **Load Planning** | จัดสินค้าใส่รถ/ตู้คอนเทนเนอร์ให้ใช้พื้นที่คุ้มค่าที่สุด (Bin Packing) |

## องค์ประกอบหลักของ GA

```
เริ่มต้น: สุ่มสร้างประชากร (Population) ของคำตอบ
   │
   ▼
┌─────────────────────────────────────────┐
│ 1. Evaluate  → คำนวณ Fitness ของแต่ละตัว │
│ 2. Selection → คัดเลือกพ่อแม่พันธุ์        │
│ 3. Crossover → ผสมคำตอบสร้างลูกใหม่       │
│ 4. Mutation  → สุ่มปรับเปลี่ยนเล็กน้อย      │
│ 5. Replace   → สร้างประชากรรุ่นถัดไป       │
└─────────────────────────────────────────┘
   │  วนซ้ำจนครบจำนวนรุ่น หรือคำตอบลู่เข้า
   ▼
ผลลัพธ์: คำตอบที่ดีที่สุดที่พบ (Best Solution)
```

### 1. Chromosome (การเข้ารหัสคำตอบ)

ในงาน Logistics มักเข้ารหัสเป็น **ลำดับการเยี่ยมลูกค้า (Permutation Encoding)** เช่น

```
Chromosome: [3, 1, 5, 2, 4]
ความหมาย:  Depot → ลูกค้า 3 → 1 → 5 → 2 → 4 → กลับ Depot
```

กรณี VRP หลายคัน อาจใส่ตัวคั่น (delimiter) หรือใช้ยีนบอกว่าลูกค้าคนไหนอยู่รถคันไหน

### 2. Fitness Function (ฟังก์ชันประเมินคุณภาพ)

ตัวชี้วัดที่นิยมใช้ในงาน Logistics:

- ระยะทางรวม / เวลาเดินทางรวม (ยิ่งน้อยยิ่งดี)
- ต้นทุนขนส่งรวม (ค่าน้ำมัน + ค่าแรง + ค่าเสื่อม)
- จำนวนรถที่ใช้
- ค่าปรับ (Penalty) เมื่อละเมิดข้อจำกัด เช่น เกินความจุรถ หรือส่งของนอกช่วงเวลาที่ลูกค้ากำหนด

ตัวอย่าง:

```
Fitness = 1 / (ระยะทางรวม + α × จำนวนรถ + β × Penalty)
```

### 3. Selection (การคัดเลือก)

- **Tournament Selection** — สุ่มมา k ตัว เลือกตัวที่ Fitness ดีสุด (นิยมที่สุด ปรับจูนง่าย)
- **Roulette Wheel** — โอกาสถูกเลือกแปรผันตามค่า Fitness
- **Elitism** — เก็บคำตอบที่ดีที่สุดข้ามรุ่นไว้เสมอ ป้องกันคำตอบดีหายไป

### 4. Crossover (การผสมพันธุ์)

เนื่องจาก Chromosome เป็นลำดับ (ห้ามมีลูกค้าซ้ำ) ต้องใช้ Crossover แบบพิเศษ:

- **OX (Order Crossover)** — ตัดช่วงจากพ่อ เติมส่วนที่เหลือตามลำดับของแม่
- **PMX (Partially Mapped Crossover)** — จับคู่ตำแหน่งยีนแล้วสลับตาม mapping
- **CX (Cycle Crossover)** — สลับยีนตามวงรอบที่ตำแหน่งตรงกัน

ตัวอย่าง OX:

```
Parent 1: [1, 2, |3, 4, 5|, 6, 7]
Parent 2: [4, 3, |1, 7, 6|, 2, 5]
Child:    [7, 6, |3, 4, 5|, 2, 1]   ← คงช่วงกลางจาก Parent 1
```

### 5. Mutation (การกลายพันธุ์)

ช่วยให้ไม่ติดอยู่ใน Local Optimum:

- **Swap** — สลับตำแหน่งลูกค้า 2 ราย
- **Insertion** — ย้ายลูกค้าหนึ่งรายไปแทรกตำแหน่งใหม่
- **Inversion (2-opt)** — กลับลำดับช่วงหนึ่งของเส้นทาง (ได้ผลดีมากกับปัญหาเส้นทาง)

## พารามิเตอร์ที่แนะนำ (จุดเริ่มต้น)

| พารามิเตอร์ | ค่าแนะนำ | หมายเหตุ |
|---|---|---|
| Population Size | 50–200 | ปัญหาใหญ่ → ประชากรมาก |
| Generations | 500–2000 | หรือหยุดเมื่อคำตอบไม่ดีขึ้นติดต่อกัน N รุ่น |
| Crossover Rate | 0.8–0.95 | |
| Mutation Rate | 0.01–0.1 | สูงไปจะกลายเป็น Random Search |
| Elitism | 1–5% ของประชากร | เก็บตัวท็อปข้ามรุ่น |

## ตัวอย่าง Pseudocode

```python
population = init_random_population(pop_size)

for generation in range(max_generations):
    fitness = [evaluate(route) for route in population]

    new_population = elitism(population, fitness)      # เก็บตัวที่ดีที่สุดไว้
    while len(new_population) < pop_size:
        p1, p2 = tournament_selection(population, fitness)
        child = order_crossover(p1, p2)                # OX
        if random() < mutation_rate:
            child = swap_mutation(child)
        new_population.append(child)

    population = new_population

best_route = argmax(population, key=evaluate)
```

## ข้อดี–ข้อจำกัดของ GA ในงาน Logistics

**ข้อดี**
- รับมือปัญหาขนาดใหญ่และข้อจำกัดซับซ้อนได้ (ความจุรถ, Time Window, หลาย Depot)
- ไม่ต้องการสูตรคณิตศาสตร์ของปัญหาแบบชัดเจน แค่ประเมิน Fitness ได้ก็พอ
- ขนานงาน (Parallelize) ได้ง่าย

**ข้อจำกัด**
- ไม่การันตีว่าได้คำตอบที่ดีที่สุดจริง (Near-optimal)
- ผลลัพธ์ขึ้นกับการจูนพารามิเตอร์
- อาจลู่เข้าช้าเมื่อปัญหาใหญ่มาก — นิยมผสมกับ Local Search (เช่น 2-opt) เป็น **Hybrid / Memetic Algorithm**

## แหล่งอ้างอิงเพิ่มเติม

- Goldberg, D. E. (1989). *Genetic Algorithms in Search, Optimization, and Machine Learning*
- Toth, P., & Vigo, D. (2014). *Vehicle Routing: Problems, Methods, and Applications*
- [DEAP](https://deap.readthedocs.io/) — ไลบรารี Python สำหรับ Evolutionary Computation
- [Google OR-Tools](https://developers.google.com/optimization) — สำหรับเปรียบเทียบผลลัพธ์กับ solver มาตรฐาน
