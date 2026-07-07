# Gemini Proxy สำหรับ Money Lover Analytics (โหมด Proxy)

โหมด **Proxy** ให้ทุกคนใช้ AI ได้โดย **ไม่ต้องมี Gemini API key เอง** — key จริงถูกเก็บไว้ฝั่งเซิร์ฟเวอร์ (Firebase Cloud Functions) เท่านั้น เบราว์เซอร์แค่ยิงคำถามมาที่ URL ของฟังก์ชันนี้

> โหมด **คีย์ส่วนตัว** ในแอปยังใช้ได้ตามปกติ — Proxy เป็นทางเลือกเสริมสำหรับกรณีอยากแชร์ให้คนอื่นใช้ร่วมกัน

การ์ดข้อมูลที่ส่งผ่าน proxy เป็น **ยอดสรุปแบบปิดบังตัวตน** เหมือนกับโหมดคีย์ส่วนตัว (ปัดเศษหลักร้อย, ชื่อบัญชีแทนด้วย A/B/C, ไม่มีรายการดิบ/โน้ต) — proxy ไม่ได้เก็บเนื้อหาคำถาม เก็บแค่ตัวนับจำนวนครั้งเพื่อทำ rate limit

---

## สิ่งที่ต้องมี

- โปรเจกต์ Firebase (จะใช้ตัวเดียวกับที่ใช้ sync อยู่ก็ได้) แผน **Blaze** (จ่ายตามใช้จริง — Cloud Functions v2 ต้องใช้ Blaze; โควตาฟรีต่อเดือนสูง ใช้ส่วนตัวแทบไม่เสียเงิน)
- [Firebase CLI](https://firebase.google.com/docs/cli): `npm i -g firebase-tools`
- Gemini API key (ขอฟรีที่ https://aistudio.google.com/apikey)

## ขั้นตอนติดตั้ง

```bash
# 1) ล็อกอินและผูกโปรเจกต์ (รันในโฟลเดอร์รากของ repo นี้ที่มี firebase.json)
firebase login
firebase use --add            # เลือกโปรเจกต์ Firebase ของคุณ

# 2) ติดตั้ง dependencies ของฟังก์ชัน
cd functions && npm install && cd ..

# 3) ตั้งค่า secrets (จะถูกถามให้พิมพ์ค่า)
firebase functions:secrets:set GEMINI_KEY       # วาง Gemini API key จริง
firebase functions:secrets:set ACCESS_CODES     # เช่น: myhome,friend2026 (คั่นด้วย comma)

# 4) deploy
firebase deploy --only functions
```

หลัง deploy สำเร็จ CLI จะพิมพ์ URL ของฟังก์ชันออกมา หน้าตาประมาณ:

```
https://asia-southeast1-<your-project-id>.cloudfunctions.net/askAI
```

## เชื่อมในแอป

1. เปิด Money Lover Analytics → กดปุ่มแชท 💬 → ⚙️ ตั้งค่า
2. เลือกโหมด **🌐 Proxy**
3. วาง **Proxy URL** (ที่ได้จากขั้นตอน deploy) และ **รหัสเข้าใช้** (ค่าใน `ACCESS_CODES`)
4. กด **บันทึก Proxy** — เสร็จแล้วถามคำถามได้เลย

## ปรับแต่งเพิ่ม (แก้ในไฟล์ `index.js`)

| ค่า | ความหมาย | ค่าเริ่มต้น |
|-----|----------|-------------|
| `DAILY_LIMIT` | จำนวนครั้งต่อรหัสต่อวัน | `60` |
| `MAX_BODY` | ขนาด body สูงสุด (bytes) | `64 KB` |
| `ALLOWED_MODELS` | โมเดลที่อนุญาต | flash/pro หลายรุ่น |
| `ALLOWED_ORIGINS` | โดเมนที่เรียกได้ (`['*']` = ทุกที่) | `['*']` |

> เพื่อความปลอดภัยขึ้น แนะนำเปลี่ยน `ALLOWED_ORIGINS` จาก `['*']` เป็น origin ของเว็บคุณจริง เช่น `['https://yourname.github.io']`

## ค่าใช้จ่าย

- Cloud Functions v2 มีโควตาฟรีต่อเดือน (2M invocations) — การใช้ส่วนตัว/ครอบครัวแทบไม่เกิน
- ค่า Gemini คิดตาม token ที่ใช้จริง (โมเดล `*-flash` ราคาถูกมาก)
- `maxInstances: 5` ในโค้ดช่วยกันบิลบานปลายกรณีโดนยิงถล่ม
