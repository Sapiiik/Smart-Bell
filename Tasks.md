# Tasks — Sistem Alarm Panic Button (PLAN B)

> **Versi:** 1.0  
> **Tanggal:** 27 Juni 2026  
> **Total Estimasi:** ~5.5 Minggu

---

## Pembagian Peran Tim

| Anggota | Peran | Tanggung Jawab Utama |
|---------|-------|----------------------|
| **Person A** | Backend Developer | Laravel API, database, WebSocket (Reverb), Tuya API, FCM server-side, deployment VPS |
| **Person B** | Frontend Web Developer | Next.js dashboard, WebSocket client, peta Leaflet, UI alert/sirine, responsive |
| **Person C** | Android Developer | Kotlin app, panic button, GPS, FCM client, Google Maps SDK, UI/UX Android |

> Semua anggota saling membantu di area overlap. Person A sebagai pemilik semua integrasi pihak ketiga karena ini titik kritis sistem.

---

## Fase 0 — Persiapan (Minggu 1)

| ID | Task | PJ | Estimasi | Dep |
|----|------|----|----------|-----|
| 0.1 | Setup repository Git (monorepo), tentukan branching strategy (GitFlow / trunk-based) | Semua | 0.5 hari | — |
| 0.2 | Setup VPS Ubuntu 22.04: install PHP 8.2, Composer, MySQL 8, Nginx, Node.js, Certbot | A | 1 hari | — |
| 0.3 | Buat akun Tuya IoT Developer, daftarkan device sirine, catat Client ID & Secret | A | 0.5 hari | — |
| 0.4 | Buat project Firebase Console, aktifkan FCM, download `google-services.json` & catat server key | A | 0.5 hari | — |
| 0.5 | Scaffold project Laravel (`laravel new backend`), install Sanctum, Reverb, konfigurasi `.env` | A | 0.5 hari | 0.2 |
| 0.6 | Scaffold project Next.js (`npx create-next-app web-dashboard --typescript`), install dependencies | B | 0.5 hari | — |
| 0.7 | Setup project Android Studio (Kotlin, Gradle KTS), tambah dependencies (Retrofit, Hilt, Maps, FCM) | C | 0.5 hari | 0.4 |
| 0.8 | Review & finalisasi PRD, StyleGuide, API contract bersama tim | Semua | 0.5 hari | — |

---

## Fase 1 — MVP Core: Alur Panic End-to-End (Minggu 2–3)

> Fokus: satu alur lengkap dari tekan tombol → semua notifikasi terkirim → petugas merespons.

### Backend (Person A)

| ID | Task | Estimasi | Dep | Ref PRD |
|----|------|----------|-----|---------|
| 1.1 | Buat migration & model: `users`, `panic_events`, `event_responses` | 1 hari | 0.5 | F-B06 |
| 1.2 | API autentikasi: register, login, logout (Laravel Sanctum) + middleware auth | 1.5 hari | 1.1 | F-B05 |
| 1.3 | API `POST /api/panic`: validasi, simpan koordinat GPS ke DB, dispatch 3 job paralel | 1 hari | 1.1 | F-B01 |
| 1.4 | Job `SendFcmNotification`: kirim push notif ke FCM topic `perumahan_{id}` (high priority) | 1 hari | 0.4, 1.3 | F-B03 |
| 1.5 | Setup Laravel Reverb, buat event `PanicTriggered`, broadcast ke channel presence | 1 hari | 1.3 | F-B02 |
| 1.6 | Service `TuyaService`: autentikasi Tuya OAuth2, command `alarm_switch=true` ke device | 1.5 hari | 0.3, 1.3 | F-B04 |
| 1.7 | API `POST /api/panic/{id}/cancel`: stop sirine (Tuya off), broadcast `PanicCancelled`, notif FCM pembatalan | 0.5 hari | 1.4, 1.5, 1.6 | F-B07 |
| 1.8 | API `PATCH /api/panic/{id}/respond` dan `/resolve`: update status, broadcast event | 0.5 hari | 1.3 | F-B08 |

### Android (Person C)

