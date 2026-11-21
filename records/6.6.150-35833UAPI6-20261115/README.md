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
- `35833`：內建的 KernelSU 版本號（KowSU Pro 35833），**不是** Oplus 韌體版本；`UAPI6` 推測是 KSU UAPI 版本
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
- KernelSU 分支：**KowSU Pro**，版本 35833，預設管理器 `com.kowx712.kowsupro`；也認 SukiSU Ultra、ReSukiSU、KSU Next、官方 KernelSU 等管理器包名
- `CONFIG_SUKISU_KPM=y`：KPM 介面在未打補丁時只是 stub，要在刷入時選擇套用 KPM patch 才真正可用
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

## 和原廠 boot 的比較（2026-11-21）

原廠 `boot.img` 放在 release `Boot-stock`。

| 項目 | 原廠 | 這個 AK3 |
|---|---|---|
| boot.img SHA-256 | `ffa909cd23f2c83f1d2568031cbbdabc265935b79cae6740f9a96c6aab4197cc` | — |
| 核心 | `6.6.118-android15-8-gb1422e39c313-abogki552867473-4k` | `6.6.150-android15-8-g3b3662ca5829-abogki537459655-4k` |
| 建置者 | `kleaf@build-host` | `emira@Lotus` |
| 建置時間（uname） | 2026-09-04 | 2025-05-12（偽造） |
| 安全性修補 | 2026-11-01 | — |
| boot header | v4，kernel 是 LZ4 legacy 壓縮，ramdisk 是 0（ramdisk 在 init_boot） | |

fingerprint：`alps/hal_mgvi_64_64only_armv82/mgvi_64_64only_armv82:15/AP3A.240617.008/1794382964403:user/release-keys`

完整的 config 差異見 [`config-diff.txt`](config-diff.txt)，原廠 config 見 [`stock-6.6.118.config`](stock-6.6.118.config)。

**AK3 有、原廠沒有的：**
- KernelSU（KowSU Pro）、SUSFS v2.4.0、KPM（這是主要的改動）
- 核心版本 6.6.118 → 6.6.150，多了 32 個 LTS 小版本的上游修正
- 預設 qdisc 改成 fq_codel
- 加了 TTL/HL 修改的 target（`IP_NF_TARGET_TTL`、`IP6_NF_TARGET_HL`），可用來改熱點 TTL
- F2FS dedup、PROC_EVENTS/CONNECTOR、SafeSetID、ANDROID_WRAPFD 等零星選項

**原廠有、AK3 沒有的：**
- `OPLUS_KSTACK_LAZY_MAP`、`OPLUS_VMALLOC_PARTIAL_MAP`：Oplus 自家的記憶體優化（kernel stack 延遲映射）。原廠 Image 裡有 35 個 oplus 字串，AK3 只剩 6 個
- `MEMFD_ASHMEM_SHIM`
- `TLS`（kTLS 模組）、`STREAM_PARSER`

**兩邊完全一樣的：** HZ=250、LRU_GEN、預設 TCP 擁塞控制 cubic（兩邊都有編入 BBR 但都沒設成預設）、ZRAM、CFI、SCS、模組簽章設定。

**結論：** 這是一個 root 核心，不是效能核心。實際價值在於 KernelSU + SUSFS 帶來的 root 和隱藏能力。排程、記憶體、網路這些效能相關設定都沒有調整，還少了 Oplus 的記憶體優化。

## 除了 SUSFS / KPM 以外的差異（逐項評估）

| 項目 | 實際用途 | 評估 |
|---|---|---|
| `IP_NF_TARGET_TTL` / `IP6_NF_TARGET_HL` | 用 iptables 改 TTL/HL，避開電信商的熱點偵測 | **唯一實用的獨有功能**，原廠核心沒有 |
| `NINTENDO_FF` | 讓 Switch 手把在手機上可以震動 | 小眾 |
| `F2FS_FS_DEDUP` | Oplus 的 F2FS 去重複，要裝置樹有 `/soc/oplus,f2fs/compress_dedup` 才會啟用 | 原廠韌體沒有開，大概不會生效 |
| `PROC_EVENTS` / `CONNECTOR` | 程序事件通知（cn_proc） | 少數守護程式會用到 |
| `SECURITY_SAFESETID` | 限制 setuid 的 LSM | 要另外寫 policy 才有作用 |
| 預設 qdisc 改 fq_codel | 網路佇列 | 原廠也有內建 fq_codel，root 後用 sysctl 就能改 |
| 6.6.118 → 6.6.150 | 上游 LTS 修正 | 有，但同時少了 Oplus 的修補 |
| `ANDROID_WRAPFD`、`MMC_PASSWORDS`、`USB_XHCI_PCI_RENESAS` | — | 手機上沒有作用 |

KernelSU 本身的功能（adb_root、selinux_hide、mount_hide、kernel_umount、sulog）是 KowSU 程式碼的一部分，不是這個 GKI 獨有的，LKM 模式應該也有。

**如果 LKM 已經有 SUSFS：** 這個 GKI 唯一明確多出來的實用功能就是改 TTL。以 LKM 跑在原廠核心上反而比較好：保留 Oplus 的記憶體優化和原廠的 uname，也不用偽裝。

## 內建 SUSFS 和 LKM 版 susfs_guard 的差別

