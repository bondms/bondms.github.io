---
title: Linux disk management
---

# Linux disk management

## Partitioning and formatting

For an encrypted partition, see "Full-disk encryption" below. For non-encrypted paritions see these examples:

Example partition table:

Partition 1 is to be encypted, the remaining three are to bo non-encrypted as per these examples.

```
Disk /dev/sdb: 931.51 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: PSSD T9
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 2097152 bytes
Disklabel type: dos
Disk identifier: 0x534ecb55

Device     Boot      Start        End   Sectors   Size Id Type
/dev/sdb1             4096  650121215 650117120   310G 83 Linux
/dev/sdb2        650121216 1300238335 650117120   310G 83 Linux
/dev/sdb3       1300238336 1308626943   8388608     4G  b W95 FAT32
/dev/sdb4       1308626944 1953525167 644898224 307.5G  7 HPFS/NTFS/exFAT
```

### Linux

Example: Create an ext4 formatted partition with volume label "Linux":

1. Use `fdisk` to create a partition of type 0x83 (Linux).
1. `sudo mkfs.ext4 -L Linux /dev/sdb2`

### Cross-platform compatibile

vFAT is highly cross-platform compatible but is limited to a maximum size of 4 GiB. The volume lavel is limited to 11 upper-case characters.

Example: Create a VFAT formatted partition with volume label "DOS".

1. Use `fdisk` to create a partition of type 0x0B (W95 FAT32).
1. `sudo mkfs.vfat -n DOS /dev/sdb3`

If a partition size greater than 4 GiB is required then exFAT can be used. It is still cross-platform compatible, but some older or simpler devices may not support it, e.g.

1. Use `fdisk` to create a parition of type 0x07 (HPFS/NTFS/exFAT).
1. `sudo mkfs.exfat -L Windows /dev/sdb4`

## Erasing data

Erasing the data from a disk before performing a fresh OS installation can be convenient. It ensures, for example, that no remnants of data (such as a bootloader) will affect the new installation. It will be truely "fresh".

Booting from a "Live" Linux distribution is helpful in this case. It allows the disk to be accessed without being in use.

### Erasing magnetic disks

A traditional magnetic disk can be erased from a terminal using commands such as
```bash
cat /dev/zero > /dev/sda
```
or
```bash
dd if=/dev/zero of=/dev/sda
```
where /dev/sda is the device of the disk to be erased.

Erasing an entire disk may take a long time. Often, it is sufficient to erase just the first few sectors. In this case the command can be cancelled with `ctrl-c` once it has progressed far enough or limited from the start, e.g.
```bash
dd if=/dev/zero of=/dev/sda bs=4096 count=1024
```

### Erasing SSDs

For modern solid-state disks, there is a better way. Using `blkdiscard` offers several advantages:
* It is quicker.
* It does not wear the disk by writing to every sector.
* It informs the disk that the sectors are no longer in use allowing the disk to make use of them (e.g. for wear-levelling).

```bash
blkdiscard /dev/sda
```

### Partitions

Both of the above commands work just as well on a single partition as on a full disk, e.g.

```bash
cat /dev/zero > /dev/sda1
```
```bash
dd if=/dev/zero of=/dev/sda1
```
```bash
blkdiscard /dev/sda1
```

### Filesystem

The `fstrim` command is used to discard blocks from a provisioned drive formatted with a filesystem. Use that instead to discard erased data but to keep active files, e.g.

```bash
fstrim -v /media/user/volume/
```

## Secure wiping

If the disk is being erased because it contains confidential or personal data which must be removed then a more through wipe may be desired.

### Secure wiping magnetic disks

For magnetic disks, writing random data rather than zeros may make data recovery more difficult, e.g.
```bash
dd if=/dev/urandom of=/dev/sda
```
Be sure to allow the command to run to completion in this case and consider running multiple passes.
> Warning: This still does not guarantee full erasure. Data may still exist is sectors that have been swapped out of use because the drive's internal error detection found them to be no longer fully reliable.

### Secure wiping SSDs

For solid-state disks, the `blkdiscard` command offers an option for a secure discard, but it does require support from the device.
```bash
blkdiscard --secure /dev/sda1
```

## Full-disk encryption

With modern hardware, use of full-disk encryption doesn't incur a noticeable performance cost.

In addition to offering increased security if the disk is lost or stolen, full-disk encryption also makes secure wiping easier since only the key needs to be erased to render the remaining data no more useful than random noise.

Most modern distribtions support full-disk encryption for internal drives out-of-the-box, but some setup may be required for removable drives.

### Removable flash

Modern distributions support full-disk encryption on removable flash drives such as USB pen drives or SD cards. For example, a password prompt may appear when the drive is inserted and the data will not be accessible until the password is provided.

For this to work, the drive may be encrypted as follows:

1. Partition as usual, e.g. `sudo fdisk /dev/sdb`
1. Format using LUKS, e.g. `sudo luksformat -t ext4 /dev/sdb1 -L SecureDisk`

Where:
* /dev/sdb is the disk device. For SD cards is will likely be something like /dev/mmcblk0.
* /dev/sdb1 is the partition. For SD cards is will likely be something like /dev/mmcblk0p1.
* ext4 is the type of filesystem.
* SecureDisk is the volume label.
