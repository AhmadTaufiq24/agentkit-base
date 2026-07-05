# PRD — Referral Hub Qita (Adaptif Berdasarkan Program Procash)

| | |
|---|---|
| **Dokumen** | Product Requirements Document (PRD) |
| **Fitur** | Referral Hub Qita |
| **Audiens dokumen** | UI/UX Designer, Frontend, Backend |
| **Status** | Draft v1.12 |
| **Product Owner** | Tim Product Qita |
| **Tanggal** | Juli 2026 |

### Changelog

| Versi | Perubahan utama |
|---|---|
| v1.0–v1.3 | Model bisnis, state referral, eligibility referee |
| v1.4 | `rewards_by_user_type[]` per tipe user |
| v1.5 | Reward nominal hanya di card |
| v1.6 | Masa berlaku hanya di card; copy card tidak aktif |
| v1.7 | Spesifikasi UI lengkap (Section 6) |
| v1.8 | Hero general tanpa reward; card hanya program aktif; satu card info saat tidak ada program |
| v1.9 | Card program: hanya map kuota, reward, periode selesai dari API; sisanya hardcode |
| v1.10 | Satu program = satu `user_type`; NTB & ETB = program terpisah |
| v1.11 | Tiga tipe referee (NTB, ETB CIF, ETB BerBRImo); unik per `user_type`; maks 3 card |
| v1.12 | Tambah `ETB_X_NTB` sebagai tipe ke-4; maks 4 program/card aktif |

---

## 1. Latar Belakang

Qita adalah aplikasi banking BRI (sejenis BRImo). Qita memiliki fitur referral yang engine-nya dikelola melalui dashboard **Procash**.

### Model Bisnis Referral

Referral di Qita terdiri dari **dua lapisan** yang harus dipisahkan dalam desain:

| Lapisan | Perilaku | Kapan aktif |
|---|---|---|
| **Infrastruktur referral** | Kode/link referral unik per user, attribution, tracking ajakan | **Selalu aktif** — tidak pernah dimatikan |
| **Program reward (kampanye)** | Nominal reward, syarat kualifikasi, periode, kuota | **Hanya saat program aktif di Procash** dan referee memenuhi syarat |

**Keputusan produk:** kode referral **tetap bisa digunakan** meskipun tidak ada program reward aktif. Teman tetap bisa mendaftar dengan kode tersebut (attribution tercatat), tetapi **tidak ada reward** kecuali ada program aktif **dan** tipe user referee cocok dengan segmen program tersebut.

### Aturan Eligibility

```
Referrer (tipe apapun)  →  share kode  →  Referee daftar
                                                  │
                                                  ▼
                                        Sistem cek tipe REFEREE
                                        (NIK → CIF)
                                                  │
                        ┌─────────────────────────┼─────────────────────────┐
                        ▼                         ▼                         ▼
                  Referee NTB               Referee ETB CIF         Referee ETB BerBRImo      Referee ETB X NTB
                  + program NTB aktif       + program ETB_CIF aktif + program ETB_BERBRIMO    + program ETB_X_NTB aktif
                        │                         │                         │                         │
                        ▼                         ▼                         ▼                         ▼
                  Reward NTB                  Reward ETB CIF            Reward ETB BerBRImo       Reward ETB X NTB
                  (referrer + referee)        (referrer + referee)      (referrer + referee)

                  Tidak ada program aktif dengan user_type cocok → Tanpa reward (attribution saja)
```

| Pihak | Dicek tipe user? | Keterangan |
|---|---|---|
| **Referrer** | **Tidak** | Siapa pun tipe user-nya (NTB, ETB, ETB BerBRImo) bisa share dan dapat reward selama temannya memenuhi syarat program aktif |
| **Referee** | **Ya** | Tipe user referee (**NTB**, **ETB CIF**, **ETB BerBRImo**, **ETB X NTB**) menentukan program mana yang apply |

### Konfigurasi Program di Procash

**Aturan wajib Procash:**

| Aturan | Keterangan |
|---|---|
| 1 program = 1 `user_type` | Saat buat program, pilih **satu** tipe user referee saja |
| 4 tipe user referee | `NTB`, `ETB_CIF`, `ETB_BERBRIMO`, `ETB_X_NTB` — masing-masing program terpisah |
| Unik per tipe | **Tidak boleh** ada 2 program aktif dengan `user_type` yang sama |
| Maks aktif bersamaan | Paling banyak **4 program aktif** — satu per tipe user |

Contoh konfigurasi:

| Program | `user_type` | Reward referrer | Reward referee |
|---|---|---|---|
| Program A — NTB Q3 2026 | NTB | Rp25.000 | Rp10.000 |
| Program B — ETB CIF Q3 2026 | ETB_CIF | Rp15.000 | Rp5.000 |
| Program C — ETB BerBRImo Q3 2026 | ETB_BERBRIMO | Rp20.000 | Rp8.000 |
| Program D — ETB X NTB Q3 2026 | ETB_X_NTB | Rp18.000 | Rp7.000 |

> Setiap tipe user = program terpisah. Program D khusus untuk teman **dormant yang daftar ulang** — bukan digabung ke Program A (NTB).

**Implikasi UI:** setiap program aktif = **satu card**. Jumlah card = jumlah program aktif (0–4). Card menampilkan kuota, reward, dan periode selesai dari API; copy lainnya **hardcode** per `user_type`.

### Tipe User Referee (konteks internal — **tidak boleh muncul sebagai istilah di UI**)

Sistem mengenali **tepat 4 tipe user referee**. Satu program hanya boleh menargetkan **satu** tipe di bawah ini:

| `user_type` | Definisi | Deteksi saat onboarding |
|---|---|---|
| `NTB` | Belum punya rekening BRI sama sekali | NIK tidak punya CIF |
| `ETB_CIF` | Sudah punya rekening BRI, belum punya BRImo | NIK punya CIF, belum BRImo |
| `ETB_BERBRIMO` | Sudah punya rekening BRI dan BRImo | NIK punya CIF + BRImo |
| `ETB_X_NTB` | Pernah punya rekening BRI (dormant), daftar ulang sebagai user baru | NIK punya riwayat CIF dormant + flow daftar ulang NTB |

### Matriks Tampilan Referral Hub

UI menyesuaikan jumlah card dari `active_programs[]` (**0–4 card program**). Hero selalu sama.

| Program aktif | Jumlah card | Contoh |
|---|---|---|
| Tidak ada | 1 card info | State `no_reward` |
| 1 tipe | 1 card program | Hanya NTB, atau hanya ETB X NTB, dll. |
| 2–3 tipe | 2–3 card program | NTB + ETB CIF, dll. |
| 4 tipe | 4 card program | NTB + ETB CIF + ETB BerBRImo + ETB X NTB |

**Urutan render card (fixed):** NTB → ETB_CIF → ETB_BERBRIMO → ETB_X_NTB.

