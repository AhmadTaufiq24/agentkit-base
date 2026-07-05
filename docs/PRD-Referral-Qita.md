# PRD — Referral Hub Qita (Adaptif Berdasarkan Program Procash)

| | |
|---|---|
| **Dokumen** | Product Requirements Document (PRD) |
| **Fitur** | Referral Hub Qita |
| **Audiens dokumen** | UI/UX Designer |
| **Status** | Draft v1.1 |
| **Product Owner** | Tim Product Qita |
| **Tanggal** | Juli 2026 |

---

## 1. Latar Belakang

Qita adalah aplikasi banking BRI (sejenis BRImo). Qita memiliki fitur referral yang engine-nya dikelola melalui dashboard **Procash**.

### Model Bisnis Referral

Referral di Qita terdiri dari **dua lapisan** yang harus dipisahkan dalam desain:

| Lapisan | Perilaku | Kapan aktif |
|---|---|---|
| **Infrastruktur referral** | Kode/link referral unik per user, attribution, tracking ajakan | **Selalu aktif** — tidak pernah dimatikan |
| **Program reward (kampanye)** | Nominal reward, syarat kualifikasi, periode, kuota | **Hanya saat program aktif di Procash** dan user memenuhi eligibility |

**Keputusan produk:** kode referral **tetap bisa digunakan** meskipun tidak ada program reward aktif. Teman tetap bisa mendaftar dengan kode tersebut (attribution tercatat), tetapi **tidak ada reward** untuk kedua pihak kecuali program aktif dan semua syarat eligibility terpenuhi.

Agar referrer mendapat reward, sebuah **program** harus dibuat di Procash **dan** tipe user referrer harus termasuk dalam segmen yang diizinkan program tersebut.

### Segmen Program (Referee)

Program yang dapat dibuat di Procash menargetkan segmen **teman yang diundang (referee)**:

- **Program NTB** — reward untuk mengajak orang yang **belum punya rekening BRI sama sekali** (New To Bank).
- **Program ETB** — reward untuk mengajak orang yang **sudah punya rekening BRI dan/atau BRImo** (Existing To Bank, mencakup seluruh varian ETB) untuk aktivasi/menggunakan Qita.

### Tipe User (konteks internal — **tidak boleh muncul sebagai istilah di UI**)

| Tipe | Definisi |
|---|---|
| NTB | Belum punya rekening BRI sama sekali |
| ETB (CIF) | Sudah punya rekening BRI, belum punya BRImo |
| ETB BerBRImo | Sudah punya rekening BRI dan BRImo |
| ETB X NTB | User dormant yang mendaftar kembali sebagai NTB |

Program di Procash dikonfigurasi untuk menentukan **tipe user referrer mana yang berhak mendapatkan reward** saat program tersebut aktif. UI hanya menampilkan program & reward jika **referrer eligible** untuk program aktif tersebut.

### Matriks State Referral Hub

UI harus mengakomodasi kombinasi berikut **tanpa app release** (server-driven dari config Procash):

| # | Program aktif | Referrer eligible | Tampilan UI |
|---|---|---|---|
| A | NTB saja | Ya | Hero reward NTB |
| B | NTB + ETB | Ya | Hero reward dual-segmen |
| C | ETB saja | Ya | Hero reward ETB |
| D | Ada (tipe lain) | **Tidak** | Mode non-monetary — kode tetap aktif |
| E | **Tidak ada** | — | Mode non-monetary — kode tetap aktif |

State D dan E terlihat mirip secara visual (tanpa janji nominal), tetapi **copy-nya berbeda** — ini wajib didesain sebagai dua varian terpisah.

## 2. Masalah yang Diselesaikan

1. Referrer tidak tahu dan tidak seharusnya perlu tahu status perbankan temannya (NTB/ETB adalah kompleksitas internal). Jika UI memaksa user memahami segmentasi, sharing rate turun dan mismatch naik.
2. Jika kriteria teman yang valid tidak terkomunikasikan jelas, terjadi **janji reward yang gagal** → komplain CS, rusaknya trust referrer terhadap Qita dan trust teman terhadap referrer.
3. Program berganti-ganti sepanjang waktu; UI statis akan menampilkan janji basi.
4. User perlu paham perbedaan **"kode masih bisa dipakai"** vs **"ada reward"** — tanpa merasa fitur mati atau ditipu.

