# Style Guide — Sistem Alarm Panic Button

> **Versi:** 1.0  
> **Tanggal:** 27 Juni 2026

---

## 1. Prinsip Desain

1. **Kejelasan dalam keadaan darurat** — UI harus bisa digunakan dalam kondisi panik, gelap, atau terburu-buru
2. **Kontras tinggi** — Tombol dan teks kritis harus terlihat jelas tanpa perlu fokus
3. **Minimal langkah** — Panic button harus bisa diakses dari layar utama tanpa navigasi tambahan
4. **Tombol besar** — Semua elemen interaktif darurat minimal 72dp (Android) / 48px (Web)

---

## 2. Palet Warna

| Peran | Nama | Hex | Penggunaan |
|-------|------|-----|------------|
| Primary | Merah Darurat | `#D32F2F` | Panic button, header alert, status darurat |
| Primary Dark | Merah Gelap | `#B71C1C` | Status bar Android, pressed state |
| Primary Light | Merah Terang | `#FF1744` | Flashing overlay dashboard, notifikasi kritis |
| Secondary | Biru Keamanan | `#1565C0` | Navigasi, tombol sekunder, link |
| Secondary Light | Biru Muda | `#1E88E5` | Hover state, info badge |
| Success | Hijau Aman | `#2E7D32` | Status aman, konfirmasi penanganan selesai |
| Warning | Kuning Amber | `#F57F17` | Status menunggu, peringatan ringan |
| Background | Putih | `#FFFFFF` | Latar belakang utama |
| Surface | Abu Muda | `#F5F5F5` | Card background, section divider |
| Text Primary | Hitam | `#212121` | Teks utama |
| Text Secondary | Abu | `#757575` | Label, teks pendukung |
| Border | Abu Border | `#E0E0E0` | Garis pemisah, border card |

---

## 3. Tipografi

### Android App
- **Font:** Roboto (system default)
- Heading: Roboto Bold, 20sp–28sp
- Body: Roboto Regular, 14sp–16sp
- Caption: Roboto Regular, 12sp
- Panic Button Label: Roboto Black, 24sp, ALL CAPS

### Web Dashboard
- **Font:** Inter (via Google Fonts)
- h1: 28px / Bold
- h2: 22px / Semibold
- h3: 18px / Semibold
- Body: 16px / Regular
- Small: 14px / Regular
- Caption: 12px / Regular
- Monospace (log/data): JetBrains Mono, 14px

---

## 4. Komponen UI

### 4.1 Panic Button (Android)

```
┌─────────────────────────┐
│                         │
│      ╭───────────╮      │
│     ╱  ───────── ╲     │
│    │   ╭───────╮  │    │  ← Lingkaran 200dp
│    │   │ PANIC │  │    │  ← Long-press 2 detik
│    │   │BUTTON │  │    │  ← Progress ring melingkar
│    │   ╰───────╯  │    │     saat ditekan
│     ╲  ─────────  ╱     │
│      ╰───────────╯      │
│                         │
│   "Tekan & tahan 2      │
│    detik untuk darurat"  │
└─────────────────────────┘
```

- **Bentuk:** Lingkaran, diameter 200dp
- **Warna idle:** `#D32F2F` dengan animasi pulse (scale 1.0 → 1.05, loop)
- **Saat ditekan:** Progress ring melingkar 2 detik, warna `#B71C1C`
- **Setelah aktif:** Berubah menjadi tombol "BATALKAN" (`#F57F17`) dengan countdown 10 detik
- **Feedback:** Vibration pattern pendek saat mulai dan saat terkirim

### 4.2 Alert Card (Web Dashboard)

```
┌──┬──────────────────────────────────┐
│▐▐│  Ahmad Rizki — Blok B No. 12    │
│▐▐│  ⏱ 14:32:05 · 📍 120m dari pos  │
│▐▐│                    [RESPONS →]  │
└──┴──────────────────────────────────┘
 ↑ border-left 4px #D32F2F
```

- Border-left: 4px solid `#D32F2F`
- Background: `#FFFFFF` dengan box-shadow
- Animasi masuk: slide-in dari kanan, 300ms ease-out

### 4.3 Flashing Alert Overlay (Web Dashboard)

- Overlay full-screen, z-index tertinggi
- Alternasi background: `#FF1744` (opacity 0.85) ↔ `#FFFFFF`, interval 500ms
- Suara sirine diputar bersamaan via Web Audio API
- Tombol "LIHAT LOKASI" di tengah overlay, ukuran besar

### 4.4 Map Marker

- **Marker darurat:** Pin merah dengan animasi bounce, radius 50m transparan di sekitar titik
- **Marker pos satpam:** Ikon shield biru
- **Auto-zoom:** Level 17 (street level) saat ada kejadian aktif

---

## 5. Konvensi Penamaan Kode

### 5.1 Laravel (Backend)

```
Controller:     PascalCase          PanicController.php
Model:          PascalCase          PanicEvent.php
Migration:      snake_case          create_panic_events_table.php
Kolom DB:       snake_case          user_id, created_at, is_resolved
Route:          kebab-case          /api/panic-events
Method:         camelCase           handlePanic(), sendNotification()
Event:          PascalCase          PanicTriggered.php
Job:            PascalCase          TriggerTuyaSiren.php
Config key:     snake_case          tuya.api_key, fcm.server_key
Env var:        UPPER_SNAKE         TUYA_CLIENT_ID, FCM_SERVER_KEY
```

### 5.2 Next.js (Web Dashboard)

