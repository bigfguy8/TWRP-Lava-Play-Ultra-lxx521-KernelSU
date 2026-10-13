# KernelSU-Next for Lava Play Ultra 5G (lxx521)

KernelSU-Next root via patched init_boot.

## Device

- Device: Lava Play Ultra 5G
- Codename: lxx521
- SoC: MediaTek Dimensity 7300 (MT6878)
- Kernel: 6.1.115-android14-11
- KMI: android14-6.1

## Install

Prerequisites: Bootloader unlocked + mtkclient OR SP Flash Tool + ADB

### 1. Backup stock init_boot
mtk r init_boot_a init_boot_stock_YOURS.img

### 2. Flash patched init_boot
mtk w init_boot_a init_boot-patched.img
mtk reset

### 3. Install Manager APK
adb install kernelsu-manager.apk

### 4. Verify
Open KernelSU-Next Manager, should show Root OK.

## Recovery if Bootloop

Phone OFF, hold Vol+ and Vol-, plug USB, then:
mtk w init_boot_a init_boot_stock.img
mtk reset

## Build from Source

GitHub Actions workflow: .github/workflows/build-ksu.yml

1. Actions tab, Run workflow
2. Inputs: ksu_version (v3.4.1), kmi (android14-6.1)
3. Wait ~2 min, download artifact

## Files

- init_boot-patched.img: flash this
- init_boot_stock.img: stock recovery
- kernelsu-manager.apk: Manager app
- kernelsu-lkm-android14-6.1.ko: LKM reference
- SHA256SUMS: hashes

## Source

- KernelSU-Next: https://github.com/KernelSU-Next/KernelSU-Next
- TWRP: https://github.com/bigfguy8/TWRP-Lava-Play-Ultra-lxx521

## Warnings

- Advanced users only
- Backup stock init_boot first
- Test at your own risk
- Only init_boot_a slot modified; init_boot_b stock

## Credits

- KernelSU-Next team
- bigfguy8 (build)
