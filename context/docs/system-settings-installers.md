# System Settings Installers

เอกสารนี้อธิบายการทำงานของตัวติดตั้งที่ปรับแต่งค่า Windows system settings ในแอปพลิเคชัน DevToolInstaller ครอบคลุม Windows Explorer settings และ WSL2 configuration

## Overview

ชุดเครื่องมือ System Settings ประกอบด้วย 2 installer:

1. **WindowsExplorerSettingsInstaller** - ปรับแต่ง Windows Explorer ให้แสดง hidden files และ file extensions
2. **WslConfigInstaller** - อัปเดต WSL และตั้งค่า memory/swap limits ผ่านไฟล์ `.wslconfig`

## WindowsExplorerSettingsInstaller

### Purpose

ปรับแต่ง Windows Explorer ให้เหมาะกับการพัฒนา โดยแสดง hidden files/folders และ file extensions ซึ่งเป็นค่าที่ Windows ซ่อนไว้เป็นค่าเริ่มต้น

### Details
- **Name**: Windows Explorer Settings
- **Description**: Configure Windows Explorer: show hidden files, show file extensions
- **Category**: CrossPlatform
- **Dependencies**: ไม่มี
- **Source file**: [`WindowsExplorerSettingsInstaller.cs`](../../Installers/WindowsExplorerSettingsInstaller.cs)

### Registry Keys ที่แก้ไข

**Registry Path**: `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced`

| Key | Value | Type | ผลลัพธ์ |
|-----|-------|------|---------|
| `Hidden` | `1` | DWORD | แสดง hidden files และ folders |
| `HideFileExt` | `0` | DWORD | แสดง file extensions (เช่น `.txt`, `.exe`) |

> ใช้ HKCU (Current User) จึงไม่ต้องการสิทธิ์ Administrator

### การตรวจสอบสถานะ (IsInstalledAsync)

อ่านค่า registry และตรวจสอบว่า:
- `Hidden` = `1` (แสดง hidden files)
- `HideFileExt` = `0` (แสดง file extensions)

หากทั้งสองค่าตรงตามที่กำหนด → ถือว่าตั้งค่าแล้ว (return `true`)

### Installation Process

1. เปิด registry key `HKCU\...\Explorer\Advanced` ในโหมด writable
2. ตั้งค่า `Hidden` = `1` (แสดง hidden files)
3. ตั้งค่า `HideFileExt` = `0` (แสดง file extensions)
4. **Refresh Explorer** — รันคำสั่ง:
   ```
   cmd /c taskkill /f /im explorer.exe & start explorer.exe
   ```
   เพื่อให้ settings มีผลทันทีโดยไม่ต้อง restart Windows

## WslConfigInstaller

### Purpose

อัปเดต WSL เป็นเวอร์ชันล่าสุด และตั้งค่า `.wslconfig` เพื่อจำกัด memory และ swap ของ WSL2 ป้องกันไม่ให้ WSL2 ใช้ RAM มากเกินไป

### Details
- **Name**: WSL2 Memory Limit
- **Description**: Configure WSL2: limit memory to 4GB, swap to 8GB (.wslconfig)
- **Category**: CrossPlatform
- **Dependencies**: ไม่มี
- **Config File Path**: `%USERPROFILE%\.wslconfig`
- **Source file**: [`WslConfigInstaller.cs`](../../Installers/WslConfigInstaller.cs)

### WSL2 Settings ที่ตั้งค่า

| Setting | Value | Description |
|---------|-------|-------------|
| `memory` | `4GB` | จำกัด RAM ที่ WSL2 ใช้ได้สูงสุด 4GB |
| `swap` | `8GB` | กำหนด swap size เป็น 8GB |
| `localhostForwarding` | `true` | เปิดให้เข้าถึง WSL2 services ผ่าน localhost ของ Windows |

### การตรวจสอบสถานะ (IsInstalledAsync)

1. ตรวจสอบว่าไฟล์ `%USERPROFILE%\.wslconfig` มีอยู่
2. อ่านเนื้อหาไฟล์และตรวจสอบว่ามี `memory=4GB` และ `swap=8GB`
3. หากพบทั้งสองค่า → ถือว่าตั้งค่าแล้ว (return `true`)

### Installation Process

#### ขั้นตอนที่ 1: อัปเดต WSL

รันคำสั่ง:
```
wsl --update
```
เพื่ออัปเดต WSL เป็นเวอร์ชันล่าสุด

#### ขั้นตอนที่ 2: สร้าง/แก้ไข .wslconfig

**กรณีไฟล์ยังไม่มี**: สร้างไฟล์ใหม่พร้อมเนื้อหา:
```ini
[wsl2]
memory=4GB
swap=8GB
localhostForwarding=true
```

**กรณีไฟล์มีอยู่แล้ว**: ใช้ smart merge logic:
1. อ่านไฟล์ทีละบรรทัด
2. ค้นหา section `[wsl2]`
3. หากพบ settings ที่ต้องการ (`memory=`, `swap=`, `localhostForwarding=`) → แทนที่ด้วยค่าใหม่
4. หากไม่พบ settings ที่ต้องการใน `[wsl2]` section → เพิ่มเข้าไป
5. หากไม่มี `[wsl2]` section → เพิ่ม section ใหม่ท้ายไฟล์
6. **รักษา settings อื่นที่มีอยู่เดิม** — ไม่ลบหรือแก้ไข settings ที่ไม่เกี่ยวข้อง

#### ขั้นตอนที่ 3: แจ้งผล

แจ้งเตือนผู้ใช้ว่าต้อง restart WSL เพื่อให้ settings มีผล:
```
wsl --shutdown
```

## Error Handling and Verification

- **WindowsExplorerSettingsInstaller**:
  - ตรวจสอบ registry values ก่อนตั้งค่าซ้ำ
  - Refresh Explorer อัตโนมัติหลังตั้งค่า
  - จัดการ error กรณีเปิด registry key ไม่ได้

- **WslConfigInstaller**:
  - Smart merge — รักษา settings เดิมที่ไม่เกี่ยวข้อง
  - รองรับทั้งกรณีไฟล์ใหม่และไฟล์ที่มีอยู่แล้ว
  - จัดการ error กรณีอ่าน/เขียนไฟล์ไม่ได้
  - แจ้งเตือนให้ restart WSL เพื่อใช้ settings ใหม่