# Xiaomi BE6500 RN02 OpenWrt Port

Public work-in-progress repo for bringing OpenWrt to the Xiaomi BE6500, model
RN02.

Current status:

- Stock rooted unit is available for read-only testing.
- Sanitized hardware intel is included in `port-intel/`.
- Local OpenWrt checkout lives in `upstream-openwrt/` and is ignored by git.
- No NAND flashing is planned until initramfs boot and recovery are proven.

Read `AGENTS.md`, `PORT_PLAN.md`, and `port-intel/FINDINGS.md` first.
