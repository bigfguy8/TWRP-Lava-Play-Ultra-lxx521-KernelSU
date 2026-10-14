# KernelSU-Next for Lava Play Ultra 5G (lxx521 / MT6878)

Custom kernel + KernelSU-Next root + TWRP recovery for **Lava Play Ultra 5G**.

## Device

| Property | Value |
|---|---|
| Model | Lava Play Ultra 5G |
| Codename | `lxx521` |
| SoC | MediaTek Dimensity 7300 (MT6878) |
| Android | 15 (VNDK 14) |
| Kernel | 6.1.115-android14-11 |
| KMI | `android14-6.1` |
| Build ID | `AP3A.240905.015.A2` |
| Storage | UFS 3.1 |
| Partition | Virtual A/B - `init_boot` + `vendor_boot` |

## Outputs

| Workflow | Artifact | Flash To |
|---|---|---|
| `build-ksu.yml` | `init_boot-patched.img` | `init_boot_a` |
| `build-twrp.yml` | `recovery.img` / `vendor_boot.img` | `vendor_boot_a` |
| `build-kernel.yml` | `kernel-lxx521-*.tar.gz` | (custom install) |

## Install - KernelSU

**Prerequisites:** bootloader unlocked, `mtkclient` or SP Flash Tool, ADB.

### 1. Backup stock init_boot

    # Phone OFF -> hold Vol+ and Vol- -> plug USB
    mtk r init_boot_a init_boot_stock_YOURS.img
    md5sum init_boot_stock_YOURS.img
    # Expect: 9c0100af87e61f8146def145aa81b266

### 2. Flash patched init_boot

    # Phone OFF -> hold Vol+ and Vol- -> plug USB
    mtk w init_boot_a init_boot-patched.img
    mtk reset

Only `init_boot_a` is written. `init_boot_b` stays stock for safety.

### 3. Install Manager

    adb install kernelsu-manager.apk

### 4. Verify

    su -c "uname -r"         # -> 6.1.115-android14-11
    su -c "ksud --version"   # -> KernelSU version

Open **KernelSU-Next Manager** -> should show **Working**.

## Recovery - Bootloop

    # Phone OFF -> hold Vol+ and Vol- -> plug USB
    mtk w init_boot_a init_boot_stock_YOURS.img
    mtk reset

Both slots if needed:

    mtk w init_boot_a init_boot_stock_YOURS.img
    mtk w init_boot_b init_boot_stock_YOURS.img
    mtk reset

## Build from Source

Actions -> Run workflow -> Select inputs -> Download artifact.

| Workflow | Inputs |
|---|---|
| `build-ksu.yml` | `ksu_version: v3.4.1` , `kmi: android14-6.1` |
| `build-twrp.yml` | `device: lxx521` , `product: twrp_lxx521` |
| `build-kernel.yml` | `kernel_source: NothingOSS/android_kernel_6.1_nothing_mt6878` , `kernel_branch: mt6878/Tetris/v` , `ksu_variant: KernelSU-Next` |

**Durations:** KernelSU ~3 min, Kernel ~30 min, TWRP ~4 h.

## SP Flash Tool Guide

**Use when:** bricked, bootloop, no fastboot, no recovery, mtkclient won't connect.

### Prerequisites

| Item | Source |
|---|---|
| SP Flash Tool v5.2216+ | https://spflashtool.com |
| MTK USB VCOM Drivers | Bundled with SP Flash Tool |
| Stock firmware | `LAVA_LXX521_SW_V13_HW_V1.0_20260811.rar` |
| Scatter | `MT6878_Android_scatter.xml` |
| DA file | `DA_BR.bin` |

### Install - Windows

1. Extract SP Flash Tool to `C:\SP_Flash_Tool`
2. Run `Drivers/InstallDriver.exe` as Administrator
3. If driver install fails: reboot -> F8 -> disable signature enforcement
4. Verify: Device Manager -> Ports -> `MediaTek USB VCOM`

### Install - Linux

    sudo apt install libusb-1.0-0 libusb-1.0-0-dev
    chmod +x flash_tool
    echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="0e8d", MODE="0666"' | sudo tee /etc/udev/rules.d/99-mtk.rules
    sudo udevadm control --reload-rules
    sudo udevadm trigger

### Firmware Prep

    unrar x LAVA_LXX521_SW_V13_HW_V1.0_20260811.rar

Delete `userdata.img` if you want to preserve user data.

### Flash Procedure

**Step 1** - Open SP Flash Tool

**Step 2** - Click **Scatter-loading File** -> select `MT6878_Android_scatter.xml`

**Step 3** - Mode: **Download Only**

| Mode | Use |
|---|---|
| Download Only | Default, safest |
| Format All + Download | Badly corrupted |
| Firmware Upgrade | Factory reset |

**Step 4** - Select partitions

- Full restore: leave all checked
- Specific: right-click -> Deselect All -> check only target
- Bootloop recovery: only `init_boot_a` -> `init_boot.img`

**Step 5** - BROM mode

1. Power off
2. Hold **Vol+ + Vol-**
3. Plug USB (screen stays black)
4. SP detects in ~2s

**Step 6** - Click **Download** (green arrow)

