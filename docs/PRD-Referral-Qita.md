# PRD — Referral Hub Qita

## Konteks Bisnis

**Goals**

1. **Tujuan referral ke ETB** — meningkatkan jumlah pengguna Qita dari segmen existing BRI. Ada target akhir tahun sebanyak **X pengguna** sebelum grand launching.

**Masalah**

1. **Reward hanya berdasarkan program yang dibuat di Procash** — jika tidak ada program aktif (periode selesai atau kuota habis), referrer dan referee **tidak mendapat reward**.
2. **Pilihan misi di Procash saat ini hanya setoran** — misi setoran sesuai untuk NTB, tetapi **tidak sesuai untuk ETB** karena ETB sudah memiliki rekening BRI.
3. **ETB saat ini tanpa misi** (akan dikembangkan di sisi Procash) — jika ETB berhasil onboarding ke Qita, referral dianggap berhasil dan referrer mendapat reward tanpa syarat setoran.

---

## 1. Deskripsi

### Fitur Referral

Fitur **referral** memungkinkan pengguna Qita (referrer) mengajak orang lain (referee) bergabung menggunakan kode referral unik miliknya. Referrer membagikan kode tersebut; referee memasukkan kode saat mendaftar atau onboarding di Qita.

Alur secara umum:
1. Referrer mendapat kode referral dan membagikannya ke teman
2. Referee mendaftar atau onboarding di Qita menggunakan kode tersebut
3. Jika memenuhi syarat program reward yang sedang aktif, referrer (dan referee) mendapat reward
4. Referrer dapat memantau status setiap ajakan di riwayat

Kode referral **selalu aktif** — user bisa share kapan saja. Reward hanya diberikan jika ada **program aktif di Procash** dan referee memenuhi syarat program tersebut.

### Halaman Ajak Teman

Halaman **Ajak Teman** di aplikasi Qita — tempat user melihat kode referral, membagikannya, dan memahami **program reward mana yang sedang berjalan** serta **tipe teman siapa saja yang bisa diajak** agar referrer dan temannya mendapat reward.

Qita adalah aplikasi banking BRI. Program reward dikonfigurasi tim marketing melalui dashboard Procash (di luar scope desain ini).

**Dua lapisan yang harus dipisahkan:**

| Lapisan | Perilaku |
|---|---|
| Kode referral | Selalu aktif — user bisa share kapan saja |
| Program reward | Hanya saat ada kampanye aktif |

Meski tidak ada program reward, **kode referral tetap bisa dibagikan**. Teman tetap bisa daftar, tetapi reward hanya ada jika ada program aktif **dan** tipe teman yang diajak cocok dengan program tersebut.

**Segmen program — 4 tipe teman**

Satu program hanya menargetkan **satu tipe teman**:

| Tipe (internal) | Siapa teman yang diajak? | Judul card di UI |
|---|---|---|
| NTB | Belum punya rekening BRI sama sekali | Ajak Teman Baru di BRI |
| ETB CIF | Sudah punya rekening BRI, belum punya BRImo | Ajak Pengguna BRI |
| ETB BerBRImo | Sudah punya rekening BRI dan BRImo | Ajak Pengguna BRImo |
| ETB X NTB | Pernah punya rekening BRI (dormant), daftar ulang di Qita | Ajak Teman Kembali |

**Aturan tampilan:**
- Hanya tampilkan card untuk program yang sedang aktif
- Program tidak aktif → card tidak muncul
- Maksimal 4 card jika keempat program aktif
- **Lebih dari 1 card aktif → tampilkan dalam carousel** (swipe horizontal + indikator halaman)
- Urutan card: NTB → ETB CIF → ETB BerBRImo → ETB X NTB
- Istilah NTB, ETB, CIF, dormant **tidak boleh** muncul di UI

**Goals (desain)**

1. Satu halaman Referral Hub adaptif — template tetap, isi card mengikuti program aktif.
2. Hero general di semua kondisi — mengajak pakai Qita tanpa menyebut reward.
3. **Hanya section program reward yang berubah** — tanpa card (tidak ada program) hingga 4 card (semua program aktif).
4. Referrer membaca di halaman referral: reward (referrer & referee), periode selesai, status program, dan **tipe user** — yaitu tipe teman mana yang eligible untuk program di card tersebut (contoh: "Teman yang belum punya rekening BRI" untuk program NTB).
5. Satu program = satu tipe teman = satu card. Tidak ada multi-reward dalam satu card.

---

## 2. Masalah

1. **Referrer tidak tahu siapa yang bisa diajak agar dapat reward** — informasi tipe teman, reward, periode, dan status program harus terbaca jelas di halaman referral.
2. Kriteria teman yang tidak jelas menimbulkan **janji reward yang gagal** → komplain CS dan rusaknya kepercayaan.
3. Program berganti-ganti sepanjang waktu — desain statis akan menampilkan informasi basi.

---

## 3. Solusi

