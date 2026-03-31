# Python Development Tool Installers

เอกสารนี้อธิบายการทำงานของตัวติดตั้งในหมวด Python ของแอปพลิเคชัน DevToolInstaller

## Overview

ชุดเครื่องมือ Python ปัจจุบันประกอบด้วย 5 installer:

1. **PythonInstaller** - ติดตั้ง Python 3.12.5 interpreter
2. **PipInstaller** - ตรวจสอบและอัปเดต pip package manager
3. **PoetryInstaller** - ติดตั้ง Poetry dependency management tool
4. **UvInstaller** - ติดตั้ง uv (Rust-based Python package installer)
5. **VisualCppBuildToolsInstaller** - ติดตั้ง Visual C++ Build Tools สำหรับคอมไพล์ Python packages

## Dependency Chain

**Python → Pip → Poetry**

รายละเอียด dependency:
- **PythonInstaller**: ไม่มี dependency
- **PipInstaller**: ขึ้นกับ Python
- **PoetryInstaller**: ขึ้นกับ Python และ Pip
- **UvInstaller**: ไม่มี dependency (ติดตั้งแยกอิสระ)
- **VisualCppBuildToolsInstaller**: ไม่มี dependency (ติดตั้งแยกอิสระ)

## PythonInstaller

### Purpose
ติดตั้ง **Python 3.12.5** (64-bit) สำหรับ Windows

### Details
- **Package Source**: ดาวน์โหลดโดยตรงจาก python.org
- **Download URL**: `https://www.python.org/ftp/python/3.12.5/python-3.12.5-amd64.exe`
- **Target Version**: `3.12.5`
- **Installation Method**: ดาวน์โหลด installer แล้วรันแบบ quiet
- **Category**: Python
- **Dependencies**: None

### Installation Process
1. ดาวน์โหลดไฟล์ `python-3.12.5-amd64.exe` จาก python.org ไปยัง temp directory
2. รัน installer ด้วย arguments: `/quiet InstallAllUsers=1 PrependPath=1 Include_test=0`
   - `InstallAllUsers=1` — ติดตั้งให้ทุก user บนเครื่อง
   - `PrependPath=1` — เพิ่ม Python ลง PATH อัตโนมัติ
   - `Include_test=0` — ไม่ติดตั้ง test suite (ประหยัดเนื้อที่)
3. ลบไฟล์ installer หลังติดตั้งเสร็จ

### IsInstalled Logic
- ตรวจสอบ `python.exe` ผ่าน `FindExecutableInPathAsync` (ค้นใน PATH)
- ตรวจสอบผ่าน `ProcessHelper.IsToolInstalled("python")` (ค้นจาก registry/where command)

## PipInstaller

### Purpose
ตรวจสอบและอัปเดต **pip** (Python Package Manager) ให้พร้อมใช้งาน

### Details
- **Installation Method**: ใช้ Python module `ensurepip` และ `pip install --upgrade pip`
- **Category**: Python
- **Dependencies**: Python

### Installation Process
1. รัน `python -m ensurepip --default-pip` เพื่อให้แน่ใจว่า pip ถูกติดตั้ง
2. หาก ensurepip ล้มเหลว จะแสดง warning แล้วลองวิธีอื่น
3. รัน `python -m pip install --upgrade pip` เพื่ออัปเกรด pip เป็นเวอร์ชันล่าสุด

### IsInstalled Logic
- **Primary check**: รัน `python -m pip --version` แล้วตรวจสอบว่ามี output หรือไม่
- **Secondary check**: ค้นหา `pip.exe` ใน PATH ผ่าน `FindExecutableInPathAsync`

## PoetryInstaller

### Purpose
ติดตั้ง **Poetry** — เครื่องมือจัดการ dependency และ packaging สำหรับ Python

### Details
- **Installation Method**: ติดตั้งผ่าน `pip install poetry`
- **Category**: Python
- **Dependencies**: Python, Pip
- **Special Features**:
  - ค้นหา poetry.exe ที่มีอยู่แล้วบนเครื่องก่อนติดตั้งใหม่
  - จัดการ User PATH อัตโนมัติ (เพิ่ม Scripts directory)
  - Broadcast `WM_SETTINGCHANGE` ให้โปรเซสอื่นรับรู้การเปลี่ยนแปลง PATH

### Installation Process
1. **ตรวจสอบ poetry.exe ที่มีอยู่**: ค้นหาตามตำแหน่งต่างๆ บนเครื่อง
   - หาก poetry.exe มีอยู่แต่ไม่อยู่ใน PATH → เพิ่ม Scripts directory เข้า User PATH แล้วจบ
2. **ติดตั้งใหม่** (ถ้าไม่พบ poetry.exe เลย):
   - อัปเดต pip ก่อน: `python -m pip install --upgrade pip`
   - ติดตั้ง Poetry: `pip install poetry`
   - ค้นหาตำแหน่ง poetry.exe ที่ถูกติดตั้ง
   - เพิ่ม Scripts directory เข้า User PATH
3. Refresh environment variables

### IsInstalled Logic
- ค้นหา `poetry.exe` ผ่าน `FindExecutableInPathAsync`
- ใช้ `where poetry` ผ่าน cmd เป็น fallback (กรณี PATH ที่ process ปัจจุบันมองไม่เห็น)
- หาก poetry.exe มีอยู่บนดิสก์แต่ไม่อยู่ใน PATH → return false (เพื่อให้ reinstall เพิ่ม PATH)

