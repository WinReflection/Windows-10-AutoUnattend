# Windows 10 AutoUnattend
This repository is for the managment of an AutoUnattend.xml answer file for the automated installation of Windows 10 via a USB drive. The answer file is created in Windows System Image Manager (WSIM) which is included in the Windows 10 ADK files. Although the repository says "Windows 10", answer files for others flavors of Windows 10 such Enterprise and Enterprise LTSC will be added in the future.

---
## Process Creation and Installation:
1. Official Windows 10 source files.
2. Windows 10 ADK and Microsoft Deploeyment Toolkit (MDT) installed on a supported version of Windows 11.
3. Generate catalog file (.clg) with Windows Image System Manager (WSIM) from source files.
4. Create answer file in WSIM from available settings in the generated catalog file (.clg).
5. Save answer file as AutoUnattend.xml.
6. Place AutoUnattend.xml on root of USB drive.
7. Have USB drive insterted into computer during install of Windows so Setup.exe can find it.
8. Windows installs automatically.

## Troubleshooting:
- If step #7 fails it's possible Windows source files in use does not contain USB controller or chipset drivers for your computer so the USB drive will not be seen. Certain USB controller hubs might not be detected, some USB ports may be detected where others are not. Additonaly the Terms of Service Agreement Windows may present itself as the installer boots before the USB drive is detected, in many cases you just have to click "OK" and automated install will continue.
## Goals
- Microsoft Deployment Toolkit (MDT) is Microsoft's official method for deploying Windows 10 to computers. Therefore, I want to make sure the Answer Files in this repository do not deviate too much or at all from Microsoft's defaults if possible. 
## Building Notes
- Answer Files where first generated in MDT and then edited with WSIM to be compatible for USB deployment (AutoUnattend.xml), keeping Microsoft's default values when images are imported into MDT. 
- Selecting in MDT: %DeploymentShare% -> Task Sequences - > Select Task Sequences -> Properties (Right-click menu) -> Edit Unattend.xml (Button) will start generating a catalog file (.clg) for the source image linked to the task sequence. Located: %deploymentshare%\Operating Systems\Windows 10 22H2 RTM x64\sources\install_Windows 10 Pro.clg.
- The default Unattend.xml file is located at: %DeploymentShare%\Control\%TaskSequenceID%\Unattend.xml.
## Files
- install_Windows 10 Pro.clg - Catlog file generted by WSIM for Windows 10 Pro.
- Unattend.xml - Default answer file generated by MDT for Windows 10 Pro.
- AutoUnattend.xml - Answer file for USB installtion, as close to the Microsoft's defaults as possible.
## Unattend.xml to AutoUnattend.xml Modications
- When deploying an image with MDT everything is permored under LiteTouchPE which accesses the MDT Deployment Share which contains many scripts. When installing Windows via AutoUnattend.xml from a USB drive we don't have access to these scripts which create some limitations when trying to imitate the default configuration.
#### Disk Configuration: Recovery Partition
 - By default the Windows partition is set to use 99% of the drive after the System partitons are created and the Recovery partition is set to use the last remianing 1% of the drive space. WSIM does not offer a way to do this or even a shrink a partiton. You would need to use manual values which won't work since you can't predict the size of every disk in a computer for imaging. To get around this, 3 more RunSyncronousCommands were added to the Specialize phase.
```
 - powershell.exe -noninteractive -command "echo 'select volume c' 'shrink minimum=300' 'create partition primary' 'format quick fs=ntfs label=Recovery' 'assign letter=R' | diskpart.exe"
 - powershell.exe -noninteractive -command "echo 'select volume r' 'set id=de94bba4-06d1-4d40-a16a-bfd50179d6ac' 'gpt attributes=0x8000000000000001' 'remove letter=R' | diskpart.exe"
 - reg delete "HKLM\SYSTEM\MountedDevices" /v "\DosDevices\R:" /f
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
