# Microsoft Edge Browser Remover & Uninstaller (2026 Edge Debloater)

Remove **Microsoft Edge**, **Edge Update**, and optionally **Edge WebView2 Runtime** from Windows 10 and Windows 11 using a single PowerShell script.

This tool combines the native Microsoft uninstaller with the **Integrated Services Region Policy** unlock used in the EU (Digital Markets Act), so the uninstall path works **outside Europe** as well — not just a brute-force folder delete.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Windows](https://img.shields.io/badge/Windows-10%20%7C%2011-0078D6?logo=windows&logoColor=white)](https://www.microsoft.com/windows)
[![PowerShell](https://img.shields.io/badge/PowerShell-5.1%2B-5391FE?logo=powershell&logoColor=white)](https://learn.microsoft.com/powershell/)

---

## Will this work on every Windows PC?

**Honestly: no.** No third-party Edge remover can guarantee 100% success on every machine. This script is designed for **Windows 10 and Windows 11 (x64)** on standard desktop/laptop installs.

| Scenario | Expected result |
|----------|-----------------|
| Windows 11 / Windows 10 (64-bit), local admin | Works in most cases |
| Already patched `IntegratedServicesRegionPolicySet.json` | Works; skips duplicate patch |
| Ghost entry in Settings after files deleted | Cleaned automatically |
| Windows Update later | May restore Edge or reset the policy JSON — re-run script |
| **Windows S Mode** | **Not supported** (blocks unsigned scripts) |
| **ARM64 Windows** | May work; less tested |
| **Enterprise / MDM-managed** | Group Policy may override or reinstall Edge |
| Apps using **WebView2** (Teams, some installers) | Keep WebView2 unless you pass `-RemoveWebView2` |

If uninstall returns exit code **532**, **reboot** and run the script again, or use **Settings → Apps → Microsoft Edge → Uninstall** (button should no longer be grayed out after the policy patch).

Method reference: [CyberCPU Tech — Uninstall Microsoft Edge For Good](https://www.youtube.com/watch?v=nE5avDD2rnY)

---

## What it removes

- Microsoft Edge (Chromium) browser
- Edge Update services (`edgeupdate`, `edgeupdatem`)
- Edge scheduled tasks
- Edge / EdgeCore / EdgeUpdate install folders
- Edge AppX and provisioned packages
- User `AppData` Edge profile folders (optional skip)
- Orphan uninstall registry entries (fixes “setup.exe not found” in Settings)
- **Optional:** Edge WebView2 Runtime (`-RemoveWebView2`)

## What it configures

- Enables non-EU uninstall policies in `C:\Windows\System32\IntegratedServicesRegionPolicySet.json`
- Sets `DoNotUpdateToEdgeWithChromium` and related keys to reduce automatic reinstall

---

## Quick start (one line)

**Review the script before running any remote code.**

```powershell
irm https://raw.githubusercontent.com/eunjunseo47/edge-remover/main/edge-remover.ps1 | iex
```

Or clone and run locally (recommended):

```powershell
git clone https://github.com/eunjunseo47/edge-remover.git
cd edge-remover
powershell -ExecutionPolicy Bypass -File .\edge-remover.ps1
```

Accept the **UAC (Administrator)** prompt when it appears.

---

## Usage

```powershell
# Remove Edge browser + Edge Update (keeps WebView2 for app compatibility)
.\edge-remover.ps1

# Also remove WebView2 Runtime
.\edge-remover.ps1 -RemoveWebView2

# No confirmation prompts
.\edge-remover.ps1 -Force

# Skip region policy edit (e.g. already in EU or already patched)
.\edge-remover.ps1 -SkipRegionPolicy

# Keep user AppData folders
.\edge-remover.ps1 -SkipAppDataCleanup
```

### Parameters

| Parameter | Description |
|-----------|-------------|
| `-RemoveWebView2` | Also uninstall Edge WebView2 Runtime |
| `-SkipRegionPolicy` | Do not edit `IntegratedServicesRegionPolicySet.json` |
| `-SkipReinstallBlock` | Do not write anti-reinstall registry keys |
| `-SkipAppDataCleanup` | Do not delete `%LocalAppData%\Microsoft\Edge*` |
| `-Force` | Skip interactive confirmation |
| `-NoRebootPrompt` | Do not offer reboot at the end |

---

## How it works

1. **Self-elevates** to Administrator (UAC)
2. **Stops** Edge processes, services, and scheduled tasks
3. **Patches** `IntegratedServicesRegionPolicySet.json` (enables “Edge is uninstallable” outside EU)
4. **Runs** native `setup.exe --uninstall --system-level --force-uninstall`
5. **Removes** AppX packages, leftover folders, and user data
6. **Blocks** automatic Edge reinstall via registry (best-effort)
7. **Cleans** orphan “Installed apps” entries when `setup.exe` is already gone

---

## Reinstall Microsoft Edge later

If you need Edge back:

1. Download the installer from Microsoft:  
   https://www.microsoft.com/edge
2. Or use winget:
   ```powershell
   winget install Microsoft.Edge
   ```
3. To restore WebView2 only (without the full browser):
   ```powershell
   winget install Microsoft.EdgeWebView2Runtime
   ```

To restore the original region policy file, use the backup created at:

```
%TEMP%\IntegratedServicesRegionPolicySet.json.YYYYMMDD-HHMMSS.bak
```

Copy it back to `C:\Windows\System32\IntegratedServicesRegionPolicySet.json` (requires Administrator + ownership change).

---

## Disclaimer

- Use **at your own risk**. Removing system components can affect Windows features, widgets, or apps that depend on Edge/WebView2.
- This project is **not affiliated with Microsoft**.
- **Read the script** before executing, especially with `irm | iex`.
- Windows Update may **reinstall Edge** or **restore the policy JSON**; re-run this tool if needed.
- Required Windows telemetry cannot be fully disabled by removing Edge alone.

---

## Repository layout

```
edge-remover/
├── edge-remover.ps1   # Main remover script
├── README.md
├── LICENSE
└── .gitignore
```

---

## Topics

`microsoft-edge` · `edge-remover` · `uninstall-edge` · `windows-debloat` · `remove-edge` · `windows-11` · `windows-10` · `powershell`

---

## License

MIT — see [LICENSE](LICENSE).