## 3. Goals & Non-Goals

### Goals

1. Satu Referral Hub adaptif: satu template, konten dinamis dari config Procash.
2. User paham dalam ≤ 5 detik: **apakah ada reward untuk saya, siapa yang bisa diajak, berapa reward masing-masing pihak**.
3. Satu kode/link referral per user, **selalu aktif** — apapun status program reward-nya.
4. Sistem (bukan user) yang menentukan eligibility referrer dan referee.
5. Transparansi status referral end-to-end (Diundang → Terdaftar → Memenuhi Syarat → Reward Cair / Tanpa Reward).
6. Konsistensi janji di 3 titik: layar referrer, pesan share, layar onboarding referee.

### Non-Goals

- Desain dashboard Procash (out of scope).
- Mekanisme anti-fraud backend (dedupe NIK/CIF) — hanya implikasi UI-nya yang dicakup.
- Reward selain yang dikonfigurasi Procash (tiering, gamifikasi lanjutan) — fase berikutnya.

## 4. Prinsip Desain (wajib dipegang designer)

1. **Kode referral selalu hidup.** Tidak pernah disembunyikan, dinonaktifkan, atau diganti meski tidak ada program reward.
2. **Reward adalah kampanye, bukan fitur inti.** UI harus membedakan jelas mode "ada reward" vs "tanpa reward" tanpa membuat fitur terasa mati.
3. **Satu kode, satu tombol share.** Jangan pernah meminta user memilih "mau ajak NTB atau ETB" sebelum share.
4. **Hide, jangan disable.** Program yang tidak aktif atau tidak relevan untuk tipe user tidak ditampilkan — bukan card yang di-gray-out atau berlabel "sedang tidak tersedia".
5. **Bahasa manusia, bukan istilah internal.** Terjemahan wajib: NTB → "teman yang belum punya rekening BRI"; ETB → "teman yang sudah punya rekening BRI atau BRImo".
6. **Kriteria teman yang valid muncul 3×** (hanya saat mode reward aktif): di headline hero, di section "Siapa yang bisa kamu ajak", dan di pesan share pre-filled.
7. **Tanpa reward = tanpa janji nominal di mana pun.** Share copy, hero, dan onboarding referee tidak boleh menyebut angka reward jika tidak ada program aktif atau referrer tidak eligible.
8. **Tidak ada layout shift antar state.** Template hub tetap; hanya konten yang berubah.
9. **Jangan pernah menampilkan nominal dari cache lama.** Jika config gagal dimuat, fallback ke mode non-monetary.

## 5. Arsitektur Konten (Server-Driven)

Procash meng-expose config yang menentukan:

- Program aktif (0–2 segmen referee: NTB dan/atau ETB)
- **Tipe user referrer yang eligible** mendapat reward per program
- Nominal reward referrer & referee
- Periode, kuota, syarat kualifikasi
- Copy S&K dan copy share message

API response harus menyertakan flag eksplisit untuk client:

```json
{
  "referral_code": "QITA-ABC123",
  "referrer_eligible": true,
  "has_active_reward_program": true,
  "active_programs": [ ... ],
  "ui_mode": "reward_ntb" | "reward_etb" | "reward_dual" | "no_reward"
}
```

Client me-render UI berdasarkan `ui_mode` — bukan mengevaluasi eligibility sendiri.

**Implikasi untuk designer:** semua komponen teks harus didesain dengan asumsi konten variabel (panjang nominal, panjang periode, jumlah program 0–2, mode reward vs non-reward). Siapkan spec untuk truncation dan dynamic type.

## 6. Struktur Halaman Referral Hub

### Mode Reward Aktif (State A / B / C)

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

### Mode Tanpa Reward (State D / E)

