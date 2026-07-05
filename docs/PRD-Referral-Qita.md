# PRD — Referral Hub Qita (Spesifikasi Desain)

| | |
|---|---|
| **Dokumen** | Product Requirements Document (PRD) |
| **Fitur** | Referral Hub Qita |
| **Audiens** | **UI/UX Designer** |
| **Status** | Draft v2.0 (Designer) |
| **Product Owner** | Tim Product Qita |
| **Tanggal** | Juli 2026 |

### Changelog

| Versi | Perubahan utama |
|---|---|
| v1.0–v1.12 | Iterasi model bisnis & spesifikasi teknis |
| **v2.0** | **PRD khusus designer** — hapus API/kode; perbarui state tanpa program (teks saja, tanpa card) |

---

## 1. Konteks Singkat

Qita adalah aplikasi banking BRI. Fitur referral dikelola melalui dashboard **Procash** (di luar scope desain ini).

Dua hal yang perlu dipisahkan dalam desain:

| Lapisan | Perilaku |
|---|---|
| **Kode referral** | Selalu aktif — user bisa share kapan saja |
| **Program reward** | Hanya saat ada kampanye aktif di Procash |

**Keputusan produk:** meski tidak ada program reward, kode referral **tetap bisa dibagikan**. Teman tetap bisa daftar, tetapi reward hanya ada jika program aktif **dan** tipe teman yang diajak cocok dengan salah satu program yang sedang berjalan.

---

## 2. Segmen Program (Tipe Teman yang Bisa Diajak)

Satu program di Procash **hanya menargetkan satu tipe teman**. Ada **empat tipe** — masing-masing punya program (dan card) terpisah jika aktif:

| Tipe (internal) | Siapa teman yang diajak? | Judul card di UI |
|---|---|---|
| **NTB** | Belum punya rekening BRI sama sekali | Ajak Teman Baru di BRI |
| **ETB CIF** | Sudah punya rekening BRI, belum punya BRImo | Ajak Pengguna BRI |
| **ETB BerBRImo** | Sudah punya rekening BRI dan BRImo | Ajak Pengguna BRImo |
| **ETB X NTB** | Pernah punya rekening BRI (dormant), daftar ulang di Qita | Ajak Teman Kembali |

**Aturan tampilan:**
- Hanya tampilkan **card untuk program yang sedang aktif**
- Program tidak aktif → **card tidak muncul** (bukan card kosong / gray-out)
- Maksimal **4 card** jika keempat program aktif bersamaan
- **Urutan card (tetap):** NTB → ETB CIF → ETB BerBRImo → ETB X NTB

> **Penting untuk designer:** istilah NTB, ETB, CIF, dormant **tidak boleh** muncul di UI. Gunakan copy manusiawi di tabel card di bawah.

---

## 3. Masalah yang Diselesaikan

1. **Referrer perlu tahu siapa yang bisa diajak agar dapat reward** — berdasarkan **program yang sedang aktif**, informasinya harus terbaca jelas di **halaman referral** (bukan ditebak sendiri).
2. Jika kriteria teman tidak jelas, terjadi janji reward yang gagal → komplain CS.
3. Program berganti-ganti; desain harus adaptif tanpa terlihat "basi".
4. User perlu paham: **kode tetap bisa dipakai** meski belum ada reward.
5. Hanya program yang berjalan yang ditampilkan sebagai card.

---

## 4. Goals

1. Satu halaman Referral Hub yang **adaptif** — template tetap, isi card mengikuti program aktif.
2. Hero **general** di semua kondisi — mengajak pakai Qita tanpa menyebut reward.
3. **Hanya section card yang berubah** — 0 card (tanpa program) hingga 4 card (semua program aktif).
4. **Referrer membaca info reward di halaman referral** — siapa yang bisa diajak, nominal, periode, kuota — langsung dari card program yang tampil.
5. Satu program = satu tipe teman. Tidak ada multi-reward dalam satu card.

---

## 5. Prinsip Desain

