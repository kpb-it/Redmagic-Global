```markdown
# Fact-Finding Checklist — RedMagic 11 Pro (Global)

Purpose
- Consolidated checklist to collect hardware facts for the RedMagic 11 Pro (Global).
- Includes tools, links, commands, and a reproducible collection plan.

Checklist items (tick when done)
- [ ] Repo access granted OR firmware files uploaded
- [ ] Identify device codename(s) and model strings
- [ ] Extract build.prop from boot or system; capture build ID and date
- [ ] Confirm SoC model & CPU info from kernel or /proc/cpuinfo
- [ ] Confirm RAM/storage (from board config or kernel/partition layout)
- [ ] Extract screen panel info (from device tree or system drivers)
- [ ] Extract battery and charging IC info
- [ ] Extract camera sensor strings from camera configs or vendor blobs
- [ ] Pull modem firmware and list supported bands (from vendor/modem/*)
- [ ] List vendor blobs and licenses (identify closed-source)
- [ ] Find kernel source tree and verify GPL files
- [ ] Find device tree and board config files for codenames
- [ ] Inspect boot/vbmeta for AVB signatures and lock state
- [ ] Produce checksums of all images in repo
- [ ] Create feature comparison row for Global vs regional variant(s)

Host-side tools (install or link)
- adb & fastboot (Android Platform Tools)
  - Download / docs: https://developer.android.com/studio/releases/platform-tools
  - Usage: device identification, getprop, adb pull, fastboot getvar, fastboot flash
- repo (AOSP manifest tool) and git
  - repo docs: https://source.android.com/setup/develop/repo
  - git: https://git-scm.com/
- simg2img (sparse image -> raw image)
  - Repo: https://github.com/anestisb/android-simg2img
  - Usage: convert system.img (sparse) to raw image for mount
- payload-dumper-go (extract payload.bin from OTA zips)
  - Repo: https://github.com/ssut/payload-dumper-go
- avbtool (Android Verified Boot)
  - Docs / source: https://source.android.com/security/verifiedboot/avb
  - avbtool source: https://android.googlesource.com/platform/external/avb/
- magisk / magiskboot (unpack and repack boot images, patching)
  - Repo: https://github.com/topjohnwu/Magisk
  - Tools: magiskboot for boot.img ramdisk operations
- Android Image Kitchen / abootimg / mkbootimg tools (boot image tooling)
  - Android Image Kitchen: https://github.com/osm0sis/Android-Image-Kitchen
  - abootimg: https://github.com/pbatard/abootimg
- dtc (Device Tree Compiler)
  - Repo: https://git.kernel.org/pub/scm/utils/dtc/dtc.git
  - Use to convert DTB -> DTS to read panel / board nodes
- binwalk (extract embedded files & firmware)
  - Repo: https://github.com/ReFirmLabs/binwalk
- readelf / strings / file / objdump (binutils)
  - GNU binutils: https://www.gnu.org/software/binutils/
- simg2img payload tools (already listed)
- 7-Zip / unzip / tar
  - 7-Zip: https://www.7-zip.org/
  - unzip (Linux): https://linux.die.net/man/1/unzip
- sha256sum / md5sum (coreutils)
  - Coreutils: https://www.gnu.org/software/coreutils/
- Ghidra / IDA (optional binary analysis)
  - Ghidra: https://ghidra-sre.org/
  - IDA: https://www.hex-rays.com/products/ida/
- TWRP (custom recovery)
  - Official: https://twrp.me/
  - Device builds: check TWRP device pages or XDA
- EDL/QFIL/QPST / Qualcomm tools (EDL flash mode — proprietary / vendor)
  - QPST / QFIL typically provided by Qualcomm/OEM; common community references:
    - QFIL guide (XDA): https://forum.xda-developers.com/t/guide-qfil-qualcomm-flash-image-loader.4120291/
    - EDL tools & community projects: https://github.com/LongSoft/edl (community repos vary)
  - WARNING: Proprietary tools and EDL flashing can brick devices if used incorrectly.
- payload & image analysis helpers
  - payload-repacker: https://github.com/anestisb/android-simg2img (and payload tools listed above)
- Cross-compile toolchains & build tools (if building kernel)
  - AOSP build docs / toolchains: https://source.android.com/setup/build
  - ccache: https://ccache.dev/

On-device convenience apps (quick checks without adb)
- AIDA64 (Android) — Play Store: https://play.google.com/store/apps/details?id=com.finalwire.aida64
- CPU-Z (Android) — Play Store: https://play.google.com/store/apps/details?id=com.cpuid.cpu_z
(Useful for a quick non-root hardware summary)

Commands and how to extract each fact (copyable)
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
  - unzip firmware.zip ; ls -l ; sha256sum *
  - payload-dumper-go payload.bin    # extract system/vendor from OTAs
  - simg2img system.img system.raw.img ; mount -o ro system.raw.img /mnt
  - magiskboot unpack boot.img ; examine zImage and ramdisk
  - avbtool info_image --image vbmeta.img / avbtool verify_image --image vbmeta.img
  - dtc -I dtb -O dts <dtb-file>
  - binwalk boot.img / vendor.img
  - strings and readelf on kernel modules
- From repository layout:
  - locate device/ vendor/ kernel/ kernel_sources/ and read README or manifests for codenames
  - list kernel config files (arch/*/config-*)
  - find fstab.* files and default.prop / build.prop

Snapdragon-focused probes (Qualcomm sysfs & props)
- Sysfs and properties commonly useful on Qualcomm devices:
  - adb shell cat /sys/devices/soc0/family
  - adb shell cat /sys/devices/soc0/machine
  - adb shell cat /sys/devices/soc0/soc_id
  - adb shell getprop ro.vendor.qti.chipid
  - adb shell getprop ro.board.platform
  - dmesg | egrep -i 'qcom|snapdragon|adreno|kgsl|msm'
  - /sys/class/kgsl/* and GPU model:
    - adb shell ls /sys/class/kgsl
    - adb shell cat /sys/class/kgsl/kgsl-3d0/gpu_model
- These help identify whether the device uses a Snapdragon 5-series SoC (look for family/soc_id strings and Adreno GPU names)

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
- Verify AVB/vbmeta signatures when available using avbtool.
- Warn about OEM unlock consequences and warranty.

Deliverables I will produce (once I can access repo or files)
- Consolidated hardware spec sheet (Global variant) with sources and checksums
- Tooling & flashing guide (fastboot and EDL flows), including exact commands
- Comparison table between Global and other regional variants in repo
- Kernel/device-tree/vendor summary and license audit
- ZIP of extracted important blobs and checksums (if allowed by licensing)

Suggested quick commands to run on a device (copyable)
- adb shell getprop ro.product.model
- adb shell getprop ro.product.device
- adb shell getprop ro.build.version.release
- adb shell cat /proc/cpuinfo
- adb shell dumpsys display
- fastboot getvar all

Suggested file commands for firmware archive
- sha256sum firmware.zip
- unzip -l firmware.zip
- payload-dumper-go payload.bin
- magiskboot unpack boot.img
- avbtool info_image --image vbmeta.img
- simg2img system.img system.raw.img ; mount -o ro system.raw.img /mnt

Reporting format I will use
- Hardware summary (one-page)
- Tools & extraction commands (copyable)
- Evidence section (file paths, checksums, exact file contents snippets)
- Comparison table (CSV/markdown)
- Actionable next steps (flash guide / kernel build steps / compliance issues)

Permissions / context I need
- Public repo link or read permission
- If private: add me (kpb-it collaborator) or paste raw file URLs
- If uploading images, include original zip and any READMEs

Output options
- Single consolidated Markdown report
- Separate files:
  - hardware-specs.md
  - flashing-guide.md
  - kernel-vendor-audit.md
  - comparison.csv (for spreadsheet import)

What to do next
- Make the repo public (or upload firmware images) and/or run the provided collection script (collect-hardware-specs.sh) and upload the resulting report folder.
- If you want, I can add a soc_id -> SoC lookup table for Snapdragon 5-series parts once you provide soc_id output.

Notes
- I will not attempt write/flash operations unless you explicitly instruct me to.
- Some sysfs paths may require root to read; the script will note permission errors.
```
