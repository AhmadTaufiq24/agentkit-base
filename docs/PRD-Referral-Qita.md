# PRD — Referral Hub Qita (Adaptif Berdasarkan Program Procash)

| | |
|---|---|
| **Dokumen** | Product Requirements Document (PRD) |
| **Fitur** | Referral Hub Qita |
| **Audiens dokumen** | UI/UX Designer, Frontend, Backend |
| **Status** | Draft v1.8 |
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
                  Referee NTB               Referee ETB              Tidak match
                  + program NTB aktif         + program ETB aktif      program aktif
                        │                         │                         │
                        ▼                         ▼                         ▼
                  Reward NTB                  Reward ETB               Tanpa reward
                  (referrer + referee)        (referrer + referee)     (attribution saja)
```

| Pihak | Dicek tipe user? | Keterangan |
|---|---|---|
| **Referrer** | **Tidak** | Siapa pun tipe user-nya (NTB, ETB, ETB BerBRImo) bisa share dan dapat reward selama temannya memenuhi syarat program aktif |
| **Referee** | **Ya** | Tipe user referee (NTB / ETB) menentukan program mana yang apply dan apakah ada reward |

### Konfigurasi Program di Procash

Saat membuat program di Procash, tim marketing **wajib mencantumkan nominal reward per tipe user**. Satu program dapat berisi **satu atau lebih tipe user** dengan reward yang berbeda-beda.

Contoh konfigurasi:

| Program | Tipe user (referee) | Reward referrer | Reward referee |
|---|---|---|---|
| NTB Q3 2026 | NTB | Rp25.000 | Rp10.000 |
| ETB Q3 2026 | ETB (CIF) | Rp15.000 | Rp5.000 |
| ETB Q3 2026 | ETB BerBRImo | Rp20.000 | Rp8.000 |

**Implikasi UI:** untuk setiap program yang **aktif**, halaman referral menampilkan reward per tipe user **hanya di card program** — bukan di hero, share copy, atau entry point.

### Segmen Program (Referee Type)

Program yang dapat dibuat di Procash menargetkan **tipe user referee**:

- **Program NTB** (`referee_type: "NTB"`) — reward jika teman yang diajak **belum punya rekening BRI sama sekali**.
- **Program ETB** (`referee_type: "ETB"`) — reward jika teman yang diajak **sudah punya rekening BRI dan/atau BRImo** dan aktivasi Qita. Dapat berisi **beberapa sub-tipe** (ETB CIF, ETB BerBRImo) dengan nominal reward berbeda per sub-tipe.

### Tipe User (konteks internal — **tidak boleh muncul sebagai istilah di UI**)

| Tipe | Definisi | Dipakai untuk |
|---|---|---|
| NTB | Belum punya rekening BRI sama sekali | Deteksi referee saat onboarding |
| ETB (CIF) | Sudah punya rekening BRI, belum punya BRImo | Deteksi referee saat onboarding |
| ETB BerBRImo | Sudah punya rekening BRI dan BRImo | Deteksi referee saat onboarding |
| ETB X NTB | User dormant yang mendaftar kembali sebagai NTB | Deteksi referee — kebijakan backend |

### Matriks State Referral Hub

UI harus mengakomodasi **4 state** berikut **tanpa app release** (server-driven dari config Procash):

| # | Program aktif | `ui_mode` | Tampilan UI |
|---|---|---|---|
| A | NTB saja | `reward_ntb` | Hero general + **1 card NTB** |
| B | NTB + ETB | `reward_dual` | Hero general + **2 card** (NTB & ETB) |
| C | ETB saja | `reward_etb` | Hero general + **1 card ETB** |
| D | Tidak ada | `no_reward` | Hero general + **1 card info** (bukan card program) |

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
3. **Hanya card program yang berubah** sesuai program aktif: 0 card program (ganti card info), 1 card, atau 2 card.
4. Referrer membaca reward, periode, dan kriteria **hanya dari card program aktif**.

## 4. Prinsip Desain (wajib dipegang designer)

1. **Kode referral selalu hidup.** Tidak pernah disembunyikan, dinonaktifkan, atau diganti meski tidak ada program reward.
2. **Hero general di semua state.** Headline & subheadline mengajak pakai Qita — **tanpa menyebut reward, periode, atau segmen program**. Tidak berubah antar `ui_mode`.
3. **Hanya tampilkan card untuk program yang aktif.** Program NTB tidak aktif → card NTB **tidak muncul**. Program ETB tidak aktif → card ETB **tidak muncul**. Jangan tampilkan card program yang sedang tidak berjalan.
4. **Saat tidak ada program aktif:** tampilkan **satu card informasi** (bukan card NTB/ETB) yang menjelaskan belum ada reward dan kode tetap bisa dibagikan.
5. **Satu kode, satu tombol share.** Jangan pernah meminta user memilih segmen sebelum share.
6. **Bahasa manusia, bukan istilah internal.** NTB → "teman yang belum punya rekening BRI"; ETB → "teman yang sudah punya rekening BRI atau BRImo".
7. **Reward referrer & referee hanya di card program aktif.** Hero, subheadline, dan share copy **tidak boleh** menyebut nominal reward.
8. **Masa berlaku hanya di card program aktif** — per program, independen jika ada 2 program.
9. **Tanpa reward di halaman referral = tanpa janji nominal** di hero & share copy. Onboarding referee tetap menampilkan janji reward setelah tipe terdeteksi.
10. **Tidak ada layout shift antar state.** Template hub tetap; yang berubah hanya jumlah & isi card.
11. **Jangan pernah menampilkan nominal dari cache lama.** Fallback ke `no_reward` jika config gagal.

## 5. Arsitektur Konten (Server-Driven)

### 5.1 Sumber Kebenaran

Procash meng-expose config program aktif. Client me-render UI berdasarkan `ui_mode` yang **dihitung di backend** — client tidak mengevaluasi eligibility sendiri.

Konten dinamis di-map dari `active_programs[]` — **hanya program yang `is_active: true`**. Client tidak me-render card untuk program tidak aktif.

| Konten UI | Sumber API |
|---|---|
| Card program (reward, periode, kriteria) | `active_programs[]` — 0, 1, atau 2 item |
| Card info tidak ada program | `no_program_card` — hanya saat `active_programs[]` kosong |
| Hero headline & subheadline | **Statis** — sama di semua `ui_mode` |

### 5.2 API Contract — `GET /referral/hub`

```json
{
  "referral_code": "QITA-ABC123",
  "ui_mode": "reward_dual",

  "hero": {
    "headline": "Ajak temanmu pakai Qita",
    "subheadline": "Bagikan kode referralmu dan ajak temanmu bergabung"
  },

  "active_programs": [
    {
      "program_id": "NTB_2026_Q3",
      "referee_type": "NTB",
      "referee_type_label": "Teman yang belum punya rekening BRI",
      "period": {
        "start_date": "2026-07-01",
        "end_date": "2026-08-31",
        "display": "Berlaku s.d. 31 Agustus 2026"
      },
      "rewards_by_user_type": [
        {
          "user_type": "NTB",
          "user_type_label": "Teman yang belum punya rekening BRI",
          "referrer_reward": {
            "amount": 25000,
            "display": "Rp25.000"
          },
          "referee_reward": {
            "amount": 10000,
            "display": "Rp10.000"
          },
          "qualification": {
            "display_summary": "Buka rekening & lakukan setoran/transaksi pertama min. Rp50.000 dalam 7 hari"
          }
        }
      ],
      "referee_criteria": [
        {
          "code": "NO_BRI_ACCOUNT",
          "display": "Belum punya rekening BRI sama sekali"
        },
        {
          "code": "NEVER_USED_QITA",
          "display": "Belum pernah pakai BRImo atau Qita"
        }
      ],
      "qualification": {
        "action_type": "OPEN_ACCOUNT_FIRST_TRANSACTION",
        "display_summary": "Buka rekening & lakukan setoran/transaksi pertama min. Rp50.000 dalam 7 hari",
        "deadline_days": 7,
        "min_amount": 50000,
        "min_amount_display": "Rp50.000",
        "reward_processing_days": 2
      },
      "quota": {
        "max_per_referrer": 200,
        "used_by_referrer": 12
      },
      "share_copy": {
        "template": "(tidak dipakai di halaman referral — gunakan share_copy_default tanpa nominal)"
      },
      "tnc": {
        "title": "Program Ajak Teman Baru",
        "sections": [
          {
            "title": "Reward",
            "content": "Pengundang: Rp25.000 per teman NTB. Teman yang diajak: Rp10.000."
          },
          {
            "title": "Syarat Teman yang Diajak",
            "content": "Belum pernah memiliki rekening BRI. Melakukan setoran/transaksi pertama min. Rp50.000 dalam 7 hari."
          }
        ]
      }
    },
    {
      "program_id": "ETB_2026_Q3",
      "referee_type": "ETB",
      "referee_type_label": "Teman pengguna BRI/BRImo",
      "period": {
        "start_date": "2026-07-01",
        "end_date": "2026-08-31",
        "display": "Berlaku s.d. 31 Agustus 2026"
      },
      "rewards_by_user_type": [
        {
          "user_type": "ETB_CIF",
          "user_type_label": "Teman yang punya rekening BRI (belum BRImo)",
          "referrer_reward": {
            "amount": 15000,
            "display": "Rp15.000"
          },
          "referee_reward": {
            "amount": 5000,
            "display": "Rp5.000"
          },
          "qualification": {
            "display_summary": "Aktivasi Qita dengan rekening BRI yang sudah ada & lakukan transaksi pertama dalam 7 hari"
          }
        },
        {
          "user_type": "ETB_BERBRIMO",
          "user_type_label": "Teman pengguna BRImo",
          "referrer_reward": {
            "amount": 20000,
            "display": "Rp20.000"
          },
          "referee_reward": {
            "amount": 8000,
            "display": "Rp8.000"
          },
          "qualification": {
            "display_summary": "Aktivasi Qita dengan rekening BRI/BRImo yang sudah ada & lakukan transaksi pertama dalam 7 hari"
          }
        }
      ],
      "referee_criteria": [
        {
          "code": "HAS_BRI_ACCOUNT",
          "display": "Sudah punya rekening BRI atau BRImo"
        },
        {
          "code": "NEVER_USED_QITA",
          "display": "Belum pernah pakai Qita"
        }
      ],
      "qualification": {
        "action_type": "ACTIVATE_EXISTING_FIRST_TRANSACTION",
        "display_summary": "Aktivasi Qita dengan rekening BRI yang sudah ada & lakukan transaksi pertama dalam 7 hari",
        "deadline_days": 7,
        "reward_processing_days": 2
      },
      "quota": {
        "max_per_referrer": 200,
        "used_by_referrer": 12
      },
      "share_copy": {
        "template": "(tidak dipakai di halaman referral — gunakan share_copy_default tanpa nominal)"
      },
      "tnc": {
        "title": "Program Ajak Pengguna BRI/BRImo",
        "sections": [ "..." ]
      }
    }
  ],

  "no_program_card": null,

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
  "no_program_card": {
    "title": "Program Reward",
    "message": "Belum ada program reward saat ini. Kamu tetap bisa mengajak teman ke Qita — kode referral kamu tetap berlaku."
  },
  "share_copy_default": {
    "no_reward": "Coba Qita, aplikasi banking BRI yang praktis. Pakai kode {referral_code} saat daftar."
  }
}
```

**Contoh response saat hanya NTB aktif** — `active_programs` berisi **1 item** (NTB saja), tanpa objek ETB.

**Catatan field wajib:**
- `active_programs[]` hanya berisi program dengan `is_active: true` di Procash (0–2 item).
- `no_program_card` hanya dikirim jika `active_programs[]` kosong; selain itu `null`.
- `hero` sama di semua response — tidak bergantung `ui_mode`.
- Client **hanya render card** untuk item di `active_programs[]`, plus `no_program_card` jika kosong.

### 5.3 Logic `ui_mode` (dihitung backend)

```javascript
function resolveUiMode(activePrograms) {
  if (!activePrograms || activePrograms.length === 0) return "no_reward";

  const types = activePrograms.map(p => p.referee_type);
  if (types.includes("NTB") && types.includes("ETB")) return "reward_dual";
  if (types.includes("NTB")) return "reward_ntb";
  if (types.includes("ETB")) return "reward_etb";
}
```

**Catatan:** Program tidak aktif **tidak dikirim** ke client. UI hanya menampilkan card untuk program di `active_programs[]`.

### 5.4 Mapping UI Component ↔ API Field

#### Hero (Headline & Subheadline)

**General — sama di semua `ui_mode`. Tidak menyebut reward, periode, atau segmen program.**

| Elemen | Copy | Sumber API |
|---|---|---|
| Headline | "Ajak temanmu pakai Qita" | `hero.headline` |
| Subheadline | "Bagikan kode referralmu dan ajak temanmu bergabung" | `hero.subheadline` |

#### Section Program Reward (Card)

**Hanya render card untuk program aktif.** Jumlah card = `active_programs.length` (0, 1, atau 2).

| Kondisi | Yang ditampilkan |
|---|---|
| `active_programs.length === 0` | **1 card informasi** dari `no_program_card` |
| `active_programs.length === 1` | **1 card program** (NTB atau ETB) |
| `active_programs.length === 2` | **2 card program** (NTB + ETB) |

#### Card Program Aktif

```
┌─────────────────────────────────────────────┐
│  Ajak Teman Baru di BRI            [Aktif]  │
│  ⏱ Berlaku s.d. 31 Agustus 2026             │  ← period.display per card
├─────────────────────────────────────────────┤
│  Teman yang belum punya rekening BRI        │
│  Kamu: Rp25.000  ·  Temanmu: Rp10.000      │
├─────────────────────────────────────────────┤
│  ✓ Belum punya rekening BRI sama sekali     │
│  ✓ Belum pernah pakai BRImo atau Qita       │
└─────────────────────────────────────────────┘
```

**Jika NTB dan ETB aktif dengan periode berbeda** — masing-masing card menampilkan `period.display` sendiri.

#### Card Informasi (Tidak Ada Program Aktif)

Digunakan **hanya** saat `active_programs[]` kosong. Bukan card NTB/ETB.

```
┌─────────────────────────────────────────────┐
│  Program Reward                             │
├─────────────────────────────────────────────┤
│  Belum ada program reward saat ini.         │
│  Kamu tetap bisa mengajak teman ke Qita —   │
│  kode referral kamu tetap berlaku.          │
└─────────────────────────────────────────────┘
```

Sumber: `no_program_card.title` + `no_program_card.message`

| `ui_mode` | Jumlah card di section |
|---|---|
| `reward_ntb` | 1 card program (NTB) |
| `reward_etb` | 1 card program (ETB) |
| `reward_dual` | 2 card program (NTB + ETB) |
| `no_reward` | 1 card informasi (`no_program_card`) |

#### Cara Kerja — Langkah 3 (Dinamis)

Di-map dari `active_programs[]` — hanya program yang ditampilkan sebagai card.

| `ui_mode` | Langkah 3 |
|---|---|
| `reward_ntb` | Satu teks dari `rewards_by_user_type[0].qualification` |
| `reward_etb` | Satu bullet per entry di `rewards_by_user_type[]` |
| `reward_dual` | Bullet per program aktif, per tipe user jika berbeda |
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
| Detail program | `active_programs[].tnc` |

### 5.5 API Contract — Onboarding Referee

Endpoint terpisah: dipanggil setelah referee input NIK (deteksi tipe referee).

```json
{
  "referral_code": "QITA-ABC123",
  "referrer_name": "Ahmad",
  "referee_type": "NTB",
  "matched_program": {
    "program_id": "NTB_2026_Q3",
    "referee_type": "NTB",
    "user_type": "NTB",
    "user_type_label": "Teman yang belum punya rekening BRI",
    "referrer_reward": {
      "amount": 25000,
      "display": "Rp25.000"
    },
    "referee_reward": {
      "amount": 10000,
      "display": "Rp10.000"
    },
    "qualification": {
      "display_summary": "Buka rekening & lakukan setoran/transaksi pertama min. Rp50.000 dalam 7 hari",
      "deadline_date": "2026-08-31"
    }
  }
}
```

Jika referee tidak match program aktif: `matched_program: null` → tidak ada janji reward di onboarding.

**Implikasi untuk designer:** semua komponen teks harus didesain dengan asumsi konten variabel. Siapkan spec untuk truncation dan dynamic type.

---

## 6. Spesifikasi UI Lengkap

Bagian ini adalah **acuan utama untuk UI/UX Designer** — merangkum seluruh keputusan desain terbaru dalam bentuk wireframe, komponen, dan aturan tampilan.

### 6.1 Aturan Penempatan Konten

| Informasi | Hero | Card Program Aktif | Card Info (no program) | Share Copy |
|---|---|---|---|---|
| Nominal reward | ❌ | ✅ | ❌ | ❌ |
| Masa berlaku | ❌ | ✅ | ❌ | ❌ |
| Kriteria teman | ❌ | ✅ | ❌ | ❌ |
| Belum ada reward / tetap bisa ajak | ❌ | — | ✅ | ❌ |
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
│  [1–2 card program / 1 card info]       │  ← hanya bagian ini yang berubah
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
| Ada program reward aktif (`reward_*`) | **Bagikan Sekarang** |
| Tidak ada program (`no_reward`) | **Bagikan ke Teman** |

Kode **selalu tampil** di semua state. Tap [Salin] → toast "Kode berhasil disalin". Tap CTA → native share sheet dengan `share_copy_default` (tanpa nominal).

### 6.6 Komponen: Card Program Aktif

**Anatomi card:**

```
┌─────────────────────────────────────────────┐
│  {referee_type_label}              [Aktif]  │  ← header + badge pill
│  ⏱ {period.display}                         │  ← masa berlaku per program
├─────────────────────────────────────────────┤
│  {user_type_label}                          │  ← per entry rewards_by_user_type
│  Kamu {referrer_reward.display}             │
│       · Temanmu {referee_reward.display}    │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─   │  ← divider jika >1 tipe user
│  {user_type_label}                          │
│  Kamu ... · Temanmu ...                     │
├─────────────────────────────────────────────┤
│  ✓ {referee_criteria[0].display}           │
│  ✓ {referee_criteria[1].display}           │
└─────────────────────────────────────────────┘
```

**Spesifikasi visual:**

| Elemen | Spec |
|---|---|
| Badge "Aktif" | Pill kecil, warna brand (hijau/biru), pojok kanan atas card |
| Card container | Background highlight subtle / border brand, elevation 1 |
| `period.display` | Ikon ⏱ + teks secondary, di bawah header |
| Baris reward | `user_type_label` regular; nominal **semibold** |
| Divider antar tipe user | Hairline 1px di dalam card |
| Checklist kriteria | Ikon ✓ + teks secondary |

**Card ETB multi tipe user (contoh):**

```
┌─────────────────────────────────────────────┐
│  Ajak Pengguna BRI/BRImo           [Aktif]  │
│  ⏱ Berlaku s.d. 15 September 2026           │
├─────────────────────────────────────────────┤
│  Teman punya rekening BRI (belum BRImo)     │
│  Kamu Rp15.000  ·  Temanmu Rp5.000          │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─   │
│  Teman pengguna BRImo                       │
│  Kamu Rp20.000  ·  Temanmu Rp8.000          │
├─────────────────────────────────────────────┤
│  ✓ Sudah punya rekening BRI atau BRImo      │
│  ✓ Belum pernah pakai Qita                  │
└─────────────────────────────────────────────┘
```

### 6.7 Komponen: Card Informasi (Tidak Ada Program)

**Satu card** — menggantikan card program saat tidak ada program aktif.

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
| Konten | Tanpa nominal, tanpa periode, tanpa checklist kriteria |
| Posisi | Menggantikan area card program — bukan di hero |

**Alasan desain:** satu card info lebih jelas daripada 2 card NTB/ETB kosong atau menghilangkan section sepenuhnya.

### 6.8 Layout Card per State

| State | Card yang ditampilkan |
|---|---|
| A — `reward_ntb` | 1× Card program NTB |
| B — `reward_dual` | 1× Card NTB + 1× Card ETB |
| C — `reward_etb` | 1× Card program ETB |
| D — `no_reward` | 1× Card informasi (`no_program_card`) |

Program tidak aktif **tidak ditampilkan** sebagai card.

### 6.9 Wireframe Lengkap — State A (Hanya NTB Aktif)

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

### 6.10 Wireframe Lengkap — State B (NTB + ETB Aktif)

> Hero **sama** dengan state lain. **2 card** — NTB dan ETB.

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
│  │ Teman belum punya rekening BRI  │    │
│  │ Kamu Rp25.000 · Teman Rp10.000  │    │
│  │ ✓ Belum punya rekening BRI      │    │
│  │ ✓ Belum pernah pakai BRImo/Qita │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │ Ajak Pengguna BRI/BRImo [Aktif]│    │
│  │ ⏱ Berlaku s.d. 31 Agustus 2026 │    │
│  │ Teman punya rekening BRI        │    │
│  │ Kamu Rp15.000 · Teman Rp5.000   │    │
│  │ ✓ Sudah punya rekening BRI      │    │
│  │ ✓ Belum pernah pakai Qita       │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Cara kerjanya                          │
│  ① Bagikan kode                         │
│  ② Teman daftar pakai kodemu            │
│  ③ Dua bullet — satu per program aktif  │
│                                         │
│  Status ajakanmu                        │
│  [list ajakan...]                       │
│                                         │
│  Pelajari Syarat & Ketentuan →          │
└─────────────────────────────────────────┘
```