`ui_mode`: `no_reward` (0 program) atau `reward_active` (1–4 program). Client render card dari `active_programs[]` — tidak perlu app release saat kombinasi program berubah.

Semua referrer melihat UI yang **sama** untuk program yang sama. Tidak ada pengecekan tipe user referrer.

## 2. Masalah yang Diselesaikan

1. **Referrer tidak tahu siapa yang bisa diajak untuk mendapatkan reward.** Di sisi referrer, perlu ada informasi yang jelas tentang **tipe teman (referee) mana saja yang valid** agar referrer dan temannya bisa mendapat reward — tanpa mengharuskan referrer mendiagnosis status perbankan temannya sendiri.
2. Jika kriteria teman yang valid tidak terkomunikasikan jelas, terjadi **janji reward yang gagal** → komplain CS, rusaknya trust.
3. Program berganti-ganti sepanjang waktu; UI statis akan menampilkan janji basi.
4. User perlu paham perbedaan **"kode masih bisa dipakai"** vs **"ada reward"** — tanpa merasa fitur mati atau ditipu.
5. Referrer hanya melihat **card program yang sedang aktif** — program yang tidak berjalan tidak ditampilkan sebagai card.

## 3. Goals

1. Satu Referral Hub adaptif: satu template, konten dinamis dari config Procash.
2. Hero **tetap general** di semua state — mengajak pakai Qita tanpa menyebut reward.
3. **Hanya card program yang berubah** sesuai program aktif: 0 card program (ganti card info), 1–4 card.
4. Referrer membaca reward, periode selesai, dan kuota **dari API**; kriteria dan copy card lainnya **hardcode** per `user_type`.
5. **Satu program = satu `user_type`.** Tidak boleh 2 program aktif dengan `user_type` sama.

## 4. Prinsip Desain (wajib dipegang designer)

1. **Kode referral selalu hidup.** Tidak pernah disembunyikan, dinonaktifkan, atau diganti meski tidak ada program reward.
2. **Hero general di semua state.** Headline & subheadline mengajak pakai Qita — **tanpa menyebut reward, periode, atau segmen program**. Tidak berubah antar `ui_mode`.
3. **Hanya tampilkan card untuk program yang aktif.** Tiap `user_type` punya card sendiri — jika program tipe tersebut tidak aktif, card-nya **tidak muncul**.
4. **Saat tidak ada program aktif:** tampilkan **satu card informasi** yang menjelaskan belum ada reward dan kode tetap bisa dibagikan.
5. **Satu kode, satu tombol share.** Jangan pernah meminta user memilih segmen sebelum share.
6. **Bahasa manusia, bukan istilah internal.** Gunakan label hardcode per `user_type` — bukan NTB/ETB/CIF di UI.
7. **Reward referrer & referee hanya di card program aktif** — di-map dari API. Hero, subheadline, dan share copy **tidak boleh** menyebut nominal reward.
8. **Periode selesai & kuota hanya di card program aktif** — di-map dari API. Judul, kriteria, label **hardcode** per `user_type`.
9. **Satu program = satu `user_type`.** Tidak boleh 2 program aktif dengan `user_type` yang sama. Maks 4 program aktif (NTB, ETB CIF, ETB BerBRImo, ETB X NTB).
10. **Tanpa reward di halaman referral = tanpa janji nominal** di hero & share copy. Onboarding referee tetap menampilkan janji reward setelah tipe terdeteksi.
11. **Tidak ada layout shift antar state.** Template hub tetap; yang berubah hanya jumlah card (0–4).
12. **Jangan pernah menampilkan nominal dari cache lama.** Fallback ke `no_reward` jika config gagal.

## 5. Arsitektur Konten (Server-Driven)

### 5.1 Sumber Kebenaran

Procash meng-expose config program aktif. Client me-render UI berdasarkan `ui_mode` yang **dihitung di backend** — client tidak mengevaluasi eligibility sendiri.

Konten dinamis di-map dari `active_programs[]` — **hanya program yang `is_active: true`**. Setiap item = **satu program = satu `user_type`**. Di dalam card, client **hanya bind field dinamis** (kuota, reward, periode selesai); copy lainnya **hardcode** berdasarkan `user_type`.

| Konten UI | Sumber |
|---|---|
| Card program — kuota, reward, periode selesai | API `active_programs[]` |
| Card program — judul, kriteria, label tipe user, badge | **Hardcode client** per `user_type` |
| Card info tidak ada program | **Hardcode client** |
| Hero headline & subheadline | **Hardcode client** (sama semua `ui_mode`) |

### 5.2 API Contract — `GET /referral/hub`

```json
{
  "referral_code": "QITA-ABC123",
  "ui_mode": "reward_active",

  "hero": {
    "headline": "Ajak temanmu pakai Qita",
    "subheadline": "Bagikan kode referralmu dan ajak temanmu bergabung"
  },

  "active_programs": [
    {
      "program_id": "NTB_2026_Q3",
      "user_type": "NTB",
      "period_end": {
        "end_date": "2026-08-31",
        "display": "Berlaku s.d. 31 Agustus 2026"
      },
      "referrer_reward": {
        "amount": 25000,
        "display": "Rp25.000"
      },
      "referee_reward": {
        "amount": 10000,
        "display": "Rp10.000"
      },
      "quota": {
        "max_per_referrer": 200,
        "used_by_referrer": 12
      }
    },
    {
      "program_id": "ETB_CIF_2026_Q3",
      "user_type": "ETB_CIF",
      "period_end": {
        "end_date": "2026-08-31",
        "display": "Berlaku s.d. 31 Agustus 2026"
      },
      "referrer_reward": {
        "amount": 15000,
        "display": "Rp15.000"
      },
      "referee_reward": {
        "amount": 5000,
        "display": "Rp5.000"
      },
      "quota": {
        "max_per_referrer": 200,
        "used_by_referrer": 12
      }
    },
    {
      "program_id": "ETB_BERBRIMO_2026_Q3",
      "user_type": "ETB_BERBRIMO",
      "period_end": {
        "end_date": "2026-08-31",
        "display": "Berlaku s.d. 31 Agustus 2026"
      },
      "referrer_reward": {
        "amount": 20000,
        "display": "Rp20.000"
      },
      "referee_reward": {
        "amount": 8000,
        "display": "Rp8.000"
      },
      "quota": {
        "max_per_referrer": 200,
        "used_by_referrer": 5
      }
    },
    {
      "program_id": "ETB_X_NTB_2026_Q3",
      "user_type": "ETB_X_NTB",
      "period_end": {
        "end_date": "2026-08-31",
        "display": "Berlaku s.d. 31 Agustus 2026"
      },
      "referrer_reward": {
        "amount": 18000,
        "display": "Rp18.000"
      },
      "referee_reward": {
        "amount": 7000,
        "display": "Rp7.000"
      },
      "quota": {
        "max_per_referrer": 100,
        "used_by_referrer": 3
      }
    }
  ],

  "general_tnc": {
    "version": "1.0",
    "sections": [
      { "title": "Definisi", "content": "..." },
      { "title": "Penggunaan Kode Referral", "content": "..." },
      { "title": "Program Reward", "content": "..." },
      { "title": "Kelayakan", "content": "..." },
      { "title": "Pencegahan Penyalahgunaan", "content": "..." },
      { "title": "Pajak dan Pemrosesan", "content": "..." },
      { "title": "Perubahan Ketentuan", "content": "..." }
    ]
  },

  "share_copy_default": {
    "no_reward": "Coba Qita, aplikasi banking BRI yang praktis. Pakai kode {referral_code} saat daftar."
  }
}
```