1. Kode referral selalu tampil — tidak pernah disembunyikan meski tidak ada program reward.
2. Hero general di semua kondisi — headline & subheadline sama, tanpa menyebut reward, periode, atau segmen program.
3. **Kode referral + CTA statis di semua kondisi** — tampilan dan tombol sama, baik ada maupun tidak ada program aktif.
4. Hanya tampilkan card untuk program aktif — tiap tipe punya card sendiri; tidak aktif = tidak ada card.
5. Saat tidak ada program aktif: tidak ada card. Ganti dengan teks singkat — belum ada reward, tetapi kode referral tetap bisa dibagikan.
6. Satu kode, satu tombol — user tidak perlu memilih segmen sebelum menyalin kode.
7. Bahasa manusia — gunakan label tipe teman di card, bukan istilah internal.
8. Nominal reward hanya di card program — tidak di hero atau banner.
9. Periode selesai & status program hanya di card program — tidak di hero.

**Aturan penempatan konten**

| Informasi | Hero | Kode + CTA | Card Program | Teks Tanpa Program |
|---|---|---|---|---|
| Nominal reward | ❌ | ❌ | ✅ | ❌ |
| Periode selesai | ❌ | ❌ | ✅ | ❌ |
| Tipe user (siapa yang bisa diajak) | ❌ | ❌ | ✅ | ❌ |
| Kriteria teman | ❌ | ❌ | ✅ | ❌ |
| Status program | ❌ | ❌ | ✅ | ❌ |
| Belum ada reward / tetap bisa share | ❌ | ❌ | ❌ | ✅ |
| Kode referral | ✅ | ✅ | — | — |

Yang berubah antar kondisi **hanya section program reward** — hero, kode referral, CTA, dan cara mendapatkan reward tetap sama.

**Hero**

| Elemen | Copy |
|---|---|
| Headline | "Ajak temanmu pakai Qita" |
| Subheadline | "Bagikan kode referralmu dan ajak temanmu bergabung" |

**Kode referral + CTA (statis — semua kondisi)**

| Elemen | Copy / perilaku |
|---|---|
| Kode referral | Selalu tampil |
| Tombol | **Copy kode referral** |

**Card program aktif**

Satu program = satu card. Field dinamis per card:

| Field | Contoh tampilan |
|---|---|
| Reward referrer | Kamu Rp25.000 |
| Reward referee | Temanmu Rp10.000 |
| Periode selesai | Berlaku s.d. 31 Agustus 2026 |
| Tipe user | Label tipe teman yang eligible |
| Status program | Lihat tabel di bawah |

Mapping tipe user per program:

| Tipe (internal) | Label di card | Kriteria |
|---|---|---|
| NTB | Teman yang belum punya rekening BRI | Belum punya rekening BRI · Belum pernah pakai BRImo/Qita |
| ETB CIF | Teman punya rekening BRI (belum BRImo) | Sudah punya rekening BRI · Belum pernah pakai Qita |
| ETB BerBRImo | Teman pengguna BRImo | Sudah punya rekening BRI/BRImo · Belum pernah pakai Qita |
| ETB X NTB | Teman yang pernah punya rekening BRI dan daftar ulang | Pernah punya rekening BRI (dormant) · Daftar ulang melalui Qita |

**Status program** ditampilkan di bagian card program:

| Kondisi | Contoh copy status |
|---|---|
| Kuota hampir habis | Reward program tinggal sedikit |
| Kuota habis | Reward program telah habis |
| Periode lewat | Program telah kadaluarsa |

**Tidak ada program aktif**

Tidak ada card. Hanya teks:

> Belum ada program reward saat ini.
> Kamu tetap bisa mengajak teman ke Qita — kode referral kamu tetap berlaku.

**Kombinasi tampilan**

| Program aktif | Yang ditampilkan |
|---|---|
| Tidak ada | Teks saja — tanpa card |
| 1 tipe | 1 card |
| 2–4 tipe | Carousel card (urut: NTB → ETB CIF → ETB BerBRImo → ETB X NTB) |

**Komponen pendukung**

**Cara mendapatkan reward (statis — semua kondisi program)**

Section general yang mengajak user mengundang teman sesuai program yang sedang aktif agar mendapat reward:

1. Lihat program reward yang sedang aktif di halaman ini
2. Ajak teman yang sesuai dengan tipe program tersebut
3. Bagikan kode referralmu — teman daftar atau onboarding di Qita menggunakan kodemu
4. Setelah teman memenuhi syarat program, kamu dan temanmu mendapat reward
5. Pantau progres setiap ajakan di riwayat

**Riwayat**

Riwayat ajakan milik referrer. Setiap entri menampilkan **nama referee** dan **step progress** sesuai misi program yang diikuti referee tersebut.

| Jenis program | Step progress |
|---|---|
| Ada misi setoran | ① Berhasil daftar → ② Berhasil setoran → ③ Reward dikasih |
| Tanpa misi setoran | ① Berhasil daftar → ② Reward dikasih |

