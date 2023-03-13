# Windows 10 AutoUnattend
This repository is for the management of AutoUnattend.xml answer files for the automated installation of Windows 10 via a USB removable flash drive. The answer file is created in Windows System Image Manager (WSIM) which is included in the Windows 10 ADK files. Although the repository says "Windows 10", answer files for other flavors of Windows 10 such as Enterprise and Enterprise LTSC will be added in the near future.

## Goal
- That an Answer File in this repository does not deviate too much or at all from Microsoft's default values if possible. The Microsoft Deployment Toolkit (MDT) is Microsoft's free official method for deploying Windows 10 to computers which can also be used in conjuction with WDS. Therefore, converting for standalone USB installation should be as close as possible to the official method.
 
## Creation and Use
1. Have official Windows 10 source media files.
2. Windows 10 ADK and Microsoft Deployment Toolkit (MDT) installed on a supported version of Windows 11.
3. Generate catalog file (.clg) with Windows Image System Manager (WSIM) from source files.
4. Create answer file in WSIM from available settings in the generated catalog file (.clg).
5. Save answer file desired and rename as AutoUnattend.xml.
6. Place AutoUnattend.xml on root of 'removable' USB drive.
7. Have USB drive insterted into computer during install of Windows so Setup.exe can find it.
8. Setup.exe finds AutoUnattend.xml and installs Windows automatically with no user intervention required.

## Building Notes
- Answer Files are first generated in MDT and then edited with WSIM to be compatible for USB deployment (AutoUnattend.xml), keeping Microsoft's default values when images are imported into MDT. 
- Selecting in MDT: %DeploymentShare% -> Task Sequences - > Select Task Sequences -> Properties (Right-click menu) -> Edit Unattend.xml (Button) will start generating a catalog file (.clg) for the source image linked to the task sequence. Located: %DeploymentShare%\Operating Systems\Windows 10 22H2 RTM x64\sources\install_Windows 10 Pro.clg.
- The default Unattend.xml file is located at: %DeploymentShare%\Control\%TaskSequenceID%\Unattend.xml.

## Files
- install_Windows 10 Pro.clg - Catalog file generted by Windows System Image Manager (WSIM) for Windows 10 Pro.
- Unattend_mdt-default-windows-10-pro-22h2-rtm.xml - Default answer file generated by MDT for Windows 10 Pro.
- AutoUnattend-mdt-defaults-windows-10-pro-22h2-rtm.xml - Answer file for USB installtion, as close to Microsoft's defaults as possible.

## Troubleshooting
- If Setup.exe does not find AutoUnattend.xml it's possible Windows source files in use does not contain USB controller or chipset drivers for your computer so the USB drive will not be seen. Certain USB controller hubs might not be detected or have detection delayed, some USB ports may be detected where others are not. Additonaly the EULA window may present itself as the installer boots before the USB drive and Answer Files is detected, in many cases you just have to click "OK" and the automated install will still continue.