### 6.11 Wireframe Lengkap — State C (Hanya ETB Aktif)

> Hero **sama** dengan state lain. Hanya **1 card ETB** — card NTB **tidak ditampilkan**.

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
│  │ Ajak Pengguna BRI/BRImo [Aktif]│    │
│  │ ⏱ Berlaku s.d. 31 Agustus 2026 │    │
│  │                                 │    │
│  │ Teman punya rekening BRI        │    │
│  │ Kamu Rp15.000 · Teman Rp5.000   │    │
│  │ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─   │    │
│  │ Teman pengguna BRImo            │    │
│  │ Kamu Rp20.000 · Teman Rp8.000   │    │
│  │                                 │    │
│  │ ✓ Sudah punya rekening BRI      │    │
│  │ ✓ Belum pernah pakai Qita       │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Cara kerjanya                          │
│  ① Bagikan kode                         │
│  ② Teman daftar pakai kodemu            │
│  ③ Teman aktivasi Qita                  │
│                                         │
│  Status ajakanmu                        │
│  [list ajakan...]                       │
│                                         │
│  Pelajari Syarat & Ketentuan →          │
└─────────────────────────────────────────┘
```

### 6.12 Wireframe Lengkap — State D (Tidak Ada Program)

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
| Loading | Skeleton mengikuti template (hero + kode + 1–2 card slot) — jumlah card slot mengikuti `ui_mode` terakhir atau default 1 card; tanpa layout shift |
| Config gagal | Fallback `no_reward` — **tidak pernah** tampilkan nominal dari cache |

---

## 7. Requirement per Skenario

### Skenario A — Program NTB Aktif (`reward_ntb`)

| Elemen | Konten | Sumber API |
|---|---|---|
| Headline | "Ajak temanmu pakai Qita" | `hero.headline` — **sama semua state** |
| Subheadline | "Bagikan kode referralmu dan ajak temanmu bergabung" | `hero.subheadline` — **sama semua state** |
| Card NTB | Reward per tipe user + **Berlaku s.d. 31 Agustus 2026** + kriteria | `active_programs[NTB]` |
| Card ETB | **Tidak ditampilkan** | — |
| Cara kerja langkah 3 | "Buka rekening & lakukan setoran/transaksi pertama..." | `active_programs[NTB].qualification.display_summary` |
| Share copy | "Coba Qita, aplikasi banking BRI yang praktis. Pakai kode {referral_code} saat daftar." — **tanpa nominal** | `share_copy_default` |

### Skenario B — Program NTB + ETB Aktif (`reward_dual`)

| Elemen | Konten | Sumber API |
|---|---|---|
| Headline | "Ajak temanmu pakai Qita" | `hero.headline` — **sama semua state** |
| Subheadline | "Bagikan kode referralmu dan ajak temanmu bergabung" | `hero.subheadline` — **sama semua state** |
| Card NTB | Reward + periode + kriteria | `active_programs[NTB]` |
| Card ETB | Reward per sub-tipe + periode + kriteria | `active_programs[ETB]` |
| Cara kerja langkah 3 | Dua bullet, satu per program | `active_programs[].qualification.display_summary` |
| Share copy | Template netral tanpa nominal spesifik | `share_copy_default` |
| Larangan | Tidak ada pemilihan segmen sebelum share | — |

### Skenario C — Program ETB Aktif (`reward_etb`)

| Elemen | Konten | Sumber API |
|---|---|---|
| Headline | "Ajak temanmu pakai Qita" | `hero.headline` — **sama semua state** |
| Subheadline | "Bagikan kode referralmu dan ajak temanmu bergabung" | `hero.subheadline` — **sama semua state** |
| Card ETB | Reward per sub-tipe + periode + kriteria | `active_programs[ETB]` |
| Card NTB | **Tidak ditampilkan** | — |
| Cara kerja langkah 3 | "Aktivasi Qita dengan rekening BRI yang sudah ada..." | `active_programs[ETB].qualification.display_summary` |
| Share copy | "Coba Qita, aplikasi banking BRI yang praktis. Pakai kode {referral_code} saat daftar." — **tanpa nominal** | `share_copy_default` |

*(Seluruh nominal adalah placeholder — nilai riil dari config Procash.)*

### Skenario D — Tidak Ada Program Aktif (`no_reward`)

| Elemen | Konten | Sumber API |
|---|---|---|
| Headline | "Ajak temanmu pakai Qita" | `hero.headline` — **sama semua state** |
| Subheadline | "Bagikan kode referralmu dan ajak temanmu bergabung" | `hero.subheadline` — **sama semua state** |
| Card program | **Tidak ditampilkan** | — |
| Card info | "Belum ada program reward saat ini. Kamu tetap bisa mengajak teman ke Qita — kode referral kamu tetap berlaku." | `no_program_card` |
| Kode & share | [Bagikan ke Teman] — bukan [Bagikan Sekarang] | `referral_code` |
| Share copy | "Coba Qita, aplikasi banking BRI yang praktis. Pakai kode {referral_code} saat daftar." | `share_copy_default.no_reward` |
| Opt-in notifikasi | "Beri tahu saya saat ada program reward" | Statis |
| Cara kerja | 2 langkah saja (tanpa langkah reward) | Statis |
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

③ Teman selesaikan syarat program          ← DINAMIS
   [active_programs[].qualification.display_summary]
   Reward cair maks. {reward_processing_days}×24 jam.
```

