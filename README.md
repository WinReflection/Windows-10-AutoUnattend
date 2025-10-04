# Windows 10 AutoUnattend

[![Latest Release](https://img.shields.io/github/v/release/WinReflection/Windows-10-AutoUnattend?label=release&logo=github)](https://github.com/WinReflection/Windows-10-AutoUnattend/releases)
[![Repo Stars](https://img.shields.io/github/stars/WinReflection/Windows-10-AutoUnattend?style=social)](https://github.com/WinReflection/Windows-10-AutoUnattend/stargazers)
[![License](https://img.shields.io/badge/License-MIT-informational.svg)](#license)

Answer files (`AutoUnattend.xml`) for hands-off Windows installations from a **USB removable drive**—kept as close to Microsoft defaults as possible for predictable, supportable deployments. Built from MDT defaults, finalized with [Windows System Image Manager (WSIM)](https://learn.microsoft.com/en-us/windows-hardware/customize/desktop/wsim/windows-system-image-manager-technical-reference).

---

## TL;DR (USB Method)

1) Get **official Windows 10 source media** (matching edition/arch).  
2) Pick the provided `AutoUnattend-*.xml` you want and rename it to **`AutoUnattend.xml`**.  
3) Copy **`AutoUnattend.xml` to the root** of your **USB *removable*** drive.  
4) Boot target PC with the USB connected. Setup finds the answer file and runs hands-off.

> Heads-up: the built-in **Administrator** autologon uses a **temporary** password (`Password01!`) so post-install steps can run. Change/disable this in your task sequence or GPO immediately after first logon.

---

## Why This Repository?

- **Stay close to defaults.** Start with MDT-generated `Unattend.xml`, then adapt only what’s needed for a USB-only flow—reducing surprises when Microsoft changes things.

- **Document the deltas.** Every deviation from MDT/WSIM defaults is called out and justified.

- **Be reproducible.** You can rebuild the exact same files yourself with ADK + MDT.

> Releases: see **Releases** → right sidebar for tagged versions and change notes.

---

## Compatibility & Requirements

| Area | Supported / Required |
|---|---|
| Windows Images | Windows 10 (Pro/Enterprise/LTSC), 64-bit |
| Source Media | Official Microsoft ISO / Media |
| Build Host | Windows 11 (for newest ADK), or Windows 10 with matching ADK |
| Tools | **Windows ADK** + **WinPE Add-on** + **MDT** + **WSIM** |
| Deployment | **USB *removable*** device with `AutoUnattend.xml` at root |

---

## Repository Layout

```
/ (root)
├─ README.md
├─ AutoUnattend-mdt-defaults-windows-10-pro-22h2-rtm.xml     # USB-ready, minimal deviations
├─ Unattend_mdt-default-windows-10-pro-22h2-rtm.xml          # Raw MDT default for reference
└─ install_Windows 10 Pro.clg                                # WSIM catalog for the image
```

**Which one do I use?**
- **USB install:** rename `AutoUnattend-*.xml` → `AutoUnattend.xml`, place at USB root.
- **MDT/WDS flow:** use `Unattend_mdt-default-*.xml` inside your Task Sequence.

---

## Step-by-Step: Build It Yourself

1. Install tools (newest stable works great for Windows 10 images):
   - Windows 11 **ADK** + **WinPE Add-on**
   - **Microsoft Deployment Toolkit (MDT)**
   
2. In MDT, import your Windows 10 media; create a Task Sequence.

3. From the TS: **Right-click → Properties → Edit Unattend.xml** (this triggers WSIM to generate a **`.clg`** catalog, stored under your `Operating Systems\...\sources` path).

4. Save the generated `Unattend.xml` and open in **WSIM** to review/adjust.

5. For USB installs, adapt as shown below (or use this repo’s `AutoUnattend-*.xml` and tweak as desired).

- The MDT generated Unattend.xml lives at:
```
%DeploymentShare%\Control\<TaskSequenceID>\Unattend.xml
```
- WSIM catalogs live at:
```
%DeploymentShare%\Operating Systems\Windows 10 22H2 RTM x64\sources\install_Windows 10 Pro.clg
```

---

## What’s Changed vs. MDT Defaults?

<details>
<summary><strong>1) Empty/Blank Values Removed</strong></summary>

- Values left empty in MDT (often filled by LiteTouch wizard) can cause WSIM validation errors for USB usage. Remove those entries entirely so Setup doesn’t choke on invalid types.
</details>

<details>
<summary><strong>2) Admin AutoLogon (temporary)</strong></summary>

- Enable AutolLogon for the built-in **Administrator** with password **`Password01!`** to run post-install steps without touch. **Change/Disable** immediately via GPO or script after first sign-in.
</details>

<details>
<summary><strong>3) WSIM Validation Fixes & Deprecated Settings</strong></summary>

- Resolve Invalid Display Elements (`ColorDepth`, `HorizontalResolution`, `RefreshRate`, `VerticalResolution`) and remove deprecated **NetworkLocation**. This keeps WSIM clean and prevents runtime surprises.

