# my-Arch-setup

## Preparation

1- **Boot the Live Environment**
- Boot into the Arch Linux ISO from a USB or CD.
- Ensure you are connected to the internet (`ip link` to check interfaces and `iwctl` for Wi-Fi setup if needed).

## SetFont

- **Youcan change size of font**:

```bach
setfont ter-132b
```

- **And if you want to disable it**

```bach
setfont
```


## Partitioning with cfdisk

1- **Launch `cfdisk`**:

```bach
cfdisk /dev/sda
```

When launching `cfdisk` on a new disk or one without a valid partition table, it prompts you to select a **label type**. Here’s how to choose:

- **Label Type Options**:

  1- **GPT (GUID Partition Table)**: Modern and recommended for most systems. Supports:
    - Drives larger than 2TB.
    - More than 4 primary partitions supports up to 128 (on Linux).
    - Compatibility with **UEFI** systems.

  2- **DOS (MBR - Master Boot Record)**: Older partitioning standard, ideal for:
    - Drives smaller than 2TB.
    - Limited to 4 primary partitions unless using extended/logical partitions.
    - Compatibility with **BIOS** systems.

  3- **SGI & SUN**:
    - google is your friend.

**In my case i chose DOS**.

2- **Partitioning Scheme**:

- **Create Primary Partition (`/boot`)**:
  - Create a 500MB primary partition, assign it as `sda1`.
  - Change the partition type to **Linux**.
- **Create Extended Partition**:
  - Create an extended partition with the rest of the disk, assign it as `sda2`.
- **Create LVM Physical Volume (Logical Partition)**:
  - Inside the extended partition, create a logical partition using the remaining space, assign it as `sda5`.
  - Change the partition type to **Linux LVM**.

<div align="center">
  <img width="900" alt="fainllok" src="https://github.com/user-attachments/assets/65cc4506-a809-4b58-a2aa-f0c666df93a2">
</div>

3- **Write and Exit**:

  - Select **Write** to save changes and confirm by typing `yes`, then **Quit**.

- type `lsblk`

<div align="center">
  <img width="491" alt="lsblk_output" src="https://github.com/user-attachments/assets/c05caad6-8935-47c7-bd44-c068565acc80">
</div>

## Setup LVM and LUKS Encryption

1- **Encrypt the LVM Physical Volume**:

  ```bash
  cryptsetup luksFormat /dev/sda5
  ```
  - confirm by typing `YES` not `yse` or `y`.
  - chose passwor
  ```bach
  cryptsetup open /dev/sda5 sda5_crypt
  ```

- type `lsblk`

<div align="center">
  <img width="491" alt="lsblk_output" src="https://github.com/user-attachments/assets/eee85098-5cb0-4f45-9538-a5989f918eee">
</div>

  - **What Happens in `cryptsetup`**?

    1- When you run the command:
    
    ```bash
    cryptsetup luksFormat /dev/sda5
    ```
    
    This:
      - Encrypte your `/dev/sda5` partition useing the LUKS.

    2- When you run the command:
    
    ```bash
    cryptsetup open /dev/sda5 sda5_crypt
    ```
    
    This:
      - Unlocks the LUKS-encrypted partition (`/dev/sda5`).
      - Maps it to a virtual device called `sda5_cryp`.
      - This new virtual device appears as `/dev/mapper/sda5_cryp`.

2- **Create the LVM Physical Volume**: (PV)

  ```bash
  pvcreate /dev/mapper/sda5_cryp
  ```

  - **Understanding `/dev/mapper/sda5_cryp`**:

    The term `/dev/mapper` refers to a directory in Linux that contains device files managed by the **device mapper** subsystem. 
    
    - **Device Mapper**:
  
      - The device mapper is a **kernel module** that provides a way to create and manage **virtual devices**, such as encrypted devices, logical volumes.
      - These devices are accessed through the `/dev/mapper` directory.
     
    - **Why Use `/dev/mapper/cryptlvm` in `pvcreate`?**:

      - The `pvcreate` command initializes a physical volume for use by LVM.
      - Since the actual data is stored in the encrypted partition, you don’t use `/dev/sda5` directly. Instead, you use the decrypted virtual device `/dev/mapper/cryptlvm` created by cryptsetup.
      - LVM works on top of this decrypted device to create flexible logical volumes.

3- **Create the Volume Group**: (VG)

  ```bash
  vgcreate LVMGroup /dev/mapper/sda5_cryp
  ```
  - Create the **Volume Group** useing the **Physical Volume** (`/dev/mapper/sda5_cryp`).