- **Single program:** satu teks di langkah 3.
- **Dual program:** dua bullet di langkah 3, masing-masing dari `active_programs[].qualification.display_summary`.

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

Sumber: `active_programs[].tnc` + detail reward dari `active_programs[].rewards_by_user_type[]` — satu section per program aktif. **Reward di S&K harus mencantumkan nominal per tipe user**, konsisten dengan yang ditampilkan di card.

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
│  [Detail Program Saat Ini]          │  ← active_programs[].tnc
│  (dinamis, per referee_type)        │     hanya jika reward aktif
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
- **Progres kuota** (mode reward aktif): "Kamu sudah mengajak 12/200" — dari `active_programs[].quota`.

| Status | Copy contoh | `ui_mode` | Aksi |
|---|---|---|---|
| Terdaftar | "Budi sudah gabung, tinggal transaksi pertama" | reward_* | [Ingatkan] |
| Memenuhi syarat | "Reward sedang diproses" | reward_* | — |
| Reward cair | "Rp25.000 · 2 Jul 2026" + label program | reward_* | — |
| Tidak memenuhi syarat | "Budi sudah gabung, tapi tidak memenuhi kriteria program (sudah punya rekening BRI)" | reward_* | Link ke S&K |
| Terdaftar, tanpa reward | "Budi sudah gabung — tidak ada program reward aktif saat ini" | no_reward | — |