對照的模組是 `susfs_guard_lkm-android15-6.6.ko`：
- SHA-256 `a6ecb1331fc5baf1d4f82b60ab09736b3c3d391112ed5b9a1f1b9b36ca4e19ed`
- description `SUSFS guard LKM (susfs_guard_lkm) v2.3.0-gki`
- vermagic `6.6.127-4k-g46a034eca005-dirty`

| | GKI 內建（SUSFS v2.4.0） | LKM（susfs_guard v2.3.0） |
|---|---|---|
| 掛鉤方式 | 原始碼層級直接修改 VFS / proc 函式 | 執行時掛鉤：kprobe/kretprobe、tracepoint、替換 syscall table 指標、`stop_machine` 改寫核心程式碼 |
| 找符號 | 編譯時就連結好 | 執行時掃 kallsyms 和 BTF 推算 offset（kallrecon） |
| 效能開銷 | 不觸發時幾乎沒有 | stat/getdents/statx/maps/mountinfo 這些熱路徑每次呼叫都要經過 probe，有小額開銷 |
| 自我隱藏 | 不需要（沒有額外模組） | 要靠 hide_modules / hide_syms 把自己從 `/proc/modules`、kallsyms 藏起來 |
| 被偵測面 | 較小 | syscall table 被改、kprobe 痕跡，理論上偵測面較大 |
| 相容性 | 只能用在這顆核心 | 跟著原廠核心走；如果核心更新後 layout 變了，可能要等模組更新 |
| 功能 | sus_path/mount/kstat/map、open_redirect、uname、cmdline、avc spoof | 大致相同，另外多了 opath、create gate、hide_modules |

兩者功能大致相同，差別在實作方式：內建版比較乾淨、開銷較低，LKM 版的優點是可以留在原廠核心上。

## 耗電 / 功耗

- 兩邊 config 的完整差異裡，**沒有任何一項跟功耗相關**。CPU 頻率調節、cpuidle、HZ、排程器選項都完全相同。
- 字串比對發現，原廠核心的 HMBird 排程器（Oplus 自家排程器）比較新，多了 `hmbird_add_max_freq_qos_req`、`hmbird_remove_max_freq_qos_req`、`hmbird_cpu_uclamp_min_valid`、`hmbird_uclamp_dec`、`hmbird_sched_cluster`，也就是頻率 QoS 和 uclamp 的控制。**AK3 的 HMBird 版本較舊，少了這些。**
- MTK 機型的功耗主要由 vendor 模組（cpufreq、thermal）和 Oplus 的使用者空間服務決定，GKI 能影響的部分本來就不多。

**結論：沒有做功耗最佳化。** 頂多跟原廠差不多，而且因為 HMBird 較舊、少了 ks_lazy，有可能還略差。

## ZRAM 風險（待實機確認）

- 兩邊都是 `ZRAM=m`、`ZSMALLOC=m`，模組來自 system_dlkm，用的是原廠核心的建置金鑰簽章。
- AK3 核心用的是自己的自動產生金鑰，`MODULE_SIG_PROTECT` 的檢查也還在（Image 裡有 `exports protected symbol` / `Protected symbol` 字串）。
- 換上 AK3 核心後，原廠的 zsmalloc.ko 會被當成未簽章模組。如果它匯出的 `zs_*` 符號在 protected exports 清單裡，就會載入失敗，zram 跟著起不來。
- 依賴 zram 的記憶體管理模組（例如程序壓縮）會因此失效。

刷完後的檢查方式：`cat /proc/swaps` 要看到 `zram0`；`dmesg | grep -iE "protected|zsmalloc|zram"` 不應該出現錯誤。

## 設備綁定（硬體 ID 白名單）

**刷機包本身不做檢查**：`anykernel.sh` 裡是 `do.devicecheck=0`。限制是寫在**核心裡**，在開機時才檢查。

從反組譯看到的流程（Image 檔案偏移）：
1. **讀序號**：`early_param("oplusboot.serialno")`，handler 在 `0x1e42f10`，用 strscpy 存到全域 buffer。原廠核心沒有這個參數。
2. **讀硬體 ID**：檢查函式在 `0x1e42f94`。它讀 bootconfig 裡的 `androidboot.` + 以下這些 key（表在 `0x16f82b0`）：
   `chipid`、`cpuid`、`emmcid`、`hw.soc.id`、`psn`、`serialno`
3. **雜湊比對**：把 ID 做雜湊後，跟內建白名單比對。白名單在 `0x16f5fd0`，共 8928 bytes = **279 筆 32-byte 雜湊**（熵 7.98 bits/byte，279 筆互不相同）。
4. **防竄改**：用 CRC 對整張白名單做校驗（`0x727740`，長度 `0x22e0`），比對結果還會跟常數 XOR 混在一起，讓人難以直接 patch。
5. **故意當機**：比對不通過，或偵測到竄改，就執行 `str 0xDEADBEEF, [0x2c1b]` 這類寫入無效位址的指令，讓核心 panic，手機開不了機。

所以不在白名單裡的設備，**可以刷入，但開不了機**，會卡開機畫面或重啟循環。救援方式是用 fastboot 刷回原廠 boot（release `Boot-stock`）。
這也代表作者事先收集了白名單設備的硬體 ID 雜湊，判斷是付費或登記制。
