# COSMIC Desktop xbps-src Templates — v1.0.14

Template xbps-src untuk membangun **COSMIC Desktop Environment** versi `epoch-1.0.14`
di Void Linux dari source code.

## Struktur Direktori

```
srcpkgs/
├── cosmic-applets/          # Applet panel (audio, baterai, jaringan, dll.)
├── cosmic-applibrary/       # App library overlay
├── cosmic-bg/               # Daemon wallpaper
├── cosmic-comp/             # Wayland compositor (inti)
├── cosmic-de-full/          # Metapackage — semua komponen
├── cosmic-de-minimal/       # Metapackage — komponen inti saja
├── cosmic-edit/             # Text editor
├── cosmic-files/            # File manager
├── cosmic-greeter/          # login and display manager
├── cosmic-launcher/         # App launcher (Super key)
├── cosmic-notifications/    # Notification daemon
├── cosmic-osd/              # On-screen display
├── cosmic-panel/            # Panel & dock
├── cosmic-player/           # Media player
├── cosmic-screenshot/       # Screenshot tool
├── cosmic-session/          # Session manager + start-cosmic
├── cosmic-settings/         # Aplikasi pengaturan sistem
├── cosmic-store/            # App store (butuh Flatpak)
├── cosmic-term/             # Terminal emulator
└── cosmic-workspaces-epoch/ # Workspaces overview
```

#### Package Yang Sudah Siap
1. cosmic-applets
2. cosmic-bg
3. cosmic-comp
4. cosmic-de-minimal
5. cosmic-edit
6. cosmic-files
7. cosmic-launcher
8. cosmic-panel
9. cosmic-sessions
10. cosmic-settings
11. cosmic-term

## Persiapan

### 1. Clone void-packages dan bootstrap

```bash
git clone --depth=1 https://github.com/void-linux/void-packages.git
cd void-packages
./xbps-src binary-bootstrap
```

### 2. Copy semua template ke srcpkgs/

```bash
cp -r srcpkgs/cosmic-* /path/to/void-packages/srcpkgs/
```

### 3. Isi checksum

Setiap template memiliki field `checksum` yang harus diisi sebelum build:

```bash
cd void-packages

for pkg in cosmic-comp cosmic-bg cosmic-launcher cosmic-applets \
           cosmic-panel cosmic-session cosmic-settings cosmic-files \
           cosmic-term cosmic-edit; do
    echo "==> Fetching $pkg..."
    ./xbps-src fetch $pkg
    xgensum -f srcpkgs/$pkg/template
done
```

## Langkah Build

Build harus dilakukan berurutan karena ada dependensi antar paket:

```bash
# 1. Compositor — tidak bergantung komponen lain
./xbps-src pkg cosmic-comp

# 2. Komponen inti — bisa paralel
./xbps-src pkg cosmic-bg
./xbps-src pkg cosmic-launcher
./xbps-src pkg cosmic-applets

# 3. Panel — butuh cosmic-applets
./xbps-src pkg cosmic-panel

# 4. Session — butuh semua di atas
./xbps-src pkg cosmic-session

# 5. Settings & aplikasi
./xbps-src pkg cosmic-settings
./xbps-src pkg cosmic-files
./xbps-src pkg cosmic-term
./xbps-src pkg cosmic-edit

# 6. Metapackage — selalu terakhir
./xbps-src pkg cosmic-de-minimal
```

## Install

```bash
# Install metapackage (otomatis tarik semua dependency)
sudo xbps-install --repository=hostdir/binpkgs cosmic-de-minimal

# Atau install manual per paket
sudo xbps-install --repository=hostdir/binpkgs \
    cosmic-comp cosmic-session cosmic-settings cosmic-applets \
    cosmic-panel cosmic-bg cosmic-launcher cosmic-files \
    cosmic-term cosmic-edit
```

## Setup & Konfigurasi

### 1. Aktifkan Services

```bash
# D-Bus — wajib
sudo ln -s /etc/sv/dbus /var/service

# elogind — wajib untuk session/seat management
sudo ln -s /etc/sv/elogind /var/service

# PipeWire — untuk audio (sangat direkomendasikan)
sudo ln -s /etc/sv/pipewire /var/service
sudo ln -s /etc/sv/wireplumber /var/service
```

### 2. Reboot

```bash
sudo reboot
```

### 3. Jalankan COSMIC

`cosmic-greeter` belum stabil di Void Linux, gunakan salah satu cara berikut:

**Cara A — Manual dari TTY (paling sederhana):**
```bash
start-cosmic
```

**Cara B — Auto-start saat login TTY:**

