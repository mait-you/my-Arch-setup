# my-Arch-setup

## Preparation

1- **Boot the Live Environment**
- Boot into the Arch Linux ISO from a USB or CD.
- Ensure you are connected to the internet (`ip link` to check interfaces and `iwctl` for Wi-Fi setup if needed).

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
  lvcreate -L 4G LVMGroup -n var-log
  ```
  - `-L`: refers to a saze of LV
  - `LVMGroup`: refers to a place of LV
  - `-n`: refers to a name of LV.

5- **Verify LVM Setup**:

  ```bash
  lvdisplay
  ```







