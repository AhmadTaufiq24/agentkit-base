# PRD — Referral Hub Qita

| | |
|---|---|
| **Dokumen** | Product Requirements Document (PRD) |
| **Fitur** | Referral Hub Qita |
| **Audiens** | UI/UX Designer |
| **Status** | Draft v2.1 (Designer) |
| **Product Owner** | Tim Product Qita |
| **Tanggal** | Juli 2026 |

---

## 1. Deskripsi

### Apa itu Referral Hub?

Halaman **Ajak Teman** di aplikasi Qita — tempat user melihat kode referral, membagikannya, dan memahami **program reward mana yang sedang berjalan** serta **tipe teman siapa saja yang bisa diajak** agar referrer dan temannya mendapat reward.

Qita adalah aplikasi banking BRI. Program reward dikonfigurasi tim marketing melalui dashboard internal (di luar scope desain ini).

### Dua hal yang harus dipisahkan dalam desain

| Lapisan | Perilaku |
|---|---|
| **Kode referral** | Selalu aktif — user bisa share kapan saja |
| **Program reward** | Hanya saat ada kampanye aktif |

Meski tidak ada program reward, **kode referral tetap bisa dibagikan**. Teman tetap bisa daftar, tetapi reward hanya ada jika ada program aktif **dan** tipe teman yang diajak cocok dengan program tersebut.

### Segmen program — 4 tipe teman

Satu program hanya menargetkan **satu tipe teman**. Empat tipe yang dikenali sistem:

| Tipe (internal) | Siapa teman yang diajak? | Judul card di UI |
|---|---|---|
| NTB | Belum punya rekening BRI sama sekali | Ajak Teman Baru di BRI |
| ETB CIF | Sudah punya rekening BRI, belum punya BRImo | Ajak Pengguna BRI |
| ETB BerBRImo | Sudah punya rekening BRI dan BRImo | Ajak Pengguna BRImo |
| ETB X NTB | Pernah punya rekening BRI (dormant), daftar ulang di Qita | Ajak Teman Kembali |

**Aturan tampilan:**
- Hanya tampilkan **card untuk program yang sedang aktif**
- Program tidak aktif → card **tidak muncul**
- Maksimal **4 card** jika keempat program aktif
- **Urutan card:** NTB → ETB CIF → ETB BerBRImo → ETB X NTB

> Istilah NTB, ETB, CIF, dormant **tidak boleh** muncul di UI. Gunakan label manusiawi di tabel di atas.

### Goals

1. Satu halaman Referral Hub **adaptif** — template tetap, isi card mengikuti program aktif.
2. Hero **general** di semua kondisi — mengajak pakai Qita tanpa menyebut reward.
3. **Hanya section card yang berubah** — tanpa card (tidak ada program) hingga 4 card (semua program aktif).
4. **Referrer membaca di halaman referral:** reward (referrer & referee), periode selesai, kuota, dan **tipe user** — yaitu informasi **tipe teman mana yang eligible** untuk program di card tersebut (contoh: "Teman yang belum punya rekening BRI" untuk program NTB).
5. Satu program = satu tipe teman = satu card. Tidak ada multi-reward dalam satu card.

---

## 2. Masalah

1. **Referrer tidak tahu siapa yang bisa diajak agar dapat reward** — padahal program yang aktif sudah bisa dilihat di halaman referral; informasi tipe teman, reward, periode, dan kuota harus terbaca jelas di sana.
2. Jika kriteria teman tidak jelas, terjadi **janji reward yang gagal** → komplain CS, rusaknya kepercayaan.
3. Program berganti-ganti sepanjang waktu — desain statis akan menampilkan informasi basi.
4. User perlu paham perbedaan **"kode masih bisa dipakai"** vs **"ada reward"** — tanpa merasa fitur mati.
5. Program yang tidak berjalan **tidak boleh** ditampilkan sebagai card kosong atau gray-out.

---

## 3. Solusi

### 3.1 Prinsip Desain