1. **Kode referral selalu tampil** — tidak pernah disembunyikan meski tidak ada program reward.
2. **Hero general di semua kondisi** — headline & subheadline sama, tanpa menyebut reward, periode, atau segmen program.
3. **Hanya tampilkan card untuk program aktif** — tiap tipe punya card sendiri; tidak aktif = tidak ada card.
4. **Saat tidak ada program aktif: tidak ada card.** Ganti dengan **teks singkat** di area program: belum ada reward, tetapi kode referral tetap bisa dibagikan. Bukan card, bukan empty state "fitur mati".
5. **Satu kode, satu tombol share** — user tidak perlu memilih segmen sebelum share.
6. **Bahasa manusia** — label dan kriteria pakai copy di Section 7, bukan istilah internal.
7. **Nominal reward hanya di card program** — tidak di hero, tidak di share copy, tidak di banner.
8. **Periode selesai & kuota hanya di card program** — tidak di hero.
9. **Tanpa program aktif = tanpa janji nominal** di hero & share copy.
10. **Jangan tampilkan nominal usang** — jika data program gagal dimuat, tampilkan kondisi tanpa program (teks + kode tetap ada).

---

## 6. Spesifikasi UI Lengkap

### 6.1 Aturan Penempatan Konten

| Informasi | Hero | Card Program | Teks Tanpa Program | Share Copy |
|---|---|---|---|---|
| Nominal reward | ❌ | ✅ | ❌ | ❌ |
| Periode selesai | ❌ | ✅ | ❌ | ❌ |
| Kuota ajakan | ❌ | ✅ | ❌ | ❌ |
| Siapa yang bisa diajak / kriteria | ❌ | ✅ | ❌ | ❌ |
| Belum ada reward / tetap bisa share | ❌ | ❌ | ✅ | ❌ |
| Kode referral | ✅ | — | — | ✅ |

**Yang berubah antar kondisi hanya section program** — hero tetap sama.

### 6.2 Visual Hierarchy

```
1. Apa yang harus saya lakukan?      → Kode referral + tombol Bagikan
2. Siapa yang bisa saya ajak?         → Card program (jika ada)
3. Bagaimana caranya?                → Cara kerja
4. Apa status ajakan saya?           → Tracker
```

### 6.3 Kerangka Halaman (Template Tetap)

```
┌─────────────────────────────────────────┐
│  ←  Ajak Teman                          │
├─────────────────────────────────────────┤
│  [Ilustrasi referral]                   │
│                                         │
│  Ajak temanmu pakai Qita                │  ← hero, sama semua kondisi
│  Bagikan kode referralmu dan ajak       │
│  temanmu bergabung                      │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │  QITA-ABC123          [Salin]   │    │
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │  [ Bagikan Sekarang /            │    │
│  │    Bagikan ke Teman ]            │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Program Reward                         │
│  [0–4 card program]                     │  ← hanya jika ada program aktif
│  ATAU                                   │
│  [Teks: belum ada reward, tetap share]  │  ← jika tidak ada program
│                                         │
│  Cara kerjanya                          │
│  Status ajakanmu                        │
│  Pelajari Syarat & Ketentuan →          │
└─────────────────────────────────────────┘
```

**Above the fold:** ilustrasi → headline → kode → tombol share.

### 6.4 Komponen: Hero

| Elemen | Copy |
|---|---|
| Headline | "Ajak temanmu pakai Qita" |
| Subheadline | "Bagikan kode referralmu dan ajak temanmu bergabung" |

**Larangan:** nominal reward, periode, segmen program, kata "reward".

### 6.5 Komponen: Kode Referral + CTA

```
┌──────────────────────────────────┐
│  QITA-ABC123            [Salin]  │
└──────────────────────────────────┘
```

| Kondisi | Label tombol |
|---|---|
| Ada program reward aktif | **Bagikan Sekarang** |
| Tidak ada program aktif | **Bagikan ke Teman** |

Tap [Salin] → toast "Kode berhasil disalin".

**Share copy (tanpa nominal):** "Coba Qita, aplikasi banking BRI yang praktis. Pakai kode {kode} saat daftar."

### 6.6 Komponen: Card Program Aktif

**Satu program = satu card = satu pasang reward.**

**Konten dinamis di card** (nilai berubah per program/kampanye):
- Periode selesai — contoh: "Berlaku s.d. 31 Agustus 2026"
- Reward referrer — contoh: "Kamu Rp25.000"
- Reward referee — contoh: "Temanmu Rp10.000"
- Kuota — contoh: "Kuota: 12/200 ajakan"

**Konten tetap di card** (copy desain per tipe program):

