# macOS Tahoe 26 on ASUS M515DA — OpenCore Offline Installation Guide

This guide documents the exact process used to install **macOS Tahoe 26** on an **ASUS M515DA** using OpenCore and a prebuilt EFI.

The installation is performed **offline**. Instead of downloading macOS through Recovery, we use the full `InstallAssistant.pkg` package and create the installer locally.

> [!IMPORTANT]
> The EFI used in this guide is already configured for this laptop. If your hardware differs, do not assume that the same EFI will work without modifications.

---

## Hardware

Laptop:

**ASUS M515DA**

Main platform:

* AMD Ryzen 5 3500U
* Radeon Vega integrated graphics
* UEFI firmware
* Realtek Wi-Fi
* OpenCore bootloader

The EFI provided with this guide was built specifically around this hardware.

---

> [!IMPORTANT]
>
> ## Current Compatibility Status
>
> Before starting, be aware of the current limitations of this EFI:
>
> * **Audio is currently not working.**
> * **Wi-Fi is working**, but it is **not natively supported by macOS**. The ASUS M515DA uses a Realtek wireless adapter, so Wi-Fi is provided through the **RTW/rtw88 driver (Feixiao)** and its separate Wi-Fi interface/application.
>
> The rest of this guide documents the exact installation used for **macOS Tahoe 26 on the ASUS M515DA with Ryzen 5 3500U**.


# 1. What You Need

Before starting, prepare:

* A USB flash drive of at least **16 GB**
* The provided **EFI folder**
* A working computer
* `InstallAssistant.pkg` for macOS Tahoe 26
* Access to the laptop's BIOS/UEFI
* An internal SSD where macOS will be installed

For Wi-Fi configuration after installation, you may also need:

* The compatible Realtek Wi-Fi driver/kext
* The Wi-Fi interface application

These steps are covered later.

---

# 2. Download macOS Tahoe

This installation was **not created by downloading macOS directly from another Mac through the App Store**.

The full macOS installer was obtained through an `InstallAssistant.pkg` file.

Search online for:

```text
macOS Tahoe 26 InstallAssistant.pkg
```

Look for a download that ultimately points to Apple's official servers.

> [!IMPORTANT]
> Prefer an `InstallAssistant.pkg` hosted on or redirected to an official Apple download server. Avoid modified macOS images, ISOs, DMGs, or installers from unknown sources.

After obtaining the package, install it on macOS.

The full installer should then appear in:

```text
/Applications/Install macOS Tahoe.app
```

Verify that the application exists before continuing.

---

# 3. Prepare the USB Drive

Connect the USB flash drive.

Open **Disk Utility** and select:

```text
View → Show All Devices
```

Select the physical USB device, not only its existing partition.

Click **Erase**.

Use:

```text
Name: MyVolume
Format: Mac OS Extended (Journaled)
Scheme: GUID Partition Map
```

Then erase the drive.

> [!NOTE]
> `MyVolume` is only an example. You can use another name, but you must adjust the commands accordingly.

---

# 4. Create the macOS Installer

Open Terminal.

Run:

```bash
sudo "/Applications/Install macOS Tahoe.app/Contents/Resources/createinstallmedia" --volume /Volumes/MyVolume
```

Enter your macOS password when requested.

Confirm the operation if prompted.

`createinstallmedia` will:

1. Erase the USB volume
2. Copy the macOS installer
3. Make the USB bootable

Wait until the process finishes completely.

You should now have a macOS Tahoe installation USB.

---

# 5. Mount the USB EFI Partition

The macOS installer alone is not enough for this AMD laptop.

OpenCore must be placed inside the USB's EFI System Partition.

Identify the USB:

```bash
diskutil list
```

Find its EFI partition.

It will normally look similar to:

```text
/dev/disk2s1
```

Mount it:

```bash
sudo diskutil mount disk2s1
```

Replace `disk2s1` with the correct identifier for your USB.

A volume named something similar to:

```text
EFI
```

should appear.

---

# 6. Copy the Prebuilt EFI

Copy the provided:

```text
EFI
```

folder to the EFI partition of the USB.

The structure must look like:

```text
EFI
├── BOOT
│   └── BOOTx64.efi
│
└── OC
    ├── ACPI
    ├── Drivers
    ├── Kexts
    ├── Resources
    ├── Tools
    ├── config.plist
    └── OpenCore.efi
```

Do **not** put the EFI folder inside the macOS installer partition.

It belongs inside the small FAT32 EFI System Partition.