```
┌─────────────────────────────────────┐
│ ① HERO: ilustrasi + headline        │  Above the fold.
│    non-monetary + penjelasan status │
│ ② KODE REFERRAL [Salin]             │  Tetap prominent.
│    [ Bagikan ke Teman ] (primary)   │
│ ③ OPT-IN NOTIFIKASI                 │  "Beri tahu saya saat ada reward"
├─────────────────────────────────────┤
│ ④ STATUS AJAKANMU (tracker)         │  Riwayat lama + ajakan tanpa reward
└─────────────────────────────────────┘
```

**Tidak ditampilkan** di mode tanpa reward: badge periode, section "Siapa yang bisa kamu ajak", cara kerja ber-reward, S&K program (kecuali riwayat program lama di tracker).

Elemen above the fold wajib terlihat tanpa scroll di device baseline.

## 7. Requirement per Skenario

### Skenario A — Program NTB Aktif & Referrer Eligible

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

### Skenario B — Program NTB + ETB Aktif & Referrer Eligible

| Elemen | Konten |
|---|---|
| Headline | "Ajak siapa saja ke Qita, dapat hingga Rp25.000 per teman" (nominal tertinggi + "hingga") |
| Siapa yang bisa diajak | **Dual-card** berdampingan: Card 1 "Teman BARU di BRI (belum punya rekening)" — Kamu Rp25.000 / Teman Rp10.000. Card 2 "Teman pengguna BRI/BRImo" — Kamu Rp15.000 / Teman Rp5.000 |
| Caption di bawah dual-card | "Nggak perlu bingung — bagikan saja kodenya, sistem kami yang menentukan reward-nya." |
| Share copy | Netral: "Gabung Qita pakai kode aku dan dapat bonus saldo — mau kamu udah punya rekening BRI atau belum!" |
| Larangan | Tidak ada pemilihan segmen sebelum share; tetap satu tombol |

### Skenario C — Program ETB Aktif & Referrer Eligible

Perlu **reframing dari "ajak orang baru" menjadi "ajak pindah/aktivasi"**.

| Elemen | Konten |
|---|---|
| Headline | "Punya teman pengguna BRI atau BRImo? Ajak mereka pakai Qita, kamu dapat Rp15.000" |
| Subheadline | "Temanmu cukup aktivasi Qita dengan rekening BRI yang sudah dia punya — dia juga dapat Rp5.000" |
| Siapa yang bisa diajak | Checklist: ✓ Sudah punya rekening BRI atau BRImo · ✓ Belum pernah pakai Qita |
| Cara kerja | 1. Bagikan kode ke teman pengguna BRI/BRImo → 2. Teman download Qita, **login/aktivasi pakai rekening BRI-nya** + kodemu → 3. Teman transaksi pertama di Qita → reward cair |
| Contoh persona | "Misalnya: temanmu yang sehari-hari pakai BRImo" |
| Share copy | "Udah punya BRImo atau rekening BRI? Cobain Qita — tinggal aktivasi pakai rekening kamu yang sekarang plus kode QITA-ABC123, dapat Rp5.000." |
| Ilustrasi hero | Visual "berpindah/mencoba app baru", bukan "buka rekening pertama" |

*(Seluruh nominal di dokumen ini adalah placeholder — nilai riil dari config Procash.)*

### Skenario D — Program Aktif, Referrer Tidak Eligible

Program reward sedang berjalan di Procash, tetapi **tipe user referrer saat ini tidak termasuk segmen yang berhak mendapat reward**.

| Elemen | Konten |
|---|---|
| Headline | "Ajak temanmu rasakan Qita" |
| Subheadline | "Saat ini belum ada program reward untuk kamu. Kode referral kamu tetap bisa dibagikan." |
| Kode & share | Tetap prominent — tombol [Bagikan ke Teman], bukan [Bagikan Sekarang] (hindari implikasi ada reward) |
| Share copy | Non-monetary: "Coba Qita, aplikasi banking BRI yang praktis. Pakai kode aku saat daftar: QITA-ABC123" — **tanpa menyebut nominal** |
| Opt-in notifikasi | "Beri tahu saya saat ada program reward untuk saya" (push) |
| Tracker | Riwayat reward lama tetap tampil. Ajakan baru: status "Teman terdaftar — tidak ada program reward untuk kamu saat ini" |
| Larangan | Jangan tampilkan detail program yang sedang aktif untuk tipe user lain (menimbulkan FOMO negatif & komplain CS). Jangan tampilkan card program NTB/ETB yang di-gray-out. |

