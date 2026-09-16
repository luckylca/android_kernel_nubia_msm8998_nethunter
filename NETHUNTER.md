# NetHunter kernel for Nubia Z17 (NX563J)

Branch `nethunter-22.2` = official LineageOS 22.2 kernel
(`LineageOS/android_kernel_nubia_msm8998` @ `cda6a278`) plus the NetHunter
capability patch set developed and hardware-verified on the Ubuntu-phase
reference platform (nubiaz17_linux project).

## Patches (on top of cda6a278)

| commit | what |
|---|---|
| 0001 | backport scoped guard/lock pointer helpers (cleanup.h) |
| 0002–0004 | touch/i2c stability fixes (resume delay, recovery reset, retry window) |
| 0005–0008 | WCN3990 Bluetooth: hci_uart stale-RX lock, btqca TLV split, LE host-supported tolerance |
| 0009 | **qcacld-3.0 monitor-mode packet injection** (monitor vdev + helper STA vdev, host-side verified on-device) |
| 0010–0012 | cfg80211 compat + ath9k_htc build fixes (external USB Wi-Fi) |

## Config

`nethunter/config/nethunter.fragment` (merged over `lineageos_nx563j_defconfig`):
USB configfs gadget (serial/ACM/ECM — HID and mass-storage already in
defconfig), WCN3990 HCIUART + RFCOMM/BNEP/HIDP, MAC80211 + ath9k_htc +
rtl8xxxu for external USB Wi-Fi, btusb USB BT dongles, SocketCAN gs_usb,
NFS client.

Note on RNDIS: legacy `f_rndis` is intentionally disabled (it would link
rndis.o into three composite objects on this 4.4 tree), but RNDIS gadget
functionality IS available via the Qualcomm GSI path — the defconfig ships
`CONFIG_USB_CONFIGFS_F_GSI=y` + `CONFIG_RNDIS_IPA=y`, and `gsi.rndis` was
verified creatable in configfs on the running kernel (2026-09-16). The
NetHunter `win,rndis*` USB attack modes work through it unchanged.

## Build

GitHub Actions (`.github/workflows/build.yml`): defconfig + fragment,
Android clang-r450784d / gcc-4.9 toolchains, reproducible timestamps.
Artifact: `Image.gz-dtb` + `kernel.config` + `SHA256SUMS`.

## Known limitations

- **2.4 GHz internal injection is unsafe**: WCN3990 firmware asserts
  (~31 s after injection, `ratectrl_11ac_`) when the helper STA vdev runs
  on a 2.4 GHz channel. Use 5 GHz (ch36+) only. 5 GHz injection verified
  stable 2026-09-16.
- External adapter drivers (ath9k_htc / rtl8xxxu / btusb / gs_usb) are
  kernel-side staged; hardware verification pending OTG hardware.
