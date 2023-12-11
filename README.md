# OMEN-Transcend-16-ACPI-fix

Fix buggy BIOS for OMEN Transcend Laptop 16-u0017TX (81M76PA).  

OMEN 9 Transcend 16 BIOS version "F.11 Rev.A" comes with buggy ACPI config. Booting linux distros with kernel 6.0 and later versions always fail, this has been tested with Fedora 38/39 and Ubuntu 22.04/23.10/24.04. Appending kernel parameter "noapic" will partially fix booting problem, but more problems remain. Below is a list of symptoms caused by this problematic ACPI config.

1. System bootup blocks forever.
2. Touchpad is not working (with kernel parameter `noapic`).
3. Unable to wakeup from S3 state sleep (with kernel parameter `noapic`).
4. Built-in speakers are not working (with kernel parameter `noapic`).

After some digging, I managed to fix first 3 problems by overriding DSDT with a patched version. Decompiled source file can be found in folder "F.11 Rev.A". Modifications can be viewed with commit logs.

This patch has been tested on Fedora 39 with kernel 6.5/6.6/6.7 and Ubuntu 24.04 with kernel 6.5.

## HOWTO USE

Compiled DSDT file can be found here [F.11 Rev.A_patch/dsdt.aml](./F.11%20Rev.A_patch/dsdt.aml). Download it to your local drive.

Following procedure is for Fedora 39, use on Ubuntu is pretty much the same except for updating grub config.

### Test the patch

1. Copy patched DSDT to your boot partition.  
`sudo cp dsdt.aml /boot`

2. Reboot and select current boot entry, press '`e`' to edit boot parameters. Insert following line before the line starts with "linux".  
`acpi /dsdt.aml`

3. Remove "`noapic`" or "`acpi=off`" or other kernel parameters which disables ACPI and APIC. Boot and check if it works.

### Make it permanent

1. Edit `/etc/grub.d/40_custom` with your favourite text editor, add following line to tail.  
`acpi /dsdt.aml`

2. Update grub config.  
`sudo grub2-mkconfig -o /boot/grub2/grub.cfg`

3. Reboot.

## Problems remaining

1. Built-in speakers still have no sound.  
This patch has fixed speaker config in DSDT and kernel stops complaining about invalid speaker parameters, but speakers are still not working in linux while they work flawlessly in windows. I guess there's more magic hidden in windows drivers.

2. Boot order.  
I work in a dual boot environment in which Windows 10 is installed first. No matter how I change boot order in BIOS or with `efibootmgr` in linux, the machine always boots into Windows automatically. But if I hit 'F9' during startup, I can see "Fedora" is selected as primary target. This might be a bios bug and not related to ACPI parameters.

## Further readings

If anyone is instrested in ACPI debugging, you can find latest ACPI specs @ [https://uefi.org/sites/default/files/resources/ACPI_Spec_6_5_Aug29.pdf](https://uefi.org/sites/default/files/resources/ACPI_Spec_6_5_Aug29.pdf).
