# BE6500 RN02 OpenWrt Port Plan

Status date: 2026-06-29.

## Current Ground Truth

- Device: Xiaomi BE6500, model RN02, stock MiWiFi/QSDK firmware, rooted.
- Live DT compatible: `qcom,ipq5332-ap-mi01.2`, `qcom,ipq5332`.
- Stock runtime: kernel 5.4.213, 32-bit armv7 userland, but silicon is ARMv8. OpenWrt should target aarch64.
- Flash: 128 MiB NAND, dual rootfs A/B:
  - `rootfs` at `0x01040000`, size `0x02a00000`
  - `rootfs_1` at `0x03a40000`, size `0x02a00000`
  - `0:ART` at `0x00e00000`, size `0x00200000`
  - `bdata` at `0x06440000`, size `0x00080000`
- Wi-Fi exposed by stock:
  - IPQ5332 built-in radio: 2.4 GHz, `phy4`
  - Active QCN9224 PCIe endpoint: vendor/device `17cb:1109`, 5 GHz 160 MHz, `phy5`
  - MLO virtual phy: `mld-phy0`
  - The live DT also has a second QCN9224 node, but it is `disabled` on this unit.
- Ethernet/switch: QCA8386 + QCA SSDK + proprietary NSS/PPE offload in stock.
- Buttons:
  - reset: GPIO 43, active-low, stock keycode `KEY_RESTART`
  - mesh: GPIO 16, active-low, stock keycode `BTN_9`
- LED:
  - stock exposes one RGB LED at `/sys/class/leds/rgb`
  - DT uses `pwm-rgb`; orange = PWM channel 0, white = PWM channel 2, period 50000 ns.

## Upstream Status Checked

- `openwrt-xiaomi/xmir-patcher` issue #171 is open and has no comments yet.
- OpenWrt `main` has:
  - `target/linux/qualcommax` with subtargets `ipq807x ipq60xx ipq50xx`, kernel 6.12.
  - IPQ5332-related patches under `qualcommax/patches-6.12`.
  - no `ipq53xx` subtarget and no RN02 device profile.
  - `target/linux/qualcommbe` exists, but currently only `ipq95xx`, kernel 6.18, source-only.
- OpenWrt code search found no RN02, no GL-BE9300/Flint 3 profile, no QCA8386 hit, and no QCN9224 hit in the official tree.

## Safety Rule

Do not flash NAND until initramfs boot is proven.

The first boot target must be an initramfs image over U-Boot/TFTP/serial. If there is no working serial/U-Boot recovery path, stop before any write to `rootfs` or `rootfs_1`.

## Port Phases

### 1. Build Tree

Create a separate OpenWrt working tree, not inside this router-notes repo.

Initial staging choice:

- Add temporary `qualcommax/ipq53xx` scaffolding, copied from the closest `ipq50xx` target structure.
- Enable only the minimum IPQ5332 kernel symbols needed to compile and boot initramfs.
- Keep Wi-Fi and NSS/PPE out of the first boot image.

Reason: official OpenWrt has IPQ5332 patches but no device target/profile to attach RN02 to.

### 2. Minimal RN02 DTS

Create `qcom-ipq5332-xiaomi-rn02.dts` from the live DT, but keep it minimal:

- model/compatible
- memory and reserved-memory required for boot
- UART
- NAND fixed partitions
- reset/mesh buttons
- one RGB LED, or skip LED if `pwm-rgb` is not upstream-friendly yet
- MDIO/switch nodes only as far as needed to attempt one Ethernet port
- PCIe/QCN9224 nodes can be present but disabled for first boot

First DTS goal is boot correctness, not full hardware.

### 3. Initramfs Gate

Build only initramfs first.

Pass condition:

- kernel boots on RN02
- serial console works
- root shell available
- `/proc/mtd` matches stock partition map
- UBI attach does not corrupt NAND
- at least one LAN path can be tested without writing flash

Fail condition:

- no serial log
- kernel panic before root shell
- NAND partitions differ from stock
- boot requires writing to inactive bank

### 4. Ethernet Gate

Ethernet is likely the hardest early blocker.

Start with the simplest target:

- get one copper port working without NSS/PPE
- do not chase 2.5G or hardware offload first
- QCA8386 may require downstream QCA SSDK or new driver work, because official OpenWrt tree currently has no obvious QCA8386 support

Pass condition:

- link up on one LAN port
- ping from host to initramfs
- no switch resets/panics under light traffic

### 5. Wi-Fi Gate

Only after initramfs + Ethernet:

- internal IPQ5332 2.4 GHz first
- active QCN9224 PCIe 5 GHz second
- MLO last

Use `wifi-board-data/` from `port-intel/` as reference. Do not publish or depend on per-unit ART/bdata unless a porter specifically needs structure, not values.

### 6. Persistent Install

Only after repeated initramfs boots and recovery are proven:

- write OpenWrt to the inactive A/B rootfs bank
- preserve the known-good stock bank
- switch `flag_boot_rootfs` only after verifying the inactive bank image layout
- document exact rollback command before first write

## Immediate Next Step

Clone OpenWrt into a separate working tree and create the minimal `ipq53xx` staging target + RN02 DTS skeleton. Do not build firmware images for flashing yet; only build initramfs.
