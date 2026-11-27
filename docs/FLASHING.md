# Flashing LineageOS 20.0 on the Infinix Smart 9 HD (X6532C)

This ROM replaces the **system side only**. The vendor partition, the vendor
policy and the recovery stay exactly as they are on the phone.

Read the two lines below before anything else; they are the whole safety story:

> **Never flash `vendor.img`.** It is the Android 12 reference this ROM is built
> against. Replacing it with a repacked copy gains nothing and loses the stock
> policy the blobs were extracted from.
>
> **Never flash `vendor_boot.img`.** TWRP lives there. Everything this ROM
> produces for that partition is a rebuild of the stock modules and would erase
> the recovery.

## What is in the artifact

`lineage-20.0-X6532C-<run>-images.zip`:

| File | Partition | Notes |
| --- | --- | --- |
| `boot.img` | `boot` | kernel + generic ramdisk, header v4 |
| `dtbo.img` | `dtbo` | stock device tree overlay |
| `vbmeta.img` | `vbmeta` | verification disabled, no AVB signing |
| `vbmeta_system.img` | `vbmeta_system` | |
| `vbmeta_vendor.img` | `vbmeta_vendor` | |
| `system.img` | `system` | LineageOS 20.0 platform |
| `system_ext.img` | `system_ext` | LineageOS system extensions |
| `product.img` | `product` | LineageOS product + Google apps |
| `super_empty.img` | — | only for `fastboot wipe-super`, not a flash target |

`SHA256SUMS` covers every file in the zip. The images are raw ext4 and are
deflate-compressed inside the archive; expect roughly 0.8–0.9 GB for the set.

`super.img` is **not** included by default. It contains `vendor` as well, so a
whole-super flash would replace the stock vendor partition. If you want it
anyway, dispatch the workflow with `artifact_super: true` and you get
`super.img.zst` (≈1.2 GB from 5 GiB).

## Requirements

- Unlocked bootloader (`fastboot flashing unlock`).
- `adb` and `fastboot` from platform-tools 34 or newer. `fastbootd` on the phone
  is what writes the logical partitions.
- The phone on the stock firmware it shipped with, so `vendor` is the original.

Check the slot before flashing. This unit was dumped with the bootloader on
slot `b`:

```sh
fastboot getvar current-slot
```

Every command below writes the **current slot** only, which is what you want.

## Flash

```sh
unzip lineage-20.0-X6532C-<run>-images.zip
cd lineage-20.0-X6532C-<run>-images
sha256sum -c ../SHA256SUMS

adb reboot bootloader
fastboot devices

# Physical partitions.
fastboot flash boot    boot.img
fastboot flash dtbo    dtbo.img
fastboot flash vbmeta  vbmeta.img

# Logical partitions live inside super and are only writable from fastbootd.
fastboot reboot fastboot
fastboot devices

fastboot flash system      system.img
fastboot flash system_ext  system_ext.img
fastboot flash product     product.img
fastboot flash vbmeta_system vbmeta_system.img
fastboot flash vbmeta_vendor vbmeta_vendor.img

# Android 13 on an Android 14 data layout is not an upgrade, it is a reinstall.
fastboot -w

fastboot reboot
```

`fastboot -w` erases `userdata` and `metadata`. It is not optional: the phone
ships with an Android 14 Go system, and the FBE keys and the ART image in
`/data` are not readable by an Android 13 framework.

### If the logical partitions are missing

If `fastboot flash system` answers `partition not found`, the super metadata is
gone or empty. Recreate it from the empty template and flash again:

```sh
fastboot wipe-super super_empty.img
```

That writes an empty super with the stock group layout. It does not touch
`vendor`; the partition is created empty and the stock vendor content is gone
only if you then flash something into it — so do not.

## Recovery

TWRP is still in `vendor_boot` and this ROM never writes there. To boot it:

```sh
adb reboot bootloader
fastboot boot twrp.img          # or hold power+volume down from off
```

Booting TWRP does not change the installed system. Do not "install recovery" from
inside TWRP to `boot` — on this device that overwrites the ramdisk the system
needs.

## First boot

The first boot takes 5–15 minutes on this SoC because the boot classpath is
compiled by the JIT (the ROM is built with `WITH_DEXPREOPT=false`). A black screen
with a backlight for the first two minutes is normal.

Google apps are preinstalled: Play Store, Play Services, Setup Wizard. If the
setup wizard never appears, the device tree domain that applies the touch
settings is permissive during bring-up and may be spewing denials; check:

```sh
adb shell dmesg | grep avc
adb logcat -d | grep -i -E 'gms|setupwizard|fatalfatal'
```

## Rollback

The stock firmware package (`X6532C-MF380...`) restores everything with SP Flash
Tool. Because this ROM never touches `vendor` or `vendor_boot`, a rollback that
only needs the three images is:

```sh
fastboot flash boot    stock-boot.img
fastboot flash system  stock-system.img      # fastbootd
fastboot flash product stock-product.img     # fastbootd
fastboot -w
```

If you kept a full dump, flashing `super.img` and `boot.img` from it returns the
phone to exactly the state it was dumped in.

## What is not covered yet

- No image from this tree has been booted on hardware yet. Everything is
  verified statically against the firmware dump.
- VINTF enforcement is disabled for the bring-up build and the device
  compatibility matrix is generated but not wired in.
- The `x6532c_boot` SELinux domain is permissive so the first boot can report
  the real denials for the Transsion touch nodes.