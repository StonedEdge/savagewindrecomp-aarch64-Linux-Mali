# Savage Wind on Mali-G610 (RK3588), aarch64 Linux

The `mali-fixes` branch of this fork is the work that got Savage Wind running on an Arm handheld: a Retro Lite CM5
(Rockchip RK3588, Mali-G610). It was merged upstream as
[PR #212](https://github.com/Revivedx/SavageWindRecomp/pull/212) on 1 November 2026, so **upstream's `main` and `test`
have all of it plus later work** (FSR 3.1 upscaling, more fixes). Use upstream unless you need this exact branch.

## Tested setup

| | |
|---|---|
| Board | Retro Lite CM5: RK3588, Mali-G610 |
| OS | Ubuntu 24.04 aarch64 (Armbian base), Wayland desktop |
| GPU driver | Arm's Mali Vulkan driver (the "blob"), through the mali-vulkan-icd-wrapper |
| Settings | 720p, FSR at ultra quality; the GPU at its stock clock (an overclock caused glitches and shutdowns on low battery) |

**PanVK (Mesa's open Mali driver) is not tested**: one report so far is a black screen on start. Reports with the
log (the Vulkan device and driver lines, any missing features or device-lost errors) are welcome.

## What the branch changes (Mali is detected by its Vulkan vendor ID; other GPUs behave as before)

- Pass barriers fragment-only unless the pass's vertex stage may read fresh GPU writes.
- Attachments that are also sampled read through snapshots (blue walls, rocks and barricades), reused within one pass.
- Colour targets a pass does not draw into left out of it.
- BC4 textures decoded to R8 on the CPU, and on the GPU for the ones the game draws into (NPC speech bubbles).
- 16 and 8-bit integer colour targets take the low bits of the shader's output (Mali saturated them).
- Occlusion query results waited for only up to their submission (Kakariko fog).
- Shader global memory bound in two parts, under `maxStorageBufferRange`.
- The performance overlay shows the Mali GPU's load, clock and temperature.
- aarch64: a register clash in the assembly batches fixed.
- FidelityFX builds on Linux (upstream's `ffx_compat.h` fixes).

## Switches (environment variables, for comparing)

| Variable | Mali default | Other values |
|---|---|---|
| `SAVAGEWIND_VK_BC4_CPU` | 1: BC4 decoded on the CPU | 0: the GPU's own BC4 |
| `SAVAGEWIND_VK_BC4_DRAWN` | decoded on the GPU into R8 | `bc4`: a real BC4 image |
| `SAVAGEWIND_VK_SNAP_REUSE` | `pass` | `frame` |
| `SAVAGEWIND_VK_BARRIER_FRAG` | 1: fragment-only where safe | 0: full barriers |
| `SAVAGEWIND_VK_HEAP` | `import` | `export`, `dmaheap`: how the guest heap reaches the GPU |

## Building

As in [linux.md](linux.md): natively on the device, or `make appimage ARCH=aarch64` cross-compiled from x86-64 with
podman and QEMU.