### Poetry Executable Search Locations
Installer จะค้นหา `poetry.exe` ตามลำดับ:
1. ผ่าน `pip show poetry` → หา Location แล้วตรวจ Scripts sibling directory
2. `%LocalAppData%\Programs\Python\Python*\Scripts\`
3. `%ProgramFiles%\Python\Python*\Scripts\`
4. `C:\Python\Python*\Scripts\`
5. `%AppData%\pypoetry\venv\Scripts\` (ตำแหน่ง official Poetry installer)
6. `%AppData%\Python\Scripts\`
7. `%AppData%\Python\Python*\Scripts\`
8. `%LocalAppData%\Python\Python*\Scripts\`

### PATH Management
- เพิ่ม directory เข้า **User PATH** ผ่าน registry (`EnvironmentVariableTarget.User`)
- อัปเดต **Process PATH** ด้วยเพื่อให้ session ปัจจุบันใช้งานได้ทันที
- Broadcast `WM_SETTINGCHANGE` ผ่าน `SendMessageTimeout` ให้โปรเซสอื่นรับรู้

## UvInstaller

### Purpose
ติดตั้ง **uv** — Python package installer ที่เขียนด้วย Rust ซึ่งเร็วกว่า pip/virtualenv อย่างมาก

### Details
- **Installation Method**: winget (primary) / PowerShell script (fallback)
- **winget Package ID**: `astral-sh.uv`
- **Category**: Python
- **Dependencies**: None (ติดตั้งแยกอิสระ ไม่ต้องมี Python ก่อน)

### Installation Process
1. **ลองผ่าน winget ก่อน**:
   - ตรวจสอบว่า winget พร้อมใช้งาน
   - รัน `winget install --id=astral-sh.uv -e --source=winget --accept-source-agreements --accept-package-agreements --force`
2. **Fallback ผ่าน official PowerShell installer**:
   - รัน `powershell -ExecutionPolicy Bypass -Command "irm https://astral.sh/uv/install.ps1 | iex"`

### IsInstalled Logic
- ค้นหา `uv.exe` ผ่าน `FindExecutableInPathAsync`
- ตรวจสอบผ่าน `ProcessHelper.IsToolInstalled("uv")`

## VisualCppBuildToolsInstaller

### Purpose
ติดตั้ง **Microsoft Visual C++ Build Tools** ซึ่งจำเป็นสำหรับคอมไพล์ Python packages ที่เขียนด้วย C/C++ (เช่น numpy, pandas, cryptography)

### Details
- **Package Source**: ดาวน์โหลดโดยตรงจาก Microsoft
- **Download URL**: `https://aka.ms/vs/17/release/vs_buildtools.exe`
- **Installation Method**: ดาวน์โหลด Visual Studio Build Tools installer แล้วรันแบบ quiet
- **Category**: Python
- **Dependencies**: None
- **Workload**: `Microsoft.VisualStudio.Workload.VCTools` พร้อม recommended components

### Installation Process
1. ดาวน์โหลด `vs_buildtools.exe` จาก Microsoft ไปยัง temp directory
2. รัน installer ด้วย arguments: `--quiet --wait --add Microsoft.VisualStudio.Workload.VCTools --includeRecommended`
   - `--quiet` — ไม่แสดง UI
   - `--wait` — รอจนติดตั้งเสร็จ
   - `--add Microsoft.VisualStudio.Workload.VCTools` — ติดตั้ง C++ build tools workload
   - `--includeRecommended` — รวม recommended components ทั้งหมด
3. ลบไฟล์ installer หลังติดตั้งเสร็จ

### IsInstalled Logic
ตรวจสอบการมีอยู่ของไฟล์ `vcvarsall.bat` ในตำแหน่งมาตรฐาน:
- `%ProgramFiles%\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build\vcvarsall.bat`
- `%ProgramFiles(x86)%\Microsoft Visual Studio\2022\BuildTools\VC\Auxiliary\Build\vcvarsall.bat`
- `%ProgramFiles%\Microsoft Visual Studio\2019\BuildTools\VC\Auxiliary\Build\vcvarsall.bat`
- `%ProgramFiles(x86)%\Microsoft Visual Studio\2019\BuildTools\VC\Auxiliary\Build\vcvarsall.bat`

> รองรับทั้ง Visual Studio 2019 และ 2022

## Usage in Application

Installers เหล่านี้ถูกลงทะเบียนใน ToolRegistry และเข้าถึงได้ผ่าน:
- เมนูหมวด "Python Development"
- การเลือกติดตั้งรายเครื่องมือ
- กลไก dependency-aware installation ตาม chain:
  - Python ก่อน
  - ตามด้วย Pip
  - แล้วจึง Poetry
  - uv และ Visual C++ Build Tools ติดตั้งแยกอิสระ

## Error Handling and Verification

Installer ในกลุ่มนี้มีการจัดการข้อผิดพลาดและการตรวจสอบผลลัพธ์:
- ตรวจสอบเครื่องมือก่อนติดตั้งซ้ำ
- จัดการไฟล์ installer ชั่วคราว (ดาวน์โหลดแล้วลบ)
- Poetry มี fallback logic สำหรับค้นหา executable ในหลายตำแหน่ง
- uv มี fallback จาก winget ไปยัง PowerShell installer
- Pip มี fallback จาก `ensurepip` ไปยัง `pip install --upgrade pip`
- รองรับการรายงานสถานะแบบละเอียดเพื่อการดีบัก