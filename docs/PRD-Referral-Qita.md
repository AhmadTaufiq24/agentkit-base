# PRD — Referral Hub Qita

## 1. Deskripsi

Halaman **Ajak Teman** di aplikasi Qita — tempat user melihat kode referral, membagikannya, dan memahami **program reward mana yang sedang berjalan** serta **tipe teman siapa saja yang bisa diajak** agar referrer dan temannya mendapat reward.

Qita adalah aplikasi banking BRI. Program reward dikonfigurasi tim marketing melalui dashboard internal (di luar scope desain ini).

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
- Urutan card: NTB → ETB CIF → ETB BerBRImo → ETB X NTB
- Istilah NTB, ETB, CIF, dormant **tidak boleh** muncul di UI

**Goals**

1. Satu halaman Referral Hub adaptif — template tetap, isi card mengikuti program aktif.
2. Hero general di semua kondisi — mengajak pakai Qita tanpa menyebut reward.
3. **Hanya section program reward yang berubah** — tanpa card (tidak ada program) hingga 4 card (semua program aktif).
4. Referrer membaca di halaman referral: reward (referrer & referee), periode selesai, kuota, dan **tipe user** — yaitu tipe teman mana yang eligible untuk program di card tersebut (contoh: "Teman yang belum punya rekening BRI" untuk program NTB).
5. Satu program = satu tipe teman = satu card. Tidak ada multi-reward dalam satu card.

---

## 2. Masalah

1. **Referrer tidak tahu siapa yang bisa diajak agar dapat reward** — informasi tipe teman, reward, periode, dan kuota harus terbaca jelas di halaman referral.
2. Kriteria teman yang tidak jelas menimbulkan **janji reward yang gagal** → komplain CS dan rusaknya kepercayaan.
3. Program berganti-ganti sepanjang waktu — desain statis akan menampilkan informasi basi.
4. User perlu paham perbedaan **"kode masih bisa dipakai"** vs **"ada reward"** — tanpa merasa fitur mati.
5. Program yang tidak berjalan **tidak boleh** ditampilkan sebagai card kosong atau gray-out.

---

## 3. Solusi

**Prinsip desain**

1. Kode referral selalu tampil — tidak pernah disembunyikan meski tidak ada program reward.
2. Hero general di semua kondisi — headline & subheadline sama, tanpa menyebut reward, periode, atau segmen program.
3. **Kode referral + CTA statis di semua kondisi** — tampilan dan label tombol share sama, baik ada maupun tidak ada program aktif.
4. Hanya tampilkan card untuk program aktif — tiap tipe punya card sendiri; tidak aktif = tidak ada card.
5. Saat tidak ada program aktif: tidak ada card. Ganti dengan teks singkat — belum ada reward, tetapi kode referral tetap bisa dibagikan.
6. Satu kode, satu tombol share — user tidak perlu memilih segmen sebelum share.
7. Bahasa manusia — gunakan label tipe teman di card, bukan istilah internal.
8. Nominal reward hanya di card program — tidak di hero, share copy, atau banner.
9. Periode selesai & kuota hanya di card program — tidak di hero.
10. Tanpa program aktif = tanpa janji nominal di hero & share copy.

**Aturan penempatan konten**

| Informasi | Hero | Kode + CTA | Card Program | Teks Tanpa Program | Share Copy |
|---|---|---|---|---|---|
| Nominal reward | ❌ | ❌ | ✅ | ❌ | ❌ |
| Periode selesai | ❌ | ❌ | ✅ | ❌ | ❌ |
| Kuota ajakan | ❌ | ❌ | ✅ | ❌ | ❌ |
| Tipe user (siapa yang bisa diajak) | ❌ | ❌ | ✅ | ❌ | ❌ |
| Kriteria teman | ❌ | ❌ | ✅ | ❌ | ❌ |
| Status program | ❌ | ❌ | ✅ | ❌ | ❌ |
| Belum ada reward / tetap bisa share | ❌ | ❌ | ❌ | ✅ | ❌ |
| Kode referral | ✅ | ✅ | — | — | ✅ |

Yang berubah antar kondisi **hanya section program reward** — hero, kode referral, CTA, dan cara kerja tetap sama.

