## Creating a Virtual SD Card for Embedded Linux Development

# To Create a Virtual SD Card for Embedded Linux Development, you can follow these steps:
1. **Create a Disk Image**: Use the `dd` command to create a disk image file that will serve as your virtual SD card. For example, to create a 1GB image:
   ```bash
   dd if=/dev/zero of=virtual_sd_card.img bs=1M count=1024
   ```

```markdown
![Virtual SD Card Creation Process](./images/sd_card_creation.png)
```

2. **Formatting and Partitioning**: Using `cfdisk` 
For example , to partition the disk image into Two Partitions, you can use the following command and then manually create the partitions:
   ```bash
   cfdisk virtual_sd_card.img
   ```
   2.1 In the `cfdisk` interface:
    - Select Label Type `dos` for MPR.
    - Create a new partition for the Raspi bootloader (e.g., 200MB) and set it as primary and 6.FAT16 Type and Select Bootable.
    - Create another partition for the root filesystem (e.g., 800MB) and set it as primary and 83 Linux Type (ext4).

```markdown
![Virtual SD Card Formatting and Partitioning](./images/sd_card_partitioning.png)
```

3. **Attach the Disk Image**: Use the `losetup` command to attach the disk image to a loop device:
   ```bash
   sudo losetup -fP virtual_sd_card.img
   ```
   -f -> Find the first available loop device.
   -P -> Automatically create loop devices for each partition.

    This will create loop devices for each partition (e.g., `/dev/loop0p1` for the boot partition and `/dev/loop0p2` for the root filesystem).

    Use `lsblk` to verify the loop devices and know the exact device names:
    ```bash
    lsblk
    ```

```markdown
![Virtual SD Card as Loop Device](./images/sd_card_loop_device.png)
```


4. **Format the Partitions**: Format the partitions using the appropriate file systems:
    select the loop devices for each partition and format them accordingly. For example:
   ```bash
    sudo mkfs.vfat -F16 -n BOOT /dev/loop0p1  # Format the boot partition
    sudo mkfs.ext4 -L FILESYS /dev/loop0p2  # Format the root filesystem partition
    ```
    Use `lsblk -f` again to verify the partitions are formatted correctly:
    ```bash
    lsblk -f
    ```

```markdown
![Partitions Formatted](./images/sd_card_formatted.png)
```

5. **Mount the Partitions**: Create mount points and mount the partitions to copy files:
   ```bash
   mkdir -p /mnt/boot
   mkdir -p /mnt/rootfs
   sudo mount /dev/loop0p1 /mnt/boot
   sudo mount /dev/loop0p2 /mnt/rootfs

   ```
```markdown
![Partitions Mounted](./images/mount_points.png)
```