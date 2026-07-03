# PRD — Referral Hub Qita (Adaptif Berdasarkan Program Procash)

| | |
|---|---|
| **Dokumen** | Product Requirements Document (PRD) |
| **Fitur** | Referral Hub Qita |
| **Audiens dokumen** | UI/UX Designer |
| **Status** | Draft v1.0 |
| **Product Owner** | Tim Product Qita |
| **Tanggal** | Juli 2026 |

---

## 1. Latar Belakang

Qita adalah aplikasi banking BRI (sejenis BRImo). Qita memiliki fitur referral yang engine-nya dikelola melalui dashboard **Procash**. Agar user mendapat reward dari referral, sebuah **program** harus dibuat di Procash.

Saat ini program yang dapat dibuat menargetkan dua segmen **teman yang diundang (referee)**:

- **Program NTB** — reward untuk mengajak orang yang **belum punya rekening BRI sama sekali** (New To Bank).
- **Program ETB** — reward untuk mengajak orang yang **sudah punya rekening BRI dan/atau BRImo** (Existing To Bank, mencakup seluruh varian ETB) untuk aktivasi/menggunakan Qita.

Tipe user di ekosistem Qita (konteks internal, **tidak boleh muncul sebagai istilah di UI**):

| Tipe | Definisi |
|---|---|
| NTB | Belum punya rekening BRI sama sekali |
| ETB (CIF) | Sudah punya rekening BRI, belum punya BRImo |
| ETB BerBRImo | Sudah punya rekening BRI dan BRImo |
| ETB X NTB | User dormant yang mendaftar kembali sebagai NTB |

Dalam satu waktu, kombinasi program yang aktif bisa berupa: **(A) hanya NTB**, **(B) NTB + ETB bersamaan**, **(C) hanya ETB**, atau **(D) tidak ada program aktif**. UI harus mengakomodasi keempatnya **tanpa app release** (server-driven dari config Procash).

## 2. Masalah yang Diselesaikan

1. Referrer tidak tahu dan tidak seharusnya perlu tahu status perbankan temannya (NTB/ETB adalah kompleksitas internal). Jika UI memaksa user memahami segmentasi, sharing rate turun dan mismatch naik.
2. Jika kriteria teman yang valid tidak terkomunikasikan jelas, terjadi **janji reward yang gagal** → komplain CS, rusaknya trust referrer terhadap Qita dan trust teman terhadap referrer.
3. Program berganti-ganti sepanjang waktu; UI statis akan menampilkan janji basi.

## 3. Goals & Non-Goals

### Goals

1. Satu Referral Hub adaptif: satu template, konten dinamis dari config Procash.
2. User paham dalam ≤ 5 detik: **siapa yang bisa diajak, apa yang harus teman lakukan, berapa reward masing-masing pihak**.
3. Satu kode/link referral per user, tidak pernah berubah, apapun program aktifnya.
4. Sistem (bukan user) yang menentukan eligibility teman saat onboarding.
5. Transparansi status referral end-to-end (Diundang → Terdaftar → Memenuhi Syarat → Reward Cair).

### Non-Goals

- Desain dashboard Procash (out of scope).
- Mekanisme anti-fraud backend (dedupe NIK/CIF) — hanya implikasi UI-nya yang dicakup.
- Reward selain yang dikonfigurasi Procash (tiering, gamifikasi lanjutan) — fase berikutnya.

## 4. Prinsip Desain (wajib dipegang designer)

1. **Satu kode, satu tombol share.** Jangan pernah meminta user memilih "mau ajak NTB atau ETB" sebelum share.
2. **Hide, jangan disable.** Program yang tidak aktif tidak ditampilkan sama sekali — bukan card yang di-gray-out atau berlabel "sedang tidak tersedia".
3. **Bahasa manusia, bukan istilah internal.** Terjemahan wajib: NTB → "teman yang belum punya rekening BRI"; ETB → "teman yang sudah punya rekening BRI atau BRImo".
4. **Kriteria teman yang valid muncul 3×:** di headline hero, di section "Siapa yang bisa kamu ajak", dan di pesan share pre-filled.
5. **Value prop terbaca dalam satu kalimat.** Detail masuk bottom sheet S&K.
6. **Tidak ada layout shift antar program.** Template hub tetap; hanya konten yang berubah.
7. **Jangan pernah menampilkan nominal dari cache lama.** Jika config gagal dimuat, fallback ke mode non-monetary.

## 5. Arsitektur Konten (Server-Driven)

Procash meng-expose config program aktif berisi: segmen target, nominal reward referrer & referee, periode, kuota, syarat kualifikasi, copy S&K, dan copy share message. Seluruh teks bernominal dan berkriteria di Referral Hub di-render dari config ini.

