# Lava Play Ultra (lxx521) - KernelSU-Next Findings

**Date:** 2026-10-15
**Device:** Lava Play Ultra 5G (lxx521 / MT6878)
**Author:** bigfguy8
**Status:** Root working, module system inactive

---

## 1. Device Fingerprint

| Property | Value |
|----------|-------|
| Model | Lava Play Ultra 5G |
| Codename | lxx521 |
| SoC | MediaTek MT6878 (Dimensity 7300) |
| Board | k6878v1_64 |
| Android | 15 (SDK 35) |
| Build ID | AP3A.240905.015.A2 |
| Kernel | 6.1.115-android14-11-g6b18f0b574ab-ab12901745 |
| KMI | android14-6.1 |
| Arch | aarch64 |
| Bootloader | Unlocked (verifiedbootstate=orange) |
| SELinux | Enforcing |

---

## 2. Partition Layout (Virtual A/B)

| Partition | Slot A SHA256 | Slot B SHA256 |
|-----------|---------------|---------------|
| init_boot | 51ff3ad4... STOCK | 6bd20e52... PATCHED |
| boot | 433b8d88... STOCK | 3b6a07d0... STOCK |
| vbmeta | 6559cc13... STOCK | N/A |
| dtbo | cc403ef2... STOCK | N/A |

**Active slot:** _a (per ro.boot.slot_suffix) - LKM early-load works from _b.

**Backup location:** /sdcard/lxx521_backup/

---

## 3. Current KernelSU-Next Status

| Component | Version | Location | Status |
|-----------|---------|----------|--------|
| LKM kernelsu.ko | v3.3.1 | kernel-space | LOADED |
| ksud binary | v3.3.1 | /data/adb/ksud | PRESENT |
| ksud daemon | N/A | N/A | NOT RUNNING |
| Manager APK | v3.3.1 | me.weishu.kernelsu | INSTALLED |
| Module folder | N/A | /data/adb/modules | MISSING |
| Overlay mount | N/A | N/A | INACTIVE |

**LKM Vermagic:** 6.1.166-dirty SMP preempt mod_unload modversions aarch64

---

## 4. Root Cause Analysis

### Problem: ksud daemon not running

1. _b slot patched init_boot has kernelsu.ko embedded
2. init.rc NOT modified -> import /init.kernelsu.rc missing
3. Android 13+ init reads /system/etc/init/*.rc, NOT /init.kernelsu.rc
4. Patched ramdisk lacks /system/etc/init/kernelsu.rc
5. Result: LKM loads (early-load), but ksud post-fs-data never runs

---

## 5. Bugs Found and Fixed

| # | Bug | Fix |
|---|-----|-----|
| 1 | init.rc hard-extract fails Android 13+ | Made optional |
| 2 | xxd not in Ubuntu runner | Use od -An -tx1 |
| 3 | Boolean input if: always truthy | Use == true explicit |
| 4 | Heredoc in YAML breaks indentation | Use printf single-line |
| 5 | Missing system/etc/init/kernelsu.rc | Added mkdir + copy |

---

## 6. Recommended Fix

### For v3.3.1 current system

Rebuild init_boot with:
- ksu_version: v3.3.1 (match current)
- kmi: android14-6.1
- LKM v3.3.1 embedded
- init.kernelsu.rc + system/etc/init/kernelsu.rc

### For v3.4.2 full upgrade

1. Manager APK v3.4.2 install
2. /data/adb/ksud update to v3.4.2
3. Rebuild init_boot v3.4.2 LKM
4. Flash in order

---

## 7. Safety Backup

/sdcard/lxx521_backup/
- init_boot_a.img  51ff3ad4...  STOCK
- boot_a.img       433b8d88...
- vbmeta_a.img     6559cc13...
- dtbo_a.img       cc403ef2...

### Recovery command if bootloop

fastboot flash init_boot_a /sdcard/lxx521_backup/init_boot_a.img
fastboot reboot

---

## 8. Known Limitations

1. bootctrl HAL missing on MT6878
2. MediaTek virtual A/B - slot_suffix unreliable
3. KernelSU-Next v3.3.1 ksud daemon init problem
4. LKM vermagic mismatch works via modversions
5. Kernel source not available from Lava (GPL pending)

---

## 9. Next Steps

- [ ] Request GPL kernel source from Lava support
- [ ] Build custom LKM with exact vermagic 6.1.115-android14-11
- [ ] Build custom TWRP recovery
- [ ] Add proper init.rc integration to ramdisk
- [ ] Embed ksud binary in ramdisk
- [ ] Test module system functionality

---

**Repo:** https://github.com/bigfguy8/TWRP-Lava-Play-Ultra-lxx521-KernelSU
