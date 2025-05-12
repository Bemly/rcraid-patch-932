# rcraid-patch-932
Inofficial patch driver AMD RAID( aka rcraid ) to work in linux 6.14 and above

The code only patch free sdk code driver, dont include rcblob.

## Foreword
Many AMD mainboards for the AM4 socket based on the following chipsets come with RAID support:
 * X370 / B350
 * X399
 * X470 / B450
 * X570 / B550

But this RAID mode, which needs to be set in the BIOS, requires a specific driver for each OS.
There is a driver for Windows, but for Linux AMD provides either a binary blob or the sources.
When following the instructions, you will need to recompile the driver and install the kernel module on each kernel update and/or upgrade.
Since we are in the 21 century and we have software like DKMS, we don't need to do this manually, but let it happen automatically.

## Install for Ubuntu
Preparation:
===========
  * burn the ISO Image to usb
  * Append `modprobe.blacklist=ahci` to `linux	/casper/vmlinuz  --- quiet splash` and `linux	/casper/vmlinuz nomodeset  --- quiet splash` in USB:/boot/grub/grub.cfg
  * Append `modprobe.blacklist=ahci` to `linux	/casper/vmlinuz  iso-scan/filename=${iso_path} --- quiet splash` and `linux	/casper/vmlinuz nomodeset  iso-scan/filename=${iso_path} --- quiet splash` in USB:/boot/grub/loopback.cfg
  * Reboot
  * Switch to RAID mode

PreInstall:
===========
  * Download 9.3.2 rcraid on AMD offical: https://www.amd.com/en/support/chipsets/amd-socket-strx4/trx40
  * Put rcraid driver into usb
  * Boot your Linux installation from a USB disk
```
sudo apt-get update
sudo apt install build-essential dwarves git mokutil
cd <your_rcraid_sdk_path>
git clone https://github.com/Bemly/rcraid-patch-932.git
patch -p1 < rcraid-patch-932/rcraid-932.patch
sudo cp /sys/kernel/btf/vmlinux /usr/lib/modules/`uname -r`/build/
sudo make clean
sudo make
sudo cp rcraid.ko /lib/modules/`uname -r`/kernel/drivers/scsi
sudo depmod -a
sudo modprobe rcraid
```

Install:
===========
* Install Ubuntu

PostInstall (don't restart):
===========
```
sudo chroot /target sed -i.bak -e '/^GRUB_CMDLINE_LINUX_DEFAULT/ s/ *modprobe.blacklist=ahci// ; /^GRUB_CMDLINE_LINUX_DEFAULT/ s/"$/ modprobe.blacklist=ahci"/' /etc/default/grub
sudo chroot /target sed -i.bak -e '/\/boot\/vmlin/ s/ *modprobe.blacklist=ahci// ; /\/boot\/vmlin/ s/$/ modprobe.blacklist=ahci/' /boot/grub/grub.cfg
sudo cp rcraid.ko /target/lib/modules/`uname -r`/kernel/drivers/scsi/rcraid.ko
sudo chroot /target depmod -a `uname -r`
sudo chroot /target cp -ap /boot/initrd.img-`uname -r` /boot/initrd.img-`uname -r`.bak
sudo chroot /target mkinitramfs -o /boot/initrd.img-`uname -r` `uname -r`
reboot
```

## Update kernel
```
install Kernel
sudo make clean
sudo make KDIR=/lib/modules/{folder-with-new-kernel}/build/
sudo cp rcraid.ko /lib/modules/{folder-with-new-kernel}/kernel/drivers/scsi/rcraid.ko
sudo depmod -a {folder-with-new-kernel}
sudo cp -ap /boot/initrd.img-{folder-with-new-kernel} /boot/initrd.img-{folder-with-new-kernel}.bak
sudo mkinitramfs -o /boot/initrd.img-{folder-with-new-kernel} {folder-with-new-kernel}
Reboot
```