**Implikasi untuk designer:** semua komponen teks harus didesain dengan asumsi konten variabel (panjang nominal, panjang periode, jumlah program 0–2). Siapkan spec untuk truncation dan dynamic type.

## 6. Struktur Halaman Referral Hub

```
┌─────────────────────────────────────┐
│ ① HERO: ilustrasi + headline +      │  Above the fold.
│    subheadline (nominal 2 pihak)    │  Menjawab: "apa untungnya?"
│ ② BADGE PERIODE                     │  "Berlaku s.d. <tanggal>"
│ ③ KODE REFERRAL [Salin]             │  Menjawab: "apa yang harus
│    [ Bagikan Sekarang ] (primary)   │   saya lakukan?"
├─────────────────────────────────────┤
│ ④ SIAPA YANG BISA KAMU AJAK         │  Checklist / dual-card
│ ⑤ CARA KERJA (3 langkah)            │  Termasuk contoh persona
│ ⑥ STATUS AJAKANMU (tracker)         │  + agregat total reward
│ ⑦ S&K (bottom sheet trigger)        │
└─────────────────────────────────────┘
```

Elemen ①–③ wajib above the fold di device baseline. ④–⑦ boleh di bawah lipatan.

## 7. Requirement per Skenario Program

### Skenario A — Hanya Program NTB Aktif

| Elemen | Konten |
|---|---|
| Headline | "Ajak temanmu buka rekening pertama di Qita, kamu dapat Rp25.000" |
| Subheadline | "Temanmu yang belum punya rekening BRI juga dapat Rp10.000" |
| Badge periode | "Berlaku s.d. 31 Agustus 2026" |
| Siapa yang bisa diajak | Checklist: ✓ Belum punya rekening BRI sama sekali · ✓ Belum pernah pakai BRImo atau Qita. Footer reward: "Kamu Rp25.000 · Temanmu Rp10.000" |
| Cara kerja | 1. Bagikan kode ke teman yang belum punya rekening BRI → 2. Teman buka rekening di Qita pakai kodemu → 3. Teman setor/transaksi pertama min. RpXX → reward cair maks. 2×24 jam |
| Contoh persona | "Misalnya: adikmu yang baru mulai kerja dan belum punya rekening" |
| Share copy | "Belum punya rekening? Buka rekening BRI pertamamu di Qita pakai kode QITA-ABC123, langsung dapat Rp10.000!" |
| Larangan | Tidak ada jejak apapun dari program ETB |

*(Seluruh nominal di dokumen ini adalah placeholder — nilai riil dari config Procash.)*

### Skenario B — Program NTB + ETB Aktif Bersamaan

| Elemen | Konten |
|---|---|
| Headline | "Ajak siapa saja ke Qita, dapat hingga Rp25.000 per teman" (nominal tertinggi + "hingga") |
| Siapa yang bisa diajak | **Dual-card** berdampingan (bukan checklist): Card 1 "Teman BARU di BRI (belum punya rekening)" — Kamu Rp25.000 / Teman Rp10.000. Card 2 "Teman pengguna BRI/BRImo" — Kamu Rp15.000 / Teman Rp5.000 |
| Caption di bawah dual-card | "Nggak perlu bingung — bagikan saja kodenya, sistem kami yang menentukan reward-nya." |
| Share copy | Netral: "Gabung Qita pakai kode aku dan dapat bonus saldo — mau kamu udah punya rekening BRI atau belum!" |
| Larangan | Tidak ada pemilihan segmen sebelum share; tetap satu tombol |

### Skenario C — Hanya Program ETB Aktif

Perlu **reframing dari "ajak orang baru" menjadi "ajak pindah/aktivasi"**. Aksi kualifikasi (aktivasi dengan rekening yang sudah ada, bukan buka rekening baru) wajib eksplisit.

| Elemen | Konten |
|---|---|
| Headline | "Punya teman pengguna BRI atau BRImo? Ajak mereka pakai Qita, kamu dapat Rp15.000" |
| Subheadline | "Temanmu cukup aktivasi Qita dengan rekening BRI yang sudah dia punya — dia juga dapat Rp5.000" |
| Siapa yang bisa diajak | Checklist: ✓ Sudah punya rekening BRI atau BRImo · ✓ Belum pernah pakai Qita |
| Cara kerja | 1. Bagikan kode ke teman pengguna BRI/BRImo → 2. Teman download Qita, **login/aktivasi pakai rekening BRI-nya** + kodemu → 3. Teman transaksi pertama di Qita → reward cair |
| Contoh persona | "Misalnya: temanmu yang sehari-hari pakai BRImo" |
| Share copy | "Udah punya BRImo atau rekening BRI? Cobain Qita — tinggal aktivasi pakai rekening kamu yang sekarang plus kode QITA-ABC123, dapat Rp5.000." |
| Ilustrasi hero | Visual "berpindah/mencoba app baru", bukan "buka rekening pertama" |

