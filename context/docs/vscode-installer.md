# Visual Studio Code Installer

เอกสารนี้อธิบายการทำงานของตัวติดตั้ง Visual Studio Code ในแอปพลิเคชัน DevToolInstaller รวมถึง extensions และ settings ที่ configure อัตโนมัติ

## Overview

VSCodeInstaller ทำ 3 สิ่งหลัก:

1. **ติดตั้ง VS Code** — ดาวน์โหลดและติดตั้ง VS Code (ข้ามถ้าติดตั้งแล้ว)
2. **จัดการ Extensions** — ลบ extensions ที่ไม่ต้องการ + ติดตั้ง extensions ที่กำหนด
3. **Configure Settings** — Merge user settings ที่กำหนดไว้เข้า settings.json

## VSCodeInstaller

### Purpose
ติดตั้ง **Visual Studio Code** พร้อม extensions และ settings ที่เหมาะสมสำหรับการพัฒนา

### Details
- **Package Source**: ดาวน์โหลดโดยตรงจาก code.visualstudio.com
- **Download URL**: `https://code.visualstudio.com/sha/download?build=stable&os=win32-x64-user`
- **Installation Method**: ดาวน์โหลด user installer แล้วรันแบบ silent
- **Category**: CrossPlatform
- **Dependencies**: None

### Installation Process
1. ตรวจสอบว่า VS Code ติดตั้งแล้วหรือไม่
   - **ถ้าติดตั้งแล้ว**: ข้ามขั้นตอนดาวน์โหลด/ติดตั้ง ไปจัดการ extensions และ settings โดยตรง
   - **ถ้ายังไม่ติดตั้ง**:
     1. ดาวน์โหลด `VSCodeSetup.exe` ไปยัง temp directory
     2. รัน installer ด้วย arguments: `/VERYSILENT /NORESTART /MERGETASKS=!runcode`
        - `/VERYSILENT` — ไม่แสดง UI เลย
        - `/NORESTART` — ไม่รีสตาร์ทเครื่อง
        - `/MERGETASKS=!runcode` — ไม่เปิด VS Code หลังติดตั้ง
     3. ลบไฟล์ installer หลังติดตั้งเสร็จ
2. ติดตั้ง extensions (ทุกครั้ง ไม่ว่า VS Code จะติดตั้งใหม่หรือมีอยู่แล้ว)
3. Configure user settings (ทุกครั้ง)

### IsInstalled Logic
- ค้นหา `code.exe` ผ่าน `FindExecutableInPathAsync`
- ตรวจสอบผ่าน `ProcessHelper.IsToolInstalled("code")`

## Extensions ที่ถูกลบ (Uninstall)

Extensions ต่อไปนี้จะถูกลบออกอัตโนมัติ:

| Extension ID | รายละเอียด |
|---|---|
| `GitHub.copilot` | GitHub Copilot |
| `GitHub.copilot-chat` | GitHub Copilot Chat |

## Extensions ที่ติดตั้ง

### C# / .NET
| Extension ID | รายละเอียด |
|---|---|
| `modelharbor.modelharbor-agent` | ModelHarbor Agent |
| `ms-dotnettools.vscode-dotnet-runtime` | .NET Runtime |
| `formulahendry.dotnet` | .NET Core Tools |
| `ms-dotnettools.csharp` | C# language support |
| `ms-dotnettools.csdevkit` | C# Dev Kit |
| `ms-dotnettools.vscodeintellicode-csharp` | IntelliCode for C# |
| `alexcvzz.vscode-sqlite` | SQLite Viewer |
| `kreativ-software.csharpextensions` | C# Extensions (snippets, quick actions) |

### Python / Jupyter
| Extension ID | รายละเอียด |
|---|---|
| `ms-python.python` | Python language support |
| `ms-python.debugpy` | Python Debugger |
| `ms-python.vscode-pylance` | Pylance (Python language server) |
| `ms-toolsai.jupyter` | Jupyter Notebook support |
| `charliermarsh.ruff` | Ruff linter/formatter |

### React / Next.js / Frontend
| Extension ID | รายละเอียด |
|---|---|
| `dsznajder.es7-react-js-snippets` | ES7+ React/Redux/React-Native snippets |
| `bradlc.vscode-tailwindcss` | Tailwind CSS IntelliSense |
| `dbaeumer.vscode-eslint` | ESLint |
| `esbenp.prettier-vscode` | Prettier - Code formatter |
| `formulahendry.auto-rename-tag` | Auto Rename Tag |
| `christian-kohler.path-intellisense` | Path IntelliSense |
| `christian-kohler.npm-intellisense` | npm IntelliSense |
| `mikestead.dotenv` | DotENV syntax highlighting |

### Vue.js
| Extension ID | รายละเอียด |
|---|---|
| `Vue.volar` | Vue - Official (Volar) |

### Svelte
| Extension ID | รายละเอียด |
|---|---|
| `svelte.svelte-vscode` | Svelte for VS Code |

### General / Markdown / DevTools
| Extension ID | รายละเอียด |
|---|---|
| `PKief.material-icon-theme` | Material Icon Theme |
| `shd101wyy.markdown-preview-enhanced` | Markdown Preview Enhanced |
| `bierner.markdown-mermaid` | Markdown Mermaid diagrams |
| `ms-vscode-remote.remote-ssh` | Remote - SSH |
| `sitoi.ai-commit` | AI Commit message generator |
| `eamodio.gitlens` | GitLens — Git supercharged |
| `usernamehw.errorlens` | Error Lens (inline error display) |
| `ms-azuretools.vscode-docker` | Docker |

