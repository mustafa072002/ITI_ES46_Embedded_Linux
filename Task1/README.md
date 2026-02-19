
<details>
<summary><b>6. What is the difference between a block device and a character device?</b></summary>

<table>
   <tr>
      <th style="text-align:left">Aspect</th>
      <th style="text-align:left">Block Device</th>
      <th style="text-align:left">Character Device</th>
   </tr>
   <tr>
      <td>Data Access</td>
      <td>Transfers data in fixed-size blocks (e.g., 512 bytes, 4KB)</td>
      <td>Transfers data as a stream of bytes (no block size)</td>
   </tr>
   <tr>
      <td>Examples</td>
      <td>Hard drives, SSDs, USB drives, loop devices</td>
      <td>Keyboards, mice, serial ports, terminals</td>
   </tr>
   <tr>
      <td>Buffering</td>
      <td>Buffered by the kernel for random access</td>
      <td>Unbuffered or line-buffered, sequential access</td>
   </tr>
   <tr>
      <td>Device Files</td>
      <td>Located in <code>/dev</code> as <code>sd*</code>, <code>loop*</code>, etc.</td>
      <td>Located in <code>/dev</code> as <code>tty*</code>, <code>serial*</code>, etc.</td>
   </tr>
   <tr>
      <td>Usage</td>
      <td>Used for filesystems, random access storage</td>
      <td>Used for direct device communication, streaming data</td>
   </tr>
</table>

<br>
<b>Summary:</b>
<ul>
   <li><b>Block devices</b> are for storage and filesystems, supporting random access in blocks.</li>
   <li><b>Character devices</b> are for byte-stream devices, supporting sequential access.</li>
</ul>

</details>
<div align="center">
   <h1>Virtual SD Card for Embedded Linux Development</h1>
</div>

---

### Step-by-Step Guide
1. **Create a Disk Image**
    - Use the `dd` command to create a disk image file that will serve as your virtual SD card.
    - Example (1GB image):
       ```bash
       dd if=/dev/zero of=virtual_sd_card.img bs=1M count=1024
       ```
    <div align="center">
       <img src="./images/sd_card_creation.png" alt="Virtual SD Card Creation Process" width="600"/>
    </div>

2. **Formatting and Partitioning**
    - Use `cfdisk` to partition the disk image into two partitions:
       ```bash
       cfdisk virtual_sd_card.img
       ```
    - In the `cfdisk` interface:
       - Select Label Type `dos` for MPR.
       - Create a new partition for the Raspi bootloader (e.g., 200MB, primary, 6.FAT16, Bootable).
       - Create another partition for the root filesystem (e.g., 800MB, primary, 83 Linux Type ext4).
    <div align="center">
       <img src="./images/sd_card_partitioning.png" alt="Virtual SD Card Partitioning" width="600"/>
    </div>

3. **Attach the Disk Image**
    - Use the `losetup` command to attach the disk image to a loop device:
       ```bash
       sudo losetup -fP virtual_sd_card.img
       ```
       - `-f`: Find the first available loop device.
       - `-P`: Automatically create loop devices for each partition.
    - This will create loop devices for each partition (e.g., `/dev/loop0p1` for the boot partition and `/dev/loop0p2` for the root filesystem).
    - Use `lsblk` to verify the loop devices:
       ```bash
       lsblk
       ```
    <div align="center">
       <img src="./images/sd_card_loop_device.png" alt="Virtual SD Card as Loop Device" width="600"/>
    </div>


4. **Format the Partitions**
    - Select the loop devices for each partition and format them accordingly:
       ```bash
       sudo mkfs.vfat -F16 -n BOOT /dev/loop0p1  # Format the boot partition
       sudo mkfs.ext4 -L FILESYS /dev/loop0p2    # Format the root filesystem partition
       ```
    - Use `lsblk -f` again to verify the partitions are formatted correctly:
       ```bash
       lsblk -f
       ```
    <div align="center">
       <img src="./images/sd_card_formatted.png" alt="Partitions Formatted" width="600"/>
    </div>

5. **Mount the Partitions**
    - Create mount points and mount the partitions to copy files:
       ```bash
       mkdir -p /mnt/boot
       mkdir -p /mnt/rootfs
       sudo mount /dev/loop0p1 /mnt/boot
       sudo mount /dev/loop0p2 /mnt/rootfs
       ```
    <div align="center">
       <img src="./images/mount_points.png" alt="Partitions Mounted" width="600"/>
    </div>

---


<div align="center">
    <h1>Questions And Answers</h1>
</div>

---

<details>
<summary><b>1. What is the difference between the DOS/MBR and GPT partition scheme/type?</b></summary>