**Contoh response saat tidak ada program aktif** (`ui_mode: "no_reward"`):

```json
{
  "referral_code": "QITA-ABC123",
  "ui_mode": "no_reward",
  "hero": {
    "headline": "Ajak temanmu pakai Qita",
    "subheadline": "Bagikan kode referralmu dan ajak temanmu bergabung"
  },
  "active_programs": [],
  "general_tnc": {
    "version": "1.0",
    "sections": [ "..." ]
  },
  "share_copy_default": {
    "no_reward": "Coba Qita, aplikasi banking BRI yang praktis. Pakai kode {referral_code} saat daftar."
  }
}
```

**Contoh response saat hanya NTB aktif** — `active_programs` berisi **1 item** (NTB saja), tanpa objek ETB.

**Catatan field wajib:**
- `active_programs[]` hanya berisi program dengan `is_active: true` di Procash (**0–4 item**).
- **Setiap `user_type` unik** dalam array — tidak boleh duplikat (Procash menolak 2 program aktif dengan `user_type` sama).
- **Satu program = satu `user_type`.** Nilai yang valid: `NTB`, `ETB_CIF`, `ETB_BERBRIMO`, `ETB_X_NTB`.
- Per item program, **field dinamis untuk card** hanya: `period_end`, `referrer_reward`, `referee_reward`, `quota`.
- `user_type` dipakai client untuk **lookup copy hardcode** dan urutan render card.
- `ui_mode`: `no_reward` (array kosong) atau `reward_active` (1–4 program).
- Client render **1 card per item** di `active_programs[]`, urut: NTB → ETB_CIF → ETB_BERBRIMO → ETB_X_NTB.

### 5.3 Logic `ui_mode` (dihitung backend)

```javascript
function resolveUiMode(activePrograms) {
  if (!activePrograms || activePrograms.length === 0) return "no_reward";
  return "reward_active";
}

// Validasi backend sebelum kirim response:
function validateActivePrograms(programs) {
  const types = programs.map(p => p.user_type);
  if (new Set(types).size !== types.length) {
    throw new Error("Duplicate user_type in active programs");
  }
  if (types.some(t => !["NTB", "ETB_CIF", "ETB_BERBRIMO", "ETB_X_NTB"].includes(t))) {
    throw new Error("Invalid user_type");
  }
}
```

**Catatan:** Program tidak aktif **tidak dikirim** ke client. `active_programs[]` paling banyak **4 item** — satu per `user_type`. Procash **tidak boleh** mengaktifkan 2 program dengan `user_type` yang sama.

### 5.4 Mapping UI Component ↔ API Field

#### Hero (Headline & Subheadline)

**General — sama di semua `ui_mode`. Tidak menyebut reward, periode, atau segmen program.**

| Elemen | Copy | Sumber API |
|---|---|---|
| Headline | "Ajak temanmu pakai Qita" | `hero.headline` |
| Subheadline | "Bagikan kode referralmu dan ajak temanmu bergabung" | `hero.subheadline` |

#### Section Program Reward (Card)

**Hanya render card untuk program aktif.** Jumlah card = `active_programs.length` (0–4).

| Kondisi | Yang ditampilkan |
|---|---|
| `active_programs.length === 0` | **1 card informasi** (hardcode client) |
| `active_programs.length === 1–4` | **1 card per program**, urut NTB → ETB_CIF → ETB_BERBRIMO → ETB_X_NTB |

#### Card Program Aktif

**Field dinamis (dari API):**

| Field API | Tampilan di card |
|---|---|
| `period_end.display` | ⏱ Berlaku s.d. 31 Agustus 2026 |
| `referrer_reward.display` | Kamu Rp25.000 |
| `referee_reward.display` | Temanmu Rp10.000 |
| `quota.used_by_referrer` / `quota.max_per_referrer` | Kuota: 12/200 ajakan |

**Hardcode di client** (lookup `user_type`):

| `user_type` | Judul card | Label tipe user | Kriteria |
|---|---|---|---|
| `NTB` | Ajak Teman Baru di BRI | Teman yang belum punya rekening BRI | Belum punya rekening BRI; Belum pernah pakai BRImo/Qita |
| `ETB_CIF` | Ajak Pengguna BRI | Teman punya rekening BRI (belum BRImo) | Sudah punya rekening BRI; Belum pernah pakai Qita |
| `ETB_BERBRIMO` | Ajak Pengguna BRImo | Teman pengguna BRImo | Sudah punya rekening BRI/BRImo; Belum pernah pakai Qita |
| `ETB_X_NTB` | Ajak Teman Kembali | Teman yang pernah punya rekening BRI dan daftar ulang | Pernah punya rekening BRI (dormant); Daftar ulang melalui Qita |

```
┌─────────────────────────────────────────────┐
│  Ajak Teman Baru di BRI            [Aktif]  │  ← hardcode (user_type: NTB)
│  ⏱ Berlaku s.d. 31 Agustus 2026             │  ← period_end.display
├─────────────────────────────────────────────┤
│  Teman yang belum punya rekening BRI        │  ← hardcode
│  Kamu: Rp25.000  ·  Temanmu: Rp10.000      │  ← referrer_reward / referee_reward
│  Kuota: 12/200 ajakan                       │  ← quota
├─────────────────────────────────────────────┤
│  ✓ Belum punya rekening BRI sama sekali     │  ← hardcode
│  ✓ Belum pernah pakai BRImo atau Qita       │  ← hardcode
└─────────────────────────────────────────────┘
```

**Satu card = satu program = satu pasang reward.** Tidak ada divider multi-tipe dalam satu card.

#### Card Informasi (Tidak Ada Program Aktif)

Digunakan **hanya** saat `active_programs[]` kosong. Bukan card NTB/ETB. **Seluruh copy hardcode di client.**

```
┌─────────────────────────────────────────────┐
│  Program Reward                             │  ← hardcode
├─────────────────────────────────────────────┤
│  Belum ada program reward saat ini.         │  ← hardcode
│  Kamu tetap bisa mengajak teman ke Qita —   │  ← hardcode
│  kode referral kamu tetap berlaku.          │  ← hardcode
└─────────────────────────────────────────────┘
```

