# HP 250 G4 (Broadwell) Opencore Hackintosh Guide

Specs:

- CPU: Intel Core i3 5005U
- GPU: Intel HD
- dGPU: AMD Radeon R5 M330 (Disabled via SSDT)
- Network: RTL810xE Ethernet controller
- Audio: Realtek ALC
- RAM: 12GB DDR
- HDD: 240GB PNY SSD
- Opencore version: 0.5.9

What's working:

- Sleep (And also by closing the lid, works on AC, and on battery didn't test, since I am
    waiting for an replacement)
- Brightness control
- CPU Power Management
- Metal


- SD Card reader
- HDMI Audio
- Built in Camera, Microphone
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
- AppleALC (Best alcid layout id for ALC282 is: 3)
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
    dGPU-OFF) and also Opencore config: https://dortania.github.io
- DSDT patches were installed through RehabMan's version of MaciASL:
    https://bitbucket.org/RehabMan/os-x-maciasl-patchmatic/downloads/
- DSDT was extracted through iasl on Windows: https://github.com/RehabMan/Intel-
    iasl
- AppleALC: https://github.com/acidanthera/AppleALC
- Lilu: https://github.com/acidanthera/lilu/releases
- RealtekRTL8100: https://www.osx86.net/files/file/4830-realtekrtl8100kext/


- VirtualSMC (Includes: SMCBatteryManager, SMCProcessor, SMCSuperIO):
    https://github.com/acidanthera/virtualsmc/releases
- USBInjectALL guide: https://dortania.github.io/USB-Map-Guide/
- VoodooPS2Controller: https://bitbucket.org/RehabMan/os-x-voodoo-ps2-
    controller/downloads/
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

```
Please note that this configuration will not work as it is, since this path in the
Method (FCPU...) SB.LID0 doesn't probaby exist in your DSDT, and then, you will have
to find the real path of your LID. The easiest way to find the path of the LID is on
Windows Device Manager under System devices/ACPI Lid, Property BIOS device
name:
```

```
In my case the ACPI path for LID0 is _SB.PCI0.LPCB.LID0, so you will have to change
all _SB.LID0 to _SB.PCI0.LPCB.LID
```
```
It should look like this
```
4. Install the ACPIPoller kext under EFI/OC/Kexts, and compile and save the patched
    DSDT to EFI/OC/ACPI, and then with ProperTree, and then go to File/OC Snapshot,
    and then save the Config.plist you have created earlier. Reboot, and you should have
    a working LID.

```
Change this one
```
```
And this one
```

For Patching the Intel GPU HD5500 With HDMI audio under Opencore:
For this you will be needing ProperTree

Under device properties, under PciRoot(0x0)/Pci(0x2,0x0) (If it doesn't exist, just create a
new key with the same type) you will insert the next properties:

Also for HDMI audio, it is necessarry to create also PciRoot(0x0)/Pci(0x03,0x0) to get audio
working, you will have to use those settings which are in the same screenshot, and also
framebuffer patches.


