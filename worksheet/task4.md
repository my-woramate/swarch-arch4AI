# Task 4: Deployment topologies and trade-offs

**Name / ชื่อ:** วรเมธ พงศาประเสริฐพร

**Student ID / รหัสนิสิต:** 6970259821

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง ใน fork ของตัวเอง*

**Scenario (A, B or C) / Scenario ที่เลือก:**
 > ข้อ A

Choose **one** scenario below. If the last digit of your student ID is odd, take A or B; if it is even, take B or C. Use your measurements from Tutorial 6 as evidence: cite run names from `results/runs.csv`. Answer in Thai or English.

เลือก scenario **หนึ่ง** ข้อด้านล่าง ถ้าเลขตัวสุดท้ายของรหัสนิสิตเป็นเลขคี่ ให้เลือก A หรือ B ถ้าเป็นเลขคู่ ให้เลือก B หรือ C ใช้ผลการวัดจาก tutorial ที่ทำไปเป็นข้อมูลอ้างอิง โดยอ้างชื่อ run จาก `results/runs.csv` จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

Facts that hold in every scenario / ข้อเท็จจริงที่ใช้กับทุก scenario:

- The large model serves at most **20 calls per second** (4 calls at once × about 200 ms each), and every call costs money.

  *large model รับได้สูงสุด **20 calls ต่อวินาที** (พร้อมกัน 4 calls × ประมาณ 200 ms ต่อ call) และทุก call มีค่าใช้จ่าย*
- The small model is fast and cheap enough to run anywhere, including inside the review service or on a kiosk, but it is much less accurate on its own.

  *small model เร็วและถูกพอที่จะรันที่ไหนก็ได้ รวมถึงภายใน review service หรือบน kiosk แต่ถ้าใช้ตัวเดียวจะแม่นน้อยกว่ามาก*

## Scenarios

**A. Owner alerts.** When a customer posts a negative review, the restaurant owner gets a push notification within **2 seconds**. At the lunch peak, CampusEats receives **40 reviews per second**. A missed alert is acceptable, but a wrong alert (for example, "your customer is unhappy" when they are happy) makes owners stop trusting alerts and, in the end, ignore them.

*เมื่อลูกค้าโพสต์รีวิวเชิงลบ เจ้าของร้านจะต้องได้รับ push notification ภายใน **2 วินาที** เวลาช่วงพีคตอนเที่ยง CampusEats ได้รับรีวิวจำนวน **40 รายการต่อวินาที** การพลาด alert ไปบ้างนั้นยอมรับได้ แต่ alert ที่ผิด (เช่น "ลูกค้าไม่พอใจ" ทั้งที่จริงๆ แล้วพอใจ) จะทำให้เจ้าของร้านไม่เชื่อถือและเลิกสนใจ alert ในที่สุด*

**B. Canteen kiosk.** A tablet in each canteen lets students leave a review on the spot. However, the canteen Wi-Fi is poor: the tablets lose their connection several times a day, for minutes at a time. The system must accept every review, even while a tablet is offline. Restaurant owners need the labels **by the end of the day**.

*มี tablet ในแต่ละโรงอาหารให้นิสิตรีวิวได้ทันที อย่างไรก็ดี การเชื่อมต่อ Wi-Fi ในโรงอาหารไม่ดี ทำให้ tablet หลุดการเชื่อมต่อวันละหลายครั้ง ครั้งละหลายนาที ระบบจะต้องรับรีวิวได้ทุกรายการแม้ขณะที่กำลัง offline อยู่ และเจ้าของร้านต้องการ label **ภายในสิ้นวัน***

**C. Weekly report.** Restaurant owners want a weekly sentiment report. Nothing in producing this report has to be real-time. **Accuracy matters most** and **cost must be minimal**.

*เจ้าของร้านต้องการรายงาน sentiment รายสัปดาห์ ไม่มีอะไรที่จะต้องทำแบบ real-time ในการออกรายงานนี้ ความแม่นยำ **(accuracy) สำคัญที่สุด** และ **ค่าใช้จ่ายต้องน้อยที่สุด***

## Topologies and patterns to choose from / Topology และ pattern ที่เลือกใช้ได้