| Tipe program | Label tipe teman | Kriteria (checklist) |
|---|---|---|
| NTB | Teman yang belum punya rekening BRI | Belum punya rekening BRI sama sekali · Belum pernah pakai BRImo atau Qita |
| ETB CIF | Teman punya rekening BRI (belum BRImo) | Sudah punya rekening BRI · Belum pernah pakai Qita |
| ETB BerBRImo | Teman pengguna BRImo | Sudah punya rekening BRI atau BRImo · Belum pernah pakai Qita |
| ETB X NTB | Teman yang pernah punya rekening BRI dan daftar ulang | Pernah punya rekening BRI (dormant) · Daftar ulang melalui Qita |

**Anatomi card:**

```
┌─────────────────────────────────────────────┐
│  Ajak Teman Baru di BRI            [Aktif]  │
│  ⏱ Berlaku s.d. 31 Agustus 2026             │
├─────────────────────────────────────────────┤
│  Teman yang belum punya rekening BRI        │
│  Kamu Rp25.000  ·  Temanmu Rp10.000         │
│  Kuota: 12/200 ajakan                       │
├─────────────────────────────────────────────┤
│  ✓ Belum punya rekening BRI sama sekali     │
│  ✓ Belum pernah pakai BRImo atau Qita       │
└─────────────────────────────────────────────┘
```

**Spesifikasi visual:**

| Elemen | Spec |
|---|---|
| Badge "Aktif" | Pill kecil, warna brand, pojok kanan atas |
| Container | Background highlight subtle / border brand |
| Periode | Ikon ⏱ + teks secondary |
| Nominal reward | Semibold |
| Kuota | Teks secondary |
| Kriteria | Ikon ✓ + teks secondary |

### 6.7 Kondisi: Tidak Ada Program Aktif

**Tidak ada card.** Hanya teks di bawah section header "Program Reward":

```
Program Reward

Belum ada program reward saat ini.
Kamu tetap bisa mengajak teman ke Qita —
kode referral kamu tetap berlaku.

[🔔 Beri tahu saya saat ada program reward]   ← opsional
```

| Elemen | Spec |
|---|---|
| Teks | Body regular, warna secondary — bukan card, bukan alert merah |
| Posisi | Menggantikan area card — bukan di hero |
| Konten | Tanpa nominal, tanpa periode, tanpa kuota |

### 6.8 Kombinasi Tampilan Card

| Program aktif | Yang ditampilkan |
|---|---|
| Tidak ada | Teks saja (Section 6.7) — **tanpa card** |
| NTB saja | 1 card NTB |
| ETB CIF saja | 1 card ETB CIF |
| NTB + ETB CIF | 2 card |
| Keempatnya | 4 card (urut: NTB → ETB CIF → ETB BerBRImo → ETB X NTB) |

---

## 7. Wireframe

### 7.1 Ada Program — Contoh 1 Card (NTB Aktif)

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
│  │ Teman belum punya rekening BRI  │    │
│  │ Kamu Rp25.000 · Teman Rp10.000  │    │
│  │ Kuota: 12/200 ajakan            │    │
│  │ ✓ Belum punya rekening BRI      │    │
│  │ ✓ Belum pernah pakai BRImo/Qita │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Cara kerjanya · Status ajakanmu        │
│  Pelajari Syarat & Ketentuan →          │
└─────────────────────────────────────────┘
```

### 7.2 Ada Program — Contoh 4 Card (Semua Aktif)

```
┌─────────────────────────────────────────┐
│  Ajak temanmu pakai Qita                │
│  [Kode + Bagikan Sekarang]              │
│                                         │
│  Program Reward                         │
│  ┌─ NTB ──────────────────────────┐      │
│  │ Ajak Teman Baru di BRI [Aktif]│      │
│  │ Kamu Rp25.000 · Teman Rp10.000 │      │
│  └───────────────────────────────┘      │
│  ┌─ ETB CIF ─────────────────────┐      │
│  │ Ajak Pengguna BRI      [Aktif] │      │
│  │ Kamu Rp15.000 · Teman Rp5.000  │      │
│  └───────────────────────────────┘      │
│  ┌─ ETB BerBRImo ────────────────┐      │
│  │ Ajak Pengguna BRImo    [Aktif] │      │
│  │ Kamu Rp20.000 · Teman Rp8.000  │      │
│  └───────────────────────────────┘      │
│  ┌─ ETB X NTB ──────────────────┐      │
│  │ Ajak Teman Kembali     [Aktif] │      │
│  │ Kamu Rp18.000 · Teman Rp7.000  │      │
│  └───────────────────────────────┘      │
└─────────────────────────────────────────┘
```

### 7.3 Tidak Ada Program Aktif

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
│  Belum ada program reward saat ini.     │  ← teks saja, BUKAN card
│  Kamu tetap bisa mengajak teman ke      │
│  Qita — kode referral kamu tetap        │
│  berlaku.                               │
│                                         │
│  🔔 Beri tahu saya saat ada program     │
│     reward                              │
│                                         │
│  Cara kerjanya (2 langkah)              │
│  Status ajakanmu (riwayat lama tetap)   │
│  Pelajari Syarat & Ketentuan →          │
└─────────────────────────────────────────┘
```

