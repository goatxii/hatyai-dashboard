# IP PA System Dashboard

React + Tailwind CSS (Vite) — ไฟล์ทั้งหมดใช้นามสกุล `.js`

## วิธีรันบน Visual Studio Code
1. ติดตั้ง Node.js (v18 ขึ้นไป) จาก https://nodejs.org
2. เปิดโฟลเดอร์นี้ใน VS Code (File > Open Folder...)
3. เปิด Terminal (Ctrl + `) แล้วรัน
   ```
   npm install
   npm run dev
   ```
4. เบราว์เซอร์จะเปิดที่ http://localhost:5173

## สร้างไฟล์สำหรับ deploy
```
npm run build
```
ได้โฟลเดอร์ `dist/`

## โครงสร้าง
- `src/PADashboard.js` — คอมโพเนนต์แดชบอร์ด (แก้ข้อมูลโซน/อุปกรณ์ได้ที่ด้านบนไฟล์)
- `src/main.js` — จุดเริ่มต้นของแอป
- `src/index.css` — directives ของ Tailwind
