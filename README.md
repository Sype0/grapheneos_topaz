# GrapheneOS for the Redmi Note 12 4G (topaz / tapas) — unofficial

An **unofficial** port of GrapheneOS (release tag in
[`.github/workflows/build.yml`](.github/workflows/build.yml), Android 17) to the
Redmi Note 12 4G, built on GitHub Actions the same way as
[crdroid_android](https://github.com/Sype0/crdroid_android) and
[aosp_topaz](https://github.com/Sype0/aosp_topaz). Not affiliated with or supported
by the GrapheneOS project, and it does **not** have GrapheneOS's security
properties: the bootloader stays unlocked, verified boot is off, the kernel and
firmware are Xiaomi/Qualcomm's (no GrapheneOS kernel hardening, no MTE, no
secure element), and builds use test keys.

## What goes into the tree

- `repo init` of `GrapheneOS/platform_manifest` at the tag, plus
  [`local_manifests.xml`](local_manifests.xml).
- Device tree `Sype0/android_device_xiaomi_topaz` branch `grapheneos-17`
  (product `topaz`, `lunch topaz-cur-<variant>`): crDroid 17 tree without
  vendor/lineage, QCOM build glue in `aosp/`, port settings in `grapheneos/`.
- Kernel: BaranAspect's prebuilt (`lineage-23.0`) via `Sype0/android_kernel_xiaomi_topaz`
  branch `aosp-16`.
- CAF/QCOM HAL projects on LineageOS 24.0 branches.

## Changes to GrapheneOS

| Project | Branch | Why |
| --- | --- | --- |
| `external/hardened_malloc` | `Sype0/hardened_malloc` `topaz-17` | 2 GiB class regions: the kernel has a 39-bit address space, the 32 GiB default cannot be reserved |
| `bionic` | `Sype0/platform_bionic` `topaz-17` | `/vendor` and `/odm` programs (Qualcomm/Xiaomi blobs) use scudo; hardened_malloc also on userdebug |
| `packages/apps/SetupWizard2` | `Sype0/platform_packages_apps_SetupWizard2` `topaz-17` | No "lock your bootloader" screen or link, no "disable OEM unlocking" option |

**Never relock the bootloader.** A locked Xiaomi bootloader only boots
Xiaomi-signed images, so relocking with this OS installed bricks the phone.

### Pixel-only features (off on this device)

GrapheneOS turns most of these off by itself on non-Google hardware; the device
overlay (`grapheneos/overlay`) pins the rest:

- USB-C port security (charging-only when locked): needs the Pixel USB HAL
- Battery charge limit, Google Settings features: `Build.MANUFACTURER == "Google"` only
- Memory tagging (MTE) options: SM6225 is ARMv8.0
- PSDS for Pixel GNSS chips, alternative touchscreen mode, TPU access for Google apps
- Updater (OTA): only in official builds; update by flashing a newer zip
- Auditor can still verify *other* phones; this one can't be attested

## Build

Actions → *Build GrapheneOS (topaz)* → Run workflow. Runs that hit the 6 h
limit save `out/` to the `ci-out-state` release and chain (`chain_left`); on a
build error the job waits for a fix pushed here or to any project in the local
manifest (`wait_for_fix`). Errors: `ci-status` branch; live log: `ci-live`.

The ROM is published as a release: flash the zip from recovery
(`adb sideload`), after formatting data when coming from another ROM.
