# Font Installer

เอกสารนี้อธิบายการทำงานของตัวติดตั้งฟอนต์ในโปรเจกต์ DevToolInstaller

## ที่มาไฟล์ฟอนต์

ตัวติดตั้งใช้ฟอนต์จาก 2 แหล่ง:

- **CascadiaMono Nerd Font** — ดาวน์โหลดที่ runtime จาก GitHub Releases:
  `https://github.com/ryanoasis/nerd-fonts/releases/download/v3.4.0/CascadiaMono.zip`
- **TH Sarabun PSK** — ฝังมากับโปรเจกต์ (bundled) ในโฟลเดอร์ `font/THSARABUN_PSK.zip`

## วิธีติดตั้ง

เมื่อเลือกเครื่องมือ **Developer Fonts** ระบบจะ:

1. ตรวจสอบสิทธิ์แอดมิน (จำเป็นสำหรับเขียนลงโฟลเดอร์ฟอนต์ของ Windows)
2. ดาวน์โหลด CascadiaMono.zip จาก GitHub แล้วแตกไฟล์ไปยังโฟลเดอร์ชั่วคราว
3. แตกไฟล์ TH Sarabun PSK จาก bundled zip ไปยังโฟลเดอร์ชั่วคราวเดียวกัน (ถ้าไม่พบ zip จะข้ามโดยแจ้งเตือน)
4. ค้นหาไฟล์ฟอนต์นามสกุล `.ttf` และ `.otf`
5. คัดลอกไฟล์ไปยังโฟลเดอร์ `C:\Windows\Fonts`
6. ลงทะเบียนฟอนต์ใน Windows Registry (`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Fonts`)
7. เรียก Win32 API (`AddFontResource` + `WM_FONTCHANGE`) เพื่อให้ฟอนต์พร้อมใช้งานทันทีโดยไม่ต้องรีบูต

## พฤติกรรมเมื่อมีฟอนต์อยู่แล้ว

ระบบใช้การคัดลอกแบบ overwrite (`File.Copy(..., overwrite: true)`)  
ดังนั้นถ้าพบไฟล์ชื่อเดียวกันใน `C:\Windows\Fonts` จะ **ถูกแทนที่ (replace)** ด้วยไฟล์จาก zip

> หมายเหตุ: การแทนที่อิงตาม “ชื่อไฟล์” ไม่ได้เทียบเวอร์ชันภายในฟอนต์

## ข้อจำกัด

- รองรับเฉพาะ Windows
- ต้องรันโปรแกรมด้วยสิทธิ์ Administrator
- หากไม่พบไฟล์ `.ttf/.otf` ใน zip ที่แตกออกมา จะรายงานข้อผิดพลาดและหยุดติดตั้ง
- หากไม่พบ bundled TH Sarabun zip จะแสดงคำเตือนแต่ยังคงติดตั้ง CascadiaMono ต่อไปได้
- ต้องมีการเชื่อมต่ออินเทอร์เน็ตสำหรับดาวน์โหลด CascadiaMono Nerd Font