| `ui_mode` | Jumlah card di section |
|---|---|
| `reward_active` | 1–4 card program (sesuai `active_programs.length`) |
| `no_reward` | 1 card informasi (hardcode) |

#### Cara Kerja — Langkah 3

**Hardcode di client** — tidak di-map dari API. Satu bullet per `user_type` yang ada di `active_programs[]`.

| `user_type` aktif | Langkah 3 (hardcode) |
|---|---|
| `NTB` | "Teman buka rekening & lakukan setoran/transaksi pertama" |
| `ETB_CIF` | "Teman aktivasi Qita dengan rekening BRI yang sudah ada & transaksi pertama" |
| `ETB_BERBRIMO` | "Teman aktivasi Qita dengan akun BRImo yang sudah ada & transaksi pertama" |
| `ETB_X_NTB` | "Teman yang daftar ulang selesaikan aktivasi & transaksi pertama" |
| `no_reward` | Langkah 3 **tidak di-render** (hanya 2 langkah) |

#### Share Copy

| Kondisi | Sumber |
|---|---|
| Program NTB aktif saja | `share_copy_default` atau template netral tanpa nominal |
| Program ETB aktif saja | Idem — tanpa nominal di share copy |
| Dual program aktif | Template netral: "Gabung Qita pakai kode {referral_code}" |
| Tidak ada program aktif | `share_copy_default.no_reward` |

Placeholder di template: `{referral_code}`, `{referee_reward}`, `{referrer_reward}` — di-replace client saat render.

#### S&K

| Lapisan | Sumber | Kapan tampil |
|---|---|---|
| Ketentuan umum | `general_tnc.sections[]` | Selalu |
| Detail program | **Hardcode client** per `user_type` + nominal dari `active_programs[]` | Saat program aktif |

### 5.5 API Contract — Onboarding Referee

Endpoint terpisah: dipanggil setelah referee input NIK (deteksi tipe referee).

```json
{
  "referral_code": "QITA-ABC123",
  "referrer_name": "Ahmad",
  "user_type": "NTB",
  "matched_program": {
    "program_id": "NTB_2026_Q3",
    "user_type": "NTB",
    "referrer_reward": {
      "amount": 25000,
      "display": "Rp25.000"
    },
    "referee_reward": {
      "amount": 10000,
      "display": "Rp10.000"
    },
    "period_end": {
      "display": "Berlaku s.d. 31 Agustus 2026"
    }
  }
}
```

Matching: backend cocokkan `referee.user_type` dengan program aktif yang memiliki **`user_type` sama** (satu program = satu tipe).

Jika tidak ada program aktif dengan `user_type` yang cocok: `matched_program: null` → tidak ada janji reward di onboarding.

**Implikasi untuk designer:** semua komponen teks harus didesain dengan asumsi konten variabel. Siapkan spec untuk truncation dan dynamic type.

---

## 6. Spesifikasi UI Lengkap

Bagian ini adalah **acuan utama untuk UI/UX Designer** — merangkum seluruh keputusan desain terbaru dalam bentuk wireframe, komponen, dan aturan tampilan.

### 6.1 Aturan Penempatan Konten

| Informasi | Hero | Card Program Aktif | Card Info (no program) | Share Copy |
|---|---|---|---|---|
| Nominal reward | ❌ | ✅ (API) | ❌ | ❌ |
| Periode selesai | ❌ | ✅ (API) | ❌ | ❌ |
| Kuota ajakan | ❌ | ✅ (API) | ❌ | ❌ |
| Kriteria teman | ❌ | ✅ (hardcode) | ❌ | ❌ |
| Judul & label tipe user | ❌ | ✅ (hardcode) | ❌ | ❌ |
| Belum ada reward / tetap bisa ajak | ❌ | — | ✅ (hardcode) | ❌ |
| Kode referral | ✅ | — | — | ✅ |

**Yang berubah antar state hanya section card** — hero tetap sama.

### 6.2 Visual Hierarchy (urutan scan user)

```
1. Apa yang harus saya lakukan?     → Kode referral + tombol Bagikan
2. Program apa yang sedang berjalan? → Card program (atau card info)
3. Bagaimana caranya?               → Cara kerja
4. Apa status ajakan saya?          → Tracker
```

Hero **tidak berubah** — tidak menjadi sumber informasi program/reward.

### 6.3 Kerangka Halaman (Template Tetap — Semua State)

```
┌─────────────────────────────────────────┐
│  ←  Ajak Teman                          │  App bar
├─────────────────────────────────────────┤
│                                         │
│  [Ilustrasi referral]                   │
│                                         │
│  HEADLINE: Ajak temanmu pakai Qita       │  ← general, sama semua state
│  Subheadline: Bagikan kode referralmu... │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │  QITA-ABC123          [Salin]   │    │
│  └─────────────────────────────────┘    │
│  ┌─────────────────────────────────┐    │
│  │   [ Bagikan Sekarang /          │    │
│  │     Bagikan ke Teman ]          │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Program Reward                         │  Section header
│  [1–4 card program / 1 card info]       │  ← jumlah card = active_programs.length
│                                         │
│  Cara kerjanya                          │
│  [Step 1] [Step 2] [Step 3 opsional]    │
│                                         │
│  Status ajakanmu                        │
│  [Tracker list]                         │
│                                         │
│  Pelajari Syarat & Ketentuan →          │
│                                         │
└─────────────────────────────────────────┘
```

**Above the fold** (tanpa scroll): ilustrasi → headline → kode → tombol share.

### 6.4 Komponen: Hero

**General — tidak berubah antar state.**

| Elemen | Copy |
|---|---|
| Headline | "Ajak temanmu pakai Qita" |
| Subheadline | "Bagikan kode referralmu dan ajak temanmu bergabung" |

**Larangan hero:** nominal reward, masa berlaku, segmen program (NTB/ETB), kata "reward".

### 6.5 Komponen: Kode Referral + CTA

```
┌──────────────────────────────────┐
│  QITA-ABC123            [Salin]  │   monospace / letter-spacing
└──────────────────────────────────┘
```

| Kondisi | Label tombol |
|---|---|
| Ada program reward aktif (`reward_active`) | **Bagikan Sekarang** |
| Tidak ada program (`no_reward`) | **Bagikan ke Teman** |

Kode **selalu tampil** di semua state. Tap [Salin] → toast "Kode berhasil disalin". Tap CTA → native share sheet dengan `share_copy_default` (tanpa nominal).

### 6.6 Komponen: Card Program Aktif

**Prinsip:** satu program = satu `user_type` = **satu card** dengan **satu pasang reward**. Client lookup copy hardcode dari `user_type`, lalu bind 4 field dinamis dari API.

