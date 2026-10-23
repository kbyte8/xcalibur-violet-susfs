# IMPORTANT: this ZIP is a GitHub Actions source project, NOT a flashable kernel

Do **not** flash `Xcalibur_SUSFS_GitHubActions_Builder_v1.zip` in OrangeFox. Upload its contents to a GitHub repository and run the included Action. The Action will produce a separate flashable kernel ZIP if the patch and build pass all checks.

# Xcalibur violet SUSFS build prepared from the Sep-30 working package

This folder was prepared from the user's known-working files:

- `KSU_Builder-3.3.0-18.zip`
- `Xcalibur_violet_KernelSU_20260930_0126.zip`

It adds one new GitHub Actions workflow:

`/.github/workflows/xcalibur-susfs.yml`

## What it intentionally keeps

- Device: Redmi Note 7 Pro (`violet`)
- Kernel source family: `Justadeayo/android_kernel_xiaomi_violet`
- Kernel branch: `17`
- Defconfig: `vendor/violet-perf_defconfig`
- Compiler: `clang-r416183b`
- The exact known-working Sep-30 `dtb`, `dtbo.img`, AnyKernel installer and tools

The generated ZIP replaces only the kernel `Image.gz` inside that proven package template. It does not use Zincore's kernel or device tree.

## Root implementation change

The original Sep-30 package uses Justadeayo's plain KernelSU fork. That fork does not expose the full SUSFS Kconfig/supercall interface needed by current SUSFS.

For the SUSFS build, this workflow uses `Justadeayo/sKernelSU` (the builder's existing `ReSukiSU` option), because it already contains the SUSFS Kconfig and SUSFS command dispatch. Hardware/kernel source remains Xcalibur branch 17.

## SUSFS patch set

The workflow pins the non-GKI Linux 4.14 SUSFS patch/hook scripts to:

`JackA1ltman/NonGKI_Kernel_Build_2nd@e2f895b`

This is a Sep-30-era patch revision and the hook script identifies itself as SUSFS 2.3.00+ for legacy kernels including 4.14.

## Enabled SUSFS features

- `CONFIG_KSU_SUSFS=y`
- suspicious path hiding
- suspicious mount hiding
- kstat spoofing
- uname spoofing
- KSU/SUSFS symbol hiding
- cmdline spoofing
- open redirect
- suspicious map hiding

SUSFS kernel logging is disabled in the build config.

## How to use

1. Create a new empty GitHub repository (private is fine).
2. Upload the **contents** of this folder to the repository root. Make sure `.github/workflows/xcalibur-susfs.yml` exists in GitHub.
3. Open **Actions**.
4. Select **Xcalibur Violet - ReSukiSU + SUSFS 2.3**.
5. Click **Run workflow**.
6. Wait for the build to finish.
7. Download the `xcalibur-violet-resukisu-susfs` artifact.
8. Do not flash it until you inspect the build log and the resulting ZIP.

No Telegram secrets are required.

## Before flashing

Keep the original working Sep-30 Xcalibur ZIP on the phone and PC. In OrangeFox, back up at least:

- Boot
- DTBO

If the experimental build does not boot, reflash the original Sep-30 Xcalibur ZIP or restore the Boot + DTBO backup.

## After a successful boot

First check in Termux:

```sh
uname -a
getenforce
su -c id
su -c 'dmesg | grep -i susfs'
```

Then install/use a manager compatible with the ReSukiSU/sKernelSU kernel. Do not install BRENE until root itself is confirmed working. After root and SUSFS are confirmed, install BRENE and reboot.

## Important

This is an experimental kernel build. The workflow is deliberately configured to **stop instead of compiling** if the 4.14 SUSFS patch does not apply cleanly or if required SUSFS options disappear from the final `.config`. If the GitHub Action fails, download the failure artifact/log and inspect the reject/build error instead of trying random patches.

## Exact-source caveat

The Sep-30 builder archive identifies the kernel repository, branch (`17`), defconfig, and compiler, but it does **not** record the exact `android_kernel_xiaomi_violet` commit SHA that was checked out during that historical Action run. The workflow therefore clones branch `17` and records the actual commit SHA in the generated `.provenance.txt` file.

The flashable package deliberately reuses the **exact Sep-30 DTB, DTBO and AnyKernel files** byte-for-byte. This reduces the hardware/device-tree variables, but the rebuilt kernel C source can still differ if branch `17` moved after Sep 30. If an old Action log or exact kernel-source SHA becomes available, pinning that SHA is the next improvement.

The workflow also switches the root core from the plain Justadeayo KernelSU fork to Justadeayo's `sKernelSU`/ReSukiSU integration because that tree already exposes the current SUSFS supercall/Kconfig interface. This is more than a one-line config change, although the hardware kernel source remains Xcalibur.
