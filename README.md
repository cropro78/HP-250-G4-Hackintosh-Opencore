# HP 250 G4 (Broadwell) Opencore Hackintosh Guide

<img src="/screenshots/macinfo1.png"/>
<img src="/screenshots/macinfo2.png"/>

Specs:

- CPU: Intel Core i3 5005U
- GPU: Intel HD 5500
- dGPU: AMD Radeon R5 M330 (Disabled via SSDT)
- Network: RTL810xE Ethernet controller
- Audio: Realtek ALC282
- RAM: 12GB DDR3
- HDD: 240GB PNY SSD
- Opencore version: 0.5.9

What's working:

- Sleep (And also by closing the lid, works on AC, and on battery I didn't test, since I am
    waiting for an replacement)
- Brightness control
- CPU Power Management
- Metal
- SD Card reader
- HDMI Audio, Video
- Built in Camera
- Sound, Microphone (With AppleALC which works better than VoodooHDA)
- VGA

Problems:

- Built in Bluetooth/Wifi card (Replaced it by TP-Link USB nano wifi adapter)
- Nothing much else...

List of Patches and kexts

- RehabMan's Clover config for HD5500 (Converted to Opencore settings)
- RehabMan's IRQ patch (For fixing AppleALC kext)
- Rehabman's Poll for LID changes patch + ACPIPoller kext (for reparing the LID)
- SSDT-dGPU-OFF (For disableing the Radeon dGPU which doesn't work in MacOS, and
    causes sleep issues if you don't disable it)
- SSDT-EC (For fixing embedded controller, build it with SSDTTime on Windows)
- SSDT-HPET (For patching IRQ conflicts, build it with SSDTTime on Windows)
- SSDT-PLUG (For fixing Power Management, build it with SSDTTime on Windows)
- SSDT-PNLF (For repairing backlight, for use with WhateverGreen)
- AppleALC (Best alcid layout id for ALC282 on this laptop is: 3)
- Lilu
- RealtekRTL
- SMCBatteryManager
- SMCProcessor
- SMCSuperIO
- USBInjectALL and USBMap kexts for fixing usb ports which also repairs wakeup issue
- VirtualSMC (instead FakeSMC, because this is recommended in Opencore, and I
    haven't seen any issues with that)
- RehabMan's version of VoodooPS2Controller (acidanthera's version works, but in
    the latest version 2.1.5 mouse buttons are not registering to anything, although for
    tracking this version is better)
- WhateverGreen

Tools/Guides used:

- For most SSDT patches(SSDT-EC, SSDT-HPET, SSDT-PLUG, SSDT-PNLF, USBmap, SSDT-
    dGPU-OFF) and also Opencore config for Broadwell: https://dortania.github.io
    - If you get a kernel panic before the first installation be sure to enable in OpenCore plist under Kernel, Quirks: EnableKernelPanic
- DSDT patches were installed through RehabMan's version of MaciASL:
    https://bitbucket.org/RehabMan/os-x-maciasl-patchmatic/downloads/
- DSDT was extracted through iasl on Windows: https://github.com/RehabMan/Intel-
    iasl
- Opencore: https://github.com/acidanthera/OpenCorePkg/releases
- AppleALC: https://github.com/acidanthera/AppleALC
- Lilu: https://github.com/acidanthera/lilu/releases
- RealtekRTL8100: https://www.osx86.net/files/file/4830-realtekrtl8100kext/
- VirtualSMC (Includes: SMCBatteryManager, SMCProcessor, SMCSuperIO):
    https://github.com/acidanthera/virtualsmc/releases
- USBInjectALL guide: https://dortania.github.io/USB-Map-Guide/
- VoodooPS2Controller: https://bitbucket.org/RehabMan/os-x-voodoo-ps2-
    controller/downloads/
- ACPIPoller: https://bitbucket.org/RehabMan/os-x-acpi-poller/downloads/
- Whatevergreen: https://github.com/acidanthera/WhateverGreen/releases
- MountEFI: https://github.com/corpnewt/MountEFI
- ProperTree: https://github.com/corpnewt/ProperTree

Tips for fixing sleep by LID, and for patching the GPU with Opencore

For fixing the lid you will have to need:

- 1. Extracted Laptop DSDT
- 2. RehabMan's version of MaciASL
1. Open with maciASL your extracted DSDT, and then click the Patch
    icon
2. Search for [misc] Poll for LID changes, and then apply the patch
3. When you apply the patch you will have a Device (LIDP) on the
    bottom on your DSDT:

<img src="/screenshots/patch2.png"/>

```
Please note that this configuration will not work as it is, since this path in the
Method (FCPU...) SB.LID0 doesn't probaby exist in your DSDT, and then, you will have
to find the real path of your LID. The easiest way to find the path of the LID is on
Windows Device Manager under System devices/ACPI Lid, Property BIOS device
name:
```
<img src="/screenshots/WindowsPath.png" width="345" height="393"/>


```
In my case the ACPI path for LID0 is _SB.PCI0.LPCB.LID0, so you will have to change
all _SB.LID0 to _SB.PCI0.LPCB.LID (Or whatever it shows as BIOS device name)
```
<img src="/screenshots/patch1.png"/>

```
It should look like this
```
<img src="/screenshots/patch0.png"/>

4. Install the ACPIPoller kext under EFI/OC/Kexts, and compile and save the patched
    DSDT to EFI/OC/ACPI, and then with ProperTree, and then go to File/OC Snapshot,
    and then save the Config.plist you have created earlier. Reboot, and you should have
    a working LID.

For Patching the Intel GPU HD5500 With HDMI audio under Opencore:
For this you will be needing ProperTree

Under device properties, under PciRoot(0x0)/Pci(0x2,0x0) (If it doesn't exist, just create a
new key with the same type) you will need to insert the next properties (Picture down below)

Also for HDMI audio, it is necessarry to create also PciRoot(0x0)/Pci(0x03,0x0) and also apply the framebuffer patches for HDMI to get the same working. You will have to use those settings which are in the same screenshot, and also
framebuffer patches. (Those settings were pulled of RehabMan's clover config)

<img src="/screenshots/gpu.png"/>


