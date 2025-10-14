# NS Small B2B – DoHome Edition (มือถือเป็นมิตร)

ฟอร์มเว็บสไตล์มินิมอล โทนสี DoHome ฟอนต์ Prompt:
- ช่องชื่อพนักงาน **ใส่ได้หลายคนในช่องเดียว** (แบบชิป/แท็ก)
- อัปโหลดรูปหน้างาน **ได้สูงสุด 4 รูป**
- ปุ่มดึงพิกัด/แปลงที่อยู่/คำนวณระยะทาง (Distance Matrix) จากพิกัดอ้างอิง (15.7053493,100.053933)
- ส่ง JSON เข้า **Power Automate** เพื่อบันทึก Excel Online

## โครงสร้าง JSON ที่ส่ง
```json
{
  "date": "2025-10-14",
  "team": "A",
  "staffNames": ["แม็กซ์","เอ","บี"],
  "staffNamesText": "แม็กซ์, เอ, บี",
  "siteName": "…",
  "customerName": "…",
  "phone": "…",
  "businessType": "…",
  "productType": "…",
  "estRevenue": "…",
  "currentSupplier": "…",
  "visitDetails": "…",
  "latlng": "15.7:100.05",
  "distanceKm": "12.34",
  "thaiAddress": "ตำบล…, อำเภอ…, จังหวัด…",
  "photosBase64": ["data:image/jpeg;base64,...", "data:image/jpeg;base64,..."]
}
```

> **ข้อดี**: Excel รับคอลัมน์ `staffNamesText` ได้ตรงๆ ส่วน `staffNames`/`photosBase64` เผื่อใช้ต่อ (เช่นสร้างไฟล์ภาพหลายไฟล์)

## สร้าง Flow (สรุปเร็ว)
1) **Trigger:** When an HTTP request is received → วาง JSON Schema ครอบคลุม fields ข้างบน (ตั้ง `photosBase64` เป็น `array` ของ `string`).
2) **สร้างไฟล์รูปหลายไฟล์ (ถ้ามี):** ใช้ **Apply to each** วน `photosBase64`
   - Compose `Base64Only = split(items('Apply_to_each'), ',')[1]`
   - Create file: `base64ToBinary(outputs('Base64Only'))`
   - สะสมลิงก์ไฟล์ด้วย Array variable `photoUrls`
3) **Add a row into a table** ใส่:
   - `Date, Team, StaffNames, SiteName, CustomerName, Phone, BusinessType, ProductType, EstRevenue, CurrentSupplier, VisitDetails, LatLng, DistanceKm, ThaiAddress, PhotoUrls`
   - `StaffNames` ← `@{triggerBody()?['staffNamesText']}`
   - `PhotoUrls` ← `join(variables('photoUrls'), '; ')` (ถ้าไม่มีให้ใส่ค่าว่าง)
4) **Response 200 OK**

## ตั้งค่าในไฟล์
แก้ 2 จุดใน `index.html`:
```js
const GOOGLE_API_KEY = "YOUR-GOOGLE-API-KEY";
const FLOW_URL = "PUT-YOUR-POWER-AUTOMATE-HTTP-POST-URL-HERE";
```

## การใช้งานบนมือถือ (แนะนำ)
- เปิดโฮสต์ HTTPS จริง เช่น **GitHub Pages, Netlify, Vercel** (ลากโฟลเดอร์นี้ขึ้นได้เลย)
- ครั้งแรกอนุญาต **Location** ในเบราว์เซอร์
- บน iPhone/Safari: Share → **Add to Home Screen** จะได้ไอคอน DoHome ใช้งานเหมือนแอป
- บน Android/Chrome: เมนู ⋮ → **Add to Home screen**
- แนะนำโหมด **หน้าจอแนวตั้ง** เพื่อฟอร์มอ่านง่าย

## โครงสร้างโฟลเดอร์
```
ns_small_b2b_dohome/
  index.html
  assets/
    dohome_logo.jpg
  README.md
```
