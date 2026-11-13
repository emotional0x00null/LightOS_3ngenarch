# LightOS 3 NGENARCH

> **New:** apps can now be separate ELF programs with their own address space, run under a
> preemptive scheduler and show up as files under `/Apps` in the explorer.
> See [docs/MULTITASKING.md](docs/MULTITASKING.md).

> A bare-metal x86_64 operating system built from scratch — no Linux, no POSIX, pure metal.

![LightOS Boot](https://img.shields.io/badge/arch-x86%2032--bit-blue)
![Language](https://img.shields.io/badge/lang-C%2B%2B%20%2B%20ASM-orange)
![License](https://img.shields.io/badge/license-MIT-green)
![Boot](https://img.shields.io/badge/boot-GRUB%20Multiboot1-yellow)

---

## Features

- **Custom GUI** — window compositing, shadows, animations, maximize/resize
- **9 Applications** — Notepad, Files, Calculator, Settings, Terminal, Paint, Clock, SysInfo, LightSurf
- **LightSurf Browser** — real HTTP client with RTL8139 NIC + TCP/IP + DNS
- **2 Themes** — Luna Steel, ColdIceBar (frosted glass)
- **3 Wallpapers** — Luna Night, Dawn, Forest
- **VFS** — RAM-based filesystem (128 nodes)
- **ACPI** — real hardware shutdown & restart
- **Audio** — AC97 + PC Speaker fallback
- **Serial Debug** — COM1 UART logging
- **Developer Mode** — 10 dev tools (F4×5 at boot)

## Quick Start

```bash
# Install dependencies (Ubuntu/WSL2)
sudo apt install build-essential nasm grub-pc-bin grub-efi-amd64-bin grub-common e2fsprogs dosfstools fdisk util-linux ovmf

# Build
./build.sh

# Run with QEMU networking + persistent disk
qemu-system-x86_64 -drive file=lightos.img,format=raw,if=ide -boot c -m 256M -vga std \
  -device rtl8139,netdev=n0 -netdev user,id=n0
```

> **Not:** Kernel 64-bit long mode'da çalışır. `qemu-system-i386` TCG modunda
> long mode'u desteklemez ve triple fault ile sürekli resetlenmeye yol açar —
> mutlaka `qemu-system-x86_64` kullanın.

> **ÖNEMLİ — `lightos.img` hem açılış hem kalıcı depolama imajıdır.**
> Notepad'de yazdığın, Luigi'nin ürettiği ya da Files'tan oluşturduğun her
> şey CandleFS bölümünde tutulur. Aynı `lightos.img` dosyasını her başlatmada
> kullan ki uygulamalar ve kullanıcı dosyaları kalıcı kalsın. `build.sh` mevcut
> tanınan imajın CandleFS bölümünü koruyup yalnızca önyükleme bölümünü günceller.

`build.sh` ISO üretmez. ISO gerekiyorsa `BUILD_ISO=1 ./build.sh` kullan.
İmaj bölümleri ve önyükleme akışı [docs/BOOT-DISK.md](docs/BOOT-DISK.md)
içinde açıklanmıştır.

EDK II removable-media loader ile UEFI imajı üretmek için `EDK2_DIR` değişkenini
TianoCore EDK II kaynak ağacına ayarla. EDK2 ön yüzü GRUB EFI'yi zincirler;
kernel hâlâ Multiboot girişini kullanır. Ayrıntılar
[docs/EDK2-UEFI.md](docs/EDK2-UEFI.md) içinde.

## Documentation

For UEFI in QEMU, use OVMF with the same image: add
`-bios /usr/share/OVMF/OVMF_CODE.fd` to the QEMU command. Add
`-serial stdio -no-reboot -no-shutdown` to see startup and panic logs.
With OVMF variables and the EDK II loader, use `bash tools/run_uefi.sh lightos.img`.

See **[HOW-TO-USE.md](HOW-TO-USE.md)** for complete documentation.
Double-click supported `.png` files in Files to open the PNG Image Viewer;
format and size limits are listed in [docs/PNG.md](docs/PNG.md).
Double-click `.mp4` files to inspect their MP4 container metadata (duration,
tracks, codec labels, and video dimensions); this does not decode or play media.
See [docs/MP4.md](docs/MP4.md) for the supported metadata and current limits.

## Architecture

```
boot.asm → kernel/init.c → start_shell()
              ↓
         GUI Loop (shell.cpp)
              ↓
    ┌─────────┴──────────┐
    │                    │
 Windows              Drivers
 compositor        PS/2 | RTL8139
    │              UART | AC97
 9 Apps + 10         ↓
 DevTools           VFS (RAM)
```

### Binary separation (initial milestone)

The ISO now contains the kernel at `/boot/kernel.bin` and a separately built
Ring 3 ELF module at `/system/apps/hello.elf`. The current loader runs this
serial-console sample once at boot. Existing GUI apps and drivers are still
kernel-linked while their syscall/API dependencies are migrated. See
[`docs/USERLAND.md`](docs/USERLAND.md) for the ABI and current limits.

## Author

**exaspot (Xaef BTL) ** — 2026