> รวมทั้งหมด **31 extensions** ที่ติดตั้ง (และ 2 extensions ที่ถูกลบ)

## User Settings ที่ Configure

Settings จะถูก **merge** เข้า `%APPDATA%\Code\User\settings.json` โดยไม่ลบ settings เดิมที่ผู้ใช้ตั้งไว้ — เฉพาะ keys ที่กำหนดเท่านั้นที่ถูกเพิ่มหรือเขียนทับ

### Appearance
| Setting | Value |
|---|---|
| `workbench.iconTheme` | `"material-icon-theme"` |

### Font
| Setting | Value |
|---|---|
| `editor.fontFamily` | `"'CaskaydiaMono Nerd Font', Consolas, 'Courier New', monospace"` |
| `editor.fontSize` | `14` |
| `editor.fontLigatures` | `true` |
| `editor.cursorSmoothCaretAnimation` | `"on"` |
| `editor.cursorBlinking` | `"smooth"` |

### Editor Behavior
| Setting | Value |
|---|---|
| `editor.formatOnSave` | `true` |
| `editor.formatOnPaste` | `true` |
| `editor.linkedEditing` | `true` |
| `editor.wordWrap` | `"on"` |
| `editor.stickyScroll.enabled` | `true` |
| `editor.guides.bracketPairs` | `true` |
| `editor.bracketPairColorization.enabled` | `true` |
| `editor.minimap.enabled` | `false` |
| `editor.renderWhitespace` | `"boundary"` |
| `editor.suggestSelection` | `"first"` |
| `editor.acceptSuggestionOnCommitCharacter` | `false` |
| `editor.inlineSuggest.enabled` | `true` |
| `editor.tabSize` | `2` |
| `editor.detectIndentation` | `true` |
| `editor.smoothScrolling` | `true` |

### File Handling
| Setting | Value |
|---|---|
| `files.autoSave` | `"afterDelay"` |
| `files.autoSaveDelay` | `1000` |
| `files.trimTrailingWhitespace` | `true` |
| `files.insertFinalNewline` | `true` |
| `files.trimFinalNewlines` | `true` |

### Explorer
| Setting | Value |
|---|---|
| `explorer.confirmDelete` | `false` |
| `explorer.confirmDragAndDrop` | `false` |
| `explorer.compactFolders` | `false` |

### Terminal
| Setting | Value |
|---|---|
| `terminal.integrated.fontFamily` | `"CaskaydiaMono Nerd Font"` |
| `terminal.integrated.fontSize` | `13` |
| `terminal.integrated.scrollback` | `10000` |
| `terminal.integrated.defaultProfile.windows` | `"PowerShell"` |
| `terminal.integrated.smoothScrolling` | `true` |

### Git
| Setting | Value |
|---|---|
| `git.autofetch` | `true` |
| `git.confirmSync` | `false` |
| `git.enableSmartCommit` | `true` |

### Workbench
| Setting | Value |
|---|---|
| `workbench.editor.enablePreview` | `false` |
| `workbench.startupEditor` | `"none"` |
| `workbench.list.smoothScrolling` | `true` |
| `workbench.tree.indent` | `16` |

### Breadcrumbs & Search
| Setting | Value |
|---|---|
| `breadcrumbs.enabled` | `true` |
| `search.smartCase` | `true` |

### Files Exclude
นอกจาก settings ข้างต้น installer ยังตั้งค่า `files.exclude` เพื่อ **แสดง .git directory** (ตั้งเป็น `false`):

```json
"files.exclude": {
    "**/.git": false
}
```

> หาก `files.exclude` มีอยู่แล้ว จะ merge เข้าไป; หากยังไม่มีจะสร้างใหม่

## Settings Merge Behavior

- อ่าน `settings.json` ที่มีอยู่ (รองรับ comments และ trailing commas)
- Merge เฉพาะ keys ที่กำหนด — **ไม่ลบ** settings เดิมของผู้ใช้
- เขียนกลับเป็น JSON แบบ indented
- หากไฟล์หรือ directory ยังไม่มี จะสร้างใหม่

## Usage in Application

Installer นี้ถูกลงทะเบียนใน ToolRegistry และเข้าถึงได้ผ่าน:
- เมนูหมวด "Cross-Platform Development"
- การเลือกติดตั้งรายเครื่องมือ
- ไม่มี dependency กับ installer อื่น — ติดตั้งได้อิสระ

## Error Handling and Verification

- ตรวจสอบ VS Code ก่อนดาวน์โหลดเพื่อข้ามการติดตั้งซ้ำ
- จัดการไฟล์ installer ชั่วคราว (ดาวน์โหลดแล้วลบ)
- Extension installation: หาก extension ใดติดตั้งไม่สำเร็จจะแสดง warning แล้วข้ามไป (ไม่หยุดทั้งหมด)
- Extension uninstall: หาก extension ที่จะลบไม่มีอยู่จะข้ามไปเงียบๆ
- Settings configuration: หากหา APPDATA ไม่ได้หรือเขียนไฟล์ไม่สำเร็จจะแสดง warning (ไม่ทำให้การติดตั้งล้มเหลว)
- มี delay 1 วินาทีระหว่างการติดตั้งแต่ละ extension เพื่อหลีกเลี่ยง rate limiting