### Skenario E — Tidak Ada Program Aktif Sama Sekali

Tidak ada program reward yang berjalan di Procash untuk siapa pun.

| Elemen | Konten |
|---|---|
| Headline | "Ajak temanmu rasakan Qita" |
| Subheadline | "Belum ada program reward saat ini. Kode referral kamu tetap bisa dibagikan." |
| Kode & share | Tetap prominent — [Bagikan ke Teman] |
| Share copy | Non-monetary: "Coba Qita, aplikasi banking BRI yang praktis. Pakai kode aku saat daftar: QITA-ABC123" |
| Opt-in notifikasi | "Beri tahu saya saat ada program reward" (push) |
| Tracker | Riwayat reward lama tetap tampil. Ajakan baru: status "Teman terdaftar — tidak ada program reward aktif saat ini" |
| Larangan | Jangan tampilkan empty state yang terkesan "fitur mati". Jangan sembunyikan menu referral. |

### Perbedaan Kunci State D vs State E

| Aspek | State D (referrer tidak eligible) | State E (tidak ada program) |
|---|---|---|
| Subheadline | "...belum ada program reward **untuk kamu**" | "...belum ada program reward **saat ini**" |
| Opt-in copy | "...saat ada program reward **untuk saya**" | "...saat ada program reward" |
| Tracker status baru | "...tidak ada program reward **untuk kamu** saat ini" | "...tidak ada program reward **aktif** saat ini" |
| Entry point homepage | Jangan tampilkan banner reward (user tidak eligible) | Jangan tampilkan banner reward (tidak ada program) |

## 8. State & Perilaku Saat User Masuk Page

### 8.1 First-time visit (sekali seumur akun)

- **Mode reward aktif:** coachmark ringan / bottom sheet "Cara Kerja" (maks. 3 langkah bergambar, dismissible).
- **Mode tanpa reward:** tidak perlu coachmark reward; cukup penjelasan singkat di hero.
- Setelah dismiss tidak muncul lagi.

### 8.2 Transisi state sejak kunjungan terakhir

Bandingkan `ui_mode` terakhir yang dilihat user (local) vs `ui_mode` saat ini:

| Perubahan | Perlakuan |
|---|---|
| Reward aktif → tanpa reward (D/E) | Bottom sheet sekali: "Program reward sudah berakhir. Kode kamu tetap bisa dibagikan." CTA: [Mengerti] — **bukan** [Bagikan Sekarang] |
| Tanpa reward → reward aktif (D/E → A/B/C) | Bottom sheet: "Program reward baru! Ajak temanmu dan dapat Rp25.000." CTA: [Mengerti, Bagikan Sekarang] |
| Ganti segmen reward (NTB ⇄ ETB) | Bottom sheet: "Program referral baru! Sekarang giliran ajak temanmu yang sudah punya rekening BRI/BRImo." |
| Hanya nominal/periode berubah | Badge "Baru" di hero, tanpa interupsi |
| State D → State E (atau sebaliknya) | Tidak perlu interupsi — copy subheadline berubah secara halus |

**Larangan umum:** tidak ada pop-up/modal promo saat page dibuka di luar kasus di atas.

### 8.3 Reward baru cair sejak kunjungan terakhir

Snackbar/banner tipis di atas hero: "🎉 Rp25.000 sudah masuk ke saldomu dari ajakan ke Budi!" — prioritas tampil lebih dulu daripada bottom sheet transisi state.

### 8.4 Ajakan menggantung (hanya mode reward aktif)

Nudge di bawah tombol share: "Budi tinggal 1 langkah lagi — ingatkan dia transaksi pertama sebelum 10 Juli. [Ingatkan]"

Tidak ditampilkan di mode tanpa reward — tidak ada reward yang dikejar.

### 8.5 Loading / config gagal