4- **Create Logical Volumes**: (LV)

  ```bash
  lvcreate -L 10G LVMGroup -n root
  lvcreate -L 2.3G LVMGroup -n swap
  lvcreate -L 5G LVMGroup -n home
  lvcreate -L 3G LVMGroup -n var
  lvcreate -L 3G LVMGroup -n srv
  lvcreate -L 3G LVMGroup -n tmp
  ```
  use `vgs` to see the Remaining free space (I hope it's more/equal than `3.99G`)
  ```bach
  lvcreate -L 3.99G LVMGroup -n var-log
  ```
  - `-L`: refers to a saze of LV
  - `LVMGroup`: refers to a place of LV
  - `-n`: refers to a name of LV.


5- **Verify LVM Setup**:

  ```bash
  lvdisplay
  ```
  or
  ```bash
  lsblk
  ```  

- type `lsblk`

<div align="center">
  <img width="491" alt="lsblk_output" src="https://github.com/user-attachments/assets/40ca7a9a-752c-4279-9987-54fba74a77ed">
</div>

## Format Partitions (File System)

1- **Format `/boot`**:

```bash
mkfs.ext4 /dev/sda1
```
- `mkfs`: Make File System.

2- **Format Logical Volumes**:

- `/root`, `/home`, `/var`, `/srv`, `/tmp`, `/var/log` as **ext4**:

  ```bash
  mkfs.ext4 /dev/LVMGroup/root
  mkfs.ext4 /dev/LVMGroup/home
  mkfs.ext4 /dev/LVMGroup/var
  mkfs.ext4 /dev/LVMGroup/srv
  mkfs.ext4 /dev/LVMGroup/tmp
  mkfs.ext4 /dev/LVMGroup/var-log
  ```

- Swap:

  ```bash
  mkswap /dev/LVMGroup/swap
  ```
  - `mkswap`: Make swap space.

- type `lsblk -f`

<div align="center">
  <img width="900" alt="lsblk-f_output" src="https://github.com/user-attachments/assets/107cc63f-4c67-4b4f-b2a4-6c3a8ac8d836">
</div>


## Mount File Systems

1- **Mount Root**:

```bash
mount /dev/LVMGroup/root /mnt
```
- **why `/mnt` and not `/`**

  During the installation of Arch Linux, the root filesystem is mounted to `/mnt` because this is where the new system you are going to install is built. The root is mounted to `/mnt` and then after the installation is complete, the system is set up to boot from `/` when you reboot.

2- **Create and Mount Subdirectories**:

```bash
mkdir /mnt/{boot,home,var,srv,tmp,var}
```
```bash
mount /dev/sda1 /mnt/boot
mount /dev/LVMGroup/home /mnt/home
mount /dev/LVMGroup/var /mnt/var
mount /dev/LVMGroup/srv /mnt/srv
mount /dev/LVMGroup/tmp /mnt/tmp
mkdir /mnt/var/log
mount /dev/LVMGroup/var-log /mnt/var/log
```

3- **Enable Swap**:

```bash
swapon /dev/LVMGroup/swap
```
<div align="center">
  <img width="491" alt="lsblk_output" src="https://github.com/user-attachments/assets/6093480f-da69-4742-96ec-e681c9e53da7">
</div>

## Install Arch Linux

1. **Select Mirrors**:

  1. **Go to [Mirrors Status](https://archlinux.org/mirrors/status/) and chose Mirrors**.
  - **How to chose the best Mirror**:
    1. **Protocol**: `HTTPS`, Because it's safer.
    2. **Country**: Choose servers close to your geographic location.
    3. **Completion**: 100%, This means that the server is fully synchronized with the latest packages.
    4. **μ Delay**: low delay, Lower latency means the server responds faster.
    5. **Mirror Score** The higher the Mirror Score, the better the server performance. You can choose servers with higher ratings.
  - **in my case**
    - `https://mirror.jingk.ai/archlinux/`
    - `https://mirror.its-tps.fr/archlinux/`
  
  2. Edit `nano /etc/pacman.d/mirrorlist` to prioritize the fastest mirrors.
  - add `Server = https://mirror.jingk.ai/archlinux/$repo/os/$arch`
  - add `Server = https://mirror.its-tps.fr/archlinux/$repo/os/$arch`

2. **config pacman.conf**

  - use
  ```bach
  nano /etc/pacman.conf
  ```
  - find `#ParallelDownloads = 5` and uncomment it `ParallelDownloads = 5`
  this allow you to download multiple packages instead of downloading them one by one.

2. **Install Base System**:

```bash
pacstrap /mnt base linux linux-firmware lvm2 networkmanager nano vim efibootmgr cryptsetup
```
Generate fstab:

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

## Configure the System

1. **Chroot into the Installation**:

```bash
arch-chroot /mnt
```

2. **Set Timezone and Locale**:

```bash
ln -sf /usr/share/zoneinfo/Africa/Casablanca /etc/localtime
hwclock --systohc
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

3. **Set Hostname**:

```bash
echo "myhostname" > /etc/hostname
```

4. **Set Hosts File**:

```bash
nano /etc/hosts
```
- and write this:
```csharp
127.0.0.1   localhost
::1         localhost
127.0.1.1   myhostname.localdomain myhostname
```

5. **Recreate Initramfs for LUKS and LVM Support**: Edit `nano /etc/mkinitcpio.conf`:

- Ensure `HOOKS` includes `encrypt`, `lvm2`, next to `block` in this order:
```csharp
HOOKS=(base udev autodetect keyboard keymap modconf block encrypt lvm2 filesystems fsck)
```

6. **Generate initramfs**:

```bash
mkinitcpio -P
```

7. **Set Root Password**:

```bash
passwd
```

8. **Install and Configure Bootloader:** Install GRUB:

```bash
pacman -S grub
```

9. **Install GRUB to the disk**:

```bash
grub-install --target=i386-pc /dev/sda
```

10. **Configure GRUB to unlock LUKS at boot**:

- **get UUID**
```bash
blkid -o value -s UUID /dev/sda5
```
```bash
blkid -o value -s UUID /dev/mapper/LVMGroup-root
```
- Edit `/etc/default/grub` and modify `GRUB_CMDLINE_LINUX`:
<div align="center">
  <img width="900" alt="lsblk_output" src="https://github.com/user-attachments/assets/31600665-4f82-4932-8a22-22023175711d">
</div>

```makefile
GRUB_CMDLINE_LINUX="cryptdevice=/dev/sda5:cryptlvm root=/dev/LVMGroup/root"
```

- Generate GRUB config:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

11. ****
```bash
systemctl enable NetworkManager
```

## Final Steps

1. **Exit Chroot and Reboot**:

```bash
exit
umount -R /mnt
swapoff -a
reboot
```
2. **Verify Installation**: After rebooting, you’ll be prompted to enter your LUKS passphrase to unlock the encrypted disk and boot into Arch Linux.






