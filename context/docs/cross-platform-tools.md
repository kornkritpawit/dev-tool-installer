# Cross-Platform Tools Installers

เอกสารนี้อธิบายการทำงานของตัวติดตั้งในหมวด Cross-Platform ของแอปพลิเคชัน DevToolInstaller ครอบคลุมเครื่องมือพื้นฐานสำหรับการพัฒนาซอฟต์แวร์ การจัดการ terminal และเครื่องมือเครือข่าย

## Overview

ชุดเครื่องมือ Cross-Platform ประกอบด้วย 7 installer:

1. **GitInstaller** - ติดตั้ง Git version control system
2. **WindowsTerminalInstaller** - ติดตั้ง Windows Terminal
3. **PowerShell7Installer** - ติดตั้ง PowerShell 7
4. **NotepadPlusPlusInstaller** - ติดตั้ง Notepad++ text editor
5. **PostmanInstaller** - ติดตั้ง Postman API client
6. **RustDeskInstaller** - ติดตั้ง RustDesk remote desktop
7. **WireGuardInstaller** - ติดตั้ง WireGuard VPN client

> หมายเหตุ: Installer ทั้งหมดในหมวดนี้ไม่มี dependency ต่อกัน สามารถติดตั้งแยกอิสระได้

## Dependency Chain

ไม่มี dependency chain ในกลุ่มนี้ — ทุก installer มี `Dependencies => new()` (empty list)

## GitInstaller

### Purpose
ติดตั้ง **Git for Windows v2.52.0** (64-bit) ระบบ version control แบบ distributed

### Details
- **Installation Method**: winget (primary) / direct download (fallback)
- **Winget ID**: `Git.Git`
- **Fallback Download URL**: `https://github.com/git-for-windows/git/releases/download/v2.52.0.windows.1/Git-2.52.0-64-bit.exe`
- **Installer Filename**: `GitSetup.exe`
- **Target Version**: `2.52.0` (64-bit)
- **Category**: CrossPlatform
- **Dependencies**: None

### IsInstalled Logic
- ค้นหา `git.exe` ใน PATH ผ่าน `ProcessHelper.FindExecutableInPathAsync`

### Installation Process
1. ตรวจสอบว่า `winget` พร้อมใช้งานหรือไม่
2. **วิธี winget**: รัน `winget install --id Git.Git -e --source winget --accept-source-agreements --accept-package-agreements`
3. **วิธี fallback** (หาก winget ล้มเหลว):
   - ดาวน์โหลด installer จาก GitHub releases ไปที่ temp directory
   - รัน installer ด้วย arguments `/VERYSILENT /NORESTART`
   - ลบไฟล์ installer หลังติดตั้งเสร็จ

### หมายเหตุ
- ใช้ Inno Setup installer สำหรับ fallback (flag `/VERYSILENT`)
- มี cleanup ลบไฟล์ installer ชั่วคราวหลังติดตั้ง

## WindowsTerminalInstaller

### Purpose
ติดตั้ง **Windows Terminal** — แอป terminal สมัยใหม่ที่รองรับ tabs, panes และ Unicode

### Details
- **Installation Method**: winget only
- **Winget ID**: `Microsoft.WindowsTerminal`
- **Category**: CrossPlatform
- **Dependencies**: None

### IsInstalled Logic
1. ตรวจสอบ MSIX package ผ่าน PowerShell command: `Get-AppxPackage -Name 'Microsoft.WindowsTerminal'`
2. หากไม่พบ — ค้นหา `wt.exe` ใน PATH เป็น secondary check

### Installation Process
1. ตรวจสอบว่า `winget` พร้อมใช้งาน
2. รัน `winget install --id=Microsoft.WindowsTerminal -e --source=winget --accept-source-agreements --accept-package-agreements --force`
3. หากไม่มี winget — แสดงคำเตือนให้ผู้ใช้ติดตั้งเองจาก Microsoft Store

### หมายเหตุ
- ไม่มี fallback download — ต้องใช้ winget เท่านั้น
- ใช้ `--force` flag เพื่อบังคับติดตั้งแม้มีเวอร์ชันเดิมอยู่
- Windows Terminal เป็น MSIX/Appx package จึงใช้ `Get-AppxPackage` ตรวจสอบ

## PowerShell7Installer

### Purpose
ติดตั้ง **PowerShell 7.5.4** — cross-platform shell และ automation framework

