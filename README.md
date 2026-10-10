# qemu-mint-live

QEMU Mint di GitHub Actions (`ubuntu-latest`) + Cloudflare Quick Tunnel untuk port `8006`.

Asal perintah:

```bash
docker run -it --rm --name qemu \
  -e "BOOT=mint" -p 8006:8006 \
  --device=/dev/kvm --device=/dev/net/tun \
  --cap-add NET_ADMIN \
  -v "${PWD:-.}/qemu:/storage" \
  --stop-timeout 120 docker.io/qemux/qemu
```

Di workflow diadaptasi ke mode `-d` agar tidak hang, dengan durasi hidup **6 jam**.

## Cara pakai

1. Buka tab **Actions** → **QEMU Mint Live** → **Run workflow**.
2. Tunggu 2-5 menit, buka log step `Start Cloudflare Quick Tunnel` atau `Summary` untuk URL `https://xxx.trycloudflare.com`.
3. Buka URL itu untuk akses QEMU `:8006`.
4. Job otomatis mati setelah ~5h40m (buffer dari `timeout-minutes: 360`).

## Catatan KVM

Runner `ubuntu-latest` x64 umumnya **ada `/dev/kvm`** (nested-virt, sejak 2024) tapi tidak otomatis writable.
Workflow mengaktifkan via udev rule `99-kvm4all` + `chmod 666` agar QEMU pakai hardware acceleration.
Jika `/dev/kvm` tidak writable, workflow otomatis fallback ke emulasi software/TCG (lambat) dengan warning.

Untuk performa penuh yang dijamin butuh larger runner 4-core+ atau self-hosted runner dengan KVM.

## File

- `.github/workflows/qemu.yml` — workflow utama (`BOOT=mint`, Quick Tunnel, keepalive 6 jam)
- `qemu/` — volume `/storage` (di runner saja, tidak di-commit)