**Hero**

| Elemen | Copy |
|---|---|
| Headline | "Ajak temanmu pakai Qita" |
| Subheadline | "Bagikan kode referralmu dan ajak temanmu bergabung" |

**Kode referral + CTA (statis — semua kondisi)**

| Elemen | Copy / perilaku |
|---|---|
| Kode referral | Selalu tampil dengan tombol Salin |
| Label tombol | **Bagikan Sekarang** |
| Share copy | *"Coba Qita, aplikasi banking BRI yang praktis. Pakai kode {kode} saat daftar."* |

**Card program aktif**

Satu program = satu card. Field dinamis per card:

| Field | Contoh tampilan |
|---|---|
| Reward referrer | Kamu Rp25.000 |
| Reward referee | Temanmu Rp10.000 |
| Periode selesai | Berlaku s.d. 31 Agustus 2026 |
| Kuota | Kuota: 12/200 ajakan |
| Tipe user | Label tipe teman yang eligible |
| Status program | Lihat tabel di bawah |

Mapping tipe user per program:

| Tipe (internal) | Label di card | Kriteria |
|---|---|---|
| NTB | Teman yang belum punya rekening BRI | Belum punya rekening BRI · Belum pernah pakai BRImo/Qita |
| ETB CIF | Teman punya rekening BRI (belum BRImo) | Sudah punya rekening BRI · Belum pernah pakai Qita |
| ETB BerBRImo | Teman pengguna BRImo | Sudah punya rekening BRI/BRImo · Belum pernah pakai Qita |
| ETB X NTB | Teman yang pernah punya rekening BRI dan daftar ulang | Pernah punya rekening BRI (dormant) · Daftar ulang melalui Qita |

**Status program** ditampilkan di bagian card program (bukan bottom sheet terpisah):

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
| 2–3 tipe | 2–3 card |
| Keempatnya | 4 card (urut: NTB → ETB CIF → ETB BerBRImo → ETB X NTB) |

**Komponen pendukung**

**Cara kerja (statis — semua kondisi program)**

Step-step general untuk membagikan kode referral, tidak berubah meski program aktif berbeda:

1. Salin atau bagikan kode referralmu ke teman
2. Ajak teman daftar di Qita menggunakan kodemu
3. Cek status ajakan di riwayat

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
│  │  QITA-ABC123          [Salin]   │    │  ← statis
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │      Bagikan Sekarang           │    │  ← statis
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
│  Cara kerjanya                          │  ← statis
│  1. Salin atau bagikan kode ke teman    │
│  2. Ajak teman daftar pakai kodemu      │
│  3. Cek status ajakan di riwayat        │
│                                         │
│  Riwayat →                              │
│  Pelajari Syarat & Ketentuan →          │
└─────────────────────────────────────────┘
```

**Empat program aktif**

```
┌─────────────────────────────────────────┐
│  Ajak temanmu pakai Qita                │
│  [Kode + Bagikan Sekarang]              │  ← statis
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

**Status program di card**

```
┌─────────────────────────────────────────┐
│  Ajak Teman Baru di BRI                 │
│  Kamu Rp25.000 · Teman Rp10.000           │
│  Kuota: 198/200 ajakan                  │
│  ⚠ Reward program tinggal sedikit         │  ← status
├─────────────────────────────────────────┤
│  Ajak Pengguna BRI                      │
│  Kamu Rp15.000 · Teman Rp5.000            │
│  Kuota: 200/200 ajakan                  │
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
│  │  QITA-ABC123          [Salin]   │    │  ← statis
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │      Bagikan Sekarang           │    │  ← statis
│  └─────────────────────────────────┘    │
│                                         │
│  Program Reward                         │
│                                         │
│  Belum ada program reward saat ini.     │  ← teks saja
│  Kamu tetap bisa mengajak teman ke      │     BUKAN card
│  Qita — kode referral kamu tetap        │
│  berlaku.                               │
│                                         │
│  Cara kerjanya                          │  ← statis
│  1. Salin atau bagikan kode ke teman    │
│  2. Ajak teman daftar pakai kodemu      │
│  3. Cek status ajakan di riwayat        │
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
