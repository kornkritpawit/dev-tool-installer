# .NET SDK Installer

เอกสารนี้อธิบายการทำงานของตัวติดตั้ง .NET SDK ในหมวด C# ของแอปพลิเคชัน DevToolInstaller

## Overview

ชุดเครื่องมือ .NET ปัจจุบันประกอบด้วย 1 installer:

1. **DotNetSdk10Installer** - ติดตั้ง .NET 10.0 SDK

## DotNetSdk10Installer

### Purpose
ติดตั้ง **.NET 10.0 SDK** สำหรับพัฒนาแอปพลิเคชัน .NET สมัยใหม่

### Details
- **Package Source**: ดาวน์โหลดโดยตรงจาก Microsoft (builds.dotnet.microsoft.com)
- **Target Version**: `10.0.100`
- **Installation Method**: ดาวน์โหลด installer แล้วรันแบบ quiet
- **Category**: CSharp
- **Dependencies**: None

### Architecture Detection
Installer ตรวจสอบ CPU architecture อัตโนมัติและเลือก installer ที่เหมาะสม:

| Architecture | Download URL | Installer File |
|---|---|---|
| **ARM64** | `https://builds.dotnet.microsoft.com/dotnet/Sdk/10.0.100/dotnet-sdk-10.0.100-win-arm64.exe` | `dotnet-sdk-10.0.100-win-arm64.exe` |
| **x64** (default) | `https://builds.dotnet.microsoft.com/dotnet/Sdk/10.0.100/dotnet-sdk-10.0.100-win-x64.exe` | `dotnet-sdk-10.0.100-win-x64.exe` |

การตรวจสอบ architecture ใช้ `RuntimeInformation.OSArchitecture` จาก `System.Runtime.InteropServices`

### Installation Process
1. ดาวน์โหลดไฟล์ installer ตาม architecture ไปยัง temp directory
2. รัน installer ด้วย arguments: `/quiet /norestart`
   - `/quiet` — ไม่แสดง UI
   - `/norestart` — ไม่รีสตาร์ทเครื่องอัตโนมัติ
3. ลบไฟล์ installer หลังติดตั้งเสร็จ
4. Refresh environment variables เพื่อให้ `dotnet` command พร้อมใช้งานทันที

### IsInstalled Logic
การตรวจสอบมี 2 ขั้นตอน:
1. ตรวจสอบว่า `dotnet.exe` อยู่ใน PATH ผ่าน `FindExecutableInPathAsync`
2. รัน `dotnet --list-sdks` แล้วตรวจสอบว่า output มี `10.0.` อยู่หรือไม่

> ทั้งสองเงื่อนไขต้องผ่านจึงจะถือว่าติดตั้งแล้ว — ต้องมีทั้ง dotnet command และ SDK เวอร์ชัน 10.0.x

## Usage in Application

Installer นี้ถูกลงทะเบียนใน ToolRegistry และเข้าถึงได้ผ่าน:
- เมนูหมวด "C# Development"
- การเลือกติดตั้งรายเครื่องมือ
- ไม่มี dependency กับ installer อื่น — ติดตั้งได้อิสระ

## Error Handling and Verification

- ตรวจสอบ architecture ก่อนดาวน์โหลดเพื่อเลือก installer ที่ถูกต้อง
- จัดการไฟล์ installer ชั่วคราว (ดาวน์โหลดแล้วลบ)
- Refresh environment variables หลังติดตั้งสำเร็จ
- รองรับการรายงานสถานะแบบละเอียดเพื่อการดีบัก