```
Component:      PascalCase.tsx      PanicAlert.tsx, MapView.tsx
Page:           kebab-case          /dashboard, /login, /history
Hook:           camelCase.ts        useWebSocket.ts, usePanicEvents.ts
Util:           camelCase.ts        formatTime.ts, playAlarm.ts
Type/Interface: PascalCase          PanicEvent, UserProfile
CSS Module:     PascalCase.module   PanicAlert.module.css
Constant:       UPPER_SNAKE         MAX_CANCEL_SECONDS, WS_URL
Props:          PascalCase          PanicAlertProps, MapViewProps
```

### 5.3 Android Kotlin

```
Activity:       PascalCase          MainActivity.kt, LoginActivity.kt
Fragment:       PascalCase          PanicFragment.kt, MapFragment.kt
ViewModel:      PascalCase          PanicViewModel.kt
Repository:     PascalCase          PanicRepository.kt
Service:        PascalCase          LocationService.kt
XML Layout:     snake_case          activity_main.xml, fragment_panic.xml
XML ID:         snake_case + prefix btn_panic, tv_status, iv_map, rv_history
Drawable:       snake_case + prefix ic_panic.xml, bg_button_red.xml
Package:        lowercase           com.panicbutton.app.ui.panic
Variable:       camelCase           currentLatitude, isPanicActive
Constant:       UPPER_SNAKE         PANIC_HOLD_DURATION_MS = 2000L
```

---

## 6. Konvensi API

### Base URL
```
Production:  https://api.{domain}.id/v1
Development: http://localhost:8000/api
```

### Endpoints

```
Auth:
  POST   /api/auth/register         Registrasi warga
  POST   /api/auth/login            Login (return token)
  POST   /api/auth/logout           Logout (revoke token)

Panic:
  POST   /api/panic                 Kirim sinyal darurat
  POST   /api/panic/{id}/cancel     Batalkan darurat
  PATCH  /api/panic/{id}/respond    Petugas merespons
  PATCH  /api/panic/{id}/resolve    Kejadian selesai
  GET    /api/panic/active          Daftar kejadian aktif
  GET    /api/panic/history         Riwayat (+ pagination & filter)

User:
  GET    /api/users/me              Profil sendiri
  PUT    /api/users/me              Update profil
  GET    /api/users                 Daftar warga (admin)
  PATCH  /api/users/{id}/approve    Approve pendaftaran
  PATCH  /api/users/{id}/reject     Reject pendaftaran
```

### WebSocket Events (Laravel Reverb)

```
Server → Client:
  PanicTriggered        Kejadian darurat baru
  PanicCancelled        Darurat dibatalkan
  PanicResponded        Petugas sedang menuju lokasi
  PanicResolved         Kejadian selesai ditangani

Client → Server:
  panic.acknowledge     Dashboard konfirmasi penerimaan
```

### Format Response Sukses

```json
{
  "success": true,
  "data": {
    "id": 1,
    "user": { "name": "Ahmad", "block": "B", "unit": "12" },
    "latitude": -6.2088,
    "longitude": 106.8456,
    "status": "active",
    "created_at": "2026-06-27T14:32:05Z"
  },
  "message": "Sinyal darurat berhasil dikirim"
}
```

### Format Response Error

```json
{
  "success": false,
  "error": {
    "code": "PANIC_COOLDOWN",
    "message": "Tunggu 30 detik sebelum mengirim sinyal darurat lagi"
  }
}
```

---

## 7. Struktur Folder Proyek

```
panic-button/
│
├── backend/                        Laravel 11
│   ├── app/
│   │   ├── Http/Controllers/
│   │   ├── Models/
│   │   ├── Events/                 PanicTriggered, PanicCancelled
│   │   ├── Jobs/                   TriggerTuyaSiren, SendFcmNotification
│   │   ├── Services/               TuyaService, FcmService
│   │   └── ...
│   ├── database/migrations/
│   ├── routes/
│   │   ├── api.php
│   │   └── channels.php
│   ├── config/
│   ├── tests/
│   ├── .env
│   └── composer.json
│
├── web-dashboard/                  Next.js 14+ (TypeScript)
│   ├── src/
│   │   ├── app/                    App Router pages
│   │   ├── components/
│   │   │   ├── PanicAlert.tsx
│   │   │   ├── MapView.tsx
│   │   │   ├── EventCard.tsx
│   │   │   └── FlashingOverlay.tsx
│   │   ├── hooks/
│   │   │   ├── useWebSocket.ts
│   │   │   └── usePanicEvents.ts
│   │   ├── services/
│   │   ├── types/
│   │   └── lib/
│   ├── public/
│   │   └── sounds/                 alarm.mp3
│   ├── package.json
│   └── next.config.ts
│
├── android-app/                    Kotlin
│   ├── app/src/main/
│   │   ├── java/com/panicbutton/app/
│   │   │   ├── ui/
│   │   │   │   ├── panic/         PanicFragment, PanicViewModel
│   │   │   │   ├── map/           MapFragment
│   │   │   │   ├── auth/          LoginActivity, RegisterActivity
│   │   │   │   └── history/       HistoryFragment
│   │   │   ├── data/
│   │   │   │   ├── api/           ApiService (Retrofit)
│   │   │   │   ├── repository/    PanicRepository
│   │   │   │   └── model/         PanicEvent, User
│   │   │   ├── service/
│   │   │   │   ├── LocationService.kt
│   │   │   │   └── FcmService.kt
│   │   │   └── di/                Hilt modules
│   │   ├── res/
│   │   └── AndroidManifest.xml
│   └── build.gradle.kts
│
├── docs/                           Dokumentasi tambahan
├── PRD.md
├── StyleGuide.md
└── Tasks.md
```
