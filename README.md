# Backup-Dual-Boot
## Part 1 — Full, Bootable Windows Backup (Veeam Agent)

This guide shows how to create a **bare-metal, bootable image** of a Windows PC (all disks/partitions, apps, settings) using **Veeam Agent for Microsoft Windows**, plus how to make and test the **Recovery Media** you’ll boot to restore it.

> Outcome: one (or more) `.vbk` backup files on your external SSD **and** a small bootable USB (or partition) you can use to restore the image to the same PC or different hardware.

---

### Table of contents

* [Before you start (safety checklist)](#before-you-start-safety-checklist)
* [Install Veeam Agent](#install-veeam-agent)
* [Create the backup job (entire computer)](#create-the-backup-job-entire-computer)
* [Optional : Restore a backup and verify](#optional--restore-a-backup-and-verify)
* [Create Recovery Media (bootable)](#create-recovery-media-bootable)
* [Test the Recovery Media](#test-the-recovery-media)
* [Space planning (how big will the backup be?)](#space-planning-how-big-will-the-backup-be)
* [Troubleshooting & tips](#troubleshooting--tips)
* [Appendix: useful commands](#appendix-useful-commands)

---

### Before you start (safety checklist)

1. **Suspend BitLocker (if enabled)**
   Control Panel → BitLocker Drive Encryption → **Suspend protection** (don’t decrypt). 
   Also note down your bitlocker key. 
   You can resume after everything is tested.

2. **Turn off hibernation & Fast Startup** (prevents “locked” NTFS):

   ```powershell
   powercfg /h off
   ```

   Control Panel → Power Options → “Choose what the power buttons do” → uncheck **Turn on fast startup** and **hibernate**.

3. Optional : **Quick health check** (optional but smart):

   ```powershell
   chkdsk C: /scan
   sfc /scannow
   ```

4. Optional : **Free space**: empty temp folders, browser caches, recycle bin. (Smaller backup.)

---

### Install Veeam Agent

1. Download **Veeam Agent for Microsoft Windows** (Free or higher).
2. Run the installer → accept defaults.
3. Launch **Veeam Agent** from Start Menu.

---

### Create the backup job (entire computer)

#### Option A (recommended): *Entire computer*

* In Veeam → **Configure backup** (or **Add New Job**).
* **Backup mode:** **Entire computer** (captures all partitions needed to boot).
* **Destination:** **Local storage** → choose a folder on your **external SSD**.
* **Storage:** leave **Compression = Optimal**; check **Encrypt backup** if you want a password.
* **Schedule:** disable if this is a one-time full, or set what you like and set the time to 9999.
* Check the box "Run at finish". 

#### Option B: *Volume-level* (manual selection of the disks)

* Choose **Volume level backup** → **Add volumes** → tick the disk you want (the one needed to boot and any others you want.
* Everything else same as above.

---

### Optional : Restore a backup and verify

1. In Veeam, click **Restore** → **File level restore**.
2. Open the latest restore point and **browse a few folders/files** to confirm the backup is readable.
3. (Optional) Enable **Verify backup** in job settings for automatic integrity checks.

---

### Create Recovery Media (bootable)

This is the small environment you’ll boot to perform **Bare Metal Recovery** if Windows won’t start or you’re restoring to a new drive/PC.

1. In Veeam → **Tools** → **Create Recovery Media**.
2. Pick **USB** (to format the usb with it and boot on it) or **ISO** (to save on ssd and to burn at a later step on a usb).
3. **Check “Include drivers from this computer”** (storage & network).
4. **Create**.

> Keep the USB with your backup drive and (if used) your **backup encryption password** and **BitLocker recovery key**.

---

### Test the Recovery Media

1. Leave the external SSD plugged in.
2. Reboot the PC → press your **Boot Menu** key (F12/F10/ESC varies).
3. Select the **Veeam Recovery Media** USB.
4. When the Veeam UI loads, confirm you can **see your external SSD** and **browse to your backup**.
5. Exit and reboot normally.

---

### Space planning (how big will the backup be?)

* Veeam does “**intelligent sector**” imaging + compression: backups are roughly your **used space** across all selected volumes, often **smaller**.
* Documents compress a lot; photos/videos hardly at all.

**Rule of thumb:** If C:+D: report **600 GB used**, expect a **\~400–600 GB** full backup. Plan headroom for at least **one** full + future incrementals.

To estimate quickly (per drive):

```powershell
# Replace C: with D: etc. to check other volumes
$vol = "C:"
$size = (Get-ChildItem $vol\ -Recurse -Force -ErrorAction SilentlyContinue | Measure-Object Length -Sum).Sum
"{0} used ≈ {1:N2} GB" -f $vol, ($size/1GB)
```

---

### Troubleshooting & tips

* **BitLocker volumes**: keep them **unlocked** during backup/restore start. You can store backups on a BitLocker-encrypted external SSD too.
* **PC boots straight to Windows after you add Linux later**: adjust UEFI boot order, or repair GRUB from Linux media.
* **Target SSD smaller than source**: OK if the **used** data fits and you allow Veeam to **shrink partitions** during restore.
* **Intel RST/RAID or unusual NVMe**: include drivers when creating Recovery Media.
* **Time saver**: make a **Standalone full** right before partitioning/dual-boot work.
* **Label everything**: tag the USB and SSD; store the **backup password** and **BitLocker key** safely.

---

### Appendix: useful commands

Turn off hibernation & Fast Startup:

```powershell
powercfg /h off
```

Quick disk checks:

```powershell
chkdsk C: /scan
sfc /scannow
```

Estimate a folder’s size (e.g., your profile):

```powershell
$path = "C:\Users\YourName"
$size = (Get-ChildItem $path -Recurse -Force -ErrorAction SilentlyContinue | Measure-Object Length -Sum).Sum
"{0:N2} GB" -f ($size/1GB)
```

---

**Next up (Part 2):** Dual-boot Windows ↔ Linux, migrating data, and later removing Windows cleanly once you’re confident on Linux.