Riwayat lintas program dipertahankan selamanya (dengan label program).

## 12. Sisi Referee (Teman yang Diundang)

1. Deep link membawa kode referral ke onboarding. Kode **selalu diterima**.
2. Setelah referee input NIK, backend deteksi `referee_type` (cek CIF).
3. **Janji reward hanya ditampilkan jika:**
   - Ada program aktif yang `referee_type`-nya match dengan tipe referee terdeteksi
   - Response `matched_program` tidak null
4. Jika tidak match: onboarding normal, **tanpa janji nominal**, tanpa pesan penolakan frontal. Attribution tetap dicatat.
5. Terms di-lock saat registrasi: syarat yang berlaku adalah syarat saat referee mendaftar.

### Flow Deteksi Referee

```
Referee input NIK → Backend cek CIF
       │
       ├── Tidak ada CIF        → referee_type = "NTB"
       ├── Ada CIF              → referee_type = "ETB"
       └── Dormant re-register  → kebijakan backend (ETB X NTB)
       │
       ▼
Match dengan active_programs[].referee_type yang aktif?
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
- **Nominal reward & masa berlaku hanya di card program aktif** — dilarang di hero, subheadline, share copy, banner, dan entry point.
- Program tidak aktif **tidak ditampilkan** sebagai card — jangan gunakan card gray-out atau `inactive_message`.
- Saat tidak ada program: satu card informasi (`no_program_card`) — tanpa nominal, tanpa periode.
- Share copy dari halaman referral: value prop produk, **zero mention nominal**.
- Alasan gagal di tracker: jujur, bahasa kriteria yang sama dengan card program aktif.
- CTA: "Bagikan Sekarang" (ada reward) vs "Bagikan ke Teman" (tanpa reward).

## 15. Edge Cases

| Kasus | Perlakuan |
|---|---|
| Referee ETB masuk saat hanya program NTB aktif | Onboarding tanpa janji reward; tracker referrer: "tidak memenuhi kriteria program" |
| Referee NTB masuk saat hanya program ETB aktif | Idem |
| Referrer share saat reward aktif, referee daftar setelah program berakhir | Lock-in terms jika dalam grace period Procash; jika tidak, tracker: tanpa reward |
| ETB X NTB (dormant daftar ulang) | Kebijakan backend/S&K; UI: "tidak memenuhi kriteria program" |
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
| K-factor per segmen | Undangan → registrasi → kualifikasi, dipisah NTB vs ETB |
| Mismatch rate | Referee masuk via link tapi `referee_type` tidak match program aktif |
| Time-to-reward | Kualifikasi → reward cair; target < 48 jam |
| Repeat referral rate | Referrer yang mengajak ≥ 2 teman |
| CS ticket rate | Tiket referral per 1.000 referral |
| False promise rate | Share copy janji reward tapi tidak ada program aktif saat referee daftar |

## 17. Deliverables yang Diminta dari Designer

> **Acuan utama desain: Section 6 — Spesifikasi UI Lengkap**

1. High-fidelity design Referral Hub untuk **4 state** (A/B/C/D) mengikuti wireframe Section 6.9–6.12.
2. Komponen card program aktif (Section 6.6) dan card informasi tidak ada program (Section 6.7).
3. Komponen hero, kode, CTA, cara kerja, tracker, bottom sheet (Section 6.4–6.5, 6.13–6.15).
4. Flow referee onboarding (Section 6.17).
5. Entry points tanpa nominal (Section 6.16).
6. Loading skeleton & error state (Section 6.18).
7. Spec komponen dinamis: truncation, dynamic type, transisi tanpa layout shift.

## 18. Open Questions

1. Nominal & syarat kualifikasi final **per tipe user** per program (menunggu konfigurasi Procash).
2. Kebijakan final eligibility ETB X NTB (dihitung NTB atau tidak) — menentukan deteksi `referee_type`.
3. Apakah reward cair otomatis atau perlu klaim manual? (Rekomendasi: otomatis.)
4. Batas kuota per referrer per periode dan per program.
5. Channel share: native share sheet vs shortcut khusus (WhatsApp-first?).
6. Grace period setelah program berakhir: berapa hari referee yang sudah terdaftar masih bisa memenuhi syarat?
