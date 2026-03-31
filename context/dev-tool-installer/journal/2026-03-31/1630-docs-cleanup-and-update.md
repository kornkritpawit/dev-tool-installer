# Docs Cleanup and Update

**วันที่:** 2026-03-31  
**เวลา:** 16:30 (UTC+7)

## สรุป

ทำความสะอาดโค้ดที่ไม่ได้ใช้งาน แก้ไขเอกสารที่ไม่ตรงกับโค้ด และเขียนเอกสารใหม่ให้ครอบคลุมทุก installer ในระบบ

## สิ่งที่ทำ

### 1. ลบ Orphan Code Files 6 ไฟล์

ไฟล์ installer ที่ไม่ได้ลงทะเบียนใน `ToolRegistry` ถูกลบออก:

- **ไม่เคยลงทะเบียน:**
  - `DBeaverInstaller.cs`
  - `FlowiseInstaller.cs`
  - `PostgreSQLInstaller.cs`

- **เวอร์ชันเก่าที่ถูกแทนที่แล้ว:**
  - `DotNetSdkInstaller.cs` (แทนที่ด้วย `DotNetSdk10Installer.cs`)
  - `NodeJsInstaller.cs` (แทนที่ด้วย `NvmWindowsInstaller.cs` + `NodeJs20Installer.cs`)
  - `NodeJs22Installer.cs` (แทนที่ด้วย `NvmWindowsInstaller.cs` + version management)

### 2. แก้ไขเอกสารที่ไม่ตรงกับโค้ด

- **`font-installer.md`** — แก้ให้ระบุว่า CascadiaMono Nerd Font ถูก download ที่ runtime (ไม่ได้ bundle มากับโปรแกรม)
- **`nodejs-installers.md`** — เพิ่ม pnpm ใน `NodeJsToolsInstaller` section ให้ตรงกับโค้ดจริง
- **`context/docs/INDEX.md`** — เพิ่ม link ไปยัง `tui-interface-design.md` ที่หายไป

### 3. เขียนเอกสาร Markdown ใหม่ 6 ไฟล์

เอกสารใหม่ครอบคลุมทุก installer ที่เหลือซึ่งยังไม่มี documentation:

| ไฟล์ | Installers ที่ครอบคลุม |
|------|----------------------|
| `python-installers.md` | Python, Pip, Poetry, Uv, VisualCppBuildTools (5) |
| `dotnet-installer.md` | DotNetSdk10 (1) |
| `vscode-installer.md` | VSCode + 31 extensions + 43 settings (1) |
| `browser-installers.md` | Chrome, Firefox, Brave, Opera, BrowserSettings (5) |
| `docker-installer.md` | DockerDesktop (1) |
| `system-settings-installers.md` | WindowsExplorerSettings, WslConfig (2) |
| `cross-platform-tools.md` | Git, WindowsTerminal, PowerShell7, Notepad++, Postman, RustDesk, WireGuard (7) |

## ผลลัพธ์

- **Docs coverage:** เพิ่มจาก 21.4% (6/28 installers) เป็น **100% (28/28 installers)**
- **Orphan code:** ลบ 6 ไฟล์ที่ไม่ได้ใช้งานออกจาก codebase
- **Docs accuracy:** แก้ไข 3 จุดที่เอกสารไม่ตรงกับโค้ดจริง

## ไฟล์ที่เปลี่ยนแปลง

### ลบ
- `Installers/DBeaverInstaller.cs`
- `Installers/FlowiseInstaller.cs`
- `Installers/PostgreSQLInstaller.cs`
- `Installers/DotNetSdkInstaller.cs`
- `Installers/NodeJsInstaller.cs`
- `Installers/NodeJs22Installer.cs`

### แก้ไข
- `context/docs/font-installer.md`
- `context/docs/nodejs-installers.md`
- `context/docs/INDEX.md`

### สร้างใหม่
- `context/docs/python-installers.md`
- `context/docs/dotnet-installer.md`
- `context/docs/vscode-installer.md`
- `context/docs/browser-installers.md`
- `context/docs/docker-installer.md`
- `context/docs/system-settings-installers.md`
- `context/docs/cross-platform-tools.md`