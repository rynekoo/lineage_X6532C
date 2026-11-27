# LineageOS 20.0 — Infinix Smart 9 HD (X6532C)

Build workspace for **LineageOS 20.0 (Android 13)** on the **Infinix Smart 9 HD
(X6532C / MT6765)**. This repository carries only the GitHub Actions workflow and
the release documentation; the device tree and the vendor blobs are pulled from
the `ryu4nn` repositories at run time.

Repositori kerja (*build workspace*) untuk membangun **LineageOS 20.0 (Android 13)**
bagi **Infinix Smart 9 HD (X6532C / MT6765)**. Repo ini hanya berisi *workflow*
GitHub Actions dan dokumentasi rilis; device tree dan blob vendor diambil dari
repositori `ryu4nn` saat build berjalan.

---

## Why 20.0 when the vendor is 12

The `vendor` partition is **Android 12 (SDK 31, VNDK 31)**. That is a
constraint on the *interface* the platform has to speak, not a cap on the
platform version — a newer platform talking to an older vendor is the normal
Treble upgrade path.

The device tree answers it in one place each:

- `PRODUCT_EXTRA_VNDK_VERSIONS := 31` installs the VNDK 31 snapshot from
  `prebuilts/vndk/v31`, so `/system/lib/vndk-31`, `/system/lib/vndk-sp-31` and
  `ld.config.31.txt` exist and the vendor linker namespace resolves its libraries
  exactly as under the stock Android 12 system. The platform itself is built at
  `BOARD_VNDK_VERSION := current` (33).
- The vendor partition is never rebuilt into the flash set. The blob repository
  exists to satisfy the build; `docs/FLASHING.md` does not flash it.

*Partisi `vendor` adalah **Android 12 (VNDK 31)**. Itu batasan pada antarmuka
yang harus dipakai platform, bukan batas versi platform: platform lebih baru
dengan vendor lebih lama adalah jalur upgrade Treble yang normal. Snapshot VNDK
31 dipasang lewat `PRODUCT_EXTRA_VNDK_VERSIONS := 31`, dan partisi `vendor` tidak
pernah ikut di-flash.*

---

## Sources

| Path in the build | Repository | Branch |
| --- | --- | --- |
| `device/infinix/X6532C` | [`ryu4nn/android_device_infinix_X6532C`](https://github.com/ryu4nn/android_device_infinix_X6532C) | `lineage-20.0` |
| `vendor/infinix/X6532C` | [`ryu4nn/android_vendor_infinix_X6532C`](https://github.com/ryu4nn/android_vendor_infinix_X6532C) | `main` |

The local manifest the workflow writes is kept for reference in
`local_manifests/X6532C.xml`.

---

## Running a build

**Actions** → **Build LineageOS 20.0 (X6532C)** → **Run workflow**.

| Input | Default | Meaning |
| --- | --- | --- |
| `target` | `droid` | Soong goal. `droid` produces raw images; `bacon` would produce a payload OTA that overwrites `vendor_boot` (and with it TWRP), so it is not the default. |
| `sync_only` | `false` | Sync the tree and stop, to measure the checkout. |
| `artifact_super` | `false` | Also ship `super.img.zst` (≈1.2 GB) for a whole-super flash. Off by default because it duplicates every partition image. |

### Artifacts

`lineage-20.0-X6532C-<run>` contains:

| File | Contents |
| --- | --- |
| `lineage-20.0-X6532C-<run>-images.zip` | `boot`, `dtbo`, `vbmeta`, `vbmeta_system`, `vbmeta_vendor`, `system`, `system_ext`, `product`, `super_empty`, deflated with `-9` (≈0.8–0.9 GB) |
| `SHA256SUMS` | digest of every file in the archive |
| `disk-monitor.log` | disk usage sampled every 10 minutes during the build |
| `super.img.zst` | only with `artifact_super: true` |

`vendor.img` and `vendor_boot.img` are deliberately excluded from both the
archive and the documentation: the vendor partition stays stock and TWRP lives in
`vendor_boot`. Flashing procedure: [`docs/FLASHING.md`](docs/FLASHING.md).

### Runner limits

A free runner has ~65 GB free and a 6-hour job limit. The workflow removes the
preinstalled toolchains, syncs with `--depth=1`, drops the `.repo` mirrors after
recording the manifest, and keeps only the two clang revisions the build actually
references. The GApps payload adds 483 MB to the checkout and 483 MB to the
images on top of that.

---

## Google apps

Included in the ROM, not a sideload. `tools/fetch-gapps.sh` in the device tree
downloads the pinned **MindTheGapps 13.0.0 arm** archive
(`sha256 9f0c3663872488acf98eb3dd548ed53a584f119161192dcd20f904b4cd05a37e`),
verifies it, and writes one `PRODUCT_COPY_FILES` line per payload file.

The arm (32-bit) archive is the correct one: userspace here is `armeabi-v7a` only
and `ro.vendor.product.cpu.abilist64` is empty, so arm64 libraries would never be
loaded. 467 MB lands in `product` and 16 MB in `system_ext`.

*GApps sudah termasuk di dalam ROM. Arsip arm (32-bit) yang dipakai karena
userspace perangkat ini murni `armeabi-v7a`.*

---

## Important notes

- The device has **no `recovery` partition**; recovery lives in `vendor_boot`.
  Nothing in this ROM writes there.
- Userspace is **32-bit only** (`ro.zygote=zygote32`), so the tree builds
  `TARGET_ARCH := arm` with a 64-bit binder.
- The kernel is **prebuilt** (GKI 5.10.237); no kernel source is published for
  this device.
- The generic ramdisk stays in `boot.img`. `init_boot_a/b` on this device is
  8 MiB of zeroes, so `BOARD_INIT_BOOT_IMAGE_PARTITION_SIZE` is deliberately
  undefined.
- VINTF enforcement is disabled for the bring-up build; the device compatibility
  matrix is generated but not wired in yet.

## Status

Structurally complete, statically verified against the firmware dump, **not yet
booted on hardware**. See the status section of the device tree README for the
open items.