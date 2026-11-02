# Tutorial 6 worksheet: serving patterns

**Name / ชื่อ:** วรเมธ พงศาประเสริฐพร

**Student ID / รหัสนิสิต:** 6970259821

This is **individual work**: each student completes and submits their own copy, in their own fork.

*งานนี้เป็น**งานเดี่ยว** นิสิตแต่ละคนทำและส่งงานของตัวเอง โดยใช้ forked repository ของตัวเอง*

Copy result rows from `results/runs.csv` (or the load test's output) into the tables. Keep answers short: one to three sentences each. Answer in Thai or English.

ตอนที่ตอบ ให้นิสิตคัดลอกผลจาก `results/runs.csv` (หรือจาก output ของ load test) ลงในตาราง ตอบสั้นๆ ข้อละ 1–3 ประโยค จะตอบเป็นภาษาไทยหรือภาษาอังกฤษก็ได้

## Task 0: Baseline (copy the baseline results from `results/runs.csv` here / copy ผลลัพธ์ของ baseline จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|
| t0-large | 492 | 592 | 93.5% | 100% |
| t0-small | 63 | 148 | 72.5% | 0% |

**Q0.** What does the large model buy, and what does it cost?

*large model ให้อะไรเพิ่ม และต้องแลกด้วยอะไร*

>สิ่งที่เห็นได้ชัดจากผลทดสอบคือ Large model นั้นให้ความแม่นย่ำที่มากกว่า โดยทำได้ถึง 93.5% ในขณะที่ Small model ทำได้เพียงแค่ 72.5% แต่สิ่งที่ต้องแลกมาด้วยนั้นคือ Latency ที่สูงกว่าอย่างมาก

## Task 1: Fallback prediction (copy the Task 1 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 1 จาก `results/runs.csv` มาวางที่นี่)

| Run | run time (s) | p50 ms | p95 ms | errors | pending | accuracy |
|---|---|---|---|---|---|---|
| t1-before | 26.2 | 236 | 361 | 18 | 0 | 93.4% |
| t1-after | 9.6 | 405 | 1008 | 0 | 20 | 92.8% |

**Q1.1** In `t1-before`, p50 can look *better* than the baseline. Why is p50 misleading here? Which column tells the real story?

*ใน `t1-before` ค่า p50 อาจดู "ดีกว่า" baseline ทำไม p50 จึงทำให้เข้าใจผิดในกรณีนี้ และ column ไหนบอกความจริง*

> ที่ p50 ดูดีกว่าเพราะใน load test มีการยิง requests พร้อมกันถึง 10 requests ทำให้เส้นที่วิ่งอยู่มีจำนวนน้อยลง และแย่งคิวของ Large model น้อยลง เลยทำให้ตอบกลับมาไวขึ้น ส่วนที่บอกความจริงคือ error ที่สูงถึง 18 รายการ และ run time ที่ใช้เวลาถึง 26.2 วินาที

**Q1.2** The hang affects only 10% of large-model calls. Why is it worse for the system than 10% of reviews being slow? Think about what a waiting request holds on to in the review service and the gateway.

*การค้างเกิดกับ calls ไปยัง large model แค่ 10% ทำไมจึงส่งผลต่อระบบแย่กว่าการที่รีวิว 10% ช้าลงเฉยๆ ลองคิดว่า request ที่รออยู่ถือทรัพยากรอะไรไว้ใน review service และ gateway*

> เพราะการที่ call หนึ่งค้างถึง 20 นาทีนั้น มันทำให้ส่วนอื่น ๆ เช่น review service และ gateway ค้างไปด้วย ทำให้เมื่อมี requests อื่น ๆ ไหลเข้ามา ก็พลอยที่จะค้างตามกันไปด้วย

**Q1.3** Pending reviews have no label. Suggest what should happen to them, and name the topology from the lecture that does it.

*รีวิวที่ pending ยังไม่มี label ควรจัดการกับรีวิวเหล่านี้อย่างไร และ topology ใดจาก lecture ที่ทำแบบนั้น*

> ใช้ background worker หรือ scheduled job ในการดึงรีวิวที่เป็น pending ออกมา และใช้ topolody -> Asynchronous / batch ที่เก็บหรือเข้าคิวไว้ก่อนและค่อยนำ label นั้นมาประมวลผล

**Q1.4** We chose a 1-second timeout. What goes wrong if it is 0.1 s? And if it is 10 s?

*เราเลือก timeout 1 วินาที จะเกิดปัญหาอะไรถ้าตั้งเป็น 0.1 วินาที และถ้าตั้งเป็น 10 วินาที*

> ถ้าตั้งเป็น 0.1 วินาที จะทำให้เกือบทุกรีวิวนั้นถูกตัดให้เป็น pending หมด และหากตั้ง 10 วินาทีซึ่งนานเกินไป จะทำให้ผู้ใช้รอนานเกินควร

## Task 2: Cascade (copy the Task 2 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 2 จาก `results/runs.csv` มาวางที่นี่)

| Run | threshold | p50 ms | p95 ms | accuracy | large calls |
|---|---|---|---|---|---|
| t2-0.6 | 0.6 | 13 | 527 | 91.5% | 38% |
| t2-0.7 | 0.7 | 392 | 577 | 93.0% | 53% |
| t2-0.8 | 0.8 | 445 | 573 | 93.5% | 64% |
| t2-0.9 | 0.9 | 476 | 601 | 93.5% | 77% |

**Q2.1** Which threshold would you deploy for CampusEats, and why? Name the quality you give up.

*จะเลือก threshold ใดสำหรับ deploy ใน CampusEats เพราะอะไร และต้องยอมเสีย quality ใดไป*

> เลือกแบบต่ำ เพราะประหยัด Large call ไปได้มาก และเร็วมาก (p50 เพียงแค่ 13ms) แต่ quality ที่ต้องเสียไปคือ Accuracy ที่ลดลงเหลือ 91.5% ซึ่งยอมรับได้สำหรับผม

**Q2.2** Look at p50 and p95 for your threshold. Why does the cascade help p50 much more than p95?

*ดูค่า p50 และ p95 ของ threshold ที่เลือก ทำไม cascade จึงช่วย p50 ได้มากกว่า p95 มาก*

> เพราะ p50 คือ reqeust ที่เป็นในส่วนของประสบการณ์ผู้ใช้ ซึ่งถ้า Small model ตอบได้ดี ค่า p50 จะลดลงอย่างมาก ส่วน p95 คือกลุ่ม 5% ที่ช้าที่สุด ซึ่งเป็นส่วนที่ตอบยากที่สุด ทำให้ Small model ต้องส่งต่อไปที่ Large model ซึ่งต้องรอค่อนข้างนาน

**Q2.3** The cascade trusts the small model's confidence. Give one situation where that trust is misplaced, and its effect on users.

*cascade เชื่อ confidence ของ small model ยกตัวอย่างหนึ่งสถานการณ์ที่ไม่ควรเชื่อ และผลที่เกิดกับผู้ใช้*

> เช่น การได้รีวิวว่า not bad ที่แแปลว่า ดี หรือ ไม่แย่ แต่ small model ตีความว่าบริบทของ bad นั้นเป็นความหมายในเชิงลบ เพราะคำว่า bad นั้นอาจะเป็นคำที่ดูแง่ลบแบบสุดโต่ง โมเดลเลยมีความมั่นใจที่จะไม่ส่งต่อไปยัง Large model  และส่งไปให้ร้านค้า ทำให้เกิดความเข้าใจผิด

## Task 3: Cached predictions (copy the Task 3 results from `results/runs.csv` here / copy ผลลัพธ์ของ Task 3 จาก `results/runs.csv` มาวางที่นี่)

| Run | p50 ms | p95 ms | accuracy | large calls | cache hits |
|---|---|---|---|---|---|
| t3-cache | 10 | 518 | 91.5% | 31% | 36% |

**Q3.1** Which reviews benefit from the cache, and which never will?

*รีวิวแบบไหนได้ประโยชน์จาก cache และแบบไหนไม่มีวันได้ประโยชน์*

> แบบที่ได้ประโยชน์คือรีวิวแบบสั้น ๆ เช่น Good, Great, Delicious ส่วนรีวิวที่ไม่ได้ประโยชน์คือรีวิวที่มีขนาดยาว เขียรบรรยาย เพราะจะทำให้เกิด cache miss

**Q3.2** Next week you deploy a better large model. What must happen to the cache, and how would you design for it?

*สัปดาห์หน้าจะ deploy large model ตัวใหม่ที่ดีกว่าเดิม ต้องทำอะไรกับ cache และจะออกแบบรองรับเรื่องนี้อย่างไร*

> เนื่องจาก cache นั้นจะเก็บค่าเดิมจากโมเดลตัวเก่าไว้ ทำให้เราต้องทำการล้างค่าเดิมจาก cache ออกเสียก่อน และทำการผูเวอร์ชันของโมเดลเข้ากับ cache key ทำให้ระบบสามารถเรียกโมเดลใหม่ได้อย่างอัตโนมัติ และกำหนดอายุของข้อมูล ที่ทำให้ cache นั้นทำการ deployment pipeline อย่างเป็นอัตโนมัติ

## Debrief

**Q4.** Name one thing you learned from these measurements that the lecture did not cover.

*ยกตัวอย่างหนึ่งสิ่งที่ได้เรียนรู้จากการวัดผลครั้งนี้ ที่ไม่มีพูดถึงใน lecture*

> การทำ Normalization เพียงเล็กน้อยให้กับ cache ก็สามารถลดค่าใช้จ่ายอง Large model ได้
