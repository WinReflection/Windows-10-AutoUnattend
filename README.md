# Windows 10 AutoUnattend
This repository is for the managment of an AutoUnattend.xml answer file for the automated installation of Windows 10 via a USB drive. The answer file is created in Windows System Image Manager (WSIM) which is included in the Windows 10 ADK files. Although the repository says "Windows 10", it will also include answer files for others flavors of Windows 10 such Enterprise and Enterprise LTSC.

---
## Process Creation and Installation:
1. Proper Windows 10 source files ready.
2. Windows 10 ADK installed on supported version of Windows 10.
3. Generate catalog file (.clg) with Windows Image System Manager (WSIM) from source files.
4. Create answer file in WSIM from available settings in the generated catalog file.
5. Save answer file as AutoUnattend.xml.
6. Place AutoUnattend.xml on root of USB drive.
7. Have USB drive insterted into computer during install of Windows so Setup.exe can find it.
8. Windows installs automatically.

## Troubleshooting:
- If step #7 fails it's possible Windows source files in use does not contain USB controller or chipset drivers for your computer so the USB drive will not be seen. Certain USB controller hubs might not be detected, some USB ports may be detected where others are not. Additonaly the Terms of Service Agreement Windows may present itself as the installer boots before the USB drive is detected, in many cases you just have to click "OK" and automated install will continue.
