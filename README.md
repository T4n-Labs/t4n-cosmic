# COSMIC Desktop xbps-src Templates — v1.0.14

Template xbps-src untuk membangun **COSMIC Desktop Environment** versi `epoch-1.0.14`
di Void Linux dari source code.

## Struktur Direktori

```
srcpkgs/
├── cosmic-desktop/          # Metapackage (dependen semua komponen)
├── cosmic-comp/             # Wayland compositor (inti)
├── cosmic-session/          # Session manager + start-cosmic
├── cosmic-settings/         # Aplikasi pengaturan sistem
├── cosmic-applets/          # Applet panel (audio, baterai, jaringan, dll.)
├── cosmic-panel/            # Panel & dock
├── cosmic-bg/               # Daemon wallpaper
├── cosmic-launcher/         # App launcher (Super key)
├── cosmic-files/            # File manager
├── cosmic-term/             # Terminal emulator
└── cosmic-edit/             # Text editor
```

Template Yang Sudah Bisa Digunakan :
1. cosmic-comp
2. cosmic-applets

## Persiapan

### 1. Clone void-packages dan bootstrap
```bash
git clone --depth=1 https://github.com/void-linux/void-packages.git
cd void-packages
./xbps-src binary-bootstrap
```

### 2. Copy semua template ke srcpkgs/
```bash
# Dari direktori ini:
cp -r srcpkgs/cosmic-* /path/to/void-packages/srcpkgs/
```

## Langkah Build

### Isi checksum terlebih dahulu
Setiap template memiliki baris `checksum="<sha256sum-here>"`.
Isi dengan perintah berikut untuk masing-masing paket:

```bash
cd void-packages

# Contoh untuk cosmic-comp:
./xbps-src fetch cosmic-comp
xgensum -f srcpkgs/cosmic-comp/template

# Ulangi untuk semua komponen:
for pkg in cosmic-comp cosmic-session cosmic-settings cosmic-applets \
           cosmic-panel cosmic-bg cosmic-launcher cosmic-files \
           cosmic-term cosmic-edit; do
    ./xbps-src fetch $pkg
    xgensum -f srcpkgs/$pkg/template
done
```

### Build urutan yang benar (dependency order)

Build harus dilakukan berurutan karena ada dependensi antar paket:

```bash
# 1. Compositor (tidak bergantung komponen lain)
./xbps-src pkg cosmic-comp

# 2. Komponen inti lainnya (bisa paralel)
./xbps-src pkg cosmic-bg
./xbps-src pkg cosmic-launcher
./xbps-src pkg cosmic-applets

# 3. Panel (butuh applets)
./xbps-src pkg cosmic-panel

# 4. Session (butuh semua di atas)
./xbps-src pkg cosmic-session

# 5. Settings & aplikasi
./xbps-src pkg cosmic-settings
./xbps-src pkg cosmic-files
./xbps-src pkg cosmic-term
./xbps-src pkg cosmic-edit

# 6. Metapackage terakhir
./xbps-src pkg cosmic-desktop
```

Atau build sekaligus (xbps-src akan resolve urutan otomatis):
```bash
./xbps-src pkg cosmic-desktop
```

### Install hasil build

```bash
# Install dari repo lokal:
sudo xbps-install --repository=hostdir/binpkgs cosmic-desktop

# Atau install per paket:
sudo xbps-install --repository=hostdir/binpkgs cosmic-comp \
    cosmic-session cosmic-settings cosmic-applets cosmic-panel \
    cosmic-bg cosmic-launcher cosmic-files cosmic-term cosmic-edit
```

## Aktifkan Services (wajib)

```bash
# D-Bus (sistem)
sudo ln -s /etc/sv/dbus /var/service

# elogind (session/seat management)
sudo ln -s /etc/sv/elogind /var/service

# PipeWire untuk audio (opsional tapi direkomendasikan)
sudo ln -s /etc/sv/pipewire /var/service
sudo ln -s /etc/sv/wireplumber /var/service

# Reboot setelah mengaktifkan service
sudo reboot
```

## Jalankan COSMIC

Karena `cosmic-greeter` belum stabil di Void Linux, jalankan secara manual:

```bash
# Dari TTY setelah login:
start-cosmic

# Atau tambahkan ke ~/.bash_profile / ~/.zprofile untuk auto-start:
if [ -z "$WAYLAND_DISPLAY" ] && [ "$XDG_VTNR" -eq 1 ]; then
    exec start-cosmic
fi
```

Jika menggunakan SDDM atau display manager lain:
```bash
sudo xbps-install sddm
sudo ln -s /etc/sv/sddm /var/service
# Pilih session "COSMIC" dari menu login
```

## Catatan Penting

| Item               | Keterangan                                                                                                                 |
|--------------------|----------------------------------------------------------------------------------------------------------------------------|
| **Rust toolchain** | Beberapa komponen memerlukan versi Rust spesifik (lihat `rust-toolchain.toml` di source). Install via `rustup` jika perlu. |
| **GPU**            | COSMIC **hanya berjalan di Wayland**. Pastikan driver GPU terinstall (`mesa-dri` untuk Intel/AMD, `nvidia` untuk NVIDIA).  |
| **cosmic-greeter** | Belum berfungsi di Void Linux — gunakan `start-cosmic` atau SDDM.                                                          |
| **musl libc**      | Template ini dibuat untuk glibc. Untuk musl, beberapa crate mungkin perlu patch tambahan.                                  |
| **checksum**       | Field `checksum` di setiap template **harus diisi** dengan `xgensum` sebelum build.                                        |
| **Upstream tag**   | Semua distfiles menggunakan tag `epoch-1.0.14` dari repo GitHub masing-masing.                                             |

## Referensi

- Upstream: https://github.com/pop-os/cosmic-epoch/releases/tag/epoch-1.0.14
- Void maintainer (community): https://codeberg.org/Bella109/void-packages
- xbps-src Manual: https://github.com/void-linux/void-packages/blob/master/Manual.md

---

<div align="center">

[@T4n-Labs](https://t4n-labs.github.io/site) · [@Gh0sT4n](https://gh0st4n.github.io/site)

</div>