**Anatomi card:**

```
┌─────────────────────────────────────────────┐
│  [Judul card — hardcode]           [Aktif]  │
│  ⏱ {period_end.display}                     │  ← API
├─────────────────────────────────────────────┤
│  [Label tipe user — hardcode]               │
│  Kamu {referrer_reward.display}             │  ← API
│       · Temanmu {referee_reward.display}    │  ← API
│  Kuota: {used}/{max} ajakan                 │  ← API
├─────────────────────────────────────────────┤
│  ✓ [Kriteria 1 — hardcode]                  │
│  ✓ [Kriteria 2 — hardcode]                  │
└─────────────────────────────────────────────┘
```

**Mapping field:**

| Elemen card | Sumber |
|---|---|
| Judul, badge "Aktif", label tipe user, checklist kriteria | **Hardcode** per `user_type` |
| `period_end.display` | **API** |
| `referrer_reward.display` | **API** |
| `referee_reward.display` | **API** |
| `quota.used_by_referrer` / `quota.max_per_referrer` | **API** |

**Spesifikasi visual:**

| Elemen | Spec |
|---|---|
| Badge "Aktif" | Pill kecil, warna brand — **hardcode** |
| Card container | Background highlight subtle / border brand, elevation 1 |
| `period_end.display` | Ikon ⏱ + teks secondary |
| Baris reward | Satu baris saja — label hardcode + nominal API |
| Baris kuota | Teks secondary: "Kuota: {used}/{max} ajakan" |
| Checklist kriteria | Ikon ✓ + teks secondary — **hardcode** |

**Copy hardcode per `user_type`:**

| `user_type` | Judul | Label | Kriteria |
|---|---|---|---|
| `NTB` | Ajak Teman Baru di BRI | Teman yang belum punya rekening BRI | Belum punya rekening BRI; Belum pernah pakai BRImo/Qita |
| `ETB_CIF` | Ajak Pengguna BRI | Teman punya rekening BRI (belum BRImo) | Sudah punya rekening BRI; Belum pernah pakai Qita |
| `ETB_BERBRIMO` | Ajak Pengguna BRImo | Teman pengguna BRImo | Sudah punya rekening BRI/BRImo; Belum pernah pakai Qita |
| `ETB_X_NTB` | Ajak Teman Kembali | Teman yang pernah punya rekening BRI dan daftar ulang | Pernah punya rekening BRI (dormant); Daftar ulang melalui Qita |

```
┌─────────────────────────────────────────────┐
│  Ajak Pengguna BRI                 [Aktif]  │  hardcode
│  ⏱ Berlaku s.d. 15 September 2026           │  API
├─────────────────────────────────────────────┤
│  Teman punya rekening BRI (belum BRImo)     │  hardcode
│  Kamu Rp15.000  ·  Temanmu Rp5.000          │  API
│  Kuota: 12/200 ajakan                       │  API
├─────────────────────────────────────────────┤
│  ✓ Sudah punya rekening BRI                 │  hardcode
│  ✓ Belum pernah pakai Qita                  │  hardcode
└─────────────────────────────────────────────┘
```

> Program ETB BerBRImo dan ETB X NTB masing-masing **program terpisah** — punya card sendiri jika aktif.

### 6.7 Komponen: Card Informasi (Tidak Ada Program)

**Satu card** — menggantikan card program saat tidak ada program aktif. **Seluruh copy hardcode di client.**

```
┌─────────────────────────────────────────────┐
│  Program Reward                             │
├─────────────────────────────────────────────┤
│  Belum ada program reward saat ini.         │
│  Kamu tetap bisa mengajak teman ke Qita —   │
│  kode referral kamu tetap berlaku.          │
└─────────────────────────────────────────────┘
```

| Elemen | Spec |
|---|---|
| Container | Background netral, bukan gray-out |
| Konten | **Hardcode** — tanpa nominal, tanpa periode, tanpa kuota |
| Posisi | Menggantikan area card program — bukan di hero |

**Alasan desain:** satu card info lebih jelas daripada 2 card NTB/ETB kosong atau menghilangkan section sepenuhnya.

### 6.8 Layout Card per State

| `active_programs` | Card yang ditampilkan |
|---|---|
| `[]` | 1× Card informasi (hardcode) |
| `[NTB]` | 1× Card NTB |
| `[ETB_CIF]` | 1× Card ETB CIF |
| `[ETB_BERBRIMO]` | 1× Card ETB BerBRImo |
| `[ETB_X_NTB]` | 1× Card ETB X NTB |
| `[NTB, ETB_CIF]` | 2× Card (NTB + ETB CIF) |
| `[NTB, ETB_CIF, ETB_BERBRIMO]` | 3× Card |
| `[NTB, ETB_CIF, ETB_BERBRIMO, ETB_X_NTB]` | 4× Card (semua tipe) |

Program tidak aktif **tidak ditampilkan**. Urutan card tetap: NTB → ETB_CIF → ETB_BERBRIMO → ETB_X_NTB.

### 6.9 Wireframe — Contoh: Hanya NTB Aktif

> Hero **sama** dengan state lain. Hanya **1 card NTB** — card ETB **tidak ditampilkan**.

```
┌─────────────────────────────────────────┐
│  ←  Ajak Teman                          │
├─────────────────────────────────────────┤
│         [ilustrasi share]               │
│                                         │
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
│                                         │
│  ┌─────────────────────────────────┐    │
│  │ Ajak Teman Baru di BRI  [Aktif]│    │
│  │ ⏱ Berlaku s.d. 31 Agustus 2026 │    │
│  │                                 │    │
│  │ Teman belum punya rekening BRI  │    │
│  │ Kamu Rp25.000 · Teman Rp10.000  │    │
│  │ Kuota: 12/200 ajakan            │    │
│  │                                 │    │
│  │ ✓ Belum punya rekening BRI      │    │
│  │ ✓ Belum pernah pakai BRImo/Qita │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Cara kerjanya                          │
│  ① Bagikan kode                         │
│  ② Teman daftar pakai kodemu            │
│  ③ Teman setor/transaksi pertama        │
│                                         │
│  Status ajakanmu                        │
│  Total reward: Rp75.000 · 12/200 ajakan │
│  [list ajakan...]                       │
│                                         │
│  Pelajari Syarat & Ketentuan →          │
└─────────────────────────────────────────┘
```

### 6.10 Wireframe — Contoh: Empat Program Aktif

> Hero **sama**. **4 card** — satu per `user_type`.