Step yang sudah selesai ditandai selesai; step berikutnya menunjukkan status progres referee saat ini.

---

## 4. Mock Up

**Satu program aktif (NTB)**

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
│  │  QITA-ABC123                    │    │  ← statis
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │    Copy kode referral           │    │  ← statis
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
│  │                                 │    │
│  │ ✓ Belum punya rekening BRI      │    │
│  │ ✓ Belum pernah pakai BRImo/Qita │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Cara mendapatkan reward                │  ← statis
│  1. Lihat program reward yang aktif     │
│  2. Ajak teman sesuai tipe program      │
│  3. Bagikan kode referralmu             │
│  4. Dapat reward setelah syarat terpenuhi│
│  5. Pantau progres di riwayat           │
│                                         │
│  Riwayat →                              │
│  Pelajari Syarat & Ketentuan →          │
└─────────────────────────────────────────┘
```

**Lebih dari 1 program aktif (carousel)**

```
┌─────────────────────────────────────────┐
│  Ajak temanmu pakai Qita                │
│  ┌─────────────────────────────────┐    │
│  │  QITA-ABC123                    │    │  ← statis
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │    Copy kode referral           │    │  ← statis
│  └─────────────────────────────────┘    │
│                                         │
│  Program Reward                         │
│  ◀ ┌─────────────────────────────┐ ▶   │  ← carousel
│    │ Ajak Teman Baru di BRI[Aktif]│   │
│    │ ⏱ Berlaku s.d. 31 Agustus    │   │
│    │ Teman yang belum punya BRI   │   │  tipe user
│    │ Kamu Rp25.000 · Teman Rp10rb │   │  reward
│    └─────────────────────────────┘     │
│              ● ○ ○ ○                    │  ← indikator
│                                         │
│  (swipe untuk card program lainnya)     │
└─────────────────────────────────────────┘
```

**Status program di card**

```
┌─────────────────────────────────────────┐
│  Ajak Teman Baru di BRI                 │
│  Kamu Rp25.000 · Teman Rp10.000           │
│  ⚠ Reward program tinggal sedikit         │  ← status
├─────────────────────────────────────────┤
│  Ajak Pengguna BRI                      │
│  Kamu Rp15.000 · Teman Rp5.000            │
│  ✕ Reward program telah habis           │  ← status
├─────────────────────────────────────────┤
│  Ajak Pengguna BRImo                    │
│  Berlaku s.d. 30 Juni 2026              │
│  ⏱ Program telah kadaluarsa              │  ← status
└─────────────────────────────────────────┘
```

**Tidak ada program aktif**

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
│  │  QITA-ABC123                    │    │  ← statis
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │    Copy kode referral           │    │  ← statis
│  └─────────────────────────────────┘    │
│                                         │
│  Program Reward                         │
│                                         │
│  Belum ada program reward saat ini.     │  ← teks saja
│  Kamu tetap bisa mengajak teman ke      │     BUKAN card
│  Qita — kode referral kamu tetap        │
│  berlaku.                               │
│                                         │
│  Cara mendapatkan reward                │  ← statis
│  1. Lihat program reward yang aktif     │
│  2. Ajak teman sesuai tipe program      │
│  3. Bagikan kode referralmu             │
│  4. Dapat reward setelah syarat terpenuhi│
│  5. Pantau progres di riwayat           │
│                                         │
│  Riwayat →                              │
│  Pelajari Syarat & Ketentuan →          │
└─────────────────────────────────────────┘
```

**Riwayat — program dengan misi setoran (3 step)**

```
┌─────────────────────────────────────────┐
│  ←  Riwayat Ajakan                      │
├─────────────────────────────────────────┤
│  👤 Budi                                │
│  Program: Ajak Teman Baru di BRI        │
│                                         │
│  ● Berhasil daftar          ✓           │
│  ○ Berhasil setoran         ← saat ini  │
│  ○ Reward dikasih                       │
├─────────────────────────────────────────┤
│  👤 Siti                                │
│  Program: Ajak Teman Baru di BRI        │
│                                         │
│  ● Berhasil daftar          ✓           │
│  ● Berhasil setoran         ✓           │
│  ● Reward dikasih           ✓  Rp25.000 │
└─────────────────────────────────────────┘
```

**Riwayat — program tanpa misi setoran (2 step)**

```
┌─────────────────────────────────────────┐
│  ←  Riwayat Ajakan                      │
├─────────────────────────────────────────┤
│  👤 Andi                                │
│  Program: Ajak Pengguna BRI             │
│                                         │
│  ● Berhasil daftar          ✓           │
│  ○ Reward dikasih           ← saat ini  │
├─────────────────────────────────────────┤
│  👤 Rina                                │
│  Program: Ajak Pengguna BRImo         │
│                                         │
│  ● Berhasil daftar          ✓           │
│  ● Reward dikasih           ✓  Rp20.000 │
└─────────────────────────────────────────┘
```