1. **Kode referral selalu tampil** — tidak pernah disembunyikan meski tidak ada program reward.
2. **Hero general di semua kondisi** — headline & subheadline sama, tanpa menyebut reward, periode, atau segmen program.
3. **Hanya tampilkan card untuk program aktif** — tiap tipe punya card sendiri; tidak aktif = tidak ada card.
4. **Saat tidak ada program aktif: tidak ada card.** Ganti dengan **teks singkat**: belum ada reward, tetapi kode referral tetap bisa dibagikan. Bukan card, bukan empty state "fitur mati".
5. **Satu kode, satu tombol share** — user tidak perlu memilih segmen sebelum share.
6. **Bahasa manusia** — gunakan label tipe teman di card, bukan istilah internal.
7. **Nominal reward hanya di card program** — tidak di hero, share copy, atau banner.
8. **Periode selesai & kuota hanya di card program** — tidak di hero.
9. **Tanpa program aktif = tanpa janji nominal** di hero & share copy.

### 3.2 Aturan Penempatan Konten

| Informasi | Hero | Card Program | Teks Tanpa Program | Share Copy |
|---|---|---|---|---|
| Nominal reward | ❌ | ✅ | ❌ | ❌ |
| Periode selesai | ❌ | ✅ | ❌ | ❌ |
| Kuota ajakan | ❌ | ✅ | ❌ | ❌ |
| Tipe user (siapa yang bisa diajak) | ❌ | ✅ | ❌ | ❌ |
| Kriteria teman | ❌ | ✅ | ❌ | ❌ |
| Belum ada reward / tetap bisa share | ❌ | ❌ | ✅ | ❌ |
| Kode referral | ✅ | — | — | ✅ |

**Yang berubah antar kondisi hanya section program** — hero tetap sama.

### 3.3 Visual Hierarchy

```
1. Apa yang harus saya lakukan?     → Kode referral + tombol Bagikan
2. Siapa yang bisa saya ajak?       → Card program (jika ada)
3. Bagaimana caranya?               → Cara kerja
4. Apa status ajakan saya?          → Tracker
```

### 3.4 Kerangka Halaman

```
┌─────────────────────────────────────────┐
│  ←  Ajak Teman                          │
├─────────────────────────────────────────┤
│  [Ilustrasi referral]                   │
│  Ajak temanmu pakai Qita                │  ← hero, sama semua kondisi
│  Bagikan kode referralmu dan ajak       │
│  temanmu bergabung                      │
│  ┌─────────────────────────────────┐    │
│  │  QITA-ABC123          [Salin]   │    │
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │  [ Bagikan Sekarang /            │    │
│  │    Bagikan ke Teman ]            │    │
│  └─────────────────────────────────┘    │
│  Program Reward                         │
│  [0–4 card program]                     │  ← jika ada program aktif
│  ATAU [teks tanpa reward]               │  ← jika tidak ada program
│  Cara kerjanya                          │
│  Status ajakanmu                        │
│  Pelajari Syarat & Ketentuan →          │
└─────────────────────────────────────────┘
```

### 3.5 Komponen: Hero

| Elemen | Copy |
|---|---|
| Headline | "Ajak temanmu pakai Qita" |
| Subheadline | "Bagikan kode referralmu dan ajak temanmu bergabung" |

### 3.6 Komponen: Kode Referral + CTA

| Kondisi | Label tombol |
|---|---|
| Ada program reward aktif | **Bagikan Sekarang** |
| Tidak ada program aktif | **Bagikan ke Teman** |

Share copy (tanpa nominal): *"Coba Qita, aplikasi banking BRI yang praktis. Pakai kode {kode} saat daftar."*

### 3.7 Komponen: Card Program Aktif

**Satu program = satu card.**

**Konten dinamis per card** (nilai berubah tiap kampanye):

| Field | Contoh tampilan |
|---|---|
| **Reward referrer** | Kamu Rp25.000 |
| **Reward referee** | Temanmu Rp10.000 |
| **Periode selesai** | Berlaku s.d. 31 Agustus 2026 |
| **Kuota** | Kuota: 12/200 ajakan |
| **Tipe user** | Label tipe teman yang eligible — lihat tabel di bawah |

**Tipe user di card** = informasi **tipe teman mana yang bisa mendapat reward** pada program tersebut. Setiap program di-map ke satu tipe user; label ditampilkan dalam bahasa manusia (bukan kode internal).

