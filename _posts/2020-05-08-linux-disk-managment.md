---
title: Linux disk management
---

# Linux disk management

## Erasing data

Erasing the data from a disk before performing a fresh OS installation can be convenient. It ensures, for example, that no remnants of data (such as a bootloader) will affect the new installation. It will be truely "fresh".

Booting from a "Live" Linux distribution is helpful in this case. It allows the disk to be accessed without being in use.

### Magnetic disks

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

### SSDs

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

## Secure wiping

If the disk is being erased because it contains confidential or personal data which must be removed then a more through wipe may be desired.

### Magnetic disk

For magnetic disks, writing random data rather than zeros may make data recovery more difficult, e.g.
```bash
dd if=/dev/urandom of=/dev/sda
```
Be sure to allow the command to run to completion in this case and consider running multiple passes.
> Warning: This still does not guarantee full erasure. Data may still exist is sectors that have been swapped out of use because the drive's internal error detection found them to be no longer fully reliable.

### SSDs

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
