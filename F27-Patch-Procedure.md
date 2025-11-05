# **HP Omen Transcend 16 Firmware vF.27 Full Patch Procedure, Changelog, & System Spec**
#### Authored by [no-hands-hand](https://github.com/no-hands-hand/OMEN-Transcend-16-ACPI-fix-f27)
## ***DISCLAIMER***
HP Omens are NOT meant to run Linux. HP has designed their firmware/components in such a way that some of them are explicitly disabled when running a non-Windows OS.</br>
As such, proceeding with this guide is not supported by HP and I expect you will void any warranty available for your machine.</br>
For all components to run successfully, you will need to modify the ACPI tables of your machine. **There is a significant chance you can mess something up**. There is also a less significant chance that *I* messed something up, and it just isn't apparent yet. Components being powered in ways they were not meant to can cause unforseen issues (although I have not seen any yet).</br>
**By proceeding with this guide, you acknowledge these risks and that you are making these changes of your own free will. I am not responsible if you mess up your machine.** 
## **Purpose**
The purpose of this document is to guide users to installing Linux on an HP Omen with modern(as of Oct. 2025) firmware. We will start at the very beginning (installing Arch Linux via bootable USB) and fix each issue as they arise.</br>
I use [Omarchy](https://omarchy.org/), which is an Arch Linux distro with Hyprland compositor built in. So this guide will cover installing specifically Arch Linux with a Limine bootloader (recommended bootloader for use with Omarchy), though it can be easily adjusted to work with other Linux distros if you put in a little effort.</br>

I apologize in advance if you are coming from a non-Arch distro or desire an Arch setup that is different from mine, as some of this may not work for you. I recommend feeding your issue, [this repo](https://github.com/no-hands-hand/OMEN-Transcend-16-ACPI-fix-f27) or [j0hnwang](https://github.com/j0hnwang/OMEN-Transcend-16-ACPI-fix)'s, and whatever other online resources you can find into a GPT-5 or Gemini 2.5 Pro deep research prompt. It may help you adapt my solution to fit your needs. I will add a list of sources at the bottom of this document.

The main concern is the firmware version - **These patches are specific to the F.27 firmware**. I will **not** be providing a compiled .aml file, as this will just hurt many people since ACPI tables can change between laptop configurations, firmware versions, etc. I will be providing the unaltered DSDT.dsl file, and my patched DSDT.dsl file. You can use these files to locate the changed blocks and fix them, but I will also include a guide on which changes to make down below. I **highly recommend** doing research into ACPI in general so you know what you are doing, and are able to detect if your specific ACPI tables differ enough that following this guide will cause more problems than it solves.

## **Issues that this Guide can help with**
When I installed Linux, I experienced the following issues:</br>
**1.** Kernel panic/system hang on install (fixed by adding `noapic` and `pci=nobar` to install & boot params)</br>
**2.** Built-in touchpad nonfunctional (with kernel param `noapic`)</br>
**3.** Internal speakers nonfunctional (with kernel param `noapic`)
</br>**4.** Internal fans working, but not correctly responding to large temp changes (In Windows, this is handled by the Omen Gaming Hub, which obviously is not a thing on Linux)
</br>**5.** Laptop failing to wake up after lid being closed (with kernel param `noapic`)

As of **Oct. 22, 2025** I have managed to get my system to this state:
- **Touchpad**: Functional, but at a basic level. More testing needed to determine if this is the full fix, or if this is more of a jerryrig solution. Scrolling seems to be locked to non-natural.
- **Internal Speakers**: Fully functional
- **Fans**: Working better but likely not due to my changes (possibly fixed by recent kernel update, or by j0hnwang's fix)
- **Wake from S3 state sleep (closed lid)**: Fully functional

## **Quick Tips, Tricks, & General Info**
- **DO NOT BOTHER** trying to extract the extra SSDT tables that will inevitably accompany your DSDT. You would need to disassemble them together with the DSDT to create one monolithic DSDT.dsl file. Normally would not be a major problem, but HP (specifically Omen) ACPI tables are littered with issues to the point of being malicious. I wasted probably two weeks just trying to disassemble all of the tables together. Just the DSDT will be fine.
- **HP does not want you to run Linux**. I'm not sure if this applies to all HP laptops, but the Omen product line ACPI tables are specifically designed to deactivate select components when the system detects an OS other than Windows. The recommended solution to this issue from [Arch Linux Wiki](https://wiki.archlinux.org/title/DSDT) is to basically spoof the OS that is reported to your firmware on boot. This can be accomplished by adding two boot params(`acpi_osi=!` and `acpi_osi="Windows 2022"`) **AFTER INSTALLING & BOOTING SUCCESSFULLY**. I recommend adding them to the **end** of your boot params. It didn't work for me, but it might work for you.
    - **IMPORTANT NOTE**: If you add the `acpi_osi=!` and `acpi_osi="Windows 2022"` boot params and they do not work, **be sure to remove them** because it can cause issues with the ACPI patches detailed in this guide (I added them after I fixed my touchpad and it immediately broke again).
    - This also will not help you with installing/booting Linux. This will only (possibly) help after you have successfully installed & booted.


# Patch Guide
## **System Specs**
- **Device**: HP Omen Transcend 16-u1000 (Product: 8M1Q6AV)
- **Linux Distro**: Omarchy 3.1.1 (Arch Linux v6.17.4-arch2-1 + Hyprland v0.51.1)
- **Bootloader**: Limine
- **Boot Image**: initramfs -> UKI
- **HP Firmware Version**: F.27 Rev.A - Sept 8, 2025 (As listed on HP website - My actual system only displays as F.27)
- **Filesystem**: btrfs
- **Disk Encryption**: LUKS full-disk (required by Omarchy)
- **Partitions**: single partition for my entire FS. Additional 1 GB boot partition (default for limine)

## **Pre-Requisites**
- **Secure Boot, TPM, Fast Boot DISABLED**
- 32 GB USB Drive
- Arch Linux .iso file
- Software for creating bootable USB device with a .iso -> Rufus, Ventoy, balenaEtcher on Windows
## **Initial Linux Install Procedure**
### **Step 1**
For initial install, use Rufus or Ventoy to flash the Arch Linux .iso onto a flash drive
- I used Ventoy because it provides a boot menu where you can modify boot params on the fly - you **absolutely need this** to successfully install Linux
### **Step 2**
When you boot up the Arch .iso, hit whatever key lets you modify the boot process (usually `e` but may be something different for you) and add the `noapic` and `pci=nobar` boot params. The .iso will never progress to a point where you can even interact with it unless you add these.
### **Step 3**
Once you arrive at the Arch shell, set up your internet access (if on ethernet, don't worry about it - this is for wifi only):
- Run `iwctl`
- Run `station wlan0 scan`
- Run `station wlan0 connect <tab>` (do a space and hit the TAB key after "connect" - this will show you the SSIDs of available wifi networks)
- Pick your network, enter password (if applicable)
### **Step 4**
Run `archinstall` and pick the options as listed on [Omarchy Manual Install](https://learn.omacom.io/2/the-omarchy-manual/96/manual-installation) - if you're not interested in Omarchy, don't worry. The linked install guide is mostly generic Arch setup, aside from a single `curl` command at the end. If you know what you want for your Arch setup, feel free to use those values instead. Know that the rest of this guide will assume you selected the options as described in the Omarchy Manual Install page. If you are fine with the Arch setup but don't want Omarchy, just skip the `curl` command at the end of the Manual Install guide.

### **Step 5**
***IMPORTANT NOTE***: This step is mainly theoretical on my part, in the sense that I don't know the specific locations/commands/etc because I did NOT do this when I installed, and it fucked everything up. I am extrapolating these instructions from my method of changing things AFTER install, as well as knowing that it is what you are supposed to do. If you miss this, you're going to need to use the bootable USB to chroot into your Arch install and fix it later. I promise it sucks so much.</br>
Once the `archinstall` run finishes, **DO NOT REBOOT YET**. The (again, theoretical) thing you need to do is cd into your `/boot` directory and add `noapic` and `pci=nobar` into your `limine.conf` file (or whatever bootloader you use - just throw those into your kernel params). Follow these steps:
- Use pacman to install nano, vim, etc. (you need a sudo-capable editor) - I recommend nano just for ease of use if you don't have vi-style editor experience
- Run `cd /boot`
- Run `ls` to check if your `limine.conf` file is here - if not, it may be located in `/boot/EFI/limine/limine.conf`. If you have both but aren't sure which is the primary (which is my case), just edit both and figure it out later. For reference, my primary one ended up being the `/boot/limine.conf` one (you can figure this out ***later*** by rebuilding your boot image and seeing which .conf file is updated with the new image hash)
- `sudo nano limine.conf` and find the primary boot entry (not the fallback).
- On the line labeled `kernel_cmdline`, go to the very end and add `noapic` and `pci=nobar`, then save the file (for nano this is Ctrl + O, then hit enter. Ctrl + X to exit afterwards)
- This will let you boot into your system once it restarts without editing boot params on the fly

#### **Step 5.5 *(Optional but Recommended)***
(THEORETICAL) If you want to make this a persistent change (meaning it will stay like this even after you update your kernel or rebuild your boot image - which you WILL be doing), you need to do `sudo nano /etc/default/limine`, find the line labeled `KERNEL_CMDLINE[default]="cryptdevice...."` and add `noapic` and `pci=nobar` to the end of that string, still inside the quotation marks. Save that file.

### **Step 6**
Now you can reboot via `sudo reboot`
- **WARNING**: You will absolutely need a USB mouse from here on out - and possibly a keyboard as well. Do not try to use a bluetooth one, get a 2.4 with the USB plugin at the very minimum. **Once you pass this point, you won't be able to use your touchpad until you complete the fix.**

### **Step 7**
For me, the next step was to install Omarchy itself via the `curl` or `wget` command. You don't have to do this. HOWEVER - Omarchy comes with alot of things built-in. I can't always differentiate between what was here already and what I installed myself. So if you end up missing some packages that I use and didn't say to install, that may be why. If you want to try Omarchy (or if installing it was already your plan), continue following the Omarchy Manual Install link in Step 4. The specific `curl` command is included there.

## **DSDT Patch Procedure**
Starting this section assumes you have successfully installed Arch Linux on your Omen, and are able to successfully login. You will need a code editor (since I use Omarchy and have a GUI, I used Zed/Cursor), sudo privileges, and some tools that we will download as part of Step 1. You will also need a package manager (pacman should be included in Arch)
### **Step 1** - Download required Tools
Run the following command to install the necessary tools for ACPI editing & debugging:
```bash
sudo pacman -Syu --noconfirm acpica fwts-git gawk coreutils grep sed findutils i2c-tools
```
The most important ones are the `acpica` and `fwts-git` packages - these will allow you to directly dump your ACPI tables and even test your changes. `i2c-tools` will allow you to debug your system after applying the patch.

### **Step 2** - Set up Working Dir & get clean ACPI dump
Run the following commands to set up your working directory. Please note this is just the advised path - you don't have to use this.
```bash
# Starting in ~/
# Make a working directory
mkdir -p omen-dsdt
cd omen-dsdt

# Make directories for your clean & patched dsdt.dsl files
mkdir -p f27_tables
mkdir -p f27_tables_patched

cd f27_tables
```
There are two ways you can go from here:
- **A**: Extract ALL your tables (dsdt & supporting sstd's) - easy to dump but hard to disassemble. This is *technically* the most cohesive way to do ACPI patching, but you are going to have an awful time trying to resolve all the duplicate namespaces that Windows firmware ships with. If you want to take this path, you need to spend alot of time researching how to cleanly resolve external SSDTs when disassembling your `dsdt.dsl`. If you are dead-set on this path, open an issue on my Github and I will provide some sources I found during my research. Please note that I never successfully pulled this off.
- **B**: Extract **only** the `dsdt.dat` ACPI tables - You will get a warning about "did not resolve external blah blah blah" on disassembly/recompile, but that is not a problem. Your computer will still load those external resources from the SSDTs already in your system even when you implement the override - so long as you don't mess with their import statements. **This is the path I will be demonstrating**.

Dump your main ACPI tables (aka DSDT):
```bash
# Extract the binary DSDT ACPI tables to your "clean" directory - in this case, ~/omen-dsdt/f27_tables
cat /sys/firmware/acpi/tables/DSDT > dsdt.dat
```
### **Step 3** - Disassemble Binary Tables into .dsl
First, we will disassemble the `dsdt.dat` file into `dsdt.dsl`. The `acpica` package is REQUIRED for this step - and any steps that use the `iasl` command.
```bash
# Disassemble the binary tables into a readable and editable format
iasl -d dsdt.dat

# This outputs a dsdt.dsl file. You should now copy this file into your patch directory - leave the clean version alone in case you mess something up.
cp dsdt.dsl ../f27_tables_patched
cd ../f27_tables_patched
```

### **Step 4** - Perform `dsdt.dsl` patches
**WARNING**: Be sure to use the `dsdt.dsl` file that was copied into your "patched" directory. Leave the original in case of emergency - if you try to extract the `dsdt.dat` later on while an override is in effect, you will not get clean results and could just mess things up further.

Now, apply the patches noted in the [DSDT File Changes](#dsdt-file-changes) section.

### **Step 5** - Recompile `dsdt.dsl` 
After applying the patches to your `dsdt.dsl` file, you will need to recompile it into something the machine can understand. Run:
```bash
iasl -tc dsdt.dsl
```
This will output two files (if successful):
- `dsdt.hex` - ignore this
- `dsdt.aml` - this is what you need

If you encounter errors during compiling, examine them and do some internet research. AI models can be very useful for debugging these issues.

### **Step 6** - Create override directory
- This part will be **heavily** dependent on your system setup, including bootloader and how your system rebuilds your boot image. If your method differs from mine: I'm truly sorry, but I only figured this out through sheer trial and error and I am not able to help you. I recommend AI assistance or Linux message boards to adapt this guide to your needs.
- Check the [System Specs](#system-specs) section for the specifics of how my boot image is created.

The only override method that worked for me is baking the ACPI override into my initramfs build. That is done by creating a new directory called `acpi_override` in `/etc/initcpio/` and then rebuilding the image.
Run:
```bash
sudo mkdir -p /etc/initcpio/acpi_override
```

### **Step 7** - Enable `acpi_override` Build Hook
- **WARNING**: Before proceeding further, I recommend setting up a manual snapshot script/command and linking it to your bootloader. Omarchy employs snapper by default, but you still have to configure/enable it.

Now that we have the standard override directory created, we need to enable the build hook that will pick it up. There are three ways this can happen:
- **A)** If your Arch/bootloader/whatever setup does not have any drop in config files (for me, they are located at `/etc/mkinitcpio.conf.d/`), you can directly edit `/etc/mkinitcpio.conf` to add `acpi_override` to the `HOOKS` section.
- **B)** If your setup is like mine and has a drop-in config file already there (the file in this case is `/etc/mkinitcpio.conf.d/omarchy_hooks.conf`) that is overriding the `HOOKS` section, you need to add it there instead.
- **C)** You can also make a new drop-in .conf file yourself. Name it something like "acpi-hooks.conf" and just add a single line: `HOOKS+=(acpi_override)`, then add it to `/etc/mkinitcpio.conf.d/`. Please note that if there are any existing drop-ins that fully overwrite the existing hooks (meaning they have `HOOKS=(...)` instead of `HOOKS+=(...)`) that this **will not work**. You will need to instead add your hook to the existing drop-in as described for **Option B**.

### **Step 8** - TAKE A SNAPSHOT
Self-explanatory section name - make sure your snapshots are available at your bootloader screen, in case your changes mess something up or lock you out

### **Step 9** - Copy the modified DSDT to the Override folder

Copy your modified & compiled `dsdt.aml` file to the override folder we created in Step 6
```bash
# running in ~/omen-dsdt/f27_tables_patched
sudo cp dsdt.aml /etc/initcpio/acpi_override/
```

### **Step 10** - Install Supporting Drivers & Kernel Modules
For the fix to truly work, we need to install any supporting drivers as well as add any necessary kernel modules. Run:
```bash
# Install necessary drivers for our components
sudo pacman -Syu linux-firmware-cirrus linux-firmware-intel linux-firmware-other
```
***Note***: The next portion will be different depending on your setup.</br>
For the kernel modules, we will be directly editing `/etc/mkinitcpio.conf` (This can also be done by adding a drop-in .conf file to `/etc/mkinitcpio.conf.d/` if you would prefer to do that. Just format it similar to **Step 7 - Option C**. That was not an option for me due to existing drop-ins.)</br>

Open your .conf file:
```bash
sudo nano /etc/mkinitcpio.conf
```
Now, you should see the "MODULES" section at the very top. You will need to modify it. There is an optional module `i915` that is VERY helpful for laptops that have an integrated Intel GPU + Nvidia GPU hybrid setup - this can prevent issues when rendering some applications on Linux. If you do add it, you have to add it at the very beginning. The other ones you can add at the end. Edit your `/etc/mkinitcpio.conf` file to reflect this:
```bash
# existing .conf:
MODULES=([existing modules, ex: nvidia, btrfs, ...])

# Updated .conf:
MODULES=(i915 [existing modules] i2c_hid_acpi i2c_i801 intel_lpss_pci)
```
Save the file and exit.

### **Step 11** - Add i8042 kernel params (OPTIONAL(?))
There were some kernel params that were recommended based on my Intel components and especially the ElanTech touchpad. You may benefit from adding them. I am not entirely sure if they are affecting anything, but they are present and my system is working so maybe consider adding them.</br>

Since we are rebuilding the image in the next step, we only need to add them to `/etc/default/limine`.</br>
Run:
```bash
sudo nano /etc/default/limine
```
and add these params to the END of your `KERNEL_CMDLINE[default]=...` line:
```bash
KERNEL_CMDLINE[default]="[existing params] noapic pci=nobar i8042.reset i8042.nomux i8042.nopnp"
```

### **Step 12** - Rebuild initramfs image & Reboot
Now we rebuild the boot image via UKI. Run:
```bash
# Rebuild your build image (command dependent on setup)
sudo limine-mkinitcpio
```

After rebuilding the image, double check your bootloader .conf file to ensure that your default `kernel_cmdline` values are still working. Now reboot:
```bash
sudo reboot
```


### **Step 13** - Check the results
I will be providing some short helper scripts for you to run in a different section, but that will allow you to check your `dmesg` logs for ACPI-related messages. You can also check yourself with the following commands:
```bash
# Get any ACPI-related messages
sudo dmesg | grep -i -e ACPI

# Check registered input devices
sudo libinput list-devices

# Check for any dmesg related to i2c & hid (i2c is for our devices I2C0 & I2C1, hid checks for any "human input devices")
sudo dmesg | grep -i -e i2c -e hid

# Checks for loaded i2c kernel modules
lsmod | grep i2c

# Collects registered hardware devices, and any kernel modules/drivers associated with them
lspci -k
```
**IMPORTANT**: It is HIGHLY likely that you will not see any change yet, and your touchpad will probably not work yet. However, your speaker may be working, so feel free to test it out. IF, by some miracle, all of your components are now working, feel free to leave this guide and enjoy your machine. If not, proceed to the next step.

### **Step 14** - Remove safety rails
I didn't talk much about what the `noapic` and `pci=nobar` kernel params do, but they are basically safety rails to prevent a kernel panic on boot due to improper routing in the ACPI files. Now that we have our override in place, we can remove them and try to boot again.</br>

- **1)** First, assuming you were able to boot successfully after the acpi_override, **MAKE ANOTHER SNAPSHOT**. Once again, ensure it is available at your boot menu - if something wasn't done correctly, removing the `noapic` and `pci=nobar` will be tough without a way to get back into your system.
- **2)** Now, ***without rebuilding the initramfs image***, edit your bootloader file (for me, this is `/boot/limine.conf`) and remove the `noapic` and `pci=nobar` params from the `kernel_cmdline: ...` line. **DO NOT REMOVE THEM FROM YOUR DEFAULTS YET**.
- **3)** Run `sudo reboot`. If your computer hangs or you get kernel panics when you boot into your system after removing those params, something is wrong. I recommend seeking help from forums or a friendly AI.
- **4)** If your computer successfully boots up, test out your touchpad. It ***should*** work, and if it does, great! If not, this is as far as I can take you. Feel free to open an issue on my Github but I am not an expert in ACPI or the Linux kernel, so the help I can give you is limited.

### **Step 15** - Clean up
If your computer was able to boot successfully without the `noapic` and `pci=nobar` kernel params, you should go ahead and remove them from `/etc/default/limine`. If you leave them there, they will be added the next time you rebuild your initramfs image.</br>

### **Step 16** - Pay it forward please!
Hopefully this was able to help you get Linux working properly on your HP Omen. The only existing patches when I started this were for F.11 and F.12 firmware versions, and I had to pretty much muddle my way through a hundred different dead-end solutions before I got to this point.</br>
If you are visiting this repo on a firmware version that releases **after F.27**, please fork this repo and do your best to apply the patches to your version. If they work, share the resulting files and a README on any relevant differences, and open a PR or just publish it yourself. Linux is community-driven; solutions to issues like the one with the HP Omen come from users like you. Share this repo and any fixes you make! 


## **DSDT File Changes**
The following changes will have 3 labels: 
- **Legacy (L)**: Changes that have been carried over from j0hnwang's previous patches. I am not sure if they are absolutely required, but things are working with them, so I suggest keeping them.
- **Necessary (N)**: These are changes I have determined to be essential to the fix.
- **Recommended (R)**: Similar to the **Legacy** changes, I am not certain that they are absolutely required BUT they are currently working without issue, so I suggest using them.

**(N) 1.** This is probably the most important change. Almost at the very top of your `DSDT.dsl`, find the `DefinitionBlock` line and uptick the last value by one. This tells the system that you are overriding the previous `DSDT` with a new one. Here are my before and after ones:</br>
*Before*
```C
DefinitionBlock ("", "DSDT", 2, "HPQOEM", "8C4D    ", 0x00000002)
```
*After*
```C
DefinitionBlock ("", "DSDT", 2, "HPQOEM", "8C4D    ", 0x00000003)
```
**(L) 2.** Find the portion labeled `Scope (_SB.PC00)`, with `Device (IC04)` right under it. ***Completely remove*** this portion. Here is the exact code block:
```C
Scope (_SB.PC00)
{
    Device (IC04)
    {
        Name (_HID, "HPIC0004")  // _HID: Hardware ID
        Method (_STA, 0, NotSerialized)  // _STA: Status
        {
            Return (0x0F)
        }
    }
}
```
- Please note that this portion is based on changes from `j0hnwang`'s original patch - not entirely sure what it does but it seemed to help.

**(N) 3.** Find the `Scope (_SB.PC00.I2C0)` (for me, it was around line 107630) that contains `Device (TPD0)`. This is your touchpad. Which specific touchpad you have varies depending on your laptop configuration, but we are going to force it to believe you have an ElanTech touchpad since Linux has firmware drivers to interact with this. I have added `[Edits]` tags around the portions I edited for ease of viewing. If you are curious about what is happening, I tried to include what each changed line does.
```C
Scope (_SB.PC00.I2C0)
{
    Name (I2CN, Zero)
    Name (I2CX, Zero)
    Name (I2CI, Zero)
    Method (_INI, 0, NotSerialized)  // _INI: Initialize
    {
        I2CN = SDS0 /* \SDS0 */
        I2CX = Zero
    }

    Device (TPD0)
    {
        Name (HID2, Zero)
        Name (SBFB, ResourceTemplate ()
        {
            I2cSerialBusV2 (0x0000, ControllerInitiated, 0x00061A80,
                AddressingMode7Bit, "NULL",
                0x00, ResourceConsumer, _Y67, Exclusive,
                )
        })
        Name (SBFG, ResourceTemplate ()
        {
            GpioInt (Level, ActiveLow, ExclusiveAndWake, PullDefault, 0x0000,
                "\\_SB.GPI0", 0x00, ResourceConsumer, ,
                )
                {   // Pin list
                    0x0000
                }
        })
        Name (SBFI, ResourceTemplate ()
        {
            Interrupt (ResourceConsumer, Level, ActiveLow, ExclusiveAndWake, ,, _Y68)
            {
                0x00000000,
            }
        })
        CreateWordField (SBFB, \_SB.PC00.I2C0.TPD0._Y67._ADR, BADR)  // _ADR: Address
        CreateDWordField (SBFB, \_SB.PC00.I2C0.TPD0._Y67._SPE, SPED)  // _SPE: Speed
        CreateWordField (SBFG, 0x17, INT1)
        CreateDWordField (SBFI, \_SB.PC00.I2C0.TPD0._Y68._INT, INT2)  // _INT: Interrupts
        Method (_INI, 0, NotSerialized)  // _INI: Initialize
        {
            /* [Edits] */
            // Set the "enabled" bit (I2CN) to true
            Or (\_SB.PC00.I2C0.I2CN, One, \_SB.PC00.I2C0.I2CN)
            // Steer firmware to ELAN07CA path on 0x15 at 400kHz, using GPIO IRQ
            Store (0x05, \TPDT)    // branch that leads to ELAN mapping in your table
            Store (0x15, \TPDB)    // 7-bit I2C address 0x15
            Store (One,  \TPDH)    // ELAN subselector
            Store (One,  \TPDS)    // speed selector: 0=100k, 1=400k, 2=1MHz → use 400k
            Store (Zero, \TPDM)    // 0 => use SBFG (GPIO) in _CRS, not GSI -> this handles interrupts

            // If firmware left placeholder HID, force ELAN vendor HID
            If (LEqual (_HID, "XXXX0000"))
            {
                Store ("ELAN07CA", _HID)
            }
            /* [End Edits] */

            // Omitted random garbage for brevity - only change things I changed.
            // If you don't know what was here previously, find it in the clean copy of my DSDT.dsl

            If ((TPDT == 0x05))
            {
                _HID = "SYNA32E5"
                If ((TPDB == 0x2C))
                {
                    If ((TPDH == 0x20))
                    {
                        _HID = "SYNA32E5"
                    }
                }

                If ((TPDB == 0x15))
                {
                    If ((TPDH == One))
                    {
                        _HID = "ELAN07CA"
                    }
                }

                HID2 = TPDH /* \TPDH */
                BADR = TPDB /* \TPDB */
                If ((TPDS == Zero))
                {
                    SPED = 0x000186A0
                }

                If ((TPDS == One))
                {
                    SPED = 0x00061A80
                }

                If ((TPDS == 0x02))
                {
                    SPED = 0x000F4240
                }

                Return (Zero)
            }
        }

        Name (_HID, "XXXX0000")  // _HID: Hardware ID
        Name (_CID, "PNP0C50" /* HID Protocol Device (I2C bus) */)  // _CID: Compatible ID
        Name (_S0W, 0x03)  // _S0W: S0 Device Wake State
        Method (_DSM, 4, Serialized)  // _DSM: Device-Specific Method
        {
            If ((Arg0 == HIDG))
            {
                Return (HIDD (Arg0, Arg1, Arg2, Arg3, HID2))
            }

            If ((Arg0 == TP7G))
            {
                Return (TP7D (Arg0, Arg1, Arg2, Arg3, SBFB, SBFG))
            }

            Return (Buffer (One)
            {
                    0x00                                             // .
            })
        }
        /* [Edits] */
        /* Previous Method (_STA ...):
        Method (_STA, 0, NotSerialized)  // _STA: Status
        {
            If (((TPDT != Zero) && (I2CN & One)))
            {
                Return (0x0F)
            }

            Return (Zero)
        }
        */
        // Updated Method(_STA, ...):
        Method (_STA, 0, NotSerialized)  // _STA: Status
        {
            Return (0x0F) // -> This hardcodes your TPD0 device as "visible" to your OS.
        }
        /* [End Edits] */

        Method (_CRS, 0, NotSerialized)  // _CRS: Current Resource Settings
        {
            If ((OSYS < 0x07DC))
            {
                Return (SBFI) /* \_SB_.PC00.I2C0.TPD0.SBFI */
            }

            If ((TPDM == Zero))
            {
                Return (ConcatenateResTemplate (I2CM (I2CX, BADR, SPED), SBFG))
            }

            Return (ConcatenateResTemplate (I2CM (I2CX, BADR, SPED), SBFI))
        }
    }
}
```

**(R) 4.** Find the `Scope (_SB.PC00.I2C1)`. Inside `Method (_INI)`, we are going to add similar logic to the previous change, but inside `\_SB.PC00.I2C1._INI` rather than inside the `TPD0._INI`.
- ***NOTE***: This is a change I am not sure about. It is leftover from one of my attempted fixes, but I haven't removed it and things are working so I am really not sure if I should remove it or not. Feel free to not include it and let me know how it goes, preferably via raising an issue on my Github.
</br>

*Before Edits:*
```C
Scope (_SB.PC00.I2C1)
{
    Name (I2CN, Zero)
    Name (I2CX, Zero)
    Name (I2CI, One)
    Method (_INI, 0, NotSerialized)  // _INI: Initialize
    {
        I2CN = SDS1 /* \SDS1 */
        I2CX = One
    }
    [...]
}
```
*After Edits:*
```C
Scope (_SB.PC00.I2C1)
{
    Name (I2CN, Zero)
    Name (I2CX, Zero)
    Name (I2CI, One)
    Method (_INI, 0, NotSerialized)  // _INI: Initialize
    {
        /* [Edits] */
        I2CN = One //SDS1 /* \SDS1 */
        I2CX = One
        // Steer firmware _INI to ELAN branch:
        Store (0x05, \TPDT)
        Store (0x15, \TPDB)
        Store (One,  \TPDH)
        Store (0x02, \TPDS)
        /* [End Edits] */
    }
    [...]
}
```
**(N) 5.** Still within the `Scope (_SB.PC00.I2C1)`, scroll down until you find the `Device (TPD0)`, and within it, `Method (_STA, ...)`. We are going to hardcode this device to be disabled, by setting the `_STA` method to return `Zero`. We are also going to disable the `Device (LLKB)` (located above `TPD0`).
- ***Background***: HP defines duplicates of many components in their ACPI tables. When a non-Windows OS is trying to run on this firmware, it can cause problems. Th(is step is about preventing that, by disabling the duplicate `TPD0` device that is housed in `I2C1`.
- ***NOTE***: If enabling the `_SB.PC00.I2C0.TPD0` does not work, you may want to try "flipping" it - meaning mirror the changes from `I2C0` (from Change #3) onto the `I2C1.TPD0` instead, and vice versa. Whether or not this is necessary for you will depend on your laptop configuration and there is truthfully no way for me to tell you at this time.

*Before edits:*
```C
Scope (_SB.PC00.I2C1)
{
    If ((ToInteger (LLKE) == One))
    {
        [...]
        Device (LLKB)
        {
            [...]
            Method (_STA, 0, NotSerialized)  // _STA: Status
            {
                Return (0x0F)
            }
            [...]
        }
    }
    Else
    {
        Device (TPD0)
        {
            [...]
            Method (_STA, 0, NotSerialized)  // _STA: Status
            {
                If (((TPDT != Zero) && (I2CN & One)))
                {
                    Return (0x0F)
                }

                Return (Zero)
            }
            [...]
        }
    }
```
*After Edits:*
```C
Scope (_SB.PC00.I2C1)
{
    If ((ToInteger (LLKE) == One))
    {
        [...]
        Device (LLKB)
        {
            [...]
            Method (_STA, 0, NotSerialized)  // _STA: Status
            {
                /* [Edits] */
                Return (Zero)
                /* [End Edits] */
            }
            [...]
        }
    }
    Else
    {
        Device (TPD0)
        {
            [...]
            Method (_STA, 0, NotSerialized)  // _STA: Status
            {
                /* [Edits] */
                Return (Zero)
                /* [End Edits] */
            }
            [...]
        }
    }
```
**(L) 6.** This last one is another patch from j0hnwang's previous fix. I'm not sure about the exact effect, but my speakers are working and they function on Cirrus audio so I figure this probably helped. Locate `Scope (_SB.PC00.I2C4)`, `Device (SPKR)`. Within the `SPKR` device, you should see `Name (_DSD, ...)`. Then, look through `Package (0x0A)`, through the nested packages, until you see the `Package (0x02)` version that contains a description with "boost-peak-milliamp" in it.
- **Shortcut**: a quick way to reach this area is just to search within your file for "cirrus". It *should* be the only portion of the file containing it.
- **Warning**: There are many copies of `Package (0x02)` within `Package (0x0A)`. Be sure you ONLY mess with the one that details "boost-peak-milliamp". Conveniently, it is also the only one that has "cirrus" in the description **twice**.

*Before Edits:*
```C
Scope (_SB.PC00.I2C4)
{
    [...]
    Device (SPKR)
    {
        [...]
        Name (_DSD, Package (0x02)
        {
            Package (0x0A)
            {
                [...]
                Package (0x02)
                {
                    "cirrus,cirrus,boost-peak-milliamp", 
                    Package (0x02)
                    {
                        0x1004, 
                        0x1004
                    }
                }, 
            }
        }
```
*After Edits:*
```C
Scope (_SB.PC00.I2C4)
{
    [...]
    Device (SPKR)
    {
        [...]
        Name (_DSD, Package (0x02)
        {
            Package (0x0A)
            {
                [...]
                Package (0x02)
                {
                    /* [Edits] */
                    "cirrus,boost-peak-milliamp", 
                    /* [End Edits] */
                    Package (0x02)
                    {
                        0x1004, 
                        0x1004
                    }
                }, 
            }
        }
```

# Helper Scripts

## Check all ACPI Kernel Messages & Modules
This is a script I wrote myself to easily run the required tests on reboot. I suggest adding it to a dir that's on your path (for example, I have a `~/bin/` dir that I added to my system path for alias'ed commands) and making an alias like "acpi" so you can run it quickly.
```bash
#!/usr/bin/env bash

# This script assumes you are using the ~/home/omen-dsdt dir I recommended to do your patching. It will create an "acpi-test-logs" directory and add the results there. They will be OVERWRITTEN each time you run it
set -euo pipefail

OMEN="${1:-$HOME}/omen-dsdt"
WORK="${OMEN}/acpi-test-logs"
LOGTAG="[acpi-test]"

mkdir -p "${WORK}"
say(){ echo "$LOGTAG $*"; }

sudo dmesg | grep -i -e ACPI > "${WORK}/dmesg-acpi.log"
sudo libinput list-devices > "${WORK}/libinput-devices.log"
sudo dmesg | grep -i -e i2c -e hid > "${WORK}/dmesg-i2c-hid.log"
lsmod | grep i2c > "${WORK}/lsmod-i2c.log"
lspci -k > "${WORK}/lspci-k.log"

say "ACPI test logs generated, check ${WORK} for details."
```

## Compile `dsdt.dat` into `dsdt.dsl` and output to logfile
There's not really much special for this one, just a simple script to capture the output from the `iasl -tc dsdt.dsl` command so you can easily search for error messages. It was a pain for me to continuously scroll through the console to find them so I made this.

```bash
#!/usr/bin/env bash

set -euo pipefail
set -o errtrace
trap 'echo "[!] Error at line $LINENO: $BASH_COMMAND" >&2' ERR

# This script assumes you are using the recommended directory structure outlined in the patch procedure. If you aren't, update the variables to reflect the correct directories.

OMEN="${1:-$HOME}/omen-dsdt"
WORK="${OMEN}/f27_tables_patched"

timestamp=$(date +"%Y%m%d_%H%M%S")
logfile="$WORK/compile-dsdt_${timestamp}.log"

exec > >(tee -i "$logfile") 2>&1
echo "[*] Compiling dsdt.dsl..."

sudo iasl -tc "$work/dsdt.dsl"
```

# Reference Resources
1. Arch Wiki – [ACPI section](https://wiki.archlinux.org/title/ACPI)
2. Kernel docs – [Documentation/admin-guide/acpi](https://www.kernel.org/doc/html/latest/admin-guide/acpi/index.html)
3. Linux Kernel issue raised with findings: https://lore.kernel.org/all/6e56ec6c-60ad-48ea-b185-19d7064a53f2@free.fr/T/#u
4. Issue raised regarding RealTek audio drivers: https://gitlab.archlinux.org/archlinux/packaging/packages/alsa-firmware/-/issues/1
5. Installing arch-linux on a Microsoft Surface: https://chrismcleod.dev/blog/installing-arch-and-omarchy-on-a-microsoft-surface-laptop-studio/#:~:text=Step%202%3A%20Secure%20Boot
6. Arch-Linux Omen 15(Intel variant) page: https://wiki.archlinux.org/title/HP_Omen_15-ek005na
7. Arch-Linux Omen 16(AMD Variant) page: https://wiki.archlinux.org/title/HP_Omen_16-c0140AX
8. Arch-Linux DSDT Page: https://wiki.archlinux.org/title/DSDT
9. HP Article on Linux install for Omen Transcend 16: https://h30434.www3.hp.com/t5/Notebook-Hardware-and-Upgrade-Questions/Solved-installing-Linux-on-HP-Omen-Laptops/m-p/9025522/highlight/false#M739408
10. HP Support Page for Omen Transcend 16: https://support.hp.com/us-en/product/details/omen-by-hp-transcend-16-inch-gaming-laptop-pc-16-u1000/2101915728
11. AI Response - Extracting ACPI Tables with Duplicates: https://search.brave.com/search?q=extracting+acpi+tables+when+there+are+AE_ALREADY_EXISTS+errors&source=desktop&summary=1&conversation=e876288d9af0ee8fbd46
12. UEFI Documentation Site: https://uefi.org/