Here are the exact blocks:
```
- The 'ColorDepth' element is invalid - The value '' is invalid according to its datatype 'ColorDepthType' - The string '' is not a valid UInt32 value.
(Components/oobeSystem/amd64_Microsoft-Windows-Shell-Setup_neutral/Display/ColorDepth)

- The 'HorizontalResolution' element is invalid - The value '' is invalid according to its datatype 'HorizontalResolutionType' - The string '' is not a valid UInt32 
value.	
(Components/oobeSystem/amd64_Microsoft-Windows-Shell-Setup_neutral/Display/HorizontalResolution)	

- The 'RefreshRate' element is invalid - The value '' is invalid according to its datatype 'RefreshRateType' - The string '' is not a valid UInt32 value.
(Components/oobeSystem/amd64_Microsoft-Windows-Shell-Setup_neutral/Display/RefreshRate)

- The 'VerticalResolution' element is invalid - The value '' is invalid according to its datatype 'VerticalResolutionType' - The string '' is not a valid UInt32 value.
(Components/oobeSystem/amd64_Microsoft-Windows-Shell-Setup_neutral/Display/VerticalResolution)

- Setting NetworkLocation is deprecated in the Windows image
(Components/oobeSystem/amd64_Microsoft-Windows-Shell-Setup_neutral/OOBE/NetworkLocation)
```
</details>

<details>
<summary><strong>4) Recovery Partition Layout (fully automated)</strong></summary>

- By default, Windows wants the OS partition to take ~99% and leave a **separate Recovery Tools** partition at the end. WSIM can’t “shrink after install,” so add `specialize` phase **RunSynchronous** commands to:
  1. Disable WinRE → shrink C: by ~768 MB → create & format Recovery (R:).
  2. Set the proper GPT type/id and attributes.
  3. Remove the temp drive letter and re-enable WinRE.

Here are the commands that complete this:
```
powershell.exe -noninteractive -command "reagentc /disable"
powershell.exe -noninteractive -command "echo 'sel volume c' 'shrink minimum=768' 'create partition primary' 'format quick fs=ntfs label=Recovery' 'assign letter=R' | diskpart.exe"
powershell.exe -noninteractive -command "echo 'sel volume r' 'set id=de94bba4-06d1-4d40-a16a-bfd50179d6ac' 'gpt attributes=0x8000000000000001' 'remove letter=R' | diskpart.exe"
powershell.exe -noninteractive -command "reagentc /enable"
reg delete "HKLM\SYSTEM\MountedDevices" /v "\DosDevices\R:" /f
```

Microsoft guidance recommends a separate Recovery partition after the OS, with minimum sizes noted for WinRE.
</details>

---

## Quick Use (USB)

1. Format a **USB removable** device as FAT32/NTFS (UEFI systems read FAT32 best for bootable media).
2. Copy Windows 10 setup files to the USB (or use the official Media Creation Tool).
3. Copy/rename the answer file:

   ```text
   AutoUnattend-mdt-defaults-windows-10-pro-22h2-rtm.xml  →  AutoUnattend.xml
   ```
4. Place `AutoUnattend.xml` at the *root* of the USB.
5. Boot the target PC from the USB (don’t unplug during PE / early OOBE). Setup should proceed automatically.

---

## Troubleshooting

### Setup Didn’t Use My `AutoUnattend.xml`
**Likely Causes:**
- The USB is not seen as a **removable** device early in setup.
- Storage/USB controller drivers missing in the image.
- The file isn’t at the root or is mis-named.

**Potential Fixes:**
- Try different USB port (rear I/O vs. hub), or a different model stick.
- Slipstream storage/USB drivers or use newer media.
- Confirm exact name: `AutoUnattend.xml` at root.

### - I Saw The EULA Once, Then Install Continued
- Early OOBE can appear before the USB is fully enumerated, then Setup finds the file and continues unattended—this can be normal. Click **OK** and let it continue.

### - MDT Errors When Generating Catalogs / Building LiteTouch PE

<details>
<summary><strong>FAILURE (5616): 15250 Verify BCDBootEx</strong></summary>

Install the relevant MDT hotfix (e.g., MDT_KB4564442).
</details>

<details>
<summary><strong>WinPE_OCs path not found</strong></summary>

Create the missing directory:
```cmd
md "C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Windows Preinstallation Environment\x86\WinPE_OCs"
```
</details>

---

## Security Notes

- The **temporary** `Password01!` is only to enable first-boot automation. Rotate/disable immediately (local policy, GPO, or post-install script).
- If you enable domain join or add packages, re-validate in WSIM to catch type/namespace changes across ADK versions.

---

## References

- [Microsoft Partitioning & WinRE Guidance (UEFI/GPT)](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/configure-uefigpt-based-hard-drive-partitions)
- [Microsoft Partitioning & WinRE Guidance (BIOS/MBR)](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/configure-biosmbr-based-hard-drive-partitions)
- [Windows Recovery Environment (Windows RE)](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-recovery-environment--windows-re--technical-reference)
- [Microsoft Deployment Toolkit Known Issues](https://learn.microsoft.com/en-us/intune/configmgr/mdt/known-issues)
- [Download and install Windows 11 Enterprise Evaluation 64-bit](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise)
- [Download and install Windows ADK for Windows 11, version 22H2](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install)
- [Download and install Windows PE add-on for the Windows ADK for Windows 11, version 22H2](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install)
- [Download and install Microsoft Deployment Toolkit](https://www.microsoft.com/en-us/download/details.aspx?id=54259)
- [Download and install MDT_KB4564442](https://support.microsoft.com/en-us/topic/windows-10-deployments-fail-with-microsoft-deployment-toolkit-on-computers-with-bios-type-firmware-70557b0b-6be3-81d2-556f-b313e29e2cb7)

- This repository’s **Releases** page for tagged versions and change logs.

---

## License

MIT — see `LICENSE` if present in the repo.

---

## Changelog

See [Releases](https://github.com/WinReflection/Windows-10-AutoUnattend/releases) for versioned notes.
