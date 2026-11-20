# LineageOS 19.1 — Infinix Smart 9 HD (X6532C)

Repositori kerja (*build workspace*) untuk membangun **LineageOS 19.1 (Android 12.1)**
bagi **Infinix Smart 9 HD (X6532C / MT6765)**. Repo ini hanya berisi *workflow*
GitHub Actions; kode device tree dan blob vendor diambil dari repositori `ryu4nn`
saat build berjalan.

Build workspace for **LineageOS 19.1 (Android 12.1)** on the **Infinix Smart 9 HD
(X6532C / MT6765)**. This repository carries only the GitHub Actions workflow; the
device tree and vendor blobs are pulled from the `ryu4nn` repositories at run time.

---

## Kenapa 19.1 / Why 19.1

Partisi `vendor` di perangkat ini adalah **Android 12 (SDK 31, VNDK 31)**. Treble
mewajibkan versi vendor ≤ versi platform, jadi platform ROM harus ≥ API 31 —
**LineageOS 19.1 (Android 12.1, API 32)** adalah target yang paling dekat dan
aman. Partisi `system` stok memang Android 14, tetapi yang mengikat adalah sisi
vendor.

*The device's `vendor` partition is **Android 12 (SDK 31, VNDK 31)**. Treble
requires vendor ≤ platform, so the ROM platform must be ≥ API 31 — **LineageOS
19.1 (Android 12.1, API 32)** is the closest safe target. The stock `system`
partition is Android 14, but the vendor side is what binds.*

---

## Sumber / Sources

| Path di build | Repositori | Branch |
| --- | --- | --- |
| `device/infinix/X6532C` | [`ryu4nn/android_device_infinix_X6532C`](https://github.com/ryu4nn/android_device_infinix_X6532C) | `lineage-19.1` |
| `vendor/infinix/X6532C` | [`ryu4nn/android_vendor_infinix_X6532C`](https://github.com/ryu4nn/android_vendor_infinix_X6532C) | `main` |

Manifest lokal yang dipakai workflow juga disimpan untuk referensi di
`local_manifests/X6532C.xml`.

---

## Menjalankan build / Running a build

1. Buka tab **Actions** → **Build LineageOS 19.1 (X6532C)** → **Run workflow**.
2. Input opsional:
   - **target** — target Soong, default `bacon` (zip OTA). Bisa `systemimage`,
     `otapackage`, dll.
   - **sync_only** — centang untuk hanya `repo sync` tanpa build (untuk mengukur
     ukuran source / memeriksa manifest).
3. Hasil ada di **Artifacts** (`lineage-19.1-X6532C-<run>`).

*Open **Actions** → **Build LineageOS 19.1 (X6532C)** → **Run workflow**. Resulting
files land under **Artifacts**.*

### Batasan runner / Runner limits

Runner gratis GitHub hanya menyediakan ~65 GB ruang kosong dan batas 6 jam per
job, sementara sinkronisasi + `out/` LineageOS 19.1 jauh lebih besar dari itu.
Workflow ini sudah menghapus toolchain yang tidak dipakai dan memakai
`repo sync --depth=1 -c`, tetapi build penuh bisa **kehabisan disk atau waktu**.
Kalau itu terjadi, opsi realistis: runner lebih besar (berbayar) atau runner
self-hosted.

*Free GitHub runners have ~65 GB free and a 6-hour job limit, while a LineageOS
19.1 sync + `out/` is far larger. This workflow trims unused toolchains and uses
`repo sync --depth=1 -c`, but a full build may still run out of disk or time. The
realistic fallbacks are a larger (paid) runner or a self-hosted one.*

---

## Catatan penting / Important notes

- Perangkat ini **tidak punya partisi `recovery`**; recovery ada di `vendor_boot`.
- Userspace stok **32-bit murni** (`ro.zygote=zygote32`), jadi tree dibangun
  `TARGET_ARCH := arm` tanpa 64-bit.
- Kernel bersifat **prebuilt** (5.10.237 GKI) — tidak ada source kernel yang
  dirilis untuk perangkat ini.
- Repositori ini belum menjamin ROM bisa boot; ini fondasi untuk iterasi
  bring-up, bukan rilis.

*This device has no `recovery` partition; recovery lives in `vendor_boot`. The
stock userspace is 32-bit only, so the tree builds `TARGET_ARCH := arm`. The
kernel is prebuilt (GKI 5.10.237) because no kernel source is published. This is a
bring-up foundation, not a boot guarantee.*