| ID | Task | Estimasi | Dep | Ref PRD |
|----|------|----------|-----|---------|
| 1.9 | UI halaman Login & Register (Material Design, sesuai StyleGuide) | 1.5 hari | — | F-A05 |
| 1.10 | Panic Button UI: lingkaran 200dp, animasi pulse, long-press 2 detik, progress ring | 1.5 hari | — | F-A01 |
| 1.11 | GPS Service: FusedLocationProviderClient, permission handling, fallback last known location | 1 hari | — | F-A02 |
| 1.12 | Integrasi API: Retrofit client, interceptor token, panggil `POST /api/panic` dengan koordinat | 1 hari | 1.2, 1.3 | F-A01 |
| 1.13 | FCM client: `FcmService.kt` extends FirebaseMessagingService, tampilkan notifikasi darurat | 1 hari | 0.4, 1.4 | F-A03 |
| 1.14 | Peta lokasi kejadian: Google Maps SDK, marker bounce di lokasi darurat, buka dari notifikasi | 1 hari | 1.13 | F-A04 |
| 1.15 | Tombol batalkan darurat: countdown 10 detik, panggil `POST /api/panic/{id}/cancel` | 0.5 hari | 1.7, 1.12 | F-A06 |

### Web Dashboard (Person B)

| ID | Task | Estimasi | Dep | Ref PRD |
|----|------|----------|-----|---------|
| 1.16 | UI halaman Login petugas (Next.js, sesuai StyleGuide) | 1 hari | — | F-W05 |
| 1.17 | Layout dashboard utama: sidebar, header, area peta, panel kejadian | 1 hari | — | F-W01 |
| 1.18 | WebSocket client: koneksi Laravel Reverb via Echo, listen event `PanicTriggered` | 1 hari | 1.5 | F-W01 |
| 1.19 | Flashing alert overlay: full-screen merah berkedip + suara sirine (Web Audio API) | 1.5 hari | 1.18 | F-W02, F-W03 |
| 1.20 | Peta Leaflet.js: auto-zoom ke lokasi kejadian, marker bounce, radius 50m | 1.5 hari | 1.18 | F-W04 |
| 1.21 | Panel daftar kejadian aktif + tombol "Respons" per kejadian | 1 hari | 1.18 | F-W06, F-W07 |

---

## Fase 2 — Fitur Pendukung (Minggu 4)

| ID | Task | PJ | Estimasi | Dep | Ref PRD |
|----|------|----|----------|-----|---------|
| 2.1 | API `GET /api/panic/history` dengan pagination, filter tanggal, search | A | 0.5 hari | 1.1 | — |
| 2.2 | API manajemen warga: `GET /api/users`, `PATCH /api/users/{id}/approve`, `/reject` | A | 1 hari | 1.2 | — |
| 2.3 | Rate limiting (throttle middleware), CORS, security headers, input sanitization | A | 1 hari | 1.2 | — |
| 2.4 | Logging & monitoring: log semua panic event, Tuya request/response, error tracking | A | 0.5 hari | 1.3 | — |
| 2.5 | Halaman riwayat kejadian di Android (RecyclerView, pagination, pull-to-refresh) | C | 1 hari | 2.1 | F-A07 |
| 2.6 | Halaman profil pengguna di Android (edit nama, nomor HP) | C | 0.5 hari | 1.9 | F-A08 |
| 2.7 | Halaman riwayat kejadian di web dashboard (tabel, filter tanggal, pagination) | B | 1 hari | 2.1 | F-W08 |
| 2.8 | Halaman manajemen warga di web dashboard (approve/reject, list warga aktif) | B | 1 hari | 2.2 | F-W09 |
| 2.9 | Responsive design dashboard (optimasi untuk monitor pos satpam & tablet) | B | 0.5 hari | 1.17 | — |

---

## Fase 3 — Testing & Deployment (Minggu 5–6)