| Tipe user (internal) | Label di card (tipe user) | Kriteria (checklist) |
|---|---|---|
| NTB | Teman yang belum punya rekening BRI | Belum punya rekening BRI · Belum pernah pakai BRImo/Qita |
| ETB CIF | Teman punya rekening BRI (belum BRImo) | Sudah punya rekening BRI · Belum pernah pakai Qita |
| ETB BerBRImo | Teman pengguna BRImo | Sudah punya rekening BRI/BRImo · Belum pernah pakai Qita |
| ETB X NTB | Teman yang pernah punya rekening BRI dan daftar ulang | Pernah punya rekening BRI (dormant) · Daftar ulang melalui Qita |

**Anatomi card:**

```
┌─────────────────────────────────────────────┐
│  Ajak Teman Baru di BRI            [Aktif]  │  ← judul (tetap per tipe)
│  ⏱ Berlaku s.d. 31 Agustus 2026             │  ← periode selesai
├─────────────────────────────────────────────┤
│  Teman yang belum punya rekening BRI        │  ← tipe user (label)
│  Kamu Rp25.000  ·  Temanmu Rp10.000         │  ← reward
│  Kuota: 12/200 ajakan                       │  ← kuota
├─────────────────────────────────────────────┤
│  ✓ Belum punya rekening BRI sama sekali     │  ← kriteria
│  ✓ Belum pernah pakai BRImo atau Qita       │
└─────────────────────────────────────────────┘
```

| Elemen visual | Spec |
|---|---|
| Badge "Aktif" | Pill kecil, warna brand |
| Container | Background highlight / border brand |
| Periode | Ikon ⏱ + teks secondary |
| Tipe user | Teks regular — menjelaskan siapa yang eligible |
| Nominal reward | Semibold |
| Kuota | Teks secondary |

### 3.8 Kondisi: Tidak Ada Program Aktif

**Tidak ada card.** Hanya teks:

```
Program Reward

Belum ada program reward saat ini.
Kamu tetap bisa mengajak teman ke Qita —
kode referral kamu tetap berlaku.

[🔔 Beri tahu saya saat ada program reward]   ← opsional
```

### 3.9 Kombinasi Tampilan

| Program aktif | Yang ditampilkan |
|---|---|
| Tidak ada | Teks saja — tanpa card |
| 1 tipe | 1 card |
| 2–3 tipe | 2–3 card |
| Keempatnya | 4 card (urut: NTB → ETB CIF → ETB BerBRImo → ETB X NTB) |

### 3.10 Komponen Pendukung

**Cara kerja** — 3 langkah (ada program) / 2 langkah (tanpa program):

| Tipe program | Copy langkah 3 |
|---|---|
| NTB | Teman buka rekening & lakukan transaksi pertama |
| ETB CIF | Teman aktivasi Qita dengan rekening BRI & transaksi pertama |
| ETB BerBRImo | Teman aktivasi Qita dengan akun BRImo & transaksi pertama |
| ETB X NTB | Teman daftar ulang selesaikan aktivasi & transaksi pertama |

**Tracker** — riwayat ajakan, total reward, status per teman.

**Bottom sheet** — program baru / program berakhir / reward cair.

**Entry points** (tanpa nominal): banner homepage, post-transaksi, menu profil.

### 3.11 Copywriting

- Dilarang di UI: NTB, ETB, CIF, dormant, Procash.
- Reward, periode, kuota, tipe user — **hanya di card program**.
- CTA: "Bagikan Sekarang" (ada program) vs "Bagikan ke Teman" (tanpa program).

### 3.12 Deliverables Designer

1. High-fidelity — kombinasi 0 card (teks saja), 1–4 card.
2. Card per empat tipe program (dengan field: reward, periode, kuota, tipe user).
3. State tanpa program (teks, bukan card).
4. Hero, kode, CTA, cara kerja, tracker, bottom sheet, entry points, loading skeleton.

---

## 4. Mock Up

### 4.1 Satu Program Aktif (NTB)

