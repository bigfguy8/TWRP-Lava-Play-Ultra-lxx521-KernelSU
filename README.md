# KernelSU-Next + TWRP for Lava Play Ultra 5G (lxx521 / MT6878)

Custom KernelSU-Next root & TWRP recovery build pipeline.

## Device

| Item | Value |
|---|---|
| Model | Lava Play Ultra 5G |
| Codename | lxx521 |
| SoC | MediaTek Dimensity 7300 (MT6878) |
| Android | 15 (SDK 35) |
| Kernel | 6.1.115-android14-11 |
| KMI | android14-6.1 |
| Partition | Virtual A/B |

## Workflows

| Workflow | Output |
|---|---|
| build-lxx521.yml | init_boot-patched.img |
| build-twrp.yml | vendor_boot.img |

## Usage

1. GitHub Actions → Run workflow
2. Download artifact
3. fastboot flash init_boot_b init_boot-patched.img
4. fastboot reboot

## Recovery

fastboot flash init_boot_b init_boot_stock.img

## Credits

- KernelSU-Next
- Magisk (magiskboot)
- TWRP
- Device tree by fahimfrk
