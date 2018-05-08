# Archlinux EFI Installation Procedure

## Table of Contents

PART 0: Preparation

- Items necessary
- Details regarding enabling EFI mode via BIOS
- Details regarding booting from USB in EFI mode
- Verify Internet connection
- Verify the Boot mode (if EFI mode is enabled)

PART 1: Disk Partition

- Finding all available drives
- Wiping the existing partition table
- Create boot partition
- Creating swap partition
- Creating root and home
- Telling linux which file systems to use for our partition

PART 2: Configure the system or Installing Arch and Making it Boot

PART 3: Setup Wifi connection AFTER initial Boot config

PART 4: i3 setup


---

## PART 0: Preparation

### Items necessary
A bootable USB (4GB or bigger) from Arch linux ISO image 

### Details regarding enabling EFI mode via BIOS
Check your motherboard manual to find out if your computer is booted/can boot in UEFI/EFI mode. It will tell you how to enable it in the BIOS.

### Details regarding booting from USB in EFI mode
Rebooting with USB stick in, you have to go to a boot menu during booting. i.e. pressing F12. Select USB that reads something like, "Arch Linux archiso x86_64 UEFI USB". Resulting screen will be,
```root@archiso ~ #```

### Verify Internet connection
Wired connection will be auth-detected, but if using wifi connection, you want to connect it via `wifi-menu`. This also creates `netctl` profile automatically: 
```
$ wifi-menu
```
Check connection with `ping`
```
$ ping -c 3 google.com // pings google.com 3 times
```

### Verify the boot mode (if EFI mode is enabled).
If UEFI mode is enabled on an UEFI motherboard, Archiso will boot Arch Linux accordingly via systemd-boot. To verify this, list the *efivars* directory:
```
# ls /sys/firmware/efi/efivars
```
If the directory does not exist, the system may be booted in BIOS or CSM mode. Refer to your motherboard's manual for details. 
Also, you can use:
```
# efivar -l
```
If it lists UEFI variables, you are using UEFI mode:

## PART 1: Disk Partition
  * We want to wipe out a drive and partition for our usage.

### Finding all available drives
In order to find out which drive to wipe out and partition, we need to see complete list of all availalbe drives. `lsblk`(List Block Devices) will list them.
```
# lsblk 
// or
# fdisk -l
```

ex)
In *gloriouseggroll* case, he has 3 drives
sda 238.5GB- this is my 256gb ssd drive
–sda1 – Existing Partition on drive
sdb 931.5GB- this is my 1tb storage drive
–sdb1 – Existing Partition on drive
sdc – this is the usb stick im using to install arch
–sdc1 – Existing Partition on drive

These are known to the system as /dev/sda, /dev/sda1, /dev/sdb, /dev/sdb1 and so forth.

ex) 

### Wiping the existing partition table
Need to wipe out the current partition table so that we can rewrite it as a GPT partition table. **This will WIPE the entire drive**. Command: `gdisk [drive]` 

ex)
```
gdisk /dev/sdX (x representing your drive. i.e. sda)
x
z
y
y
```

### Create boot partition
Start partitioning:
```
$ cgdisk /dev/sdX
```
The order you create partitions is the order they will be listed in. 

ex) If I create three partitions in order of boot, swap, and root + home on /dev/sda, and then `lsblk` will give:
```
sda
- sda1 (boot partition)
- sda2 (swap partition)
- sda3 (root + home partition)
```

First of all, create boot partition. Since we are using EFI, EF00 will be our hex code, not EF02. Dedicate 1GiB (1024MiB) of space to the boot sector to have some room to breathe in case we want to change anything or add multiple boot kernels, although arch wiki recommends only 200~300Mb.

ex)
```
Select [New] -> press Enter
First Sector: (Leave this blank) -> press Enter
Size in sector: 1024MiB -> press Enter
Hex Code: EF00 -> press Enter
Enter new partition name: boot -> press Enter
```

Note there is 1007KiB existing before boot partition we just created. This is where Protective MBR is, and is present on all GPT partition tables. This cannot be removed/deleted. It is OK to ignore.

### Create swap partition

Always create a swap partition. The way Linux kernel works, swap isn't only used when you have exhuasted all physical memory. The Linux kernel will take applications that are not active, or sleeping, and after a period of time, move the application to swap from real memory. The result is when you need that application, there will be a momentary delay (usually a second or two) while the application's memory is read back from swap to RAM

If you are running a laptop or a desktop that you might want to put in 'hibernate' mode (suspend to disk), then you always want at least as much swap as you have memory. Swap space will be used to store the contents of the RAM in the computer while it 'sleeps'. Additionally, this allows you to put inactive applications to "sleep", giving your active applications access to additional RAM.

In the unlikely event that you ran our of RAM - perhaps opening a big file, perhaps a long running tab in firefox, it doesn't matter, in that event your kernel OOM killer will kick in and start killing applications to get memory back. From a developer's perspective, you also need to have substantial swap space if you are running Java/Java apps.

Reference:
- GloriousEggroll : https://www.gloriouseggroll.tv/arch-linux-efi-install-guide/
- http://serverfault.com/questions/5841/how-much-swap-space-on-a-high-memory-system
- http://askubuntu.com/a/49130

Now, how big should swap partition be? Recommended as following:
| Amount of RAM      | Recommended swap space      | Recommended swap space if allowing for hibernation|
| ------------------ | --------------------------- | --------------------------------------------------|
| 2GB or RAM or less | 2 times the amount of RAM   | 3 times the amount of RAM                         |
| 2GB to 8GB         | Equal to the amount of RAM  | 2 times the amount of RAM                         |
| 8GB to 64GB        | 0.5 times the amount of RAM | 1.5 times the amount of RAM                       |
| 64GB or more       | 4GB of swap space           | No extra space needed                             |

ex)
```
Select [New] -> press Enter
First Sector: (Leave this blank) -> press Enter
Size in sector: 8GiB -> press Enter
Hex Code: 8200  -> press Enter
Enter new partition name: swap -> press Enter
```

### Create root and home
Even though it's usually the case where you keep /home inside root partition, you can separate root and home. Difference is that, in comparison to windows, Root is like your C: drive. Generally you don't want to mess with anything inside unless you know what you're doing. Home is where your user files are stored. In windows that would be C:\Users\someusername, which would then contain My Documents, Downloads, Pictures, etc. 


ex)
```
Select [New]
First Sector: (Leave this blank) -> press Enter
Size in sector: (Leave this blank) -> press Enter 
Hex Code: (Leave this blank) -> press Enter 
Enter new partition name: root -> press Enter 
```

Arrow over to [Write] to save your new partitions, hit enter, type "yes", hit enter again. And then [Quit].


### Telling linux which file systems to use for our partition

We now need to let linux know the file system for our partitions. For EFI with GPT, boot needs to be Fat32. For swap we simply use mkswap. The rest are default ext4 file systems:

ex)
```
mkfs.fat -F32 /dev/sda1
mkswap /dev/sda2
swapon /dev/sda2
mkfs.ext4 /dev/sda3
```




### Thanks to:
- GloriousEggroll : https://www.gloriouseggroll.tv/arch-linux-efi-install-guide/
- Archlinux installation guide : https://wiki.archlinux.org/index.php/Installation_guide
