# PRD — Sistem Alarm Panic Button (PLAN B)

> **Versi:** 1.0  
> **Tanggal:** 27 Juni 2026  
> **Status:** Draft

---

## Ringkasan Eksekutif

Sistem Alarm Panic Button adalah solusi keamanan digital untuk kompleks perumahan yang memungkinkan warga mengirim sinyal darurat secara instan melalui aplikasi Android. Saat tombol panik ditekan, sistem secara simultan mengirim notifikasi ke seluruh warga, menampilkan alert di dashboard pos satpam, dan mengaktifkan sirine fisik di pos keamanan melalui Tuya Cloud API.

---

## Visi Produk

Menyediakan sistem respons darurat real-time yang menghubungkan warga, petugas keamanan, dan perangkat sirine dalam satu ekosistem terintegrasi — sehingga bantuan dapat datang dari berbagai pihak secara bersamaan dalam hitungan detik.

---

## Target Pengguna

| Pengguna | Deskripsi | Platform |
|----------|-----------|----------|
| **Warga Perumahan** | Penghuni yang membutuhkan bantuan darurat atau menerima informasi kejadian di lingkungan | Aplikasi Android |
| **Petugas Keamanan (Satpam)** | Petugas yang bertugas di pos keamanan untuk merespons kejadian darurat | Web Dashboard |
| **Administrator (Pengurus RT/RW)** | Pengelola yang mengatur data warga dan konfigurasi sistem | Web Dashboard |

---

## Pernyataan Masalah

1. Waktu respons keamanan di perumahan lambat karena bergantung pada komunikasi manual (teriak, telepon, ketuk pintu pos)
2. Tidak ada sistem notifikasi massal yang cepat ke seluruh warga saat terjadi kejadian darurat
3. Petugas keamanan tidak memiliki informasi lokasi pasti kejadian sehingga waktu pencarian terbuang
4. Warga sekitar lokasi kejadian sering tidak mengetahui adanya bahaya di dekat mereka

---

## Alur Komunikasi

```
WARGA (Android App)
    │
    │ Tekan Panic Button (2 detik)
    │ + Koordinat GPS terkunci
    │
    ▼
POST /api/panic ──────► CLOUD SERVER (Laravel)
                              │
                              ├──► [Jalur 1] Firebase Cloud Messaging
                              │         │
                              │         ▼
                              │    Semua HP Warga Berbunyi
                              │    + Peta Lokasi Kejadian
                              │
                              ├──► [Jalur 2] WebSocket (Laravel Reverb)
                              │         │
                              │         ▼
                              │    Dashboard Pos Satpam
                              │    ✓ Layar berkedip merah
                              │    ✓ Sirine browser berbunyi
                              │    ✓ Peta auto-zoom ke lokasi
                              │
                              └──► [Jalur 3] HTTP POST Tuya Cloud API
                                        │
                                        ▼
                                   Sirine Fisik 120dB
                                   di Pos Satpam Menyala
```

---

## Fitur — Aplikasi Android (Warga)

| ID | Fitur | Prioritas | Deskripsi |
|----|-------|-----------|-----------|
| F-A01 | Panic Button | P0 (MVP) | Tombol darurat lingkaran besar, long-press 2 detik dengan progress indicator. Mengunci koordinat GPS saat ditekan. |
| F-A02 | Pengambilan GPS | P0 (MVP) | FusedLocationProviderClient untuk lokasi akurat. Permission ACCESS_FINE_LOCATION. |
| F-A03 | Push Notification | P0 (MVP) | Menerima notifikasi darurat dari warga lain via FCM. High-priority agar tembus Doze mode. |
| F-A04 | Peta Lokasi Kejadian | P0 (MVP) | Google Maps SDK menampilkan marker lokasi kejadian darurat. |
| F-A05 | Registrasi & Login | P0 (MVP) | Pendaftaran: nama, nomor blok/unit, nomor HP. Login via token. |
| F-A06 | Batalkan Darurat | P0 (MVP) | Membatalkan alarm dalam 10 detik pertama untuk menangani false alarm. |
| F-A07 | Riwayat Kejadian | P1 | Daftar kejadian darurat sebelumnya dengan timestamp dan status penanganan. |
| F-A08 | Profil Pengguna | P1 | Edit data pribadi dan pengaturan notifikasi. |

## Fitur — Web Dashboard (Pos Satpam)