### Skenario D — Tidak Ada Program Aktif

| Elemen | Konten |
|---|---|
| Hero | Non-monetary: "Ajak temanmu rasakan Qita" — tanpa janji nominal apa pun |
| Kode & share | Tetap tersedia (attribution & network effect tetap berjalan) |
| Tambahan | Opt-in: "Beri tahu saya saat ada program reward" (push) |
| Tracker | Riwayat reward lama tetap tampil |
| Larangan | Jangan tampilkan empty state yang terkesan "fitur mati" |

## 8. State & Perilaku Saat User Masuk Page

### 8.1 First-time visit (sekali seumur akun)

Coachmark ringan / bottom sheet "Cara Kerja" (maks. 3 langkah bergambar, dismissible). Setelah dismiss tidak muncul lagi; konten sama tetap tersedia di section Cara Kerja.

### 8.2 Program berganti sejak kunjungan terakhir (satu-satunya interupsi yang dibolehkan)

Bandingkan program ID terakhir yang dilihat user (local) vs program aktif:

| Perubahan | Perlakuan |
|---|---|
| Ganti segmen (NTB ⇄ ETB) | Bottom sheet sekali: "Program referral baru! Sekarang giliran ajak temanmu yang sudah punya rekening BRI/BRImo. Kamu dapat Rp15.000, temanmu Rp5.000." CTA: [Mengerti, Bagikan Sekarang] |
| Hanya nominal/periode berubah | Badge "Baru" di hero, tanpa interupsi |
| Program bertambah (→ NTB+ETB) | Bottom sheet framing positif: "Kabar baik! Sekarang kamu bisa ajak siapa saja." |
| Program berkurang | Tanpa framing kehilangan; langsung tampilkan hero program yang aktif. Riwayat reward lama utuh |

**Larangan umum:** tidak ada pop-up/modal promo saat page dibuka di luar kasus di atas.

### 8.3 Reward baru cair sejak kunjungan terakhir

Snackbar/banner tipis di atas hero: "🎉 Rp25.000 sudah masuk ke saldomu dari ajakan ke Budi!" — prioritas tampil lebih dulu daripada bottom sheet ganti program jika keduanya terjadi.

### 8.4 Ajakan menggantung

Nudge di bawah tombol share: "Budi tinggal 1 langkah lagi — ingatkan dia transaksi pertama sebelum 10 Juli. [Ingatkan]" (tombol Ingatkan → share sheet dengan pesan pre-filled ke teman ybs.).

### 8.5 Loading / config gagal

- Loading: skeleton mengikuti template hub (tanpa layout shift saat konten masuk).
- Gagal total: fallback ke mode Skenario D (non-monetary). **Tidak pernah** menampilkan nominal dari cache.

## 9. Tracker "Status Ajakanmu"

- **Agregat di atas list:** "Total reward kamu: Rp75.000 dari 3 teman".
- **Status per teman** (tanpa membocorkan data finansial teman):

| Status | Copy contoh | Aksi |
|---|---|---|
| Terdaftar | "Budi sudah gabung, tinggal transaksi pertama" | [Ingatkan] |
| Memenuhi syarat | "Reward sedang diproses" | — |
| Reward cair | "Rp25.000 · 2 Jul 2026" + label kecil nama program | — |
| Tidak memenuhi syarat | "Budi sudah gabung, tapi tidak memenuhi kriteria program (sudah punya rekening BRI)" | Link ke S&K |

- Riwayat lintas program dipertahankan selamanya (dengan label program) — menghilangkan riwayat = tiket CS.
- Jika program berkuota (mis. maks. 200 teman/periode): tampilkan progres "Kamu sudah mengajak 12/200" jauh sebelum mentok.

## 10. Sisi Referee (Teman yang Diundang)

1. Deep link (deferred deep linking) membawa kode referral ke onboarding.
2. **Janji reward hanya ditampilkan setelah sistem mendeteksi tipe user cocok dengan program aktif** (cek NIK → CIF, sebelum layar janji reward).
   - Cocok: "Kamu diajak [Nama]! Selesaikan pendaftaran & transaksi pertama untuk dapat Rp10.000" + tracker progres syarat.
   - Tidak cocok (mismatch): onboarding berjalan normal dengan value prop produk, **tanpa pernah menampilkan janji nominal** dan tanpa pesan penolakan frontal. Attribution tetap dicatat.
