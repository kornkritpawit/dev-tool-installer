# Docker Desktop Installer

เอกสารนี้อธิบายการทำงานของตัวติดตั้ง Docker Desktop ในแอปพลิเคชัน DevToolInstaller รวมถึงการ auto-configuration, การ start on boot, และการ pull Docker image เริ่มต้น

## Overview

Docker Desktop installer ทำงาน 3 ขั้นตอนหลัก:

1. **ติดตั้ง Docker Desktop** — ผ่าน winget หรือ direct download
2. **ตั้งค่า Docker Desktop** — กำหนด memory, CPU, swap ผ่าน settings.json
3. **Pull Docker image** — ดึง pgvector/pgvector:pg17 image อัตโนมัติ

## DockerDesktopInstaller

### Details
- **Name**: Docker Desktop
- **Description**: Container platform for developing, shipping, and running applications
- **Category**: CrossPlatform
- **Dependencies**: ไม่มี
- **Source file**: [`DockerDesktopInstaller.cs`](../../Installers/DockerDesktopInstaller.cs)

### การตรวจสอบสถานะ (IsInstalledAsync)

ตรวจสอบ 2 วิธี:
1. **PATH lookup** — ค้นหา `docker.exe` ใน system PATH
2. **Common install paths** — ตรวจสอบ:
   - `%ProgramFiles%\Docker\Docker\resources\bin\docker.exe`
   - `%ProgramFiles%\Docker\Docker\Docker Desktop.exe`

> Docker Desktop ไม่ได้เพิ่ม PATH ทันทีหลังติดตั้ง จึงต้องตรวจสอบ common paths ด้วย

### Installation Process

#### วิธีที่ 1: ผ่าน winget (Primary)
1. ตรวจสอบว่า `winget` พร้อมใช้งาน
2. รันคำสั่ง: `winget install --id=Docker.DockerDesktop -e --source=winget --accept-source-agreements --accept-package-agreements --force`
3. หากสำเร็จ → ดำเนินการ auto-configuration

#### วิธีที่ 2: Direct Download (Fallback)
1. ดาวน์โหลด installer จาก: `https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe`
2. บันทึกไฟล์ไปที่ `%TEMP%\DockerDesktopInstaller.exe`
3. รัน installer ด้วย argument: `install --quiet`
4. ลบไฟล์ installer หลังเสร็จ
5. หากสำเร็จ → ดำเนินการ auto-configuration

### Auto-Configuration (ConfigureDockerAsync)

หลังจากติดตั้งสำเร็จ installer จะตั้งค่าอัตโนมัติ 4 ส่วน:

#### 1. Docker Desktop Settings

แก้ไขไฟล์ `%AppData%\Docker Desktop\settings.json`:

| Setting | Value | Description |
|---------|-------|-------------|
| `memoryMiB` | `2048` | จำกัด memory เป็น 2GB |
| `cpus` | `Environment.ProcessorCount` | ใช้ CPU ทั้งหมดที่มี |
| `swapMiB` | `1024` | กำหนด swap เป็น 1GB |

การอ่าน/เขียน JSON ใช้ `System.Text.Json` ร่วมกับ [`DockerSettingsContext`](../../DockerSettingsContext.cs) (source-generated JSON serializer สำหรับ AOT compatibility)

> หมายเหตุ: จะแก้ไข settings เฉพาะเมื่อไฟล์ settings.json มีอยู่แล้ว (Docker Desktop ต้องถูกเปิดอย่างน้อย 1 ครั้งเพื่อสร้างไฟล์)

#### 2. Start on Boot

เพิ่ม registry entry เพื่อให้ Docker Desktop เปิดอัตโนมัติเมื่อ Windows เริ่มต้น:

- **Registry Path**: `HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`
- **Value Name**: `Docker Desktop`
- **Value Data**: `"%ProgramFiles%\Docker\Docker\Docker Desktop.exe"`

#### 3. Start Docker Desktop

เปิด Docker Desktop ในโหมด minimized:
```
"Docker Desktop.exe" --minimized
```

#### 4. Pull Default Image

รอให้ Docker พร้อมใช้งาน (retry สูงสุด 10 ครั้ง, แต่ละครั้งรอ 5 วินาที) แล้ว pull image:
```
docker pull pgvector/pgvector:pg17
```

- หาก Docker ไม่พร้อมภายในเวลาที่กำหนด (~50 วินาที) จะข้าม image pull พร้อมแจ้งเตือน

## DockerSettingsContext

### Details
- **Source file**: [`DockerSettingsContext.cs`](../../DockerSettingsContext.cs)
- **Purpose**: Source-generated JSON serializer context สำหรับอ่าน/เขียน Docker Desktop settings.json
- **Supported Types**: `Dictionary<string, object>`, `int`, `long`, `string`, `bool`, `double`

ใช้ `[JsonSerializable]` attributes สำหรับ .NET AOT (Ahead-of-Time) compilation compatibility แทนการใช้ reflection-based JSON serialization

## Error Handling and Verification

- ตรวจสอบ Docker ก่อนติดตั้งซ้ำ (ทั้ง PATH และ common install paths)
- Fallback จาก winget ไป direct download อัตโนมัติ
- ลบไฟล์ installer ชั่วคราวหลังติดตั้งเสร็จ
- ConfigureDockerAsync จัดการ error แบบ graceful — ไม่ทำให้การติดตั้งหลักล้มเหลว
- รอ Docker พร้อมใช้งานด้วย retry mechanism ก่อน pull image