# Browser Installers

เอกสารนี้อธิบายการทำงานของตัวติดตั้งในหมวด Browser ของแอปพลิเคชัน DevToolInstaller ครอบคลุมการติดตั้ง browser ผ่าน winget และการตั้งค่า privacy policies ผ่าน Windows Registry

## Overview

ชุดเครื่องมือ Browser ประกอบด้วย 5 installer:

1. **ChromeInstaller** - ติดตั้ง Google Chrome
2. **FirefoxInstaller** - ติดตั้ง Mozilla Firefox
3. **BraveInstaller** - ติดตั้ง Brave Browser
4. **OperaInstaller** - ติดตั้ง Opera Browser
5. **BrowserSettingsInstaller** - ตั้งค่า privacy policies สำหรับ Chromium-based browsers ผ่าน Registry

## Architecture

### WingetBrowserInstallerBase (Base Class)

Browser installer ทั้ง 4 ตัว (Chrome, Firefox, Brave, Opera) สืบทอดจาก abstract class [`WingetBrowserInstallerBase`](../../Installers/BrowserInstallers.cs:3) ซึ่ง implement [`IInstaller`](../../IInstaller.cs) interface

- **Category**: CrossPlatform
- **Dependencies**: ไม่มี (ทุก browser ติดตั้งได้อิสระ)
- **Installation Method**: winget (silent/non-interactive)

### การตรวจสอบสถานะ (IsInstalledAsync)

Base class ตรวจสอบ 2 วิธี:
1. **PATH lookup** — ค้นหา executable ใน system PATH ผ่าน `ProcessHelper.FindExecutableInPathAsync()`
2. **Common install paths** — ตรวจสอบ path ที่ browser มักถูกติดตั้ง (Program Files, LocalAppData ฯลฯ)

### กระบวนการติดตั้ง (InstallAsync)

1. ตรวจสอบว่า `winget` พร้อมใช้งาน
2. รันคำสั่ง: `winget install --id={PackageId} -e --source=winget --accept-source-agreements --accept-package-agreements --force`
3. รายงานผลสำเร็จหรือล้มเหลว
4. หาก winget ไม่พบ จะแจ้งเตือนให้ติดตั้งด้วยตนเอง

## ChromeInstaller

### Details
- **Name**: Google Chrome
- **Description**: Fast, secure web browser from Google
- **Package ID**: `Google.Chrome`
- **Executable Names**: `chrome.exe`
- **Common Install Paths**:
  - `%ProgramFiles%\Google\Chrome\Application\chrome.exe`
  - `%ProgramFiles(x86)%\Google\Chrome\Application\chrome.exe`

## FirefoxInstaller

### Details
- **Name**: Mozilla Firefox
- **Description**: Privacy-focused open source web browser
- **Package ID**: `Mozilla.Firefox`
- **Executable Names**: `firefox.exe`
- **Common Install Paths**:
  - `%ProgramFiles%\Mozilla Firefox\firefox.exe`
  - `%ProgramFiles(x86)%\Mozilla Firefox\firefox.exe`

## BraveInstaller

### Details
- **Name**: Brave Browser
- **Description**: Privacy-focused Chromium browser with built-in ad blocking
- **Package ID**: `Brave.Brave`
- **Executable Names**: `brave.exe`
- **Common Install Paths**:
  - `%ProgramFiles%\BraveSoftware\Brave-Browser\Application\brave.exe`
  - `%ProgramFiles(x86)%\BraveSoftware\Brave-Browser\Application\brave.exe`
  - `%LocalAppData%\BraveSoftware\Brave-Browser\Application\brave.exe`

## OperaInstaller

### Details
- **Name**: Opera Browser
- **Description**: Feature-rich web browser with built-in VPN and productivity tools
- **Package ID**: `Opera.Opera`
- **Executable Names**: `opera.exe`, `launcher.exe`
- **Common Install Paths**:
  - `%LocalAppData%\Programs\Opera\opera.exe`
  - `%LocalAppData%\Programs\Opera\launcher.exe`
  - `%ProgramFiles%\Opera\opera.exe`
  - `%ProgramFiles(x86)%\Opera\opera.exe`

## BrowserSettingsInstaller

### Purpose

ตั้งค่า privacy และ performance policies สำหรับ Chromium-based browsers ผ่าน Windows Registry (HKCU) รวมถึงลบ startup entries ของ browser ออกจาก Windows Run registry

### Details
- **Name**: Browser Privacy Settings
- **Category**: CrossPlatform
- **Dependencies**: ไม่มี
- **AlwaysRun**: `true` — จะถูกรันทุกครั้ง (IsInstalledAsync คืนค่า `false` เสมอ เพราะ registry policies เป็น idempotent)

### Browsers ที่ได้รับการตั้งค่า

| Browser | Registry Policy Path |
|---------|---------------------|
| Google Chrome | `HKCU\SOFTWARE\Policies\Google\Chrome` |
| Microsoft Edge | `HKCU\SOFTWARE\Policies\Microsoft\Edge` |
| Brave | `HKCU\SOFTWARE\Policies\BraveSoftware\Brave` |
| Opera | `HKCU\SOFTWARE\Policies\Opera Software\Opera` |

> หมายเหตุ: ใช้ HKCU (Current User) จึงไม่ต้องการสิทธิ์ Administrator

### Registry Policies ที่ตั้งค่า

| Policy Key | Description | Value | ผลลัพธ์ |
|-----------|-------------|-------|---------|
| `PromptForDownloadLocation` | Ask where to save each download | `1` (DWORD) | Enabled |
| `BackgroundModeEnabled` | Disable background mode | `0` (DWORD) | Disabled |
| `MetricsReportingEnabled` | Disable usage analytics & crash reporting | `0` (DWORD) | Disabled |
| `StartupBoostEnabled` | Disable startup boost (pre-launch on login) | `0` (DWORD) | Disabled |
| `AutofillAddressEnabled` | Disable address autofill | `0` (DWORD) | Disabled |
| `AutofillCreditCardEnabled` | Disable credit card autofill | `0` (DWORD) | Disabled |
| `PasswordManagerEnabled` | Disable built-in password manager | `0` (DWORD) | Disabled |
| `HardwareAccelerationModeEnabled` | Keep hardware acceleration enabled | `1` (DWORD) | Enabled |

> อ้างอิง: [Chrome Enterprise Policies](https://chromeenterprise.google/policies/)

### การลบ Browser Startup Entries

หลังจากตั้งค่า policies แล้ว installer จะลบ auto-launch entries ออกจาก registry key:

**Path**: `HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`

**Startup value prefixes ที่ถูกลบ**:
- `GoogleChromeAutoLaunch*`
- `MicrosoftEdgeAutoLaunch*`
- `BraveSoftware*`
- `Opera Browser Assistant*`

### Installation Process

1. วนลูปตั้งค่า registry policies ให้แต่ละ browser (8 policies ต่อ browser)
2. ลบ browser startup entries จาก Windows Run registry
3. รายงานสรุปจำนวน browser ที่ตั้งค่าสำเร็จ/ล้มเหลว
4. แจ้งเตือนให้ restart browsers เพื่อให้ settings มีผล

## Error Handling and Verification

- Browser installers ตรวจสอบทั้ง PATH และ common install paths ก่อนติดตั้งซ้ำ
- BrowserSettingsInstaller ทำงานแบบ idempotent — การรันซ้ำปลอดภัยและจะเขียนทับค่าเดิม
- จัดการ error แต่ละ browser แยกกัน — browser หนึ่งล้มเหลวไม่กระทบ browser อื่น
- รายงาน browser ที่ล้มเหลวแยกต่างหาก