3. Terms di-lock saat registrasi: jika teman mendaftar H-1 sebelum program berakhir, syarat kualifikasi yang berlaku adalah syarat saat ia mendaftar (requirement ke Procash; UI menampilkan deadline personal si teman).

## 11. Entry Points (di luar Referral Hub)

| Entry point | Requirement |
|---|---|
| Banner homepage / section promo | Copy spesifik program aktif ("Ajak teman buka rekening, dapat Rp25rb"), bukan generik ("Referral") |
| Post-transaksi sukses | Card kecil: "Suka pakai Qita? Ajak temanmu & dapat Rp25.000" — momen konversi tertinggi |
| Push notification saat program launch | Deep link ke Referral Hub; konten page harus persis sesuai janji notifikasi |
| Menu profil | Entry point permanen (muscle memory), tetap ada di Skenario D |

## 12. Copywriting Guidelines

- Dilarang menampilkan istilah: NTB, ETB, CIF, dormant, ETB X NTB, Procash.
- Nominal selalu ditulis untuk **kedua pihak** (double-sided) — referrer harus tahu apa yang bisa ia "tawarkan".
- Periode program selalu tampil di/dekat hero.
- Share copy ditulis dari sudut pandang penerima dan mengandung kriteria eligibility sebagai filter alami.
- Alasan gagal di tracker: jujur, bahasa kriteria yang sama dengan section "Siapa yang bisa kamu ajak", tanpa kode error.

## 13. Edge Cases

| Kasus | Perlakuan |
|---|---|
| Referee ETB masuk saat hanya program NTB aktif (atau sebaliknya) | Onboarding normal tanpa janji reward; attribution dicatat sebagai data mismatch |
| ETB X NTB (dormant daftar ulang) | Kebijakan eligibility diatur backend/S&K; di UI cukup status "tidak memenuhi kriteria program" + rujukan S&K. Taxonomy segmen tidak pernah terlihat user |
| Kuota program habis | Hero berubah ke state "kuota periode ini sudah penuh" + tanggal periode berikutnya (jika ada); tombol share tetap aktif tanpa janji nominal |
| Program berakhir saat teman di tengah funnel | Honor terms saat registrasi (lock-in); tracker teman menampilkan deadline personalnya |
| Config berubah saat user sedang di page | Refresh konten saat page re-focus; jangan swap konten di depan mata tanpa transisi |

## 14. Metrik Keberhasilan

| Metrik | Definisi |
|---|---|
| Share rate | Buka hub → tap Bagikan, dipisah per skenario program |
| K-factor per segmen | Undangan → registrasi → kualifikasi, dipisah NTB vs ETB |
| Mismatch rate | Referee masuk via link tapi tipe tidak match program aktif (indikator kejelasan copy + sinyal demand segmen non-aktif) |
| Time-to-reward | Kualifikasi → reward cair; target < 48 jam |
| Repeat referral rate | Referrer yang mengajak ≥ 2 teman |
| CS ticket rate | Tiket bertopik referral per 1.000 referral |

## 15. Deliverables yang Diminta dari Designer

1. High-fidelity design Referral Hub untuk **4 skenario program** (A/B/C/D) dari satu template komponen yang sama.
2. State: first-visit coachmark, bottom sheet ganti program (3 varian), snackbar reward cair, nudge ajakan menggantung, skeleton loading, fallback config gagal, kuota habis.
3. Flow referee: deep link → onboarding dengan janji reward (match) dan tanpa janji (mismatch), termasuk tracker syarat sisi referee.
4. Tracker referrer: list status 4 varian + agregat + progres kuota.
5. Bottom sheet S&K.
6. Entry points: banner homepage, card post-transaksi, template push notification.
7. Spec komponen dinamis: perilaku teks variabel (nominal, tanggal, nama), truncation, dynamic type, dan aturan transisi konten antar program (tanpa layout shift).

## 16. Open Questions

1. Nominal & syarat kualifikasi final per program (menunggu konfigurasi Procash).
2. Kebijakan final eligibility ETB X NTB (dihitung NTB atau tidak) — menentukan copy S&K.
3. Apakah reward cair otomatis atau perlu klaim manual di app? (Rekomendasi: otomatis; jika klaim manual, perlu tambahan state "Siap diklaim" di tracker.)
4. Batas kuota per user per periode dan per program.
5. Channel share yang didukung native share sheet vs shortcut khusus (WhatsApp-first?).
