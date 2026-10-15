# LineageOS 24.0 for OnePlus 15R (macan) — OSS kernel

Local manifest to build **unofficial** LineageOS 24.0 (Android 17) for the
OnePlus 15R (`macan`, CPH2767) with the kernel built **from source** instead
of the prebuilt stock kernel.

## What this is

- Common tree, kernel, devicetrees, modules and `hardware/oplus` come straight
  from the [LineageOS](https://github.com/LineageOS) org.
- The `macan` device tree and vendor are forks of
  [OnePlus-SM8850-Development](https://github.com/OnePlus-SM8850-Development)
  (by chandu078) with the changes below, until an official `macan` tree lands
  in LineageOS.

### Changes on top of the upstream macan tree

Device tree ([`lineage-24.0-oss`](https://github.com/cfernande1408/android_device_oneplus_macan/tree/lineage-24.0-oss)):

| Commit | Why |
|---|---|
| `macan: Build kernel from LineageOS sources` | Disable the prebuilt kernel and select `macan_dtbo_config` via `TARGET_KERNEL_BAZEL_FLAGS`, like fairlady/infiniti. |
| `macan: Ship face unlock algorithm library` | The common face HAL links against a device-specific `libstfaceunlockocl_uff` (DUMMY in common); the build fails `check_elf` without it. |
| `macan: Install ACDB where AGM expects it` | AGM looks for calibration in `/vendor/etc/acdbdata/alor_mtp`; without it the audio HAL aborts and the device bootloops. |
| `macan: Enable burn-in protection` | Shift status/navigation bar icons to reduce OLED burn-in. |

Vendor ([`lineage-24.0-oss`](https://github.com/cfernande1408/proprietary_vendor_oneplus_macan/tree/lineage-24.0-oss)):

| Commit | Why |
|---|---|
| `macan: Add face unlock algorithm library` | `libstfaceunlockocl_uff.so` taken from infiniti (TheMuppets), which pairs the same common face HAL with it. |

## Build

Requirements: Linux x86_64, ~400 GB free on ext4 (1 TB recommended), 32 GB RAM
(+ swap).

```bash
mkdir -p ~/android/lineage && cd ~/android/lineage
repo init -u https://github.com/LineageOS/android.git -b lineage-24.0 --git-lfs --no-clone-bundle
mkdir -p .repo/local_manifests
curl -L https://raw.githubusercontent.com/cfernande1408/local_manifests_macan/lineage-24.0/macan.xml \
  -o .repo/local_manifests/macan.xml
repo sync -c -j8 --force-sync --no-tags --no-clone-bundle

source build/envsetup.sh
source vendor/lineage/vars/aosp_target_release
lunch lineage_macan-${aosp_target_release}-userdebug
m bacon
```

The first `lunch` also syncs the kernel 6.12 build environment
(`kernel/platform/kernel-6.12`) through roomservice.

Tips:

- If `systemd-oomd` kills the build, stop it while building:
  `sudo systemctl mask --runtime systemd-oomd.service systemd-oomd.socket`
- With limited RAM use `m -j8 bacon` (or lower).
- To debug a bootloop, build with `export WITH_ADB_INSECURE=true` to get adb
  without authorization during boot. Never ship such a build.

## Credits

- [chandu078](https://github.com/chandu078) and the OnePlus SM8850 maintainers
  for the device trees and the LineageOS kernel work
- [LineageOS](https://lineageos.org)
- [TheMuppets](https://github.com/TheMuppets) for the vendor blobs
- OnePlus for the [kernel sources](https://github.com/OnePlusOSS)