```
┌─────────────────────────────────────────┐
│  ←  Ajak Teman                          │
├─────────────────────────────────────────┤
│         [ilustrasi]                     │
│  Ajak temanmu pakai Qita                │
│  Bagikan kode referralmu dan ajak       │
│  temanmu bergabung                      │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │  QITA-ABC123          [Salin]   │    │
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │      Bagikan Sekarang           │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Program Reward                         │
│  ┌─────────────────────────────────┐    │
│  │ Ajak Teman Baru di BRI  [Aktif]│    │
│  │ ⏱ Berlaku s.d. 31 Agustus 2026 │    │
│  │                                 │    │
│  │ Teman yang belum punya          │    │  ← tipe user
│  │ rekening BRI                    │    │
│  │ Kamu Rp25.000 · Teman Rp10.000  │    │  ← reward
│  │ Kuota: 12/200 ajakan            │    │  ← kuota
│  │                                 │    │
│  │ ✓ Belum punya rekening BRI      │    │
│  │ ✓ Belum pernah pakai BRImo/Qita │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Cara kerjanya                          │
│  Status ajakanmu                        │
│  Pelajari Syarat & Ketentuan →          │
└─────────────────────────────────────────┘
```

### 4.2 Empat Program Aktif

```
┌─────────────────────────────────────────┐
│  Ajak temanmu pakai Qita                │
│  [Kode + Bagikan Sekarang]              │
│                                         │
│  Program Reward                         │
│  ┌─ NTB ─────────────────────────┐     │
│  │ Ajak Teman Baru di BRI [Aktif]│     │
│  │ Teman yang belum punya BRI    │     │  tipe user
│  │ Kamu Rp25.000 · Teman Rp10.000│     │  reward
│  │ Kuota: 12/200 · s.d. 31 Agt   │     │  kuota + periode
│  └───────────────────────────────┘     │
│  ┌─ ETB CIF ────────────────────┐     │
│  │ Ajak Pengguna BRI      [Aktif]│     │
│  │ Teman punya rekening BRI      │     │
│  │ Kamu Rp15.000 · Teman Rp5.000 │     │
│  └───────────────────────────────┘     │
│  ┌─ ETB BerBRImo ───────────────┐     │
│  │ Ajak Pengguna BRImo    [Aktif]│     │
│  │ Teman pengguna BRImo          │     │
│  │ Kamu Rp20.000 · Teman Rp8.000 │     │
│  └───────────────────────────────┘     │
│  ┌─ ETB X NTB ──────────────────┐     │
│  │ Ajak Teman Kembali     [Aktif]│     │
│  │ Teman daftar ulang di Qita    │     │
│  │ Kamu Rp18.000 · Teman Rp7.000 │     │
│  └───────────────────────────────┘     │
└─────────────────────────────────────────┘
```

### 4.3 Tidak Ada Program Aktif

```
┌─────────────────────────────────────────┐
│  ←  Ajak Teman                          │
├─────────────────────────────────────────┤
│         [ilustrasi]                     │
│  Ajak temanmu pakai Qita                │
│  Bagikan kode referralmu dan ajak       │
│  temanmu bergabung                      │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │  QITA-ABC123          [Salin]   │    │
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │      Bagikan ke Teman           │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Program Reward                         │
│                                         │
│  Belum ada program reward saat ini.     │  ← teks saja
│  Kamu tetap bisa mengajak teman ke      │     BUKAN card
│  Qita — kode referral kamu tetap        │
│  berlaku.                               │
│                                         │
│  🔔 Beri tahu saya saat ada program     │
│     reward                              │
│                                         │
│  Cara kerjanya (2 langkah)              │
│  Status ajakanmu                        │
│  Pelajari Syarat & Ketentuan →          │
└─────────────────────────────────────────┘
```

### 4.4 Tracker

```
┌─────────────────────────────────────────┐
│  Total reward kamu: Rp75.000            │
│  Kamu sudah mengajak 12/200             │
├─────────────────────────────────────────┤
│  👤 Budi                                │
│  Sudah gabung, tinggal transaksi pertama│  [Ingatkan]
├─────────────────────────────────────────┤
│  👤 Siti · Rp25.000 · 2 Jul 2026       │
│  Program Ajak Teman Baru                │
├─────────────────────────────────────────┤
│  👤 Andi                                │
│  Tidak memenuhi kriteria program        │
└─────────────────────────────────────────┘
```

### 4.5 Placeholder Desain

| Field | Nilai contoh |
|---|---|
| Reward referrer | Rp25.000 |
| Reward referee | Rp10.000 |
| Periode selesai | Berlaku s.d. 31 Agustus 2026 |
| Kuota | 12/200 ajakan |
| Tipe user (NTB) | Teman yang belum punya rekening BRI |

Nilai riil ditentukan tim marketing — gunakan placeholder saat desain high-fidelity.
