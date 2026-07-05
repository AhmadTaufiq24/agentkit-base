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
3. Hanya section card yang berubah — tanpa card (tidak ada program) hingga 4 card (semua program aktif).
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
3. Hanya tampilkan card untuk program aktif — tiap tipe punya card sendiri; tidak aktif = tidak ada card.
4. Saat tidak ada program aktif: tidak ada card. Ganti dengan teks singkat — belum ada reward, tetapi kode referral tetap bisa dibagikan.
5. Satu kode, satu tombol share — user tidak perlu memilih segmen sebelum share.
6. Bahasa manusia — gunakan label tipe teman di card, bukan istilah internal.
7. Nominal reward hanya di card program — tidak di hero, share copy, atau banner.
8. Periode selesai & kuota hanya di card program — tidak di hero.
9. Tanpa program aktif = tanpa janji nominal di hero & share copy.

**Aturan penempatan konten**

| Informasi | Hero | Card Program | Teks Tanpa Program | Share Copy |
|---|---|---|---|---|
| Nominal reward | ❌ | ✅ | ❌ | ❌ |
| Periode selesai | ❌ | ✅ | ❌ | ❌ |
| Kuota ajakan | ❌ | ✅ | ❌ | ❌ |
| Tipe user (siapa yang bisa diajak) | ❌ | ✅ | ❌ | ❌ |
| Kriteria teman | ❌ | ✅ | ❌ | ❌ |
| Belum ada reward / tetap bisa share | ❌ | ❌ | ✅ | ❌ |
| Kode referral | ✅ | — | — | ✅ |

Yang berubah antar kondisi hanya section program — hero tetap sama.

**Hero**

| Elemen | Copy |
|---|---|
| Headline | "Ajak temanmu pakai Qita" |
| Subheadline | "Bagikan kode referralmu dan ajak temanmu bergabung" |

**Kode referral + CTA**

| Kondisi | Label tombol |
|---|---|
| Ada program reward aktif | Bagikan Sekarang |
| Tidak ada program aktif | Bagikan ke Teman |

Share copy (tanpa nominal): *"Coba Qita, aplikasi banking BRI yang praktis. Pakai kode {kode} saat daftar."*

**Card program aktif**

Satu program = satu card. Field dinamis per card:

| Field | Contoh tampilan |
|---|---|
| Reward referrer | Kamu Rp25.000 |
| Reward referee | Temanmu Rp10.000 |
| Periode selesai | Berlaku s.d. 31 Agustus 2026 |
| Kuota | Kuota: 12/200 ajakan |
| Tipe user | Label tipe teman yang eligible |

Mapping tipe user per program:

| Tipe (internal) | Label di card | Kriteria |
|---|---|---|
| NTB | Teman yang belum punya rekening BRI | Belum punya rekening BRI · Belum pernah pakai BRImo/Qita |
| ETB CIF | Teman punya rekening BRI (belum BRImo) | Sudah punya rekening BRI · Belum pernah pakai Qita |
| ETB BerBRImo | Teman pengguna BRImo | Sudah punya rekening BRI/BRImo · Belum pernah pakai Qita |
| ETB X NTB | Teman yang pernah punya rekening BRI dan daftar ulang | Pernah punya rekening BRI (dormant) · Daftar ulang melalui Qita |

**Tidak ada program aktif**

Tidak ada card. Hanya teks:

> Belum ada program reward saat ini.
> Kamu tetap bisa mengajak teman ke Qita — kode referral kamu tetap berlaku.

Opsional: tombol "Beri tahu saya saat ada program reward".

**Kombinasi tampilan**

| Program aktif | Yang ditampilkan |
|---|---|
| Tidak ada | Teks saja — tanpa card |
| 1 tipe | 1 card |
| 2–3 tipe | 2–3 card |
| Keempatnya | 4 card (urut: NTB → ETB CIF → ETB BerBRImo → ETB X NTB) |

**Komponen pendukung**

- **Cara kerja** — 3 langkah (ada program) / 2 langkah (tanpa program)
- **Tracker** — riwayat ajakan, total reward, status per teman
- **Bottom sheet** — program baru / program berakhir / reward cair
- **Entry points** (tanpa nominal) — banner homepage, post-transaksi, menu profil

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

**Empat program aktif**

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

**Tracker**

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
