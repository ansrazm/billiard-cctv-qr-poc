# Billiard CCTV QR Access — Proof of Concept

POC untuk sistem "Web QR-Code & Live Stream CCTV Meja Billiard (Stand-Alone)". Mendemonstrasikan logika inti yang paling teknis dari project: **QR code generation, token berbasis waktu, dan auto cut-off saat sesi habis** — tanpa backend/server, murni client-side.

## Cara Pakai

1. Buka `index.html` (atau domain hasil deploy) di browser.
2. **Tampilan Admin** akan muncul otomatis (tidak ada `?token=` di URL):
   - Pilih nomor meja
   - Input durasi main (menit)
   - Tap **Generate QR Code**
3. QR code yang muncul meng-encode link unik berisi nomor meja + waktu expiry.
4. Scan QR pakai HP lain (atau tap "Buka sebagai Pelanggan" untuk test di device yang sama).
5. **Tampilan Pelanggan** akan:
   - Meminta izin kamera (mensimulasikan feed CCTV)
   - Menampilkan countdown timer sisa waktu
   - Otomatis mematikan kamera + video begitu waktu habis, menampilkan pesan token expired

## Yang Disimulasikan vs. Yang Real

| Bagian | Status di POC ini |
|---|---|
| QR code generation per meja + durasi | ✅ Real |
| Token berisi waktu expiry | ✅ Real |
| Countdown timer | ✅ Real |
| Auto cut-off saat waktu habis | ✅ Real (video track di-`stop()`) |
| Sumber video | ⚠️ Simulasi — pakai kamera device pelanggan, bukan CCTV RTSP asli |
| Validasi token di server | ⚠️ Simulasi — validasi terjadi di browser (client-side), bukan di server lokal |

## Arsitektur Rencana untuk Implementasi Produksi

```
[IP Camera RTSP/ONVIF]  →  [Mini PC lokal: MediaMTX]  →  [WebRTC/HLS]  →  [Browser HP Pelanggan]
     (Hikvision/                (convert RTSP ke
      TP-Link Vigi)              format web-friendly)
```

- **Media server**: [MediaMTX](https://github.com/bluenviron/mediamtx) (open-source) untuk convert stream RTSP kamera menjadi WebRTC (latency rendah) atau HLS (fallback, lebih stabil di jaringan kurang bagus).
- **Validasi token**: dipindah ke server lokal (mini PC) — token diverifikasi di server sebelum stream URL dikirim ke browser, bukan cuma dicek di frontend seperti di POC ini. Ini penting supaya token tidak bisa "dipalsukan" dengan mengubah waktu di client.
- **Fitur clip 30 detik**: buffer rolling di server pakai `ffmpeg`, dipicu dari tombol di halaman pelanggan.
- **Jaringan**: semua jalan di Wi-Fi lokal venue, tidak bergantung ke internet luar untuk streaming — hemat bandwidth.

## Deploy

File `index.html` ini statis, bisa langsung di-deploy ke:

- **Vercel** (disarankan — HTTPS otomatis & valid, penting karena butuh akses kamera):
  1. Import repo ini di [vercel.com/new](https://vercel.com/new), atau
  2. `npx vercel` dari folder ini
- **Cloudflare Pages**: `npx wrangler pages deploy .` dari folder ini, atau upload manual lewat dashboard.

> **Catatan penting:** fitur akses kamera (`getUserMedia`) butuh HTTPS yang valid. Kalau deploy ke hosting gratisan tanpa HTTPS proper, kamera tidak akan berfungsi.

## Tech Stack

- Vanilla HTML/CSS/JS, single file, tanpa dependency build
- [qrcodejs](https://github.com/davidshimjs/qrcodejs) via CDN untuk generate QR code