### Details
- **Installation Method**: winget (primary) / direct MSI download (fallback)
- **Winget ID**: `Microsoft.PowerShell`
- **Fallback Download URL**: `https://github.com/PowerShell/PowerShell/releases/download/v7.5.4/PowerShell-7.5.4-win-x64.msi`
- **Installer Filename**: `PowerShell7Setup.msi`
- **Target Version**: `7.5.4` (x64)
- **Category**: CrossPlatform
- **Dependencies**: None

### IsInstalled Logic
- ค้นหา `pwsh.exe` ใน PATH ผ่าน `ProcessHelper.FindExecutableInPathAsync`
- หรือตรวจสอบผ่าน `ProcessHelper.IsToolInstalled("pwsh")`

### Installation Process
1. ตรวจสอบว่า `winget` พร้อมใช้งาน (ผ่าน `ProcessHelper.IsToolInstalled`)
2. **วิธี winget**: รัน `winget install --id=Microsoft.PowerShell -e --source=winget --accept-source-agreements --accept-package-agreements --force`
3. **วิธี fallback** (หาก winget ล้มเหลว):
   - ดาวน์โหลด MSI installer จาก GitHub releases
   - รัน MSI installer ผ่าน `ProcessHelper.ExecuteMsiInstaller`
   - ลบไฟล์ installer หลังติดตั้งเสร็จ

### หมายเหตุ
- ใช้ MSI installer สำหรับ fallback (ไม่ใช่ EXE)
- มี cleanup ลบไฟล์ installer ชั่วคราวหลังติดตั้ง

## NotepadPlusPlusInstaller

### Purpose
ติดตั้ง **Notepad++** — text/source code editor ฟรีพร้อม syntax highlighting

### Details
- **Installation Method**: winget only
- **Winget ID**: `Notepad++.Notepad++`
- **Category**: CrossPlatform
- **Dependencies**: None

### IsInstalled Logic
1. ค้นหา `notepad++.exe` ใน PATH
2. ตรวจสอบ common install locations:
   - `%ProgramFiles%\Notepad++\notepad++.exe`
   - `%ProgramFiles(x86)%\Notepad++\notepad++.exe`

### Installation Process
1. ตรวจสอบว่า `winget` พร้อมใช้งาน
2. รัน `winget install --id=Notepad++.Notepad++ -e --source=winget --accept-source-agreements --accept-package-agreements --force`
3. หากไม่มี winget — แสดงคำเตือนให้ผู้ใช้ติดตั้งเอง

### หมายเหตุ
- ไม่มี fallback download — ต้องใช้ winget เท่านั้น
- ใช้ `--force` flag

## PostmanInstaller

### Purpose
ติดตั้ง **Postman** — แพลตฟอร์มสำหรับสร้าง ทดสอบ และเอกสาร APIs

### Details
- **Installation Method**: winget only
- **Winget ID**: `Postman.Postman`
- **Category**: CrossPlatform
- **Dependencies**: None

### IsInstalled Logic
1. ค้นหา `Postman.exe` ใน PATH
2. ตรวจสอบ common install locations (Postman ติดตั้งใน LocalAppData):
   - `%LocalAppData%\Postman\Postman.exe`
   - `%LocalAppData%\Programs\Postman\Postman.exe`

### Installation Process
1. ตรวจสอบว่า `winget` พร้อมใช้งาน
2. รัน `winget install --id=Postman.Postman -e --source=winget --accept-source-agreements --accept-package-agreements --force`
3. หากไม่มี winget — แสดงคำเตือนให้ผู้ใช้ติดตั้งเอง

### หมายเหตุ
- ไม่มี fallback download — ต้องใช้ winget เท่านั้น
- Postman ติดตั้งใน `LocalAppData` ไม่ใช่ `ProgramFiles` — IsInstalled ต้องตรวจสอบ path พิเศษ

## RustDeskInstaller

### Purpose
ติดตั้ง **RustDesk** — remote desktop client แบบ open-source รองรับ self-hosted server

### Details
- **Installation Method**: winget only
- **Winget ID**: `RustDesk.RustDesk`
- **Category**: CrossPlatform
- **Dependencies**: None

### IsInstalled Logic
1. ค้นหา `rustdesk.exe` ใน PATH
2. ตรวจสอบผ่าน `ProcessHelper.IsToolInstalled("RustDesk")`
3. ตรวจสอบ common install locations:
   - `%ProgramFiles%\RustDesk\rustdesk.exe`
   - `%ProgramFiles(x86)%\RustDesk\rustdesk.exe`