| | Topology | Where the model runs / โมเดลรันที่ไหน |
|---|---|---|
| T1 | Model as a service, synchronous | Inference service, called for each review (today's baseline)<br>*inference service ถูกเรียกทุกรีวิว (baseline ของวันนี้)* |
| T2 | Model as a library | Inside the review service, or on the device<br>*ภายใน review service หรือบนอุปกรณ์* |
| T3 | Hybrid | Small model locally, large model as a remote service<br>*small model รันในเครื่อง large model เป็น remote service* |
| T4 | Asynchronous / batch | Reviews are queued (broker, W10) or stored, and scored later in bulk<br>*รีวิวถูกเข้าคิว (broker, W10) หรือเก็บไว้ แล้วติด label ทีหลังเป็นชุด* |

Combine one topology with any of the patterns from Tutorial 6: **fallback**, **cascade** and **cache**.

*เลือก topology หนึ่งแบบ แล้วผสมกับ pattern จาก Tutorial 6 ได้ตามต้องการ: **fallback**, **cascade** และ **cache***

## 1. Quality requirements

The three most important quality requirements for your scenario, each with a number where possible.

*quality requirements ที่สำคัญที่สุด 3 ข้อของ scenario ที่เลือก ระบุตัวเลขเมื่อทำได้*

| # | Quality | Requirement |
|---|---|---|
| 1 | Performance| Latency (p95 latency < 2,000 ms)|
| 2 | Capacity| Scalability (ส่งให้ Large model ไม่เกิน 20 calls/วินาที)| 
| 3 | Accuracy | >= 90%–95% |

## 2. Design / การออกแบบ

Your topology, the patterns you add, and a container-level diagram (C4 level 2). The C4 diagram must be written in Mermaid.

*topology ที่เลือก pattern ที่เพิ่ม และ diagram ระดับ container (C4 level 2) เขียนด้วย Mermaid*

```mermaid
    C4Container
        title CampusEats: container diagram
        Person(customer, "Customer", "เขียนรีวิวร้าน (จำลองด้วย tools/loadtest.py)")
        Person(owner, "Restaurant owner", "ดูรีวิวและ label ของร้านตัวเอง")
        System_Boundary(campuseats, "CampusEats") {
            Container(gateway, "API gateway", "Python, FastAPI, :8000", "จุดเข้าเดียวของ client ส่งต่อ /api/reviews และ /api/restaurants")
            Container(review, "Review service", "Python, FastAPI, :8001", "เก็บรีวิวในหน่วยความจำ")
            Container(inference, "Inference service", "Python, FastAPI, scikit-learn, :8002", "small model และ large model")
        }
        Rel(customer, gateway, "ส่งรีวิว", "HTTP/JSON")
        Rel(owner, gateway, "ดูจำนวนรีวิวตาม label", "HTTP/JSON")
        Rel(gateway, review, "ส่งต่อ request", "HTTP/JSON")
        Rel(review, inference, "POST /predict", "REST")
        UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

## 3. Evidence / หลักฐาน

Which runs support your choice? Show the numbers and, for scenario A, the capacity arithmetic: how many large-model calls per second does your design need at the peak?

*run ไหนที่สนับสนุนการตัดสินใจของนิสิต ให้นิสิตแสดงตัวเลขประกอบ และสำหรับ scenario A ให้นิสิตแสดงการคำนวณ capacity ด้วย เพื่อแสดงว่าในช่วงพีคนั้น design ของนิสิตต้องใช้ large model กี่ calls ต่อวินาที*

| Run | What it shows / แสดงอะไร |
|---|---|
| t3-cache | p50 = 10 ms, p95 = 518 ms, accuracy = 91.5%, cache_hits = 36%, large_calls = 31% เมื่อมีรีวิวพีค 40 รีวิว/วินาที จะเรียก Large model เพียง 40 × 31% = 12.4 calls/วินาที|
| t1-after | errors = 0, pending = 20, p95 = 1,008 ms, accuracy = 92.8% เมื่อ Large model ค้าง Fallback ช่วยตัด timeout ใน 1 วินาที ท​ำให้ระบบไม่ล่มและยังตอบทันภายใน 2 วินาที|

## 4. ADR

Use the ADR format from W5.

*ใช้รูปแบบ ADR จาก W5*

**Title:** Topology และ Serving Patterns ส​ำหรับระบบแจ้งเตือนรีวิวเชิงลบ

**Status:** Proposed

**Context:** ช่วงพีคตอนเที่ยงมีรีวิวเข้ามา 40 รีวิว/วินาที และต้องแจ้งเตือนเจ้าของร้านภายใน 2 วินาที โดยห้ามทายผิดบ่อย แต่ Large model รับโหลดได้สูงสุดแค่ 20 calls/วินาที และมีโอกาสค้างได้

**Decision:** เลือกใช้ Topology T1 (Model as a service) (หรือ T3 Hybrid ตามกราฟในข้อ 2) ร่วมกับ 3 patterns ได้แก่ Prediction Cache, Cascade (threshold 0.6) และ Fallback (timeout 1 วินาที)

**Consequences:**
- Positive: ลดการเรียก Large model เหลือเพียง 31% (12.4 calls/วินาที) ซึ่งไม่เกินเพดาน 20 calls/วินาที, ความเร็ว p95 อยู่ที่ 518 ms (ทันภายใน 2 วินาที), ความแม่นย​ำสูง 91.5%, และไม่เกิด error เมื่อโมเดลค้าง
- Negative: บอกข้อเสียหรือสิ่งที่ต้องแลก: ความแม่นย​ำลดลงเล็กน้อยจาก 93.5% เหลือ 91.5%, รีวิวที่เกิด timeout จะถูกเก็บเป็น pending ท​ำให้พลาดการแจ้งเตือนทันทีไปบ้าง, และต้องคอยล้าง Cache เมื่อมีการเปลี่ยนโมเดลใหม่

**Alternatives considered:** at least one topology you rejected, and why.

*อย่างน้อยหนึ่ง topology ที่ไม่เลือก และเหตุผล*

> ไม่เลือก T4 เพราะมีการประมวลผลเป็นรอบ ๆ (Asynchronous / batch) ทำให้ส่งแจ้งเตือนภายใน 2 วินาทีไม่ได้
