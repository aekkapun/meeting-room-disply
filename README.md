# Digital Signage — ตารางการประชุม

จอแสดงตารางการประชุมสำหรับติดหน้าโถง/หน้าห้อง ดึงข้อมูลจาก **Digital Signage API**
ของระบบจองห้องประชุม (`/meet-reserve/api/signage`)

## ตั้งค่า

คัดลอก `.env.example` เป็น `.env` แล้วแก้ค่า:

| ตัวแปร | ค่าเริ่มต้น | ความหมาย |
|---|---|---|
| `VITE_SIGNAGE_API` | `https://it-asset.dlt.go.th/meet-reserve/api/signage` | URL ของ API |
| `VITE_SIGNAGE_KEY` | (ว่าง) | ใส่เมื่อระบบจองห้องตั้ง `SIGNAGE_API_KEY` |
| `VITE_REFRESH_SECONDS` | `60` | ดึงข้อมูลใหม่ทุกกี่วินาที |

ตอนพัฒนาในเครื่อง `.env.development` ชี้ไปที่ `http://localhost:4500/meet-reserve/api/signage` อยู่แล้ว

## รัน

```bash
npm install
npm run dev      # http://localhost:5173/digital-signage/
npm run build    # ได้ไฟล์ใน dist/ เอาไปวางบนเว็บเซิร์ฟเวอร์ที่ path /digital-signage/
```

## การใช้งานบนจอ

- ปุ่มมุมขวาบน: เลือกห้องที่ต้องการแสดง (จำไว้ใน localStorage ของเครื่องนั้น) และปุ่มเต็มจอ
- รายการเลื่อนอัตโนมัติเมื่อเนื้อหายาวเกินจอ และหยุดชั่วคราวเมื่อมีคนแตะ
- ถ้าดึงข้อมูลรอบใหม่ไม่สำเร็จ จอจะคงข้อมูลเดิมไว้ ไม่ขึ้นจอว่าง

ข้อมูลที่ API ส่งมาไม่มีเลขบัตรประชาชน เบอร์โทร และรายชื่อผู้เข้าร่วม