- Loading: skeleton mengikuti template hub (tanpa layout shift saat konten masuk).
- Gagal total: fallback ke mode Skenario E (non-monetary). **Tidak pernah** menampilkan nominal dari cache.

## 9. Tracker "Status Ajakanmu"

- **Agregat di atas list** (jika pernah dapat reward): "Total reward kamu: Rp75.000 dari 3 teman".
- **Status per teman** (tanpa membocorkan data finansial teman):

| Status | Copy contoh | Mode | Aksi |
|---|---|---|---|
| Terdaftar | "Budi sudah gabung, tinggal transaksi pertama" | Reward aktif | [Ingatkan] |
| Memenuhi syarat | "Reward sedang diproses" | Reward aktif | — |
| Reward cair | "Rp25.000 · 2 Jul 2026" + label program | Reward aktif | — |
| Tidak memenuhi syarat | "Budi sudah gabung, tapi tidak memenuhi kriteria program (sudah punya rekening BRI)" | Reward aktif | Link ke S&K |
| Terdaftar, tanpa reward (State D) | "Budi sudah gabung — tidak ada program reward untuk kamu saat ini" | Tanpa reward | — |
| Terdaftar, tanpa reward (State E) | "Budi sudah gabung — tidak ada program reward aktif saat ini" | Tanpa reward | — |

- Riwayat lintas program dipertahankan selamanya (dengan label program) — menghilangkan riwayat = tiket CS.
- Jika program berkuota (mis. maks. 200 teman/periode): tampilkan progres "Kamu sudah mengajak 12/200" jauh sebelum mentok (hanya mode reward aktif).

## 10. Sisi Referee (Teman yang Diundang)

1. Deep link (deferred deep linking) membawa kode referral ke onboarding. Kode **selalu diterima** — apapun status program.
2. **Janji reward hanya ditampilkan jika ketiga kondisi terpenuhi secara bersamaan:**
   - Ada program reward aktif di Procash
   - Referrer eligible untuk program tersebut
   - Tipe referee cocok dengan segmen program (cek NIK → CIF, sebelum layar janji reward)
3. Jika salah satu kondisi tidak terpenuhi: onboarding berjalan normal dengan value prop produk, **tanpa pernah menampilkan janji nominal** dan tanpa pesan penolakan frontal. Attribution tetap dicatat.
4. Terms di-lock saat registrasi: jika teman mendaftar saat program masih aktif dan semua eligibility terpenuhi, syarat kualifikasi yang berlaku adalah syarat saat ia mendaftar (requirement ke Procash; UI menampilkan deadline personal si teman).

## 11. Entry Points (di luar Referral Hub)

| Entry point | Mode reward aktif | Mode tanpa reward (D/E) |
|---|---|---|
| Banner homepage | Copy spesifik program ("Ajak teman buka rekening, dapat Rp25rb") | **Sembunyikan** banner reward — jangan tampilkan janji nominal |
| Post-transaksi sukses | Card: "Suka pakai Qita? Ajak temanmu & dapat Rp25.000" | Card non-monetary: "Kenalkan Qita ke temanmu" (tanpa nominal) |
| Push notification program launch | Deep link ke Referral Hub; hanya ke segmen referrer eligible | Tidak dikirim ke referrer tidak eligible |
| Menu profil | Entry point permanen — label "Ajak Teman" (bukan "Dapat Reward") | Sama — tetap ada |

## 12. Copywriting Guidelines

- Dilarang menampilkan istilah: NTB, ETB, CIF, dormant, ETB X NTB, Procash.
- Nominal selalu ditulis untuk **kedua pihak** (double-sided) — hanya saat mode reward aktif.
- Periode program selalu tampil di/dekat hero — hanya saat mode reward aktif.
- Share copy ditulis dari sudut pandang penerima. Saat tanpa reward: fokus value prop produk, **zero mention nominal**.
- Alasan gagal di tracker: jujur, bahasa kriteria yang sama dengan section "Siapa yang bisa kamu ajak", tanpa kode error.
- Tombol CTA: "Bagikan Sekarang" (ada reward) vs "Bagikan ke Teman" (tanpa reward) — perbedaan kecil tapi meaningful.