```
┌─────────────────────────────────────────┐
│  ←  Ajak Teman                          │
├─────────────────────────────────────────┤
│  Ajak temanmu pakai Qita                │
│  [Kode + Bagikan Sekarang]              │
│                                         │
│  Program Reward                         │
│  ┌─ Card NTB ─────────────────────┐    │
│  │ Ajak Teman Baru di BRI [Aktif] │    │
│  │ Kamu Rp25.000 · Teman Rp10.000  │    │
│  │ Kuota: 12/200                    │    │
│  └─────────────────────────────────┘    │
│  ┌─ Card ETB CIF ────────────────┐    │
│  │ Ajak Pengguna BRI      [Aktif] │    │
│  │ Kamu Rp15.000 · Teman Rp5.000   │    │
│  │ Kuota: 12/200                    │    │
│  └─────────────────────────────────┘    │
│  ┌─ Card ETB BerBRImo ───────────┐    │
│  │ Ajak Pengguna BRImo    [Aktif] │    │
│  │ Kamu Rp20.000 · Teman Rp8.000   │    │
│  └─────────────────────────────────┘    │
│  ┌─ Card ETB X NTB ─────────────┐    │
│  │ Ajak Teman Kembali     [Aktif] │    │
│  │ Kamu Rp18.000 · Teman Rp7.000   │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Cara kerjanya — 4 bullet (satu/tipe)   │
└─────────────────────────────────────────┘
```

### 6.11 Wireframe — Contoh: Hanya ETB CIF Aktif

> Hero **sama**. **1 card** — hanya program `ETB_CIF` yang aktif.

```
┌─────────────────────────────────────────┐
│  Program Reward                         │
│  ┌─────────────────────────────────┐    │
│  │ Ajak Pengguna BRI       [Aktif] │    │
│  │ ⏱ Berlaku s.d. 31 Agustus 2026 │    │
│  │ Teman punya rekening BRI        │    │
│  │ (belum BRImo)                   │    │
│  │ Kamu Rp15.000 · Teman Rp5.000   │    │
│  │ Kuota: 12/200 ajakan            │    │
│  │ ✓ Sudah punya rekening BRI      │    │
│  │ ✓ Belum pernah pakai Qita       │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

### 6.12 Wireframe — Tidak Ada Program Aktif

> Hero **sama** dengan state lain. **1 card informasi** — bukan card NTB/ETB.

```
┌─────────────────────────────────────────┐
│  ←  Ajak Teman                          │
├─────────────────────────────────────────┤
│         [ilustrasi share]               │
│                                         │
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
│  ┌─────────────────────────────────┐    │
│  │ Belum ada program reward saat   │    │
│  │ ini.                            │    │
│  │                                 │    │
│  │ Kamu tetap bisa mengajak teman  │    │
│  │ ke Qita — kode referral kamu    │    │
│  │ tetap berlaku.                  │    │
│  └─────────────────────────────────┘    │
│                                         │
│  🔔 Beri tahu saya saat ada program     │
│     reward                              │
│                                         │
│  Cara kerjanya (2 langkah saja)         │
│  ① Bagikan kode  ② Teman daftar         │
│                                         │
│  Status ajakanmu (riwayat lama tetap)   │
│  Pelajari Syarat & Ketentuan →          │
└─────────────────────────────────────────┘
```

### 6.13 Komponen: Cara Kerja

**3 langkah** (ada program aktif) / **2 langkah** (tidak ada program).

```
  ①  Bagikan kode referral kamu
  ②  Teman gabung Qita pakai kodemu
  ③  Teman selesaikan syarat program     ← dinamis, hanya jika reward aktif
```

Visual: numbered circle + teks. Opsional ilustrasi kecil per langkah.

### 6.14 Komponen: Tracker

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

### 6.15 Komponen: Bottom Sheet & Interupsi

| Trigger | Konten |
|---|---|
| Program reward baru | "Program reward baru! Cek detail reward di halaman Ajak Teman." [Mengerti, Bagikan Sekarang] |
| Program berakhir | "Program reward sudah berakhir. Kode kamu tetap bisa dibagikan." [Mengerti] |
| Reward baru cair | Snackbar di atas hero: "🎉 Rp25.000 sudah masuk dari ajakan ke Budi!" |
| First-time visit | Coachmark 3 langkah cara kerja (dismissible, sekali seumur akun) |

### 6.16 Entry Points (di luar Referral Hub)

| Entry point | Copy (tanpa nominal) | Aksi |
|---|---|---|
| Banner homepage | "Ajak temanmu ke Qita — ada program reward!" | Deep link ke Referral Hub |
| Post-transaksi | "Kenalkan Qita ke temanmu & cek reward-nya" | Deep link |
| Menu profil | "Ajak Teman" | Navigasi ke Referral Hub |
| Push notifikasi | "Program reward baru! Cek sekarang." | Deep link |

### 6.17 Onboarding Referee (layar terpisah)

Setelah NIK → deteksi tipe referee:

**Match program aktif:**
```
┌─────────────────────────────────────────┐
│  Kamu diajak Ahmad!                     │
│  Selesaikan pendaftaran & transaksi     │
│  pertama untuk dapat Rp10.000           │  ← nominal OK di sini
│  [Tracker syarat + deadline]            │
└─────────────────────────────────────────┘
```

**Tidak match:** onboarding normal, **tanpa janji nominal**.

### 6.18 Loading & Error State

| State | Perlakuan |
|---|---|
| Loading | Skeleton: hero + kode + 1–4 card slot (sesuai `active_programs.length` terakhir); tanpa layout shift |
| Config gagal | Fallback `no_reward` — **tidak pernah** tampilkan nominal dari cache |

---

## 7. Requirement per Skenario

## 7. Requirement per Skenario

### Skenario — Satu program aktif (`reward_active`, 1 card)

Contoh: hanya Program A (NTB) aktif.

| Elemen | Konten | Sumber |
|---|---|---|
| Headline / Subheadline | General | Hardcode |
| Card | 1 card sesuai `user_type` program aktif | Hardcode + API |
| Card lain | **Tidak ditampilkan** | — |
| Cara kerja langkah 3 | 1 bullet hardcode sesuai `user_type` | Hardcode |

### Skenario — Beberapa program aktif (`reward_active`, 2–4 card)

Contoh: keempat program (NTB, ETB_CIF, ETB_BERBRIMO, ETB_X_NTB) aktif.

| Elemen | Konten | Sumber |
|---|---|---|
| Headline / Subheadline | General | Hardcode |
| Cards | 1 card per program, urut NTB → ETB_CIF → ETB_BERBRIMO → ETB_X_NTB | Hardcode + API |
| Cara kerja langkah 3 | 1 bullet per `user_type` aktif | Hardcode |
| Larangan | Tidak ada pemilihan segmen sebelum share | — |

### Skenario — Tidak ada program aktif (`no_reward`)

| Elemen | Konten | Sumber |
|---|---|---|
| Headline / Subheadline | General — sama semua state | Hardcode |
| Card program | **Tidak ditampilkan** | — |
| Card info | "Belum ada program reward saat ini..." | Hardcode |
| Kode & share | [Bagikan ke Teman] | `referral_code` |
| Share copy | Tanpa nominal | `share_copy_default.no_reward` |
| Opt-in notifikasi | "Beri tahu saya saat ada program reward" | Hardcode |
| Cara kerja | 2 langkah saja | Hardcode |
| Tracker ajakan baru | "Teman terdaftar — tidak ada program reward aktif saat ini" | — |
| Larangan | Jangan tampilkan empty state "fitur mati". Jangan sembunyikan menu referral. | — |

## 8. Cara Kerja — Template General

Cara kerja menggunakan **template tetap** dengan slot dinamis di langkah 3. Tidak dibuat terpisah per skenario.

### Saat ada program reward aktif (3 langkah)

```
① Bagikan kode referral kamu
   Kirim kode {referral_code} ke teman lewat WhatsApp,
   media sosial, atau cara lainnya.