- Wait 3-10 min
- Green circle = success

**Step 7** - Disconnect USB -> hold **Power** 10s -> boot

First boot: 5-10 min (dalvik rebuild).

### Troubleshooting

| Error | Fix |
|---|---|
| `S_BROM_CMD_STARTCMD_FAIL` | USB 2.0 port, reinstall VCOM drivers |
| `S_BROM_CMD_JUMP_DA_FAIL` | Verify `DA_BR.bin` for MT6878 |
| `S_FT_ENABLE_DRAM_FAIL` | Use Format All + Download |
| `S_COM_PORT_OPEN_FAIL` | Check cable, retry BROM |
| `S_SECURITY_SLA_FAIL` | Load `.auth` file (vendor) |
| Stuck at 0% | Re-enter BROM mode |
| Not detected | Different port, reinstall drivers |

### Emergency Scenarios

**Bootloop after bad init_boot:**

    Deselect all -> check init_boot_a -> init_boot.img -> Download

**Dead device:**

    Format All + Download -> all partitions -> wait 10-15 min

**Stuck at vendor logo:**

    Flash only: boot, init_boot, vendor_boot, vbmeta
    Do NOT flash super.img

### Pre-Flash Checklist

- [ ] Device >50% charged
- [ ] Original USB cable
- [ ] Firmware matches LXX521
- [ ] Scatter + DA files present
- [ ] USB 2.0 port (not 3.0)
- [ ] Antivirus disabled
- [ ] Only needed partitions selected

### mtkclient Alternative

    pip3 install mtkclient

    mtk r init_boot_a init_boot_a.img       # read
    mtk w init_boot_a init_boot-patched.img # write
    mtk e userdata                          # format
    mtk printgpt                            # list partitions
    mtk reset

### Critical Warnings

- Never unplug USB during flash
- Never flash another device's firmware
- Never use Format All without backup
- Verify MD5 before flashing

## Partition Layout

| Partition | Block | Size |
|---|---|---|
| `boot_a` | `/dev/block/sdc36` | 64 MB |
| `boot_b` | `/dev/block/sdc65` | 64 MB |
| `init_boot_a` | `/dev/block/sdc38` | 8 MB |
| `init_boot_b` | `/dev/block/sdc67` | 8 MB |
| `vendor_boot_a` | `/dev/block/sdc37` | 64 MB |
| `vendor_boot_b` | `/dev/block/sdc66` | 64 MB |

**Stock MD5s:**

- `init_boot_a`: `529e38438f8fe148b0b250619081538b`
- `init_boot_b`: `f04f0d15e015f090f41b677f1d781145`
- Real stock: `9c0100af87e61f8146def145aa81b266`

## Known Issues

| Issue | Workaround |
|---|---|
| LKM vermagic mismatch (6.1.115 vs 6.1.166) | Build kernel from source |
| `ksud --partition` missing on musl | magiskboot + cpio |
| QEMU can't run Android binary | Missing `/system/bin/linker64` |
| No lxx521-specific kernel source | Use NothingOSS MT6878 base |
| FBE/FSCRYPT v3 on A15 | `TW_USE_FSCRYPT_V3` flag |

## References

**Device**

- Device tree: https://github.com/fahimfrk/Dimensity7300-Device-Tree-Lava-Play-Ultra
- Kernel config: https://github.com/SiliconxLab/android_device_lava_lxx521-kernel

**Kernel Sources (MT6878)**

- NothingOSS/android_kernel_6.1_nothing_mt6878 (`mt6878/Tetris/v`)
- mt6878-devs/android_kernel_device_modules
- CMF-Phone-1/android_kernel_6.1_nothing_mt6878

**KernelSU**

- KernelSU-Next: https://github.com/KernelSU-Next/KernelSU-Next
- SukiSU-Ultra: https://github.com/SukiSU-Ultra/SukiSU-Ultra

**Tools**

- magiskboot: https://github.com/Uevo001/magiskboot-linux
- mtkclient: https://github.com/bkerler/mtkclient
- AOSP clang: https://android.googlesource.com/platform/prebuilts/clang/host/linux-x86

**Firmware**

- LAVA dumps: https://gitlab.com/noescape-firmware-dumps/LAVA/LXX521

**TWRP**

- Manifest: https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp (`twrp-14`)

## Roadmap

- [x] Stock firmware extraction
- [x] Device tree identification
- [x] KernelSU init_boot patch
- [ ] Custom kernel boots
- [ ] KernelSU integrated in kernel (no LKM)
- [ ] Custom ROM (LineageOS 22)
- [ ] Full OTA zip

## Credits

- Device tree: fahimfrk
- Kernel config: SiliconxLab
- KernelSU: tiann + KernelSU-Next team
- MT6878 kernel: NothingOSS, mt6878-devs
- Workflows: bigfguy8

## License

Workflows: MIT. Kernel source: GPL-2.0. Device tree: original repo license.

## Disclaimer

Flashing custom images voids warranty and can brick your device. Backup first, verify MD5s, keep stock firmware available. Use at your own risk.

---

**Last updated:** 2026-10-14 | **Maintainer:** [@bigfguy8](https://github.com/bigfguy8)