## 13. Edge Cases

| Kasus | Perlakuan |
|---|---|
| Referee ETB masuk saat hanya program NTB aktif | Onboarding normal tanpa janji reward; attribution dicatat sebagai data mismatch |
| Referrer share saat reward aktif, referee daftar setelah program berakhir | Lock-in terms saat registrasi jika masih dalam grace period Procash; jika tidak, tracker: "tidak ada program reward aktif saat ini" |
| Referrer tidak eligible, referee eligible untuk program aktif | Referee bisa dapat reward (jika Procash mengizinkan); referrer tidak. Tracker referrer: status tanpa reward. **Jangan tampilkan reward ke referrer.** |
| ETB X NTB (dormant daftar ulang) | Kebijakan eligibility diatur backend/S&K; di UI cukup status "tidak memenuhi kriteria program" + rujukan S&K |
| Kuota program habis | Mode mirip tanpa reward untuk referrer: kode tetap aktif, hero "Kuota program periode ini sudah penuh", tombol share tanpa janji nominal |
| Program berakhir saat teman di tengah funnel | Honor terms saat registrasi (lock-in); tracker menampilkan deadline personal si teman |
| Config berubah saat user sedang di page | Refresh konten saat page re-focus; jangan swap konten di depan mata tanpa transisi |
| User pernah dapat reward, sekarang tanpa program | Riwayat reward tetap tampil; hero mode tanpa reward — jangan framing kehilangan |

## 14. Metrik Keberhasilan

| Metrik | Definisi |
|---|---|
| Share rate | Buka hub → tap Bagikan, dipisah per `ui_mode` |
| Share rate tanpa reward | Khusus State D/E — indikator apakah fitur tetap valuable tanpa insentif |
| Opt-in notifikasi rate | State D/E → user tap "Beri tahu saya saat ada reward" |
| K-factor per segmen | Undangan → registrasi → kualifikasi, dipisah NTB vs ETB |
| Mismatch rate | Referee masuk via link tapi tipe tidak match program aktif |
| Time-to-reward | Kualifikasi → reward cair; target < 48 jam |
| Repeat referral rate | Referrer yang mengajak ≥ 2 teman |
| CS ticket rate | Tiket bertopik referral per 1.000 referral |
| False promise rate | Ajakan dengan janji reward di share copy tapi tidak ada program aktif saat referee daftar |

## 15. Deliverables yang Diminta dari Designer

1. High-fidelity design Referral Hub untuk **5 skenario** (A/B/C/D/E) dari satu template komponen yang sama.
2. State transisi: reward aktif ↔ tanpa reward, ganti segmen, reward cair, ajakan menggantung.
3. Perbedaan visual State D vs State E (copy berbeda, layout sama).
4. Flow referee: deep link → onboarding dengan janji reward (semua eligibility terpenuhi) dan tanpa janji (salah satu tidak terpenuhi).
5. Tracker referrer: 6 varian status + agregat + progres kuota.
6. Entry points: banner homepage (reward vs non-reward), card post-transaksi (reward vs non-reward), template push notification.
7. Bottom sheet S&K (mode reward) dan bottom sheet transisi state.
8. Spec komponen dinamis: perilaku teks variabel, truncation, dynamic type, aturan transisi tanpa layout shift.
9. Spec tombol CTA: "Bagikan Sekarang" vs "Bagikan ke Teman".

## 16. Open Questions

1. Nominal & syarat kualifikasi final per program (menunggu konfigurasi Procash).
2. Kebijakan final eligibility ETB X NTB (dihitung NTB atau tidak) — menentukan copy S&K.
3. Apakah reward cair otomatis atau perlu klaim manual di app? (Rekomendasi: otomatis.)
4. Batas kuota per user per periode dan per program.
5. Channel share yang didukung: native share sheet vs shortcut khusus (WhatsApp-first?).
6. Apakah referee tetap bisa dapat reward jika referrer tidak eligible? (Menentukan apakah State D perlu copy berbeda di sisi referee.)
7. Grace period setelah program berakhir: berapa hari referee yang sudah terdaftar masih bisa memenuhi syarat?