② Teman gabung Qita pakai kodemu
   Teman download Qita dan masukkan kode referral
   kamu saat pendaftaran atau aktivasi.

③ Teman selesaikan syarat program          ← HARDCODE per user_type
   NTB: "Buka rekening & lakukan setoran/transaksi pertama"
   ETB_CIF: "Aktivasi Qita dengan rekening BRI & transaksi pertama"
   ETB_BERBRIMO: "Aktivasi Qita dengan akun BRImo & transaksi pertama"
   ETB_X_NTB: "Teman yang daftar ulang selesaikan aktivasi & transaksi pertama"
   (satu bullet per user_type di active_programs[])
   Reward cair maks. 2×24 jam.
```

- **1 program aktif:** satu bullet di langkah 3.
- **2–4 program aktif:** satu bullet per `user_type`, urut NTB → ETB_CIF → ETB_BERBRIMO → ETB_X_NTB.

### Saat tidak ada program aktif (2 langkah)

```
① Bagikan kode referral kamu
   Kirim kode {referral_code} ke teman lewat WhatsApp,
   media sosial, atau cara lainnya.

② Teman gabung Qita pakai kodemu
   Teman download Qita dan masukkan kode referral
   kamu saat pendaftaran atau aktivasi.
```

## 9. S&K — Struktur Dua Lapisan

### Lapisan 1: Ketentuan Umum (selalu tampil)

Sumber: `general_tnc.sections[]` — satu dokumen statis, berlaku untuk semua program dan semua state.

Isi minimal:
1. **Definisi** — pengundang, teman yang diajak, kode referral
2. **Penggunaan Kode Referral** — kode permanen, bisa dibagikan kapan saja terlepas dari ketersediaan program reward
3. **Program Reward** — reward hanya jika program aktif saat teman memenuhi syarat; tidak ada jaminan program di masa depan
4. **Kelayakan** — ditentukan berdasarkan tipe teman yang diajak dan status perbankan
5. **Pencegahan Penyalahgunaan** — satu orang satu kali, anti-fraud, keputusan final
6. **Pajak dan Pemrosesan** — mekanisme pencairan reward
7. **Perubahan Ketentuan** — hak Qita mengubah dengan pemberitahuan

### Lapisan 2: Detail Program Aktif (hanya saat `ui_mode` ≠ `no_reward`)

Sumber: **Hardcode client** per `user_type` + nominal dari `active_programs[]` — satu section per program aktif.

```
┌─────────────────────────────────────┐
│  Syarat & Ketentuan                 │
├─────────────────────────────────────┤
│  [Ketentuan Umum — selalu ada]      │  ← general_tnc
│  1. Definisi                        │
│  2. Penggunaan Kode Referral        │
│  3. Program Reward                  │
│  ...                                │
│                                     │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─   │
│                                     │
│  [Detail Program Saat Ini]          │  ← hardcode + nominal dari API
│  (satu section per user_type aktif) │     hanya jika reward aktif
│                                     │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─   │
│                                     │
│  [Riwayat Program Sebelumnya ▾]     │  ← accordion, opsional
│                                     │
└─────────────────────────────────────┘
```

Saat `no_reward`: hanya Lapisan 1 yang ditampilkan.

## 10. State & Perilaku Saat User Masuk Page

### 10.1 First-time visit (sekali seumur akun)

- **Mode reward aktif:** coachmark ringan / bottom sheet "Cara Kerja" (maks. 3 langkah, dismissible).
- **Mode tanpa reward:** tidak perlu coachmark reward; cukup penjelasan singkat di card info.
- Setelah dismiss tidak muncul lagi.

### 10.2 Transisi state sejak kunjungan terakhir

Bandingkan `ui_mode` terakhir (local) vs `ui_mode` saat ini:

| Perubahan | Perlakuan |
|---|---|
| Reward aktif → tanpa reward | Bottom sheet: "Program reward sudah berakhir. Kode kamu tetap bisa dibagikan." CTA: [Mengerti] |
| Tanpa reward → reward aktif | Bottom sheet: "Program reward baru! Cek detail reward di halaman Ajak Teman." CTA: [Mengerti, Bagikan Sekarang] |
| Ganti segmen (NTB ⇄ ETB) | Bottom sheet: "Program referral baru! Sekarang giliran ajak temanmu yang sudah punya rekening BRI/BRImo." |
| Hanya periode berubah | Badge "Baru" di card terkait, tanpa interupsi |

**Larangan:** tidak ada pop-up/modal promo saat page dibuka di luar kasus di atas.

### 10.3 Reward baru cair sejak kunjungan terakhir

Snackbar di atas hero: "🎉 Rp25.000 sudah masuk ke saldomu dari ajakan ke Budi!"

### 10.4 Ajakan menggantung (hanya mode reward aktif)

Nudge: "Budi tinggal 1 langkah lagi — ingatkan dia transaksi pertama sebelum 10 Juli. [Ingatkan]"

### 10.5 Loading / config gagal

- Loading: skeleton mengikuti template hub.
- Gagal total: fallback ke `no_reward`. **Tidak pernah** menampilkan nominal dari cache.

## 11. Tracker "Status Ajakanmu"

Endpoint terpisah: `GET /referral/invitations`

- **Agregat:** "Total reward kamu: Rp75.000 dari 3 teman" (jika pernah dapat reward).
- **Progres kuota** (mode reward aktif): "Kamu sudah mengajak 12/200" — dari `active_programs[].quota` (sama field dengan card).

| Status | Copy contoh | `ui_mode` | Aksi |
|---|---|---|---|
| Terdaftar | "Budi sudah gabung, tinggal transaksi pertama" | reward_active | [Ingatkan] |
| Memenuhi syarat | "Reward sedang diproses" | reward_active | — |
| Reward cair | "Rp25.000 · 2 Jul 2026" + label program | reward_active | — |
| Tidak memenuhi syarat | "Budi sudah gabung, tapi tidak memenuhi kriteria program" | reward_active | Link ke S&K |
| Terdaftar, tanpa reward | "Budi sudah gabung — tidak ada program reward aktif saat ini" | no_reward | — |

Riwayat lintas program dipertahankan selamanya (dengan label program).

## 12. Sisi Referee (Teman yang Diundang)

1. Deep link membawa kode referral ke onboarding. Kode **selalu diterima**.
2. Setelah referee input NIK, backend deteksi `user_type` (NTB / ETB_CIF / ETB_BERBRIMO / ETB_X_NTB).
3. **Janji reward hanya ditampilkan jika:**
   - Ada program aktif dengan **`user_type` yang sama persis** dengan tipe referee terdeteksi
   - Response `matched_program` tidak null
4. Jika tidak match: onboarding normal, **tanpa janji nominal**, tanpa pesan penolakan frontal. Attribution tetap dicatat.
5. Terms di-lock saat registrasi: syarat yang berlaku adalah syarat saat referee mendaftar.

### Flow Deteksi Referee

```
Referee input NIK → Backend deteksi status perbankan
       │
       ├── Tidak ada CIF              → user_type = "NTB"
       ├── Ada CIF, belum BRImo         → user_type = "ETB_CIF"
       ├── Ada CIF + BRImo              → user_type = "ETB_BERBRIMO"
       └── Riwayat CIF dormant +        → user_type = "ETB_X_NTB"
           daftar ulang flow NTB
       │
       ▼
