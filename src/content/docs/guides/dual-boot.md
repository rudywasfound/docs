---
title: Dual boot
description: How do do a dual boot with Windows (or any other OS)
---

# Dual Boot Guide: Windows + AxOS

How to do a full **dual boot installation** with **Windows 11** and **AxOS** (Windows 10 should work as well).

> **System Requirements**: [Check here](https://www.axos-project.com/docs/get-started/installation/#required)
> Minimum disk space: **10 GB** (but **50 GB or more is strongly recommended** for a smoother experience)

## Step 0: Before You Begin

> I will **skip the steps** for flashing AxOS to a USB. Please make sure you’ve already created a **bootable USB** by following [this guide](https://www.axos-project.com/docs/get-started/installation/#create-the-installation-media).

## Step 1: Create Free Space on Your Disk

We need to shrink an existing partition to make room for AxOS.

### Windows

1. Right click on the Windows Icon -> choose Disk management from the menu.
2. In the Disk Management window:
   * Right-click on a partition with enough free space (e.g. your D: drive)
   * Click **"Shrink Volume"**
   * Enter how much you want to shrink (in MB). For example, `50000` for 50 GB (Note: This is the minimum required space to install AxOS. You can increase it as much as you like.)
   * Click **"Shrink"**

This will create **unallocated space** which we’ll use to install AxOS.

### Linux

1. Open a terminal.
2. Type in `lsblk -f` to see all the partitions.
3. In the `lsblk -f` output:
   * Decide what partition you want to shrink.
   * You can choose the root partition (/) to make room for AxOS.
4. You’ll need to shrink the partition using a Live USB and a tool like **GParted**.

→ Don’t worry, we cover this in more detail in [Step 4](#step-4-create-partitions-with-gdisk).


## Step 2: Boot into AxOS from USB

1. Reboot your computer
2. Enter your **BIOS/UEFI menu**
3. Select your **bootable USB** device
4. Boot to AxOS


## Step 3: Verify Free Space in AxOS
> **Note:** Take note of your disk’s name. It could be something like:  
> - `/dev/sda`  
> - `/dev/nvme0n1`  
>
> We’ll use this name in the next steps.  
> If you’re not sure which disk is which:  
> 1. Open a terminal (`Windows key + Enter`)  
> 2. Type `lsblk` and press Enter  
> 3. Look for your main disk (`/dev/sda` or `/dev/nvme0n1`)  
>
>  **Important:** Do **not** modify `/dev/sdb` or your USB drive. Only partition your main disk.  

Once inside the live AxOS, open the terminal (`Win (Mod) + Enter (Return)`) and run:

```bash
sudo cfdisk /dev/<your disk. i.e - /dev/sda or /dev/nvme0n1>
```

This will list all available disks and their partitions.

Look for something like:

```
Free Space with Size type 50GB
```

Or run:

```bash
sudo parted -l | grep "Unallocated"
```


## Step 4: Create Partitions with `cfdisk`

Why `cfdisk`? This is because `cfdisk` is the lightest and the most beginner-friendly tool to manually partition your disks , you can alternatively use `gparted`, `parted`, etc.

We’ll create **three partitions**:

1. **EFI System** (`/boot/efi`) - required for UEFI boot
2. **Linux Root** (`/`) - where AxOS will be installed
3. **Swap** (Optional) - used as extra memory when RAM is full (recommended size = 1–2× your RAM)

First, open `cfdisk` (replace `yourdiskname` with your disk, e.g. `/dev/sda` or `/dev/nvme0n1`):

```bash
sudo cfdisk /dev/yourdiskname
```

---

### Inside `cfdisk`

Use the **arrow keys** to move and **Enter** to confirm.

#### 1. EFI Partition

1. Select **Free space** → choose **New**.
2. Enter `512M` for size.
3. Highlight the new partition → choose **Type** → select **EFI System**.

EFI partition ready.

---

#### 2. Root Partition (`/`)

1. Select remaining free space → choose **New**.
2. Enter a size (e.g. `50G`) or use all remaining space except what you want for Swap.
3. Highlight the new partition -> choose **Type** -> select **Linux filesystem**.

Root partition ready.

---

#### 3. Swap Partition (Optional)

1. Select leftover free space → choose **New**.
2. Enter the desired size (e.g. `8G`).
3. Highlight it -> choose **Type** -> select **Linux swap**.

Swap partition ready.

---

### Save and Exit

1. Choose **Write** → type `yes` to confirm.
2. Then select **Quit**.

---

### Partition Naming

The partition names depend on your disk type:

* **SATA disks:** `/dev/sda1`, `/dev/sda2`, `/dev/sda3`
* **NVMe disks:** `/dev/nvme0n1p1`, `/dev/nvme0n1p2`, `/dev/nvme0n1p3`
  *(Notice the **“p”** before the number on NVMe.)*

You now have:

* `/dev/...1` -> EFI (512 MB or more)
* `/dev/...2` -> Root (Linux `/`)
* `/dev/...3` -> Swap (Optional)


## Step 5: Mount the Partitions

We’ll now mount and prepare the partitions for installation.
Adjust disk and partition names as needed (`/dev/sdaX` or `/dev/nvme0n1pX`).

---

### 1. Mount the Root Partition (Linux filesystem)

```bash
sudo mount /dev/nvme0n1p2 /mnt
```

---

### 2. Create and Mount the EFI Directory

```bash
sudo mkdir -p /mnt/boot/efi
sudo mount /dev/nvme0n1p1 /mnt/boot/efi
```

---

### 3. Enable the Swap Partition (Optional)

First, format it as swap:

```bash
sudo mkswap /dev/nvme0n1p3
```

Then activate it:

```bash
sudo swapon /dev/nvme0n1p3
```


Now we’re ready to launch the installer.

---

## Step 6: Launch the AxOS Installer

1. Open the **AxOS Install** application from the menu.
2. Proceed through the installation.
3. When you get to **"Installation disk and partitioning"**, choose **Manual Partitioning**.

### Configure the Mount Points:

* For the **EFI partition**:

  * Set **FAT32** format
  * Set mount point to `/boot/efi`

* For the **Linux root partition**:

  * Set **ext4** format (ext4 is an example, but it's also recommended)
  * Set mount point to `/`

4. Continue through the installer
5. Once installation finishes, reboot

> **Adjust disk and partition names and numbers as needed.**


## Step 8: Configure GRUB to Detect Windows

After rebooting into AxOS:

### 1. Open a terminal and edit the GRUB config:

```bash
sudo nano /etc/default/grub
```

### 2. Find this line:

```bash
GRUB_DISABLE_OS_PROBER
```

Change it to:

```bash
GRUB_DISABLE_OS_PROBER=false
```

> **Save and Exit Nano**:
>
> * Press `Ctrl + O` (to save)
> * Press `Enter` (to confirm filename)
> * Press `Ctrl + X` (to exit)

### 3. Detect Windows:

```bash
sudo os-prober
```

If it returns your Windows installation, proceed.


## Step 9: Generate GRUB Config

Choose the appropriate command depending on your system:

### BIOS Systems:

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### UEFI Systems:

```bash
sudo grub-mkconfig -o /boot/efi/EFI/grub/grub.cfg
```

> ⚠️ If you are not sure what to choost stick to BIOS System.


## Final Step: Reboot

Now reboot your system:

```bash
sudo reboot now
```

You should now see the **GRUB boot menu**, with both **AxOS** and **Windows** listed.

---

## Done
