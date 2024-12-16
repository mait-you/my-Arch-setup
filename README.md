# my-Arch-setup

## Preparation

1- **Boot the Live Environment**
- Boot into the Arch Linux ISO from a USB or CD.
- Ensure you are connected to the internet (`ip link` to check interfaces and `iwctl` for Wi-Fi setup if needed).

2- **Synchronize Time**:
```bach
timedatectl set-ntp true
```

## Partitioning with cfdisk

1- **Launch cfdisk**:

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

- **Create `/boot` (Primary Partition)**:
  - Create a 500MB primary partition, assign it as `sda1`.
  - Change the partition type to **Linux**.
- **Create Extended Partition**:
  - Create an extended partition with the rest of the disk, assign it as `sda2`.
- **Create LVM Physical Volume (Logical Partition)**:
  - Inside the extended partition, create a logical partition using the remaining space, assign it as `sda5`.
  - Change the partition type to **Linux LVM**.

![image](https://github.com/user-attachments/assets/65cc4506-a809-4b58-a2aa-f0c666df93a2)

3- **Write and Exit**:

  - Select **Write** to save changes and confirm by typing `yes`, then **Quit**.

- type `lsblk`

div align="center">
  <img width="491" alt="lsblk_output" src="[https://github.com/user-attachments/assets/75e871dc-a40c-4348-8095-d8c51f1e05ce](https://github.com/user-attachments/assets/c05caad6-8935-47c7-bd44-c068565acc80)">
</div>





## Setup LVM and LUKS Encryption










