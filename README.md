# screen-wall

แขกสแกน QR ส่งข้อความและรูปขึ้นจอในงาน

- ระบบทำงานยังไง ทำอะไรได้บ้าง แยกตาม module (เปิดในเบราว์เซอร์): [docs/system-overview.html](docs/system-overview.html)
- วิธีเปิด-ปิดและใช้งานทีละขั้น: [GUIDE.md](GUIDE.md)
- การออกแบบทั้งหมดและสถานะของแต่ละขั้น: [DESIGN.md](DESIGN.md)
- ขึ้นเซิร์ฟเวอร์จริง (VPS): [DEPLOY.md](DEPLOY.md) สคริปต์อยู่ใน `deploy/`

## รันในเครื่อง

```bash
npm run db:init       # ครั้งแรกบนเครื่องใหม่เท่านั้น: สร้าง .pgdata ตาราง frame และพรีเซ็ตสำเร็จรูป
npm run admin:create  # ครั้งแรกเท่านั้น: สร้างบัญชีผู้ดูแล (ระบบมีบัญชีเดียว)
npm run db:start      # เปิด Postgres เฉพาะโปรเจกต์ (.pgdata พอร์ต 54417)
npm run db:migrate    # อัปเดตตารางเมื่อมี migration ใหม่
npm run dev           # http://localhost:3100
```

เข้าหน้าผู้จัดที่ http://localhost:3100/admin ด้วยชื่อผู้ใช้และรหัสผ่าน ลืมรหัสผ่านใช้ `npm run admin:reset-password`

หน้าผู้จัดเป็น dashboard มีเมนู: งาน (รายการ ค้นหา สร้าง แก้ไข ทำสำเนา ลบ) / พรีเซ็ต (สร้าง แก้ไข ลบ) / Frame / สื่อพักจอ / บัญชี แต่ละงานมี 3 ลิงก์: หน้าแขก (`/e/<slug>`), หน้าผู้คัดกรอง (`/m/<token>`), หน้าจอ (`/d/<token>`)

`npm run purge` รันงานลบข้อมูลที่หมดอายุหนึ่งครั้ง (ต้องมี `CRON_SECRET` ใน `.env.local` และเซิร์ฟเวอร์ต้องเปิดอยู่) บน Vercel ตั้งเวลาไว้แล้วใน `vercel.json`

ตัวแปรใน `.env.local`: `DATABASE_URL`, `CRON_SECRET`, `PUBLIC_BASE_URL` (ไม่บังคับ: โดเมนจริงที่ใส่ใน QR), `DEV_ORIGINS` (ไม่บังคับ: เลข IP ของเครื่องสำหรับทดสอบกับมือถือ)

อย่ารัน `npm run build` ขณะที่ `npm run dev` เปิดอยู่ ทั้งสองใช้โฟลเดอร์ `.next` ร่วมกัน บางครั้ง dev ไม่รับรู้ไฟล์ที่แก้ (เจอทั้ง CSS และโค้ดฝั่งเซิร์ฟเวอร์) ถ้าหน้าตาหรือ API ไม่อัปเดต ให้หยุด dev ลบ `.next` แล้วเปิดใหม่

## โครงสร้าง

- `db/migrations/` — schema, frame และพรีเซ็ตสำเร็จรูป
- `src/lib/auth.ts`, `scripts/password.mjs` — เข้าสู่ระบบ session และรหัสผ่าน
- `src/lib/queue.ts` — วงจรสถานะของรายการ คิวของจอ และการวน frame
- `src/lib/frames.ts` — โครงสร้าง frame และการตรวจข้อมูล
- `src/lib/frame-store.ts` — อ่านเขียน frame ในฐานข้อมูล
- `src/lib/presets.ts` — พรีเซ็ตในฐานข้อมูล
- `src/lib/events.ts` — สร้าง ทำสำเนา ลบงาน ตั้งค่า ลิงก์ ข้อความทดสอบ ลบข้อมูลอัตโนมัติ
- `src/lib/export.ts`, `src/lib/zip.ts` — ไฟล์ ZIP ของที่ระลึก
- `src/lib/idle.ts` — คลังสื่อพักจอ เพลย์ลิสต์ และชุดข้อมูลพักจอที่จอโหลด
- `src/lib/storage.ts` — ที่เก็บรูปแขก ภาพของ frame และสื่อพักจอ (ดิสก์ในเครื่อง)
- `src/components/frame.tsx` — ตัววาด frame ที่จอ ตัวอย่างบนมือถือ และตัวแก้ไขใช้ร่วมกัน
- `src/app/api/` — API ของแขก ผู้คัดกรอง จอ และผู้จัด (`api/admin/*` ต้องเข้าสู่ระบบ)
- `src/app/admin/login` — หน้าเข้าสู่ระบบ
- `src/app/admin/(dash)/` — หน้า dashboard ทั้งหมด ส่วนประกอบที่ใช้ร่วมกันอยู่ใน `src/app/admin/_components/`
- `src/app/e`, `src/app/m`, `src/app/d` — หน้าแขก ผู้คัดกรอง จอ
# screen-wall
