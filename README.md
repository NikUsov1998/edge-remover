# Microsoft Edge Browser Remover & Uninstaller (2026 Edge Debloater)

Remove **Microsoft Edge**, **Edge Update**, and optionally **Edge WebView2 Runtime** from Windows 10 and Windows 11 using a single PowerShell script.

This tool combines the native Microsoft uninstaller with the **Integrated Services Region Policy** unlock used in the EU (Digital Markets Act), so the uninstall path works **outside Europe** as well — not just a brute-force folder delete.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Windows](https://img.shields.io/badge/Windows-10%20%7C%2011-0078D6?logo=windows&logoColor=white)](https://www.microsoft.com/windows)
[![PowerShell](https://img.shields.io/badge/PowerShell-5.1%2B-5391FE?logo=powershell&logoColor=white)](https://learn.microsoft.com/powershell/)

---

## Important: this does NOT work on every Windows machine

**No third-party Edge remover can guarantee success on every PC.** This project targets a specific, well-supported environment:

| Supported target | Notes |
|------------------|-------|
| **Windows 10 / 11 x64** | Primary supported platform |
| **Local Administrator** | Script self-elevates via UAC |
| **Non-S Mode** | S Mode blocks unsigned/local scripts |

| Not supported or unreliable | Notes |
|-----------------------------|-------|
| **Windows S Mode** | Script execution is blocked |
| **Enterprise / MDM-managed PCs** | Group Policy or Intune may override registry changes and reinstall Edge |
| **ARM64 Windows** | May work; not fully validated |
| **Windows Server** | Edge may not be preinstalled; behavior varies |

**Windows Update may restore Microsoft Edge** or reset `IntegratedServicesRegionPolicySet.json` after a major upgrade. Re-run this script if Edge returns.

Method reference: [CyberCPU Tech — Uninstall Microsoft Edge For Good](https://www.youtube.com/watch?v=nE5avDD2rnY)

---

## Recommended: clone and run locally

Review the script on disk before executing anything with administrator privileges.

```powershell
git clone https://github.com/eunjunseo47/edge-remover.git
cd edge-remover
powershell -ExecutionPolicy Bypass -File .\edge-remover.ps1
```

Accept the **UAC (Administrator)** prompt when it appears.

### Convenience one-liner (review first)

Only use this if you have read the script on GitHub and trust the source:

```powershell
irm https://raw.githubusercontent.com/eunjunseo47/edge-remover/main/edge-remover.ps1 | iex
```

Running remote code with `irm | iex` carries inherent risk. Prefer cloning the repository when possible.

---

## Usage

```powershell
# Remove Edge browser + Edge Update (keeps WebView2 by default)
.\edge-remover.ps1

# Also remove WebView2 Runtime — RISKY: may break dependent apps
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
| `-RemoveWebView2` | **Optional and risky.** Also uninstall Edge WebView2 Runtime. Many apps embed WebView2. |
| `-SkipRegionPolicy` | Do not edit `IntegratedServicesRegionPolicySet.json` |
| `-SkipReinstallBlock` | Do not write anti-reinstall registry keys |
| `-SkipAppDataCleanup` | Do not delete `%LocalAppData%\Microsoft\Edge*` |
| `-Force` | Skip interactive confirmation |
| `-NoRebootPrompt` | Do not offer reboot at the end |

### WebView2 removal warning

By default, **WebView2 is kept** for application compatibility. Only pass `-RemoveWebView2` if you understand that installers, login flows, and desktop apps may stop working until WebView2 is reinstalled.

---

## What it removes

- Microsoft Edge (Chromium) browser
- Edge Update services (`edgeupdate`, `edgeupdatem`)
- Edge scheduled tasks
- Edge / EdgeCore / EdgeUpdate install folders
- Edge AppX and provisioned packages
- User `AppData` Edge profile folders (unless `-SkipAppDataCleanup`)
- Orphan uninstall registry entries (fixes “setup.exe not found” in Settings)
- **Optional:** Edge WebView2 Runtime (`-RemoveWebView2`)

## What it configures

- Enables non-EU uninstall policies in `C:\Windows\System32\IntegratedServicesRegionPolicySet.json`
- Sets `DoNotUpdateToEdgeWithChromium` and EdgeUpdate policy keys using the correct product GUIDs:
  - Edge Stable: `{56EB18F8-B008-4CBD-B6D2-8C97FE7E9062}`
  - WebView2 (only when `-RemoveWebView2`): `{F3017226-FE2A-4295-8BDF-00C3A9A7E4C5}`

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

## Tested on

| Environment | Result |
|-------------|--------|
| Windows 11 24H2 / 25H2 (x64), US region | Edge + WebView2 removed; orphan Settings entry cleaned |
| Windows 10 22H2 (x64) | Expected to work; community reports welcome via Issues |
| Windows 11 ARM64 | Not validated in CI — use at your own risk |
| Windows S Mode | **Not supported** |

CI validates PowerShell syntax on `windows-latest` for every push and pull request.

---

## Known limitations

- Does **not** disable all Windows telemetry — only removes Edge-related components.
- **Enterprise / MDM** devices may ignore or revert local policy and registry changes.
- **Windows Update** can reinstall Edge or restore the region policy JSON file.
- **WebView2 removal** is optional because many applications depend on it.
- Exit code **532** from `setup.exe` often means a **reboot is required** before uninstall completes.
- The script makes **no network calls** during execution; only you choose whether to download it from GitHub.

---

## Restore / reinstall

### Reinstall Microsoft Edge

1. Download from Microsoft: https://www.microsoft.com/edge
2. Or use winget:

```powershell
winget install Microsoft.Edge
```

### Reinstall WebView2 only

```powershell
winget install Microsoft.EdgeWebView2Runtime
```

### Restore the region policy file

The script saves a backup before editing:

```text
%TEMP%\IntegratedServicesRegionPolicySet.json.YYYYMMDD-HHMMSS.bak
```

Copy it back to `C:\Windows\System32\IntegratedServicesRegionPolicySet.json` (Administrator + ownership change required).

---

## Security note

- **No data collection.** The script does not phone home, log telemetry, or upload files.
- **No network activity** during script execution.
- The only network interaction is if **you** choose to download the script (for example via `git clone` or `irm` from GitHub).
- Read `edge-remover.ps1` before running it, especially with `irm | iex`.

---

## Troubleshooting

### Exit code 532 from setup.exe

Reboot the PC, then either:

- Run `.\edge-remover.ps1` again, or
- Use **Settings → Apps → Installed apps → Microsoft Edge → Uninstall**

After the region policy patch, the Uninstall button should no longer be grayed out outside the EU.

### Ghost entry in Settings (“setup.exe not found”)

The files are already gone but Windows still lists Edge. Re-run the script — it removes orphan uninstall registry keys automatically. A reboot may also clear stale UI entries.

### Access denied / TrustedInstaller

Run PowerShell **as Administrator** and accept the UAC prompt. The script takes ownership of `IntegratedServicesRegionPolicySet.json` before editing. If access still fails, reboot into an admin account and retry.

### Edge returned after Windows Update

Windows Update may restore Edge or reset the policy JSON. Re-run:

```powershell
powershell -ExecutionPolicy Bypass -File .\edge-remover.ps1
```

Keep a copy of your patched JSON backup if Start menu web-search behavior changes after an update.

---

## Disclaimer

- Use **at your own risk**. Removing system components can affect Windows features, widgets, or apps that depend on Edge/WebView2.
- This project is **not affiliated with Microsoft**.
- **Read the script** before executing, especially with `irm | iex`.
- Required Windows diagnostic data cannot be fully disabled by removing Edge alone.

---

## Repository layout

```text
edge-remover/
├── .github/workflows/powershell.yml
├── edge-remover.ps1
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
