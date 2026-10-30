# Exynos 2100 video flickering — experimental build

Source: FlopKernel-Series/flop_exynos2100_kernel, branch floppy-main, SHA 4553117a5014928dff55595f46627e0d77bcf133.

Symptoms reported: Instagram Reels and TikTok alternate the first decoded frame with subsequent frames; YouTube works.

## Changes
1. DPU: treat `dma_fence_wait_timeout()` zero return as timeout and return `-ETIMEDOUT`, rather than treating it as success. This prevents presenting a potentially pending buffer after a genuine 600ms timeout. (Not confirmed as cause.)
2. MFC: compare height with `sbwc_dec_max_height` instead of max width. (Logic defect; causal role unproven.)
3. Experimental isolation: default `default_sbwc_mode=2` to disable **both** SBWC and SBWCL for MFC hardware decode. This is a diagnostic workaround, not a general performance improvement.

## Build
The branch replaces `.github/workflows/build.yml` with a single **BakaSU + SUSFS** build that emits an AnyKernel3 flashable ZIP (a ZIP inside the GitHub Actions artifact archive). A release is **not** published.

## BakaSU
- Root implementation: vendored **BakaSU**, based on [Baka-SU/BakaSU](https://github.com/Baka-SU/BakaSU), with Kbuild identity `REPO_NAME := BakaSU`, pinned version `v4.2.0-rc3` (upstream commit `239e1e8871b8fcd51a6e5b3002e0ba522fdd99fb`).
- Active symbols: `CONFIG_KSU_SUKI=y` (historical internal name), `CONFIG_KSU=y`, `CONFIG_KSU_MANUAL_HOOK=y`, `CONFIG_KSU_SUSFS=y`.
- Integrated kernel SUSFS: `fs/susfs.c`, `include/linux/susfs.h` (v2.3.0).
- `CONFIG_KSU_NEXT` and `CONFIG_KSU_XXKSU` are not selected. Multi-manager compatibility is disabled.
- `s` is the current build-script selector for the vendored BakaSU implementation. `bash do_build.sh sq` produces the BakaSU-SUSFS ZIP. Do not confuse this build selector with a second root implementation.
- **Important:** BakaSU is itself a derivative of KernelSU and retains `KSU` symbol names; this does **not** mean a separate KernelSU Next instance is installed.
- CI checks the effective `out/.config` for correct variant and SUSFS settings before uploading.


## Important
- Only install if device is Exynos 2100 and its codename is supported by the AnyKernel3 package: o1s, r9s, p3s, t2s. Confirm exact ROM, vendor compatibility, bootloader unlock, and rollback recovery path.
- This build is experimental and **has not been validated on physical hardware**.
- Take an appropriate full backup, especially boot and vendor_boot, and retain the last known-good kernel for recovery.
- SBWC being disabled may increase memory bandwidth and power use; do not treat this as a permanent fix without tests.
- Capture `adb shell su -c dmesg` while reproducing if issues persist.
- You can override the SBWC setting using the upstream AnyKernel3 `/cache/fk_feat` mechanism on supported recovery setups (`default_sbwc_mode=0`), or revert the init/main.c default and rebuild.

## Test
Compare Instagram Reels, TikTok, and YouTube at 60Hz and 120Hz; try Android Developer options > Disable HW overlays. Note whether a screen recording captures the flicker. Check kernel logs for DPP, DECON, fences, MFC, IOMMU, and shadow update failures.