### Installation Process
1. ตรวจสอบว่า `winget` พร้อมใช้งาน
2. รัน `winget install --id=RustDesk.RustDesk -e --source=winget --accept-source-agreements --accept-package-agreements --force`
3. หากไม่มี winget — แสดงคำเตือนพร้อม link ดาวน์โหลด: `https://github.com/rustdesk/rustdesk/releases`

### หมายเหตุ
- ไม่มี fallback download อัตโนมัติ — แต่แสดง URL สำหรับดาวน์โหลดเองหากไม่มี winget
- ใช้ `IsToolInstalled` เป็น check เพิ่มเติมนอกเหนือจาก PATH

## WireGuardInstaller

### Purpose
ติดตั้ง **WireGuard** — VPN tunnel ที่เร็ว ทันสมัย และปลอดภัย

### Details
- **Installation Method**: winget (primary) / bundled installer (fallback)
- **Winget ID**: `WireGuard.WireGuard`
- **Bundled Installer**: `wireguard-installer.exe` (อยู่ในโฟลเดอร์เดียวกับแอปพลิเคชัน)
- **Category**: CrossPlatform
- **Dependencies**: None

### IsInstalled Logic
1. ค้นหา `wireguard.exe` ใน PATH
2. ตรวจสอบ common install locations:
   - `%ProgramFiles%\WireGuard\wireguard.exe`
   - `%ProgramFiles(x86)%\WireGuard\wireguard.exe`

### Installation Process
1. **วิธี winget** (preferred): รัน `winget install --id=WireGuard.WireGuard -e --source=winget --accept-source-agreements --accept-package-agreements --force`
2. **วิธี fallback** (bundled installer):
   - ค้นหา `wireguard-installer.exe` ในโฟลเดอร์เดียวกับ executable ของแอป (`Environment.ProcessPath`)
   - รัน installer ด้วย argument `/S` (silent install)
3. หากทั้ง winget และ bundled installer ไม่สำเร็จ — แสดง error

### หมายเหตุ
- เป็น installer เดียวในกลุ่มนี้ที่ใช้ **bundled installer** เป็น fallback (ไฟล์ `wireguard-installer.exe` รวมมากับโปรเจกต์)
- ใช้ NSIS installer สำหรับ fallback (flag `/S` สำหรับ silent)
- หาก bundled installer ไม่อยู่ในตำแหน่งที่คาดไว้ จะแสดง path ที่ค้นหาในคำเตือน

## สรุปวิธีการติดตั้ง

| Installer | winget ID | Fallback Method | Silent Args |
|-----------|-----------|-----------------|-------------|
| Git | `Git.Git` | Direct download (EXE) | `/VERYSILENT /NORESTART` |
| Windows Terminal | `Microsoft.WindowsTerminal` | ไม่มี (ต้องใช้ winget) | — |
| PowerShell 7 | `Microsoft.PowerShell` | Direct download (MSI) | MSI default |
| Notepad++ | `Notepad++.Notepad++` | ไม่มี (ต้องใช้ winget) | — |
| Postman | `Postman.Postman` | ไม่มี (ต้องใช้ winget) | — |
| RustDesk | `RustDesk.RustDesk` | ไม่มี (แสดง URL เท่านั้น) | — |
| WireGuard | `WireGuard.WireGuard` | Bundled installer (EXE) | `/S` |

## Usage in Application

Installers เหล่านี้ถูกลงทะเบียนใน ToolRegistry ภายใต้หมวด `DevelopmentCategory.CrossPlatform` และเข้าถึงได้ผ่าน:
- เมนูหมวด "Cross-Platform Tools"
- การเลือกติดตั้งรายเครื่องมือ
- ไม่มี dependency chain — ทุกตัวติดตั้งอิสระ

## Error Handling and Verification

Installer ในกลุ่มนี้มีรูปแบบการจัดการข้อผิดพลาดที่สอดคล้องกัน:
- ตรวจสอบเครื่องมือก่อนติดตั้ง (IsInstalled) เพื่อหลีกเลี่ยงการติดตั้งซ้ำ
- ลองติดตั้งผ่าน winget ก่อน แล้ว fallback ไปวิธีอื่นหากไม่สำเร็จ (Git, PowerShell 7, WireGuard)
- จัดการ exception ด้วย try-catch ครอบทั้ง method
- รายงานสถานะและ progress ผ่าน `IProgressReporter` อย่างละเอียด
- Cleanup ไฟล์ชั่วคราวหลังติดตั้ง (Git, PowerShell 7)