| ID | Fitur | Prioritas | Deskripsi |
|----|-------|-----------|-----------|
| F-W01 | Dashboard Real-time | P0 (MVP) | Tampilan utama monitoring. Koneksi WebSocket (Laravel Reverb) ke server. |
| F-W02 | Visual Alert | P0 (MVP) | Layar berkedip merah (overlay full-screen, alternasi merah-putih 500ms) saat ada darurat. |
| F-W03 | Alarm Sound Browser | P0 (MVP) | Memutar suara sirine via Web Audio API saat menerima sinyal darurat. |
| F-W04 | Auto-zoom Peta | P0 (MVP) | Peta (Leaflet.js) otomatis zoom ke koordinat GPS pengirim sinyal. |
| F-W05 | Login Petugas | P0 (MVP) | Autentikasi petugas keamanan. |
| F-W06 | Daftar Kejadian Aktif | P0 (MVP) | List kejadian darurat yang sedang berlangsung. |
| F-W07 | Tombol Respons | P0 (MVP) | Petugas menekan saat sudah merespons / tiba di lokasi. |
| F-W08 | Riwayat Kejadian | P1 | Log semua kejadian dengan filter tanggal dan pagination. |
| F-W09 | Manajemen Warga | P1 | CRUD data warga, approve/reject pendaftaran baru. |

## Fitur — Backend (Cloud Server)

| ID | Fitur | Prioritas | Deskripsi |
|----|-------|-----------|-----------|
| F-B01 | API Panic | P0 (MVP) | `POST /api/panic` — terima koordinat GPS + user ID, simpan ke DB, trigger 3 jalur paralel. |
| F-B02 | WebSocket Server | P0 (MVP) | Laravel Reverb untuk koneksi real-time ke web dashboard. Broadcast event `PanicTriggered`. |
| F-B03 | FCM Integration | P0 (MVP) | Kirim push notification high-priority ke FCM topic `perumahan_{id}`. |
| F-B04 | Tuya Cloud API | P0 (MVP) | HTTP POST ke Tuya untuk mengaktifkan sirine. `{"code": "alarm_switch", "value": true}`. |
| F-B05 | Autentikasi | P0 (MVP) | Laravel Sanctum — token-based auth untuk Android app, session-based untuk web dashboard. |
| F-B06 | Database | P0 (MVP) | MySQL — tabel users, panic_events, event_responses. |
| F-B07 | API Batalkan | P0 (MVP) | `POST /api/panic/{id}/cancel` — stop sirine, kirim notifikasi pembatalan ke semua. |
| F-B08 | API Respons | P0 (MVP) | `PATCH /api/panic/{id}/respond` dan `/resolve` — update status penanganan. |

---

## Persyaratan Teknis

| Komponen | Teknologi |
|----------|-----------|
| Backend | Laravel 11 (PHP 8.2+) |
| Database | MySQL 8.0+ |
| Real-time | Laravel Reverb (WebSocket) |
| Web Dashboard | Next.js 14+ (TypeScript) |
| Android App | Kotlin, min SDK 24 (Android 7.0), target SDK 34+ |
| Push Notification | Firebase Cloud Messaging (FCM) |
| IoT Integration | Tuya Cloud API |
| Maps (Android) | Google Maps SDK |
| Maps (Web) | Leaflet.js + OpenStreetMap |
| Server | VPS Ubuntu 22.04 LTS, 2 vCPU, 2GB RAM, 30GB SSD |
| Domain | .id TLD |
| SSL | Let's Encrypt (wajib untuk WSS) |
| Auth | Laravel Sanctum |

---

## Persyaratan Non-Fungsional

| Aspek | Requirement |
|-------|-------------|
| **Latensi** | Maksimal 3 detik dari tekan tombol hingga notifikasi diterima |
| **Ketersediaan** | 99.5% uptime — sistem keamanan kritis |
| **Keamanan** | HTTPS/WSS wajib, token auth dengan expiry, rate limiting pada endpoint panic |
| **Skalabilitas** | Mendukung 1 perumahan (100-500 unit) untuk MVP |
| **Kompatibilitas** | Android 7.0+ (Nougat) — mayoritas perangkat di Indonesia |
| **Offline** | Tampilkan pesan error jika tidak ada koneksi internet |
| **False Alarm** | Long-press 2 detik + cancel window 10 detik + rate limit 1 panic / 30 detik per user |

---

## Batasan Lingkup (Out of Scope)

- Pembelian, instalasi, dan konfigurasi hardware sirine (Almo Tuya WiFi Smart Siren)
- Infrastruktur jaringan WiFi/LAN di Pos Satpam
- Aplikasi iOS
- Panggilan telepon darurat otomatis ke polisi/pemadam
- Integrasi dengan instansi pemerintah
- Dukungan multi-perumahan (versi awal untuk 1 perumahan)
- IP Horn Speaker outdoor dan lampu strobo di gang (Plan A feature)
- Mode siaran manual / broadcast suara petugas (Plan A feature)

---

## Metrik Keberhasilan

| Metrik | Target |
|--------|--------|
| Waktu respons end-to-end | < 3 detik |
| Delivery rate notifikasi FCM | 100% perangkat terdaftar |
| Aktivasi sirine fisik | < 5 detik setelah panic ditekan |
| False alarm rate | < 5% dari total alarm |
| Waktu respons petugas | < 2 menit setelah alarm |
