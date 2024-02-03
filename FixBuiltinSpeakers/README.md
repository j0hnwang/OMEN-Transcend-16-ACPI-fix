# Kernel driver patch for HP Transcend laptop speakers

After applying my ACPI config patch, configs for internal speakers (and amplifiers) are fixed. The reason builtin speakers are still not working is, they are not bound to any audio codecs yet. This patch binds builtin speakers (and amplifiers) to realtek audio codec when realtek driver is loaded.

This patch has been tested on Fedora 39 and Ubuntu 22.04.

## HOWTO Use

1. Apply my ACPI config patch. **This is a MUST**, otherwise I2C devices are not correctly configured and speakers will not work.
2. Download kernel source for your distro, apply this patch, build and install new kernel image.
3. Reboot with new kernel image, builtin speakers should be working now. Enjoy. :)

## I am filing this patch to Ubuntu kernel team, hopefully it will be included in next kernel release for Ubuntu 22.04.
