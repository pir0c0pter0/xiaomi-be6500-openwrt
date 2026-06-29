# AGENTS.md

## Goal

Work on an OpenWrt port for Xiaomi BE6500 RN02.

## Local Layout

- `PORT_PLAN.md`: current phased plan and safety gates.
- `port-intel/`: sanitized hardware dump from the rooted stock unit.
- `upstream-openwrt/`: local OpenWrt clone for implementation work. This is intentionally ignored by git.

## Rules

- Do not flash NAND.
- Do not write to the router unless explicitly requested.
- First target is initramfs boot only.
- Keep stock A/B rootfs untouched until serial/U-Boot recovery and initramfs boot are proven.
- Use `port-intel/` as the source of truth for DT, MTD, GPIO, LEDs, Wi-Fi, and stock logs.
- Do not add per-unit ART, bdata, serials, MACs, SSIDs, private IPs, or secrets to this public repo.

## Verified Facts

- RN02 live DT compatible: `qcom,ipq5332-ap-mi01.2`, `qcom,ipq5332`.
- OpenWrt main has IPQ5332-related patches, but no official `ipq53xx` subtarget or RN02 profile yet.
- Stock exposes one active QCN9224 PCIe radio plus the IPQ5332 built-in 2.4 GHz radio.
- QCA8386 Ethernet/switch support is the likely early blocker.