---

## 8. Komponen Lainnya

### 8.1 Cara Kerja

**3 langkah** jika ada program aktif / **2 langkah** jika tidak ada program.

```
①  Bagikan kode referral kamu
②  Teman gabung Qita pakai kodemu
③  Teman selesaikan syarat program     ← hanya jika ada program
```

Langkah 3 — satu bullet per tipe program yang aktif:

| Tipe program | Copy langkah 3 |
|---|---|
| NTB | Teman buka rekening & lakukan setoran/transaksi pertama |
| ETB CIF | Teman aktivasi Qita dengan rekening BRI & transaksi pertama |
| ETB BerBRImo | Teman aktivasi Qita dengan akun BRImo & transaksi pertama |
| ETB X NTB | Teman yang daftar ulang selesaikan aktivasi & transaksi pertama |

### 8.2 Tracker "Status Ajakanmu"

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

### 8.3 Bottom Sheet & Notifikasi

| Trigger | Konten |
|---|---|
| Program reward baru | "Program reward baru! Cek detail di halaman Ajak Teman." [Mengerti, Bagikan Sekarang] |
| Program berakhir | "Program reward sudah berakhir. Kode kamu tetap bisa dibagikan." [Mengerti] |
| Reward cair | Snackbar: "🎉 Rp25.000 sudah masuk dari ajakan ke Budi!" |
| First-time visit | Coachmark cara kerja (sekali, dismissible) |

### 8.4 Entry Points (di luar Referral Hub)

| Entry point | Ada program | Tanpa program |
|---|---|---|
| Banner homepage | "Ajak temanmu ke Qita — ada program reward!" | Sembunyikan banner |
| Post-transaksi | "Kenalkan Qita ke temanmu & cek reward-nya" | "Kenalkan Qita ke temanmu" |
| Menu profil | "Ajak Teman" | Sama — selalu ada |

**Semua entry point tanpa nominal reward.**

### 8.5 Loading State

Skeleton: hero + kode + slot card (1–4) sesuai kondisi terakhir user.

---

## 9. Copywriting Guidelines

- **Dilarang di UI:** NTB, ETB, CIF, dormant, Procash.
- **Nominal reward & periode** hanya di card program.
- **Program tidak aktif** → card tidak muncul.
- **Tanpa program** → teks informatif, bukan card.
- **Share copy** tanpa nominal — fokus value prop Qita.
- **CTA:** "Bagikan Sekarang" (ada program) vs "Bagikan ke Teman" (tanpa program).
- **Tracker:** gunakan bahasa kriteria yang sama dengan checklist di card.

---

## 10. Deliverables Designer

1. High-fidelity Referral Hub — semua kombinasi: 0 card (teks saja), 1 card, 2 card, 3 card, 4 card.
2. Komponen card per empat tipe program (Section 6.6).
3. State tanpa program — teks informatif, bukan card (Section 6.7 & wireframe 7.3).
4. Hero, kode, CTA, cara kerja, tracker, bottom sheet.
5. Entry points tanpa nominal.
6. Loading skeleton.
7. Spec truncation & dynamic type untuk nominal/kuota yang panjang.

---

## 11. Referensi Visual (Opsional)

Gunakan placeholder untuk konten dinamis saat desain:

| Field | Placeholder desain |
|---|---|
| Reward referrer | Rp25.000 |
| Reward referee | Rp10.000 |
| Periode | Berlaku s.d. 31 Agustus 2026 |
| Kuota | 12/200 ajakan |

Nilai riil ditentukan tim marketing via Procash — bukan scope desain.
