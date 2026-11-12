# Xiaomi Redmi Note 10 5G (camellia)

Device configuration for the Xiaomi Redmi Note 10 5G. The Redmi Note 10T 5G and
the POCO M3 Pro 5G are the same hardware under different names and are covered
by this tree.

```
#
# Copyright (C) 2026 The LineageOS Project
#
# SPDX-License-Identifier: Apache-2.0
#
```

## Specifications

| | |
|---|---|
| SoC | MediaTek MT6833 Dimensity 700 |
| CPU | 2x2.2 GHz Cortex-A76 + 6x2.0 GHz Cortex-A55 |
| GPU | Mali-G57 MC2 |
| Memory | 4/6/8 GB LPDDR4X |
| Storage | 64/128 GB UFS 2.2, microSD |
| Display | 6.5" 1080x2400 IPS LCD, 90 Hz |
| Battery | 5000 mAh |
| Rear camera | 48 MP wide + 2 MP depth + 2 MP macro |
| Front camera | 8 MP |
| Shipped with | Android 11, MIUI 12 |

## Models covered

| Model | Name | SKU |
|---|---|---|
| M2103K19C | Redmi Note 10 5G | `camellia` |
| M2103K19I | Redmi Note 10T 5G | `camellia` |
| M2103K19G | Redmi Note 10 5G | `camellian` |
| M2103K19PI | POCO M3 Pro 5G | `camelliap` |
| M2103K19PG | POCO M3 Pro 5G | `camellianp` |

One build serves all five. Both touchscreen controllers, both fingerprint
readers, both backlight ICs, all five display panels and every camera module
combination are compiled in and selected at boot from the bootloader's hardware
IDs.

## Unofficial LineageOS 23.2

This tree is not part of official LineageOS, and no official build exists for
this device.