## Building Answer Files Yourself
- Download and install Windows 11 Enterprise Evaluation 64-bit, [here.](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise)
- Download and install Windows ADK for Windows 11, version 22H2, [here.](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install)
- Download and install Windows PE add-on for the Windows ADK for Windows 11, version 22H2, [here.](https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install)
- Download and install Microsoft Deployment Toolkit, [here.](https://www.microsoft.com/en-us/download/details.aspx?id=54259)
#### Patches:
###### FAILURE ( 5616 ): 15250: Verify BCDBootEx
- Download and install MDT_KB4564442, [here.](https://support.microsoft.com/en-us/topic/windows-10-deployments-fail-with-microsoft-deployment-toolkit-on-computers-with-bios-type-firmware-70557b0b-6be3-81d2-556f-b313e29e2cb7)
###### Could not find a part of the path 'C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Windows Preinstallation Environment\x86\WinPE_OCs'.
 - Run the following command in CMD:
 ```
 md "C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Windows Preinstallation Environment\x86\WinPE_OCs"
 ```
 
## Unattend.xml to AutoUnattend.xml Modications
- When deploying an image with MDT everything is performed under LiteTouchPE which accesses the MDT deployment share which contains many scripts. When installing Windows via AutoUnattend.xml from a USB 'removable' flash drive we don't have access to these scripts which creates some limitations when trying to clone the default configuration.

#### Empty/Blank Values Removed
 - Emtpy or blank values have been removed as they causes issues, they're usually filled out mnaually during the LiteTouchPE dpeloymwnt wizard, for USB deployment they can be removed.

#### Administrator AutoLogin Password
- The password for the Built-in Local Administrator account used for AutoLogin is "Password01!".

#### Error & Depreciated Values Resolved
 - The default Unattend.xml answer file had validation errors in WSIM, these have been fixed.
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
#### Disk Configuration: Recovery Partition
 - By default the Windows partition is set to use 99% of the drive after the System partitons are created and the Recovery partition is set to use the last remianing 1% of the drive space. WSIM does not offer a way to do this or even shrink a partiton. You would need to use manual values which won't work since you can't predict the size of every disk in a computer for imaging. To get around this, 3 more RunSyncronousCommands were added to the Specialize phase.
```
 powershell.exe -noninteractive -command "echo 'select volume c' 'shrink minimum=300' 'create partition primary' 'format quick fs=ntfs label=Recovery' 'assign letter=R' | diskpart.exe"
 powershell.exe -noninteractive -command "echo 'select volume r' 'set id=de94bba4-06d1-4d40-a16a-bfd50179d6ac' 'gpt attributes=0x8000000000000001' 'remove letter=R' | diskpart.exe"
 reg delete "HKLM\SYSTEM\MountedDevices" /v "\DosDevices\R:" /f
 ```
###### Recovery Tools Partition Information 
- [UEFI/GPT-based hard drive partitions](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/configure-uefigpt-based-hard-drive-partitions)
- [BIOS/MBR-based hard drive partitions](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/configure-biosmbr-based-hard-drive-partitions)
- [Windows Recovery Environment (Windows RE)](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-recovery-environment--windows-re--technical-reference)
> Recovery tools partition
> This partition must be at least 300 MB.
>
> The Windows Recovery Environment (Windows RE) tools require additional free space:
>
> A minimum of 52 MB is required but 250 MB is recommended, to accomodate future updates, especially with custom partition layouts.
>
> When calculating free space, note:
>
> The recovery image, winre.wim, is typically between 250-300MB, depending on what drivers, 
> languages, and customizations you add.
> The file system itself can take up additional space. 
> For example, NTFS may reserve 5-15MB or more on a 750MB partition.
> This partition must use the Type ID: DE94BBA4-06D1-4D40-A16A-BFD50179D6AC.
>
> The recovery tools should be in a separate partition than the Windows partition to support automatic failover and to support booting partitions encrypted with Windows BitLocker Drive Encryption.
>
> Create a separate recovery partition to support automatic failover and to support booting Windows BitLocker Drive Encryption-encrypted partitions.
>
> We recommend that you place this partition in a separate partition, immediately after the Windows partition. 
> This allows Windows to modify and recreate the partition later if future updates require a larger recovery image.
>
>The Windows Recovery Environment (Windows RE) tools require additional free space:
>
> A minimum of 52 MB is required but 250 MB is recommended, to accomodate future updates, especially with custom partition layouts.
>
> When calculating free space, note:
>
> The recovery image, winre.wim, is typically between 250-300MB, depending on what drivers, languages, and customizations you add.
> The file system itself can take up additional space. 
> For example, NTFS may reserve 5-15MB or more on a 750MB partition.
>
> The Windows RE update process makes every effort to reuse the existing Windows RE partition without any modification. 
> However, in some rare situations where the new Windows RE image (along with the migrated/injected contents) does not fit in the existing Windows RE partition, the > update process will behave as follows:
>
> If the existing Windows RE partition is located immediately after the Windows partition, the Windows partition will be shrunk and space will be added to the Windows > RE partition. The new Windows RE image will be installed onto the expanded Windows RE partition.
>
> If the existing Windows RE partition is not located immediately after the Windows partition, the Windows partition will be shrunk and a new Windows RE partition will > be created. 
> The new Windows RE image will be installed onto this new Windows RE partition. 
>
> The existing Windows RE partition will be orphaned.
> If the existing Windows RE partition cannot be reused and the Windows partition cannot successfully be shrunk, the new Windows RE image will be installed onto the
> Windows partition. 
> The existing Windows RE partition will be orphaned.
