# AK3 6.6.150 · Oplus k6991v1_64 · 35833UAPI6 · 2026-11-15

收到日期：2026-11-21（只做記錄，尚未刷機或測試）

## 刷機包

| 項目 | 值 |
|---|---|
| 檔名 | `AK3-6.6.150-g3b3662ca5829-Oplus-k6991v1_64-gki-35833UAPI6-20261115-2014-common.zip` |
| 大小 | 22,441,461 bytes |
| SHA-256 (zip) | `8d160bccd297894f070df9a0b97a164571baeb680696146f255f97209abb00fb` |
| 打包時間 | 2026-11-15 12:14（zip 內時間戳） |
| 打包方式 | AnyKernel3（`AK_BASE_VERSION=20260704`），schqiushui 修改版，提示有中英雙語 |
| kernel.string | `SpiderDroid Kernel by schqiushui@github.com` |

檔名各欄位的意思：
- `k6991v1_64`：MediaTek 平台代號（MT6991，天璣 9400 系列），Oplus 機型
- `35833UAPI6`：對應的 Oplus 韌體版本代號
- `20261115-2014`：建置時間 2026-11-15 20:14
- `common`：通用 GKI，沒有綁特定機型

## 核心

```
Linux version 6.6.150-android15-8-g3b3662ca5829-abogki537459655-4k (emira@Lotus)
clang 18.0.0 (Android r510928, 11368308)
#1 SMP PREEMPT Mon May 12 09:09:59 UTC 2025
```

- KMI：`android15-6.6`，sublevel 150，commit `g3b3662ca5829`
- 4K page，`ARM64_VA_BITS=39`
- uname 裡的建置日期（2025-05-12）比實際建置日期早，應該是偽裝成官方 GKI 的時間戳
- `Image` SHA-256：`1617efb1b02b6d05c540b4884f4a98841382e56390a7ac92168e37fb6998311a`
- 內含 IKCONFIG，完整設定已解出存成 [`kernel.config`](kernel.config)

## Root / 隱藏相關設定

- `CONFIG_KSU=y`：KernelSU，內建功能有 adb_root、avc spoof、kernel_umount、mount_hide、selinux_hide、sulog、su_compat、shell_su_adb_only
- `CONFIG_SUKISU_KPM=y`：支援 KPM
- `CONFIG_KSU_SUSFS=y`：SUSFS **v2.4.0**，有開啟以下子功能：
  - SUS_PATH、SUS_MOUNT、SUS_KSTAT、SUS_MAP
  - SPOOF_UNAME、SPOOF_CMDLINE_OR_BOOTCONFIG
  - OPEN_REDIRECT、PRIORITY_UID_FILTER
  - HIDE_KSU_SUSFS_SYMBOLS、ENABLE_LOG

## 其他設定

- `HZ=250`，`LRU_GEN` 已開啟
- TCP：有編入 BBR，但預設擁塞控制仍是 `cubic`
- ZRAM 是模組（`=m`），預設 lzo-rle，支援 writeback 和 multi-comp
- `OVERLAY_FS=y`，`BPF_SYSCALL=y`，`NETFILTER_XT_MATCH_BPF=y`

## 刷入流程（anykernel.sh）

1. 詢問是否套用 KPM patch（`tools/patch_android`），用音量鍵選擇，10 秒沒按就預設「否」
   - 只給 **KowSU Pro** 用，用 Kpatch-Next 的要選「否」
2. 偵測到有 `init_boot_a` 分區時走 `split_boot` + `flash_boot`（只換 kernel，不動 ramdisk）；沒有的話走 `dump_boot` + `write_boot`
3. `BLOCK=boot`、`IS_SLOT_DEVICE=1`，不做機型檢查（`do.devicecheck=0`），不 patch vbmeta

## 附檔

- [`kernel.config`](kernel.config)：從 Image 解出的完整 `.config`
- [`SHA256SUMS`](SHA256SUMS)：zip 內每個檔案的雜湊值

原始 zip 有 22 MB，沒有放進 repo。
