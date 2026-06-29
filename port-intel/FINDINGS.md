# Xiaomi BE6500 (RN02) — port intel dump

The device runs the **stock Xiaomi MiWiFi firmware** — it is NOT running a
community OpenWrt build. This data was pulled read-only over SSH from the rooted
stock unit to help anyone working on an OpenWrt port. All per-unit data (serial,
MACs, SSIDs, calibration secrets, keys) has been **redacted/removed**. The WiFi
board-data files are board-generic (shipped identically on every unit).

## Hardware (verified live, not from spec sheets)

| | |
|---|---|
| Device | Xiaomi BE6500, model **RN02**, `machid 0x8060001` |
| SoC | **Qualcomm IPQ5332** — DT compatible `qcom,ipq5332-ap-mi01.2`, Cortex-A53 |
| Userland (stock) | 32-bit **armv7** (`ipq53xx/ipq53xx_32`) — silicon is ARMv8, port should target aarch64 |
| RAM / Flash | 512 MiB / 128 MiB **NAND**, dual-boot A/B |
| WiFi 7 radios | **1× active QCN9224 over PCIe** + IPQ5332 built-in radio. The live DT also contains a second QCN9224 node, but it is `disabled` on this unit. **MLO enabled** (`cnss2.enable_mlo_support=1`) |
| Switch / Eth | **QCA8386** (qca_ssdk), 2.5GbE WAN + LAN, NSS/PPE HW offload (proprietary `qca_nss_*`) |
| Bootloader | U-Boot, `bootcmd=bootmiwifi`, env in `mtd13 0:APPSBLENV`, A/B flag `flag_boot_rootfs` |
| Stock firmware | OpenWrt **18.06 QSDK**, kernel **5.4.213** |
| LEDs | single RGB via `soc:pwm-rgb` (`/sys/class/leds/rgb`) |

## Why this matters for the OpenWrt port

- OpenWrt mainline already carries IPQ5332-related patches in `qualcommax`, but
  there is no official `ipq53xx` subtarget or RN02 device profile yet.
- The 5GHz radio is **QCN9224 (PCIe)** — an **ath12k-supported** family (sibling
  of QCN9274), *not* the "QCN6224" that TechInfoDepot lists. This gives the
  Wi-Fi 7 a real upstream path.
- The **`wifi-board-data/`** files (`qcn9224_bdwlan.b0002`, `ipq5332_bdwlan.b12`,
  `regdb.bin`, `rxgainlut*`, `firmware_rdp_feature.ini`) are the board-2.bin
  equivalents porters have reported missing from upstream linux-firmware.

## Known hard parts (set expectations)

- **NSS/PPE hardware offload** is proprietary (`qca_nss_ppe`, `qca_ssdk`) — not in
  mainline; expect lower wirespeed routing than stock until/if SFE is wired up.
- ath12k **MLO / tri-band** tuning is still maturing upstream.
- Big Qualcomm firmware blobs (`amss.bin`, `m3.bin`, `q6_fw*`, `Data.msc`) exist on
  the unit but are **not** included here (licensing). Available on request — they
  match the QSDK 11.x `IPQ5332/WIFI_FW/` set.

## Contents

- `device-tree.tar.gz` — full live `/sys/firmware/devicetree/base` (partitions,
  GPIO, pinctrl, PCIe radios, reserved-memory, switch). **The crown jewel.**
- `intel/` — mtd/partition/ubi layout, cmdline, cpuinfo, lsmod, iw list, gpio,
  scrubbed dmesg (wifi+eth), whitelisted nvram (`nvram-safe.txt`, no secrets).
- `wifi-board-data/` — board-generic WiFi calibration/config.

## NOT included (privacy)

Serial, MAC addresses, SSIDs, `nv_channel_secret`, `rand_key/nonce`,
`nv_device_id`, per-unit `0:ART` + `bdata` calibration partitions, LAN IPs.
Ask the owner if per-unit ART structure is needed for caldata bring-up.