Ada program aktif dengan user_type yang sama?
       │
       ├── Ya  → tampilkan janji reward + tracker syarat
       └── Tidak → onboarding normal, matched_program: null
```

## 13. Entry Points (di luar Referral Hub)

| Entry point | Mode reward aktif | Mode tanpa reward |
|---|---|---|
| Banner homepage | "Ajak temanmu ke Qita — ada program reward!" (tanpa nominal) | **Sembunyikan** banner reward |
| Post-transaksi sukses | "Kenalkan Qita ke temanmu & cek reward-nya" (tanpa nominal) | "Kenalkan Qita ke temanmu" |
| Push notification program launch | Deep link ke Referral Hub — ke **semua user** | Tidak dikirim |
| Menu profil | "Ajak Teman" — permanen, semua state | Sama |

## 14. Copywriting Guidelines

- Dilarang menampilkan istilah: NTB, ETB, CIF, dormant, ETB X NTB, Procash.
- **Card program — dari API:** kuota, `referrer_reward`, `referee_reward`, periode selesai.
- **Card program — hardcode:** judul, badge, label tipe user, checklist kriteria — lookup `user_type`.
- **Satu program = satu `user_type`.** Tidak boleh 2 program aktif dengan `user_type` sama.
- **Nominal reward & periode selesai hanya di card program** — dilarang di hero, share copy, banner, dan entry point.
- Program tidak aktif **tidak ditampilkan** sebagai card.
- Saat tidak ada program: card informasi **hardcode** — tanpa nominal, tanpa periode, tanpa kuota.
- Share copy dari halaman referral: value prop produk, **zero mention nominal**.
- Alasan gagal di tracker: jujur, bahasa kriteria yang sama dengan checklist hardcode di card.
- CTA: "Bagikan Sekarang" (ada reward) vs "Bagikan ke Teman" (tanpa reward).

## 15. Edge Cases

| Kasus | Perlakuan |
|---|---|
| Procash coba aktifkan 2 program dengan `user_type` sama | **Ditolak** di Procash — hanya satu yang boleh aktif per tipe |
| Referee ETB BerBRImo masuk saat hanya program ETB_CIF aktif | `matched_program: null`; tracker: "tidak memenuhi kriteria program" |
| NTB + ETB CIF + ETB BerBRImo + ETB X NTB semua aktif | 4 card di Referral Hub — urut NTB → ETB_CIF → ETB_BERBRIMO → ETB_X_NTB |
| Referee NTB masuk saat hanya program ETB_CIF/ETB_BERBRIMO/ETB_X_NTB aktif | `matched_program: null`; tracker: "tidak memenuhi kriteria program" |
| Referee ETB X NTB masuk saat hanya program NTB aktif | `matched_program: null` — tipe berbeda |
| Referrer share saat reward aktif, referee daftar setelah program berakhir | Lock-in terms jika dalam grace period Procash; jika tidak, tracker: tanpa reward |
| Kuota program habis | Kode tetap aktif; info di card program terkait; share tanpa janji nominal |
| Program berakhir saat teman di tengah funnel | Honor terms saat registrasi (lock-in) |
| Config berubah saat user di page | Refresh saat re-focus |
| User pernah dapat reward, sekarang tanpa program | Riwayat tetap tampil; `ui_mode: no_reward` + card info |

## 16. Metrik Keberhasilan

| Metrik | Definisi |
|---|---|
| Share rate | Buka hub → tap Bagikan, dipisah per `ui_mode` |
| Share rate tanpa reward | Khusus `no_reward` — indikator value fitur tanpa insentif |
| Opt-in notifikasi rate | `no_reward` → tap "Beri tahu saya saat ada reward" |
| K-factor per segmen | Undangan → registrasi → kualifikasi, dipisah per `user_type` |
| Mismatch rate | Referee masuk via link tapi `user_type` tidak match program aktif |
| Time-to-reward | Kualifikasi → reward cair; target < 48 jam |
| Repeat referral rate | Referrer yang mengajak ≥ 2 teman |
| CS ticket rate | Tiket referral per 1.000 referral |
| False promise rate | Share copy janji reward tapi tidak ada program aktif saat referee daftar |

## 17. Deliverables yang Diminta dari Designer

> **Acuan utama desain: Section 6 — Spesifikasi UI Lengkap**

1. High-fidelity design Referral Hub untuk kombinasi 0–4 card program + wireframe Section 6.9–6.12.
2. Komponen card program aktif (Section 6.6) — spec field dinamis vs hardcode — dan card informasi (Section 6.7).
3. Komponen hero, kode, CTA, cara kerja, tracker, bottom sheet (Section 6.4–6.5, 6.13–6.15).
4. Flow referee onboarding (Section 6.17).
5. Entry points tanpa nominal (Section 6.16).
6. Loading skeleton & error state (Section 6.18).
7. Spec komponen dinamis: truncation, dynamic type, transisi tanpa layout shift.

## 18. Open Questions

1. Nominal & syarat kualifikasi final **per tipe user** per program (menunggu konfigurasi Procash).
2. Kriteria deteksi `ETB_X_NTB` di backend (flag dormant + flow daftar ulang) — sinkronkan dengan tim core banking.
3. Apakah reward cair otomatis atau perlu klaim manual? (Rekomendasi: otomatis.)
4. Batas kuota per referrer per periode dan per program.
5. Channel share: native share sheet vs shortcut khusus (WhatsApp-first?).
6. Grace period setelah program berakhir: berapa hari referee yang sudah terdaftar masih bisa memenuhi syarat?