<table>
   <tr>
      <th style="text-align:left">Feature</th>
      <th style="text-align:left">DOS/MBR (Master Boot Record)</th>
      <th style="text-align:left">GPT (GUID Partition Table)</th>
   </tr>
   <tr>
      <td>Partition Limit</td>
      <td>Up to 4 primary partitions (or 3 primary + 1 extended with logicals)</td>
      <td>Up to 128 partitions (no extended/logical concept)</td>
   </tr>
   <tr>
      <td>Disk Size Support</td>
      <td>Up to 2 TB</td>
      <td>Up to 9.4 ZB (zettabytes)</td>
   </tr>
   <tr>
      <td>Boot Data Location</td>
      <td>First 512 bytes (MBR sector)</td>
      <td>Multiple locations (protective MBR, GPT header, partition table)</td>
   </tr>
   <tr>
      <td>Data Redundancy</td>
      <td>No redundancy</td>
      <td>Backup GPT header and partition table at end of disk</td>
   </tr>
   <tr>
      <td>Compatibility</td>
      <td>Older BIOS systems</td>
      <td>Modern UEFI systems</td>
   </tr>
   <tr>
      <td>Corruption Recovery</td>
      <td>Limited</td>
      <td>Better (backup header/table)</td>
   </tr>
   <tr>
      <td>Partition Identification</td>
      <td>Type codes</td>
      <td>GUIDs (Globally Unique Identifiers)</td>
   </tr>
</table>

<br>
<b>Summary:</b> <br>
<ul>
   <li><b>MBR</b> is older, limited to 4 primary partitions and 2TB disks, used in legacy BIOS systems.</li>
   <li><b>GPT</b> is newer, supports more partitions and larger disks, offers redundancy and is required for UEFI systems.</li>
</ul>

</details>

<details>
<summary><b>2. What is the difference between different File systems and their usage (FAT16, FAT32, EXT4)?</b></summary>

<table>
   <tr>
      <th style="text-align:left">File System</th>
      <th style="text-align:left">Description</th>
      <th style="text-align:left">Usage</th>
   </tr>
   <tr>
      <td><b>FAT16</b></td>
      <td>Older file system, supports up to 2GB partitions, simple structure, limited features.</td>
      <td>Used in small storage devices, boot partitions, and legacy systems.</td>
   </tr>
   <tr>
      <td><b>FAT32</b></td>
      <td>Improved FAT version, supports up to 2TB partitions, larger file support, more efficient allocation.</td>
      <td>Common in USB drives, SD cards, and compatibility across many OSes.</td>
   </tr>
   <tr>
      <td><b>EXT4</b></td>
      <td>Modern Linux file system, supports very large partitions and files, journaling, high performance and reliability.</td>
      <td>Default for most Linux distributions, used for root and data partitions.</td>
   </tr>
</table>

<br>
<b>Summary:</b>
<ul>
   <li><b>FAT16</b>: Simple, legacy, small devices and boot partitions.</li>
   <li><b>FAT32</b>: Larger devices, cross-platform, removable media.</li>
   <li><b>EXT4</b>: Advanced, robust, Linux systems, large files and partitions.</li>
</ul>

</details>

<details>
<summary><b>3. What is a Loop Device and why does Linux use them?</b></summary>

<b>Definition:</b> <br>
A loop device is a pseudo-device in Linux that allows a file to be mounted and accessed as a block device (like a disk or partition). This is useful for mounting disk images, ISO files, or creating virtual disks for testing and development.

<b>Why Linux uses them:</b>
<ul>
   <li>To mount disk images as if they were real disks.</li>
   <li>For testing, development, and embedded systems.</li>
   <li>To access filesystems inside image files without burning them to physical media.</li>
</ul>

<b>Common Commands:</b>
<ol>
   <li><b>Create a loop device:</b><br>
         <code>sudo losetup -fP &lt;image_file&gt;</code>
   </li>
   <li><b>List all loop devices currently in use:</b><br>
         <code>losetup -a</code> <br>or<br> <code>lsblk | grep loop</code>
   </li>
   <li><b>Detach a (mounted) loop device:</b><br>
         <code>sudo losetup -d &lt;loop_device&gt;</code>
   </li>
</ol>
</details>

<details>
<summary><b>4. How can you check the current loop device limit?</b></summary>

<b>Command:</b><br>
<code>cat /sys/module/loop/parameters/max_loop</code>
<br>
This file shows the maximum number of loop devices allowed by the kernel.

</details>

<details>
<summary><b>5. Can you expand the number of loop devices in Linux?</b></summary>

Yes, you can increase the number of loop devices by changing the kernel parameter.<br>
<b>Command:</b><br>
<code>sudo sh -c 'echo &lt;new_limit&gt; &gt; /sys/module/loop/parameters/max_loop'</code>
<br>
Replace <code>&lt;new_limit&gt;</code> with the desired number (e.g., 128).<br>
For permanent changes, add <code>dev.loop.max_loop=&lt;new_limit&gt;</code> to your kernel boot parameters or set it in <code>/etc/sysctl.conf</code>.

</details>



---

<div align="center">
   <sub>Created by Mustafa Mahgoub &copy; 2026</sub>
</div>