The USB now contains two important components:

```text
USB
├── EFI partition
│   └── EFI
│       ├── BOOT
│       └── OC
│
└── macOS Installer partition
    └── Install macOS Tahoe
```

---

# 7. BIOS Configuration

Restart the ASUS and enter BIOS/UEFI setup.

On ASUS laptops this is normally done using:

```text
F2
```

during startup.

Make sure the machine is configured for UEFI boot.

Depending on the BIOS version, disable options that interfere with OpenCore, especially **Secure Boot**.

Save the settings and restart.

Open the ASUS boot menu and select the USB's UEFI/OpenCore entry.

---

# 8. Boot OpenCore

OpenCore should appear.

Select the macOS installer.

If the installer entry does not immediately appear, use OpenCore's auxiliary entries/options if enabled in the provided EFI.

The system should begin booting macOS.

Verbose output may appear depending on how the EFI is configured.

Wait until the macOS installer interface loads.

---

# 9. Format the Internal SSD

From the macOS installer, open:

```text
Disk Utility
```

Then select:

```text
View → Show All Devices
```

Select the physical internal SSD.

Erase it using:

```text
Name: macintosh
Format: APFS
Scheme: GUID Partition Map
```

The volume name used during our installation was:

```text
macintosh
```

You can use another name, but remember to modify later commands if necessary.

Close Disk Utility.

---

# 10. Install macOS

Select:

```text
Install macOS Tahoe
```

Choose:

```text
macintosh
```

as the destination.

Start the installation.

The computer will reboot several times.

> [!IMPORTANT]
> Keep the USB connected during the entire installation.

After each reboot, boot through OpenCore again.

Depending on the installation stage, OpenCore may show entries such as:

```text
Install macOS
macOS Installer
macintosh
```

Select the installation-related entry while installation is still in progress.

Once installation is complete, boot:

```text
macintosh
```

---

# 11. Setup Assistant Freeze / Black Screen Fix

This was one of the important issues encountered during the installation.

The system could boot macOS, but the initial Setup Assistant could become unusable because of the graphics/Metal-related behavior.

The fix that allowed us to continue was **not simply disabling graphics acceleration**.

From Terminal, run:

```bash
defaults write "/Volumes/macintosh/Library/Preferences/com.apple.coremedia" allowMetalTransferSession -bool NO
```

This writes the following preference:

```text
allowMetalTransferSession = NO
```

to:

```text
/Library/Preferences/com.apple.coremedia
```

on the installed macOS volume.

> [!IMPORTANT]
> If your macOS volume is not called `macintosh`, replace `/Volumes/macintosh` with the correct volume name.

For example:

```bash
defaults write "/Volumes/YOUR_VOLUME/Library/Preferences/com.apple.coremedia" allowMetalTransferSession -bool NO
```

After applying the command, reboot:

```bash
reboot
```

Boot macOS again through OpenCore.

The Setup Assistant should now be able to continue.

Complete the normal macOS configuration.

---

# 12. First macOS Boot

After reaching the desktop, verify that the basic system is operational.

At this stage you should have:

* macOS Tahoe booting
* Keyboard working
* Trackpad working
* USB working
* Internal SSD working
* Basic graphics output
* OpenCore booting the installation

However, the laptop still depends on the USB for OpenCore.

The next step fixes that.

---

# 13. Install OpenCore to the Internal SSD

Mount the internal SSD's EFI partition.

First:

```bash
diskutil list
```

Find the EFI partition belonging to the internal SSD.

For example:

```text
/dev/disk0s1
```

Mount it:

```bash
sudo diskutil mount disk0s1
```

Replace the identifier if necessary.

Now copy the same working:

```text
EFI
```

folder from the USB to the internal SSD's EFI partition.

The SSD EFI should contain:

```text
EFI
├── BOOT
└── OC
```

Shut down the computer.

Remove the USB.

Turn the laptop on again.

If everything is correct, OpenCore should now boot directly from the internal SSD.

---

# 14. Realtek Wi-Fi

The ASUS uses a Realtek wireless adapter that is not natively supported by macOS.

For our configuration, Wi-Fi support was added separately after macOS was already installed.

The driver used was based on the **Feixiao / rtw88** project.

The package contains:

```text
rtw88.kext
```

Install the compatible kext according to the version included with this build.

If the kext is being loaded through OpenCore, place it inside:

```text
EFI/OC/Kexts/
```

Then add the corresponding entry under:

```text
Kernel → Add
```

in:

```text
config.plist
```

The kext entry must be enabled:

```text
Enabled = True
```

After modifying the EFI, reboot.

---

# 15. Wi-Fi Interface

The Realtek driver provides the hardware support, but macOS may not expose the adapter exactly like a native Apple Wi-Fi card.

For this reason, the Wi-Fi interface/application used with the driver must also be installed.

Use the interface compatible with the Feixiao/rtw88 driver included with your setup.

After installing it, reboot if required.

The application can then be used to:

* Scan Wi-Fi networks
* Select an SSID
* Enter the Wi-Fi password
* Connect/disconnect from networks

---

# 16. Optional — RTW Configuration

Additional RTW-related configuration is **optional**.

If Wi-Fi already works correctly using the supplied driver and interface, you can skip additional RTW modifications.

Only apply them if they are required by your specific wireless configuration.

Do not change a working Wi-Fi configuration unnecessarily.

---

# 17. Optional — Rename the Wi-Fi Interface

Renaming or changing how the Realtek interface appears in macOS is also **optional**.

It is not required for the basic installation.

If the adapter works and you only care about network connectivity, skip this section.

Renaming is mainly useful for cosmetic integration or for configurations where another application expects a specific interface name.

---

# 18. Optional — Clean Up the OpenCore Boot Screen

Once the Hackintosh is confirmed to be stable, you can optionally clean up the OpenCore boot experience.

For example, you can disable unnecessary verbose/debug output and hide auxiliary OpenCore entries.

Do this **only after confirming that the machine boots reliably**.

Keeping verbose boot enabled during the initial installation is useful because it makes troubleshooting much easier.

---

# 19. Final Boot Test

Perform a complete test without the installation USB.

Shut down the laptop.

Remove the USB.

Turn it back on.

The expected sequence is:

```text
ASUS UEFI
   ↓
Internal EFI
   ↓
OpenCore
   ↓
macOS Tahoe
   ↓
macintosh
   ↓
Desktop
```

Verify:

* macOS boots without the USB
* Keyboard works
* Trackpad works
* USB ports work
* Wi-Fi works
* Internal SSD is detected correctly
* OpenCore loads from the internal EFI
* The Setup Assistant issue does not return

---

# 20. Important Files to Back Up

Once everything works, make a backup of the working EFI.

At minimum, keep a copy of:

```text
EFI/
├── BOOT/
└── OC/
    ├── ACPI/
    ├── Drivers/
    ├── Kexts/
    └── config.plist
```

This is especially important before:

* Updating macOS
* Updating OpenCore
* Updating kexts
* Changing ACPI
* Changing AMD kernel patches
* Modifying graphics settings

A working EFI backup can save a lot of troubleshooting later.

---

# Installation Summary

The complete installation process used on this ASUS M515DA was:

```text
Download InstallAssistant.pkg
        ↓
Install macOS Tahoe.app
        ↓
Format USB as GUID
        ↓
Run createinstallmedia
        ↓
Mount USB EFI
        ↓
Copy prebuilt OpenCore EFI
        ↓
Boot ASUS from USB
        ↓
Format SSD as APFS
        ↓
Install macOS Tahoe
        ↓
Boot installer through OpenCore after each reboot
        ↓
Apply allowMetalTransferSession workaround
        ↓
Complete Setup Assistant
        ↓
Copy EFI to internal SSD
        ↓
Boot without USB
        ↓
Install Realtek/rtw88 Wi-Fi support
        ↓
Install Wi-Fi interface
        ↓
Apply optional RTW / rename tweaks
        ↓
Done
```

---

## Final Notes

This guide is based on an **actual working installation**, not a generic OpenCore procedure.

The most important parts specific to this installation are:

1. Using the provided EFI configured for the **ASUS M515DA / Ryzen 5 3500U**.
2. Using the full `InstallAssistant.pkg` for an **offline macOS Tahoe installation**.
3. Creating the installer using Apple's `createinstallmedia`.
4. Keeping OpenCore on the USB during all installation reboots.
5. Applying:

```bash
defaults write "/Volumes/macintosh/Library/Preferences/com.apple.coremedia" allowMetalTransferSession -bool NO
```

when the initial macOS configuration cannot proceed correctly.
6. Moving the working EFI to the internal SSD only after macOS successfully boots.
7. Installing the Realtek Wi-Fi driver and interface after the main installation.
8. Treating additional **RTW configuration and interface renaming as optional steps**.

Keep your original working EFI backed up before experimenting with any further changes.
