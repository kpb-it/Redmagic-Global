# RedMagic 11 Pro (Global) — Hardware & Tools Fact-Finding Guide

Purpose
- Gather authoritative hardware specs and tooling needed to extract firmware/drivers and to flash, compare, or build for the Global variant.

What I will extract (priority order)
1. Device identifiers
   - ro.product.model, ro.product.manufacturer, ro.product.device, ro.boot.serialno
   - OEM model numbers (e.g., NM-XXXX or RM-XXXX) and any variant names
2. Codename(s) and board
   - device codename used in device trees and kernel sources (e.g., codename_xyz)
3. SoC and chipset
   - Exact chipset model (e.g., Qualcomm Snapdragon version) and SoC model number
4. Memory / Storage
   - RAM size + type, internal storage size and eMMC / UFS type
5. Display
   - Resolution, refresh rate, panel vendor, peak brightness
6. Battery & charging
   - Capacity (mAh), charging tech (W), battery part numbers if available
7. Cameras
   - Sensor models for main/wide/ultra/tele/selfie, optical stabilization, max video modes
8. Radios & Connectivity
   - Cellular bands for Global variant, Wi‑Fi type, Bluetooth version, NFC, GPS chipset
9. Sensors & misc
   - Fingerprint type, gyro, compass, infrared, headphone jack etc.
10. Firmware components in repo
    - boot.img, recovery.img, vendor.img, vbmeta.img, dtbo.img, super/plat images, payload.bin
11. Kernel/device tree/vendor blobs
    - paths and license files
12. Partition layout and fstab/scatter files
13. Release/build metadata
    - build.prop values, build number, build date, OTA changelogs

Tools I will use (installed on analysis machine)
- adb (Android Debug Bridge) — adb shell getprop, adb pull, adb logcat
- fastboot — fastboot getvar all, fastboot flash <partition> <image>
- simg2img / losetup / mount tools — convert and mount system images
- payload-dumper-go — extract payload.bin from OTA zips
- avbtool (from Android source) — inspect vbmeta, verify AVB signatures
- abootimg/mkbootimg/mkbootimg_tools or magiskboot — examine/edit boot images
- 7zip/unzip/tar — extract zip/tar archives
- sha256sum / md5sum — checksums
- binwalk, strings, readelf, file — inspect binaries
- Ghidra/IDA (optional, for blob reverse-engineering)
- QPST/QFIL/Edl/Qualcomm tools (for EDL flashing where needed) — caution: use only if appropriate
- TWRP / OpenRecovery tools (if present)
- repo/git/gerrit / ccache / cross toolchains (for kernel building)

Commands and how I’ll extract each fact
- From a live device (Global variant):
  - adb shell getprop ro.product.model
  - adb shell getprop ro.product.device
  - adb shell getprop ro.boot.hardware
  - adb shell getprop ro.build.display.id
  - adb shell cat /proc/cpuinfo
  - adb shell dumpsys battery
  - adb shell getprop | grep -i radio
  - fastboot getvar all
- From firmware images and zips:
  - unzip <firmware.zip> ; ls -l ; sha256sum *
  - payload-dumper-go payload.bin -> extract system, vendor blocks
  - simg2img system.img sparse -> raw.img ; mount -o ro raw.img /mnt
  - magiskboot or unpack boot.img: magiskboot unpack boot.img ; examine kernel (zImage) and ramdisk
  - avbtool verify_image --image vbmeta.img
  - read device tree blobs (dtb/dtb.img) with dtc: dtc -I dtb -O dts <dtb>
  - binwalk boot.img / vendor.img to find embedded firmware blobs
  - strings and readelf on kernel modules to find variants & drivers
- From repository layout:
  - locate device/ vendor/ kernel/ kernel_sources/ and read README or manifests for codenames
  - list kernel config files (arch/*/config-*) and examine CONFIG_ options
  - find fstab.* files and default.prop / build.prop

Comparison matrix fields (for feature comparison across variants)
- Model / Codename / Region
- SoC / CPU cores / GPU
- RAM / Storage options
- Display (size / res / refresh / panel vendor)
- Battery (mAh / charging W / charging IC)
- Main camera (sensor model / OIS / aperture / pixels)
- Ultrawide / Tele / Macro sensors
- Connectivity (bands LTE / 5G, Wi‑Fi, BT)
- Bootloader lock method / AVB presence (vbmeta)
- Partition layout (A/B, super, dynamic)
- Stock firmware availability (global images present? yes/no)
- Kernel source availability (GPL compliance)
- Root / custom recovery status (TWRP known? Magisk compatibility)
- Known issues & notes

Safety & signature checks
- Verify SHA-256 checksums for every downloaded image.
- Verify AVB/vbmeta signatures when available.
- Warn about OEM unlock consequences and warranty.

Deliverables I will produce (once I can access repo or files)
- Consolidated hardware spec sheet (Global variant) with sources and checksums
- Tooling & flashing guide (fastboot and EDL flows), including exact commands
- Comparison table between Global and other regional variants in repo
- Kernel/device-tree/vendor summary and license audit
- ZIP of extracted important blobs and checksums (if allowed by licensing)

Notes
- I will not attempt any write/flash operations on a device without explicit instruction from you.
- If firmware files are proprietary and you cannot share them, give me file names / checksums or allow me read access to the repo.