Tambahkan ke `~/.bash_profile` (bash) atau `~/.zprofile` (zsh):
```bash
if [ -z "$WAYLAND_DISPLAY" ] && [ "$XDG_VTNR" -eq 1 ]; then
    exec start-cosmic
fi
```

**Cara C — Menggunakan LIGHTDM:**
```bash
sudo xbps-install lightdm
sudo ln -s /etc/sv/lightdm /var/service
# Pilih session "COSMIC" dari menu login LIGHTDM
```

## Troubleshooting Build

Berikut daftar error yang umum ditemui beserta solusinya.

### `vinstall: cannot find 'LICENSE'`

File license di source COSMIC umumnya bernama `LICENSE.md`, bukan `LICENSE`.
Cek nama aslinya lalu sesuaikan template:

```bash
ls masterdir/builddir/<pkgname>-1.0.14/ | grep -i license

# Fix di template:
# Ganti: vlicense LICENSE
# Jadi:  vlicense LICENSE.md
```

### `build_style=meta is deprecated`

Terjadi pada metapackage. Ganti `build_style=meta` dengan `metapackage=yes`:

```bash
sed -i '/^build_style=meta/d' srcpkgs/<pkgname>/template
# Pastikan metapackage=yes sudah ada, jika belum tambahkan manual
```

### `Unable to find libclang`

Terjadi saat build package yang menggunakan `bindgen` (misal: `cosmic-settings`
yang butuh FFI binding untuk PipeWire). Tambahkan `clang` ke `hostmakedepends`:

```
hostmakedepends="pkg-config cargo rust clang"
```

### `cannot find -linput`

Missing `libinput`. Tambahkan ke `makedepends`:

```
makedepends="... libinput-devel"
```

### `Package oniguruma was not found`

Terjadi pada `cosmic-edit`. Tambahkan ke `makedepends`:

```
makedepends="... oniguruma-devel"
```

### `found a virtual manifest instead of a package manifest`

Terjadi saat package menggunakan struktur Cargo workspace (root `Cargo.toml`
adalah workspace, bukan package). Override `do_install` secara manual:

```bash
do_install() {
    vbin target/${RUST_TARGET}/release/${pkgname}
    vlicense LICENSE.md
}
```

Lihat `cosmic-panel` sebagai contoh — binary-nya ada di subdirektori
`cosmic-panel-bin/` dalam workspace.

### `binary already exists in destination`

Sisa build sebelumnya masih ada di destdir. Jalankan clean terlebih dahulu:

```bash
./xbps-src clean <pkgname> && ./xbps-src pkg <pkgname>
```

### Warning Rust compiler

Warning seperti `unused variable`, `dead_code`, `unused import` dari Rust
compiler **tidak perlu dikhawatirkan**. Itu berasal dari kode upstream dan
tidak mempengaruhi hasil build. Selama output diakhiri dengan:
```
Finished `release` profile [optimized] target(s) in Xm Xs
```
berarti build berhasil.

## Catatan Penting

| Item               | Keterangan                                                                                                                 |
|--------------------|----------------------------------------------------------------------------------------------------------------------------|
| **Rust toolchain** | Beberapa komponen memerlukan versi Rust spesifik (lihat `rust-toolchain.toml` di source). Install via `rustup` jika perlu. |
| **GPU**            | COSMIC **hanya berjalan di Wayland**. Pastikan driver GPU terinstall (`mesa-dri` untuk Intel/AMD, `nvidia` untuk NVIDIA).  |
| **cosmic-greeter** | Belum berfungsi di Void Linux — gunakan `start-cosmic` atau SDDM.                                                          |
| **cosmic-store**   | Butuh `flatpak` dan AppStream — tidak disertakan di `cosmic-de-minimal`.                                                   |
| **musl libc**      | Template ini dibuat untuk **glibc**. Untuk musl, beberapa crate mungkin perlu patch tambahan.                              |
| **checksum**       | Field `checksum` di setiap template **harus diisi** dengan `xgensum` sebelum build.                                        |
| **Upstream tag**   | Semua distfiles menggunakan tag `epoch-1.0.14` dari repo GitHub masing-masing.                                             |

## Referensi

- Upstream release: https://github.com/pop-os/cosmic-epoch/releases/tag/epoch-1.0.14
- Void community packages: https://codeberg.org/Bella109/void-packages
- xbps-src Manual: https://github.com/void-linux/void-packages/blob/master/Manual.md

---

<div align="center">

[@T4n-Labs](https://t4n-labs.github.io/site) · [@Gh0sT4n](https://gh0st4n.github.io/site)

</div>