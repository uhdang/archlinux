# Archlinux EFI Installation Procedure

## Table of Contents

PART 0: Preparation

- Items necessary
- Details regarding enabling EFI mode via BIOS
- Details regarding booting from USB in EFI mode
- Verify Internet connection
- Verifying EFI mode is enabled

PART 1: Disk Partition

- Finding all available drives
- Wiping the existing partition table
- Creating boot partition and the difference between EF00 and EF02 Hex code
- Creating swap partition, the swap debate, choosing a swap size, and the swap 8200 hex code
- Creating root and home, the differences between them, and choosing whether to keep them on the same partition
- Telling linux which file systems to use for our partition

PART 2: Installing Arch and Making it Boot

PART 3: Setup Wifi connection AFTER initial Boot config

PART 4: i3 setup


---

### PART 0: Preparation

- Items necessary
A bootable USB (4GB or bigger) from Arch linux ISO image 

- Details regarding enabling EFI mode via BIOS
Check your motherboard manual to find out if your computer is booted/can boot in UEFI/EFI mode. It will tell you how to enable it in the BIOS.

- Details regarding booting from USB in EFI mode
Rebooting with USB stick in, you have to go to a boot menu during booting. i.e. pressing F12. Select USB that reads something like, "Arch Linux archiso x86_64 UEFI USB". Resulting screen will be,
root@archiso ~ #

- Verify Internet connection
Wired connection will be auth-detected, but if using wifi connection, you want to connect it via `wifi-menu`. This also creates `netctl` profile automatically. Check connection with `ping`. 
ex1) `ping -c 3 google.com`

- Verify EFI mode is enabled
Test if you are using UEFI mode with `efivar -l`. If it lists UEFI variables, you are using UEFI mode.


### PART 1: Disk Partition
  * We want to wipe out a drive and partition for our usage.

- Finding all available drives
In order to find out which drive to wipe out and partition, we need to see complete list of all availalbe drives. `lsblk` command will list them.

ex1)
In *gloriouseggroll* case, he has 3 drives
sda 238.5GB- this is my 256gb ssd drive
–sda1 – Existing Partition on drive
sdb 931.5GB- this is my 1tb storage drive
–sdb1 – Existing Partition on drive
sdc – this is the usb stick im using to install arch
–sdc1 – Existing Partition on drive

These are known to the system as /dev/sda, /dev/sda1, /dev/sdb, /dev/sdb1 and so forth.

ex2) 


- Wiping the existing partition table
Need to wipe out the current partition table so that we can rewrite it as a GPT partition table. **This will WIPE the entire drive**. Command: `gdisk [drive]` 

ex1)
```
gdisk /dev/sdX (x representing your drive. i.e. sda)
x
z
y
y
```





### Thanks to:
- GloriousEggroll : https://www.gloriouseggroll.tv/arch-linux-efi-install-guide/