- Downloads and checksums for this fork: [Releases](../../releases)
- cristidclxvi's builds, which this fork is based on:
  [his Releases](https://github.com/cristidclxvi/android_device_xiaomi_camellia/releases)
- Installing: [docs/INSTALL.md](docs/INSTALL.md)
- Why the tree looks the way it does: [docs/NOTES.md](docs/NOTES.md)

### Status

| | |
|---|---|
| Boot, telephony, 5G, mobile data, SMS | works |
| VoLTE | works |
| Wi-Fi 2.4/5 GHz, Bluetooth, GPS, FM radio | works |
| Cameras, main and front | works |
| Depth and macro sensors | auxiliary, not app-visible, same as stock |
| Audio, sensors, fingerprint, IR blaster | works |
| 90 Hz display, touch, auto-brightness | works |
| LiveDisplay including Outdoor mode | works |
| NFC (`camellian` / `camellianp` only) | works on `camellian`; `camellianp` untested; hardware absent on other SKUs |
| Per-app firewall, Data Saver | works |
| Wi-Fi hotspot including tethered DNS | works |
| 32-bit apps (`armeabi-v7a`) | works |

Built `userdebug`, but not a debuggable build: `ro.debuggable=0`, `ro.secure=1`,
`ro.build.tags=release-keys`. `boot` and `vbmeta` are signed with a test key, so
**never relock the bootloader while this is installed**.

cristidclxvi tested his builds on one M2103K19C. This fork is tested on one
M2103K19G (`camellian`), dual SIM, on two UK carriers. The other three models
are expected to work and nothing is known to be wrong with them, but none has
been verified.

Builds of this fork are signed with this fork's own release keys, not
cristidclxvi's. Installing one over one of his builds, or the other way round,
needs a data wipe, and a recovery from the other build will warn that the
package signature does not verify.

## Building

The kernel is not in the LineageOS organisation, so it needs a local manifest.
The device tree below is this fork; the kernel, modules and IMS repositories
are cristidclxvi's, unchanged.
Save this as `.repo/local_manifests/camellia.xml` in a `lineage-23.2` tree:

```xml
<manifest>
  <project name="fieldmar/android_device_xiaomi_camellia"
           path="device/xiaomi/camellia" remote="github" revision="camellian-23.2" />
  <project name="cristidclxvi/android_kernel_xiaomi_camellia"
           path="kernel/xiaomi/camellia" remote="github" revision="lineage-23.2" />
  <project name="cristidclxvi/android_kernel_modules_xiaomi_camellia"
           path="kernel/xiaomi/vendor" remote="github" revision="lineage-23.2" />
  <project name="LineageOS/android_device_mediatek_sepolicy_vndr"
           path="device/mediatek/sepolicy_vndr" remote="github" />
  <project name="LineageOS/android_hardware_mediatek"
           path="hardware/mediatek" remote="github" />
  <project name="LineageOS/android_hardware_xiaomi"
           path="hardware/xiaomi" remote="github" />

  <!-- MediaTek IMS. Provides com.mediatek.ims, which binds the
       IRadio/imsAospSlot1|2 instances the MTK RIL registers. Without it no
       ImsService is bound and calls fall back to GSM instead of using VoLTE.
       Pinned to a SHA on purpose: the payload is a prebuilt APK that gets our
       platform signature, so it must not change under us. -->
  <project name="cristidclxvi/android_vendor_mediatek_ims"
           path="vendor/mediatek/ims" remote="github"
           revision="32a265afc6a297b20e8c8a4c870133de0e188884" />

  <!-- Kernel toolchain. LineageOS ships clang r547379 and newer; this 4.14
       tree needs r383902, which AOSP keeps only on its Android 12 branches. -->
  <project name="platform/prebuilts/clang/host/linux-x86"
           path="prebuilts/clang/host/linux-x86-r383902"
           remote="aosp" revision="refs/tags/android-12.1.0_r27"
           clone-depth="1" />
</manifest>
```

That branch of the toolchain also carries an `Android.mk` which the current
build system refuses to see under `prebuilts/`, and which would pull in test
binaries this tree does not ship. Nothing in it is meant to be built, so mark
the directory once after syncing:

```
touch prebuilts/clang/host/linux-x86-r383902/.find-ignore
```

Apply the patches in [`patches/`](patches). They change repositories the
LineageOS manifest already provides and they are not optional. Then:

```
source build/envsetup.sh
lunch lineage_camellia-bp4a-userdebug
mka bacon
```

Proprietary blobs are not included. Extract them from stock MIUI 14 V14.0.6.0,
either a running device or an unpacked ROM, with `./extract-files.py`.

Nine connectivity firmware files are the exception. `proprietary-files.txt`
pins them by SHA-1 to the gold set described under [Credits](#credits), and the
V14.0.6.0 image carries different versions of most of them. Take those from a
crDroid 14 camellia vendor image or from the gold ROM named there. Bluetooth,
MCU and Wi-Fi firmware must stay a matched set.

A build from these repositories is functionally identical to a release but not
bit-identical. `CONFIG_LTO_CLANG` and `CONFIG_CFI_CLANG` make the kernel
non-deterministic, the build date and version strings are baked into the system
properties, and released images are signed with private release keys that are
not published, so your own build gets whichever keys you sign it with.

## Related

- Kernel: [android_kernel_xiaomi_camellia](https://github.com/cristidclxvi/android_kernel_xiaomi_camellia) — 4.14.357-openela, branch `lineage-23.2`
- Connectivity modules: [android_kernel_modules_xiaomi_camellia](https://github.com/cristidclxvi/android_kernel_modules_xiaomi_camellia), branch `lineage-23.2`

The kernel branch started from MiCode's `camellian-t-oss` drop and has since
been merged up to the OpenELA 4.14.357 stable tag, so the MediaTek and Xiaomi
history is intact alongside upstream's. To see what was changed for this
device on top of the merge:

```
git log --oneline v4.14.357-openela..lineage-23.2
```

The Wi-Fi, Bluetooth, GPS, FM and WMT/FEM modules are built from the sources in
the modules repository rather than shipped as prebuilt binaries. `lsmod` on a
running device lists exactly what that produces.

## Credits

This tree is the work of three groups, one built on the other. None of it
should read as though it started here.

### dm700-devs: the original camellia tree

The tree began as a snapshot of the `lineage-23.2` branch of
[dm700-devs/device_xiaomi_camellia](https://github.com/dm700-devs/device_xiaomi_camellia)
(commit `f6e2bf4`). Of the 99 files in this repository's first commit, 86 were
byte-identical to that branch and 11 more were the same files with edits. That
import was a single commit, so the history here does not carry the original
authors. Their repository does, and it is the place to look for who wrote what.

Still carried from it, unmodified and with copyright headers intact:

- the power HAL extension under `power/`
- most of the vendor SELinux policy under `sepolicy/vendor/`
- several of the `rootdir/etc/init.*.rc` files
- the extract-utils scaffolding
- the resource overlays, which now live under `rro_overlays/` and were under
  `overlay/` in their tree

The top-level makefiles, `vendor.prop`, `manifest.xml` and
`proprietary-files.txt` also started as theirs and have been edited since.

Thanks to everyone who wrote that tree, among them rio004, Sushrut1101,
bengris32, Zinadin Zidan, SahilSonar, Matsvei Niaverau, Jerry Zhang, Arne
Coucheron and Arian.

The files under `configs/` come out of the stock MIUI vendor image and are the
same in any camellia tree for that reason. They arrived here by way of the
dm700-devs tree all the same.

Anyone can check exactly what is shared and what is not:

```
git clone https://github.com/dm700-devs/device_xiaomi_camellia /tmp/dm700
diff -rq . /tmp/dm700 --exclude=.git
```

### cristidclxvi: the Android 16 port and the kernel

[cristidclxvi](https://github.com/cristidclxvi/android_device_xiaomi_camellia)
brought the tree up to LineageOS 23.2: the patches under `patches/` that let
Android 16 run on a 4.14 kernel, `libinit` and the single build for all five
models, and the fixes for camera, Wi-Fi, Bluetooth, GPS, FM and VoLTE calling.
The kernel and connectivity module repositories under [Related](#related) are
his too. The kernel is Xiaomi's MediaTek source with the OpenELA 4.14.357
stable tag merged into it.

### dm700-devs and crDroid: the connectivity firmware set

The Bluetooth, MCU and Wi-Fi connsys firmware is pinned to the matched "gold"
set (`Redmi/vnd_gold/gold:12/TP1A.220624.014/V14.0.1.0.TNQCNXM`).

Choosing that set was Sushrut1101's work in the dm700-devs tree: commit
`8b32ce3`, "Update connectivity firmware from gold", October 2023, which
replaced an earlier firmware set to fix weak Bluetooth signal. crDroid 14
shipped it from the same commit in
[crdroidandroid/android_device_xiaomi_camellia](https://github.com/crdroidandroid/android_device_xiaomi_camellia),
and a crDroid build is where the files in this fork's builds were taken from.

The dm700-devs `lineage-23.2` branch kept the gold pins for one chip revision
only, and cristidclxvi's tree then moved all of them to the stock V14.0.6.0
firmware. This fork pins the full 2023 set again. The firmware itself is
MediaTek's and Xiaomi's.

### This fork

[fieldmar/android_device_xiaomi_camellia](https://github.com/fieldmar/android_device_xiaomi_camellia)
adds a small number of commits on top of cristidclxvi's branch:

- ViLTE turned off so the modem registers IMS, which makes VoLTE work on
  carriers where it did not
- the gold connectivity firmware restored as above, which fixes Bluetooth
  earbuds that dropped audio after a pause and lost the microphone on repeat
  calls
- stock `atcid`, `ipsec_mon` and the modem's `mdota` and `cacerts` files

`git log github/lineage-23.2..HEAD` lists them, with `github` being
cristidclxvi's remote.

## License

Apache-2.0, see [LICENSE](LICENSE). Two files under `prebuilt/` are not covered
by it:

- `prebuilt/dtbo.img` is Xiaomi's factory device tree overlay, shipped byte for
  byte because the published kernel sources do not contain what it is built
  from.
- `prebuilt/dtb/camellia.dtb` is GPL-2.0-only, compiled from the kernel device
  tree sources published in the kernel repository above.

Configuration files taken from the stock vendor image — the audio, media,
thermal and regional property files under `configs/` — remain the property of
Xiaomi and MediaTek and are redistributed unmodified.