| ID | Task | PJ | Estimasi | Dep |
|----|------|----|----------|-----|
| 3.1 | Unit test backend: PanicController, TuyaService, FcmService (PHPUnit) | A | 1.5 hari | Fase 1–2 |
| 3.2 | Integration test: alur panic end-to-end (dari API call sampai broadcast event) | A | 1 hari | Fase 1–2 |
| 3.3 | Deploy backend ke VPS: Nginx reverse proxy, PHP-FPM, MySQL, Reverb, SSL Let's Encrypt | A | 1 hari | 3.1 |
| 3.4 | Setup CI/CD sederhana: GitHub Actions auto-deploy ke VPS via SSH | A | 0.5 hari | 3.3 |
| 3.5 | Testing Android: unit test ViewModel, instrument test panic flow | C | 1.5 hari | Fase 1–2 |
| 3.6 | Build APK signed release, distribusi via Firebase App Distribution untuk testing | C | 0.5 hari | 3.3, 3.5 |
| 3.7 | Testing web dashboard: component test (React Testing Library), WebSocket mock test | B | 1 hari | Fase 1–2 |
| 3.8 | Build & deploy web dashboard ke VPS (Nginx static / Node.js server) | B | 0.5 hari | 3.3, 3.7 |
| 3.9 | **UAT (User Acceptance Testing)**: test bersama perwakilan warga & satpam di lokasi | Semua | 2 hari | 3.3, 3.6, 3.8 |
| 3.10 | Bug fixing dari hasil UAT | Semua | 2 hari | 3.9 |
| 3.11 | Dokumentasi: panduan deployment, panduan pengguna (warga & satpam) | Semua | 1 hari | 3.10 |

---

## Critical Path

```
0.5 (Laravel scaffold)
  → 1.1 (DB migration)
    → 1.3 (API Panic)
      ├── 1.4 (FCM server) → 1.13 (FCM client Android)
      ├── 1.5 (Reverb WebSocket) → 1.18 (WS client Next.js) → 1.19 (Alert)
      └── 1.6 (Tuya API) → sirine fisik aktif
```

Tiga cabang paralel setelah task 1.3 mencerminkan arsitektur inti PLAN B: notifikasi simultan ke (1) HP warga, (2) dashboard satpam, (3) sirine fisik.

---

## Risiko & Mitigasi

| Risiko | Dampak | Probabilitas | Mitigasi |
|--------|--------|--------------|----------|
| Tuya API tidak stabil / sirine tidak merespons | Sirine fisik tidak berbunyi | Sedang | Retry dengan exponential backoff. Fallback: suara sirine di speaker monitor pos (Web Audio API sudah ada). Log semua request. |
| FCM notification terlambat (Doze mode) | Warga tidak segera ternotifikasi | Sedang | Gunakan high-priority FCM. Instruksikan warga disable battery optimization untuk app ini. |
| GPS tidak akurat (indoor/sinyal lemah) | Lokasi kejadian salah | Tinggi | FusedLocationProvider (GPS + WiFi + cell). Tampilkan accuracy radius di peta. Fallback ke last known location. |
| VPS down | Seluruh sistem mati | Rendah | Monitoring via UptimeRobot. Auto-restart PHP-FPM & Reverb via supervisor. |
| False alarm (tidak sengaja) | Kepanikan warga | Sedang | Long-press 2 detik + cancel 10 detik + rate limit 1 panic/30 detik per user. |
| WebSocket disconnect (koneksi internet pos) | Dashboard tidak menerima alert | Sedang | Auto-reconnect dengan backoff. Polling fallback `GET /api/panic/active` setiap 10 detik jika WS putus. |

---

## Timeline Ringkasan

| Fase | Durasi | Minggu |
|------|--------|--------|
| Fase 0: Persiapan | 1 minggu | Minggu 1 |
| Fase 1: MVP Core | 2 minggu | Minggu 2–3 |
| Fase 2: Fitur Pendukung | 1 minggu | Minggu 4 |
| Fase 3: Testing & Deployment | 1.5 minggu | Minggu 5–6 |
| **Total** | **~5.5 minggu** | |

---

## Checklist Milestone

- [ ] **M1 (Akhir Minggu 1):** Semua project ter-scaffold, VPS ready, akun Firebase & Tuya aktif
- [ ] **M2 (Akhir Minggu 3):** Demo end-to-end: tekan panic di HP → dashboard berkedip + sirine browser + sirine Tuya menyala + HP warga lain ternotifikasi
- [ ] **M3 (Akhir Minggu 4):** Fitur riwayat, manajemen warga, profil selesai
- [ ] **M4 (Akhir Minggu 5):** Deployed ke VPS, APK terdistribusi, web dashboard live
- [ ] **M5 (Akhir Minggu 6):** UAT selesai, bug fix done, dokumentasi lengkap, **READY FOR PRODUCTION**
