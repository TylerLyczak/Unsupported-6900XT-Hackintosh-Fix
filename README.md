# Unsupported-6900XT-Hackintosh-Fix
---

This guide is made to get certain RX 6900XT cards working in Mac OS.
Currently, 6900XT cards of the XTX varaint work fine under Mac OS Big Sur and Monterey. However, the XTXH variant of the 6900XT, such as the PowerColor RX 6900XT Ultimate, are not natively supported currently (Mac OS Monterey 12.0.1 as of writting this guide).

I have posted my full EFI config with my RX 6900XT working in another [repository](https://github.com/TylerLyczak/Hackintosh-10850k-ASUS-Z490-XII-Hero-6900XT) if you would like to look over that too.


## Prerequisites
---

* Follow the [OpenCore dortania guide](https://dortania.github.io/OpenCore-Install-Guide/) to get the right config set-up for your system
* Get the latest [Whatevergreen](https://github.com/acidanthera/WhateverGreen) kext as support for device-id GPU spoofing was added in 1.5.2
* Install [gfxutil](https://github.com/acidanthera/gfxutil)
* Install [IORegistryExplorer](https://github.com/vulgo/IORegistryExplorer)
* Install [MaciASL](https://github.com/acidanthera/MaciASL)
* Install [ProperTree](https://github.com/corpnewt/ProperTree) or any program to edit plist file


## Getting Info

#### Gfxutil
---

* Run `gfxutil` (./gfxutil)
* Look for the entry that ends with GFX0@0
    * For example, my line is `03:00.0 1002:73bf /PCI0@0/PEG1@1/PEGP@0/BRG0@0/GFX0@0 = PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)`
* Write down the path to GFX0@0
    * Etc. `/PCI0@0/PEG1@1/PEGP@0/BRG0@0/GFX0@0`
* ![GFXUTIL Output](/assets/gfxutil_pic.png)


#### IORegestryExplorer
---

* Run `IORegistryExplorer` app
* On the top left, search for `GFX0@0`
* This will bring up a path to GFX0@0
    * If the gpu is not supported, you will see a different path than what gfxutil gave you
    * For example, my path was `PCI0@0/PEG1@1/PEP@0/pci-device@0/GFX@0`
* Write down this path as well


## Making SSDT-BRG0.aml
---

* Download the SSDT-BRG0.dsl from the [OpenCore repository](https://github.com/acidanthera/OpenCorePkg)
    * If downloaded a release of OpenCore, get the file from `(OpenCoreFolderPath)/Docs/AcpiSamples/Source`
    * If cloned the repo, get the file from `(OpenCoreRepoPath)/Docs/AcpiSamples/Source`
    * I included my own modified version of this file in this repo. Use it if you want
* Open this file with MaciASL
* On lines 12 and 14 (13 and 16 for my file), you will see `External` and `Scope`
    * The default path listed is `PCI0.PEG0.PEGP`
    * You need to change it to the path given by `gfxutil`
        * For example, my path above was `PCI0@0/PEG1@1/PEP@0/pci-device@0/GFX@0`, so I would change `PCI0.PEG0.PEGP` to `PCI0.PEG1.PEGP`
* On line 20 (22 for my file), there is `Device`.
    * If `gfxutil` path to `GFX0@0` says something different then `BRG0`, then change this as well
* After you are done with the changes, Save-As the file as `SSDT-BRG0.aml` with the file format as `ACPI Machine Language Binary`
* Add the file in you EFI/OC/ACPI folder


## Edit config.plist
---

* Open your config.plist file with a plist editor program
* Under `ACPI -> Add -> (next biggest number)`, add the SSDT-BRG0.aml file
    * ProperTree can add this automatically by using Cmd+R
    * ![ACPI Section](/assets/acpi_pic.png)
* Under `DeviceProperties -> Add`, make a new child with the name of the path of the gfx card found with `gfxutil` with it being a Dictionary type
    * It will be the second value found by `gfxutil`
        * Example, `PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)`
    * Make two children under this new dictionary, `device-id` and `model`
        * `device-id` will be Data type
        * `model` will be String type
    * Make the `device-id` have the value of `BF730000`
    * Make the `model` have the value `Radeon RX 6900 XT (XTXH)`
    * ![DeviceProperties Section](/assets/device_pic.png)
* Under `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> boot-args`, append the boot arg `agdpmod=pikera -radcodec`
    * `-radcodec`: Used for allowing officially unsupported AMD GPUs (spoofed) to use the Hardware Video Encoder
    * ![NVRAM Section](/assets/nvram_pic.png)
* Save the file


## Test config
---

* Restart your hackintosh and see the results
* Check metal scores with the gpu to see if its working

## Additional Options
---

* If using VDADecoder, it may not say you have video acceleration
    * To fix this, type in the terminal `defaults write com.apple.AppleGVA gvaForceAMDAVCDecode -boolean yes`


## Feedback
---

* If theres any problems with this guide, please open an issue and I'll get to it as soon as possible.