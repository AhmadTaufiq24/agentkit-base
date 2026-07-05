# PRD — Referral Hub Qita (Adaptif Berdasarkan Program Procash)

| | |
|---|---|
| **Dokumen** | Product Requirements Document (PRD) |
| **Fitur** | Referral Hub Qita |
| **Audiens dokumen** | UI/UX Designer, Frontend, Backend |
| **Status** | Draft v1.2 |
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

### Segmen Program (Referee Type)

Program yang dapat dibuat di Procash menargetkan **tipe user referee**:

- **Program NTB** (`referee_type: "NTB"`) — reward jika teman yang diajak **belum punya rekening BRI sama sekali**.
- **Program ETB** (`referee_type: "ETB"`) — reward jika teman yang diajak **sudah punya rekening BRI dan/atau BRImo** dan aktivasi Qita.

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
| A | NTB saja | `reward_ntb` | Hero reward NTB |
| B | NTB + ETB | `reward_dual` | Hero reward dual-segmen |
| C | ETB saja | `reward_etb` | Hero reward ETB |
| D | Tidak ada | `no_reward` | Mode non-monetary — kode tetap aktif |

Semua referrer melihat UI yang **sama** untuk program yang sama. Tidak ada pengecekan tipe user referrer.

## 2. Masalah yang Diselesaikan

1. Referrer tidak tahu dan tidak seharusnya perlu tahu status perbankan temannya. Jika UI memaksa user memahami segmentasi, sharing rate turun dan mismatch naik.
2. Jika kriteria teman yang valid tidak terkomunikasikan jelas, terjadi **janji reward yang gagal** → komplain CS, rusaknya trust.
3. Program berganti-ganti sepanjang waktu; UI statis akan menampilkan janji basi.
4. User perlu paham perbedaan **"kode masih bisa dipakai"** vs **"ada reward"** — tanpa merasa fitur mati atau ditipu.

## 3. Goals & Non-Goals

### Goals

1. Satu Referral Hub adaptif: satu template, konten dinamis dari config Procash.
2. User paham dalam ≤ 5 detik: **apakah ada reward, siapa yang bisa diajak, berapa reward masing-masing pihak**.
3. Satu kode/link referral per user, **selalu aktif** — apapun status program reward-nya.
4. Sistem (bukan user) yang menentukan tipe referee dan mencocokkan dengan program aktif.
5. Transparansi status referral end-to-end (Diundang → Terdaftar → Memenuhi Syarat → Reward Cair / Tanpa Reward).
6. Konsistensi janji di 3 titik: layar referrer, pesan share, layar onboarding referee.

### Non-Goals

- Desain dashboard Procash (out of scope).
- Mekanisme anti-fraud backend (dedupe NIK/CIF) — hanya implikasi UI-nya yang dicakup.
- Pengecekan tipe user referrer untuk eligibility reward.
- Reward selain yang dikonfigurasi Procash (tiering, gamifikasi lanjutan) — fase berikutnya.

## 4. Prinsip Desain (wajib dipegang designer)

1. **Kode referral selalu hidup.** Tidak pernah disembunyikan, dinonaktifkan, atau diganti meski tidak ada program reward.
2. **Reward adalah kampanye, bukan fitur inti.** UI membedakan jelas mode "ada reward" vs "tanpa reward" tanpa membuat fitur terasa mati.
3. **Satu kode, satu tombol share.** Jangan pernah meminta user memilih "mau ajak NTB atau ETB" sebelum share.
4. **Hide, jangan disable.** Program yang tidak aktif tidak ditampilkan — bukan card yang di-gray-out.
5. **Bahasa manusia, bukan istilah internal.** NTB → "teman yang belum punya rekening BRI"; ETB → "teman yang sudah punya rekening BRI atau BRImo".
6. **Kriteria teman yang valid muncul 3×** (hanya saat mode reward aktif): di headline hero, di section "Siapa yang bisa kamu ajak", dan di pesan share pre-filled.
7. **Tanpa reward = tanpa janji nominal di mana pun.** Share copy, hero, dan onboarding referee tidak boleh menyebut angka reward jika tidak ada program aktif.
8. **Tidak ada layout shift antar state.** Template hub tetap; hanya konten yang berubah.
9. **Jangan pernah menampilkan nominal dari cache lama.** Jika config gagal dimuat, fallback ke mode non-monetary.

## 5. Arsitektur Konten (Server-Driven)

### 5.1 Sumber Kebenaran

Procash meng-expose config program aktif. Client me-render UI berdasarkan `ui_mode` yang **dihitung di backend** — client tidak mengevaluasi eligibility sendiri.

Konten dinamis di-map dari array `programs[]`, di-index oleh `referee_type` (`NTB` / `ETB`):

| Konten UI | Sumber API |
|---|---|
| Nominal reward | `programs[].reward` |
| Periode | `programs[].period` |
| Syarat siapa yang bisa diajak | `programs[].referee_criteria` |
| Syarat apa yang harus dilakukan | `programs[].qualification` |
| Share copy | `programs[].share_copy` |
| S&K detail program | `programs[].tnc` |
| Kuota | `programs[].quota` |

### 5.2 API Contract — `GET /referral/hub`

```json
{
  "referral_code": "QITA-ABC123",
  "ui_mode": "reward_dual",

  "programs": [
    {
      "program_id": "NTB_2026_Q3",
      "referee_type": "NTB",
      "referee_type_label": "Teman yang belum punya rekening BRI",
      "period": {
        "start_date": "2026-07-01",
        "end_date": "2026-08-31",
        "display": "Berlaku s.d. 31 Agustus 2026"
      },
      "reward": {
        "referrer_amount": 25000,
        "referee_amount": 10000,
        "currency": "IDR",
        "display_referrer": "Rp25.000",
        "display_referee": "Rp10.000"
      },
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
        "template": "Belum punya rekening? Buka rekening BRI pertamamu di Qita pakai kode {referral_code}, langsung dapat {referee_reward}!"
      },
      "tnc": {
        "title": "Program Ajak Teman Baru",
        "sections": [
          {
            "title": "Reward",
            "content": "Pengundang: Rp25.000 per teman. Teman yang diajak: Rp10.000."
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
      "reward": {
        "referrer_amount": 15000,
        "referee_amount": 5000,
        "display_referrer": "Rp15.000",
        "display_referee": "Rp5.000"
      },
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
        "template": "Udah punya BRImo atau rekening BRI? Cobain Qita — aktivasi pakai kode {referral_code}, dapat {referee_reward}!"
      },
      "tnc": {
        "title": "Program Ajak Pengguna BRI/BRImo",
        "sections": [ "..." ]
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

### 5.3 Logic `ui_mode` (dihitung backend)

```javascript
function resolveUiMode(programs) {
  if (!programs || programs.length === 0) return "no_reward";

  const types = programs.map(p => p.referee_type);
  if (types.includes("NTB") && types.includes("ETB")) return "reward_dual";
  if (types.includes("NTB")) return "reward_ntb";
  if (types.includes("ETB")) return "reward_etb";
}
```

### 5.4 Mapping UI Component ↔ API Field

#### Hero (Headline & Subheadline)

| `ui_mode` | Sumber field | Logic render |
|---|---|---|
| `reward_ntb` | `programs[referee_type=NTB].reward` | "Ajak temanmu buka rekening pertama di Qita, kamu dapat **{display_referrer}**" |
| `reward_etb` | `programs[referee_type=ETB].reward` | "Punya teman pengguna BRI/BRImo? Ajak mereka pakai Qita, kamu dapat **{display_referrer}**" |
| `reward_dual` | `max(programs[].reward.referrer_amount)` | "Ajak siapa saja ke Qita, dapat hingga **{max_display_referrer}** per teman" |
| `no_reward` | — | Headline statis: "Ajak temanmu rasakan Qita" |

#### Badge Periode

| Kondisi | Mapping |
|---|---|
| 1 program aktif | `programs[0].period.display` |
| 2 program, periode sama | `programs[0].period.display` |
| 2 program, periode beda | `Berlaku s.d. {formatDate(max(programs[].period.end_date))}` |

#### "Siapa yang Bisa Kamu Ajak"

Di-map dari `programs[].referee_criteria` per `referee_type`:

| `ui_mode` | Render |
|---|---|
| `reward_ntb` / `reward_etb` | Checklist: `programs[0].referee_criteria[].display` |
| `reward_dual` | Dual-card: satu card per item `programs[]` dengan `referee_type_label` + `reward` |

#### Cara Kerja — Langkah 3 (Dinamis)

Di-map dari `programs[].qualification.display_summary` per `referee_type`:

| `ui_mode` | Langkah 3 |
|---|---|
| `reward_ntb` / `reward_etb` | `{qualification.display_summary}` + "Reward cair maks. {reward_processing_days}×24 jam" |
| `reward_dual` | Dua bullet, satu per program |
| `no_reward` | Langkah 3 **tidak di-render** (hanya 2 langkah) |

#### Share Copy

| Kondisi | Sumber |
|---|---|
| Program NTB aktif | `programs[referee_type=NTB].share_copy.template` |
| Program ETB aktif | `programs[referee_type=ETB].share_copy.template` |
| Dual program | Template netral dari Procash atau `share_copy_default` |
| Tidak ada program | `share_copy_default.no_reward` |

Placeholder di template: `{referral_code}`, `{referee_reward}`, `{referrer_reward}` — di-replace client saat render.

#### S&K

| Lapisan | Sumber | Kapan tampil |
|---|---|---|
| Ketentuan umum | `general_tnc.sections[]` | Selalu |
| Detail program | `programs[].tnc` | Saat `ui_mode` ≠ `no_reward` (1 atau 2 section) |

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
    "reward": {
      "referee_amount": 10000,
      "display_referee": "Rp10.000"
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

## 6. Struktur Halaman Referral Hub

### Mode Reward Aktif (State A / B / C)

```
┌─────────────────────────────────────┐
│ ① HERO: ilustrasi + headline +      │  Above the fold.
│    subheadline (nominal 2 pihak)    │  Menjawab: "apa untungnya?"
│ ② BADGE PERIODE                     │  ← programs[].period.display
│ ③ KODE REFERRAL [Salin]             │  Menjawab: "apa yang harus
│    [ Bagikan Sekarang ] (primary)   │   saya lakukan?"
├─────────────────────────────────────┤
│ ④ SIAPA YANG BISA KAMU AJAK         │  ← programs[].referee_criteria
│ ⑤ CARA KERJA (3 langkah)            │  ← langkah 3: programs[].qualification
│ ⑥ STATUS AJAKANMU (tracker)         │  + agregat total reward
│ ⑦ S&K (bottom sheet trigger)        │  ← general_tnc + programs[].tnc
└─────────────────────────────────────┘
```

### Mode Tanpa Reward (State D)

```
┌─────────────────────────────────────┐
│ ① HERO: ilustrasi + headline        │  Above the fold.
│    non-monetary + penjelasan status │
│ ② KODE REFERRAL [Salin]             │  Tetap prominent.
│    [ Bagikan ke Teman ] (primary)   │
│ ③ OPT-IN NOTIFIKASI                 │  "Beri tahu saya saat ada reward"
├─────────────────────────────────────┤
│ ④ CARA KERJA (2 langkah)            │  Tanpa langkah reward
│ ⑤ STATUS AJAKANMU (tracker)         │  Riwayat lama + ajakan tanpa reward
│ ⑥ S&K (bottom sheet trigger)        │  ← general_tnc saja
└─────────────────────────────────────┘
```

Elemen above the fold wajib terlihat tanpa scroll di device baseline.

## 7. Requirement per Skenario

### Skenario A — Program NTB Aktif (`reward_ntb`)

| Elemen | Konten | Sumber API |
|---|---|---|
| Headline | "Ajak temanmu buka rekening pertama di Qita, kamu dapat Rp25.000" | `programs[NTB].reward.display_referrer` |
| Subheadline | "Temanmu yang belum punya rekening BRI juga dapat Rp10.000" | `programs[NTB].reward.display_referee` |
| Badge periode | "Berlaku s.d. 31 Agustus 2026" | `programs[NTB].period.display` |
| Siapa yang bisa diajak | Checklist dari `referee_criteria` | `programs[NTB].referee_criteria[].display` |
| Cara kerja langkah 3 | "Buka rekening & lakukan setoran/transaksi pertama..." | `programs[NTB].qualification.display_summary` |
| Share copy | "Belum punya rekening? Buka rekening BRI pertamamu..." | `programs[NTB].share_copy.template` |
| Larangan | Tidak ada jejak program ETB | — |

### Skenario B — Program NTB + ETB Aktif (`reward_dual`)

| Elemen | Konten | Sumber API |
|---|---|---|
| Headline | "Ajak siapa saja ke Qita, dapat hingga Rp25.000 per teman" | `max(programs[].reward.referrer_amount)` |
| Siapa yang bisa diajak | Dual-card per `programs[]` | `referee_type_label` + `reward` + `referee_criteria` |
| Caption | "Nggak perlu bingung — bagikan saja kodenya, sistem kami yang menentukan reward-nya." | Statis |
| Cara kerja langkah 3 | Dua bullet, satu per program | `programs[].qualification.display_summary` |
| Share copy | Template netral tanpa nominal spesifik | `share_copy_default` atau template Procash |
| Larangan | Tidak ada pemilihan segmen sebelum share | — |

### Skenario C — Program ETB Aktif (`reward_etb`)

| Elemen | Konten | Sumber API |
|---|---|---|
| Headline | "Punya teman pengguna BRI/BRImo? Ajak mereka pakai Qita, kamu dapat Rp15.000" | `programs[ETB].reward.display_referrer` |
| Subheadline | "Temanmu cukup aktivasi Qita dengan rekening BRI yang sudah dia punya — dia juga dapat Rp5.000" | `programs[ETB].reward.display_referee` |
| Siapa yang bisa diajak | Checklist dari `referee_criteria` | `programs[ETB].referee_criteria[].display` |
| Cara kerja langkah 3 | "Aktivasi Qita dengan rekening BRI yang sudah ada..." | `programs[ETB].qualification.display_summary` |
| Share copy | "Udah punya BRImo atau rekening BRI? Cobain Qita..." | `programs[ETB].share_copy.template` |
| Ilustrasi hero | Visual "berpindah/mencoba app baru" | — |

*(Seluruh nominal adalah placeholder — nilai riil dari config Procash.)*

### Skenario D — Tidak Ada Program Aktif (`no_reward`)

| Elemen | Konten | Sumber API |
|---|---|---|
| Headline | "Ajak temanmu rasakan Qita" | Statis |
| Subheadline | "Belum ada program reward saat ini. Kode referral kamu tetap bisa dibagikan." | Statis |
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
   [programs[].qualification.display_summary]
   Reward cair maks. {reward_processing_days}×24 jam.
```

- **Single program:** satu teks di langkah 3.
- **Dual program:** dua bullet di langkah 3, masing-masing dari `programs[].qualification.display_summary`.

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

Sumber: `programs[].tnc` — satu section per program yang aktif, di-render di bawah ketentuan umum.

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
│  [Detail Program Saat Ini]          │  ← programs[].tnc
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
- **Mode tanpa reward:** tidak perlu coachmark reward; cukup penjelasan singkat di hero.
- Setelah dismiss tidak muncul lagi.

### 10.2 Transisi state sejak kunjungan terakhir

Bandingkan `ui_mode` terakhir (local) vs `ui_mode` saat ini:

| Perubahan | Perlakuan |
|---|---|
| Reward aktif → tanpa reward | Bottom sheet: "Program reward sudah berakhir. Kode kamu tetap bisa dibagikan." CTA: [Mengerti] |
| Tanpa reward → reward aktif | Bottom sheet: "Program reward baru! Ajak temanmu dan dapat {display_referrer}." CTA: [Mengerti, Bagikan Sekarang] |
| Ganti segmen (NTB ⇄ ETB) | Bottom sheet: "Program referral baru! Sekarang giliran ajak temanmu yang sudah punya rekening BRI/BRImo." |
| Hanya nominal/periode berubah | Badge "Baru" di hero, tanpa interupsi |

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
- **Progres kuota** (mode reward aktif): "Kamu sudah mengajak 12/200" — dari `programs[].quota`.

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
Match dengan programs[].referee_type yang aktif?
       │
       ├── Ya  → tampilkan janji reward + tracker syarat
       └── Tidak → onboarding normal, matched_program: null
```

## 13. Entry Points (di luar Referral Hub)

| Entry point | Mode reward aktif | Mode tanpa reward |
|---|---|---|
| Banner homepage | Copy spesifik program ("Ajak teman buka rekening, dapat Rp25rb") | **Sembunyikan** banner reward |
| Post-transaksi sukses | "Suka pakai Qita? Ajak temanmu & dapat Rp25.000" | "Kenalkan Qita ke temanmu" (tanpa nominal) |
| Push notification program launch | Deep link ke Referral Hub — ke **semua user** | Tidak dikirim |
| Menu profil | "Ajak Teman" — permanen, semua state | Sama |

## 14. Copywriting Guidelines

- Dilarang menampilkan istilah: NTB, ETB, CIF, dormant, ETB X NTB, Procash.
- Nominal selalu untuk **kedua pihak** — hanya saat mode reward aktif.
- Periode program selalu tampil di/dekat hero — hanya saat mode reward aktif.
- Share copy dari sudut pandang penerima. Tanpa reward: zero mention nominal.
- Alasan gagal di tracker: jujur, bahasa kriteria yang sama dengan "Siapa yang bisa kamu ajak".
- CTA: "Bagikan Sekarang" (ada reward) vs "Bagikan ke Teman" (tanpa reward).

## 15. Edge Cases

| Kasus | Perlakuan |
|---|---|
| Referee ETB masuk saat hanya program NTB aktif | Onboarding tanpa janji reward; tracker referrer: "tidak memenuhi kriteria program" |
| Referee NTB masuk saat hanya program ETB aktif | Idem |
| Referrer share saat reward aktif, referee daftar setelah program berakhir | Lock-in terms jika dalam grace period Procash; jika tidak, tracker: tanpa reward |
| ETB X NTB (dormant daftar ulang) | Kebijakan backend/S&K; UI: "tidak memenuhi kriteria program" |
| Kuota program habis | Kode tetap aktif; hero "Kuota program periode ini sudah penuh"; share tanpa janji nominal |
| Program berakhir saat teman di tengah funnel | Honor terms saat registrasi (lock-in) |
| Config berubah saat user di page | Refresh saat re-focus |
| User pernah dapat reward, sekarang tanpa program | Riwayat tetap tampil; hero `no_reward` |

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

1. High-fidelity design Referral Hub untuk **4 skenario** (A/B/C/D) dari satu template komponen.
2. State transisi: reward aktif ↔ tanpa reward, ganti segmen, reward cair, ajakan menggantung.
3. Cara kerja: template 3 langkah (reward) dan 2 langkah (tanpa reward) dengan slot dinamis langkah 3.
4. S&K bottom sheet: Lapisan 1 (umum) + Lapisan 2 (detail program dinamis).
5. Flow referee: deep link → deteksi `referee_type` → janji reward (match) / tanpa janji (mismatch).
6. Tracker referrer: 5 varian status + agregat + progres kuota.
7. Entry points: banner homepage, card post-transaksi, push notification (reward vs non-reward).
8. Spec komponen dinamis: teks variabel, truncation, dynamic type, transisi tanpa layout shift.
9. Spec tombol CTA: "Bagikan Sekarang" vs "Bagikan ke Teman".

## 18. Open Questions

1. Nominal & syarat kualifikasi final per program (menunggu konfigurasi Procash).
2. Kebijakan final eligibility ETB X NTB (dihitung NTB atau tidak) — menentukan deteksi `referee_type`.
3. Apakah reward cair otomatis atau perlu klaim manual? (Rekomendasi: otomatis.)
4. Batas kuota per referrer per periode dan per program.
5. Channel share: native share sheet vs shortcut khusus (WhatsApp-first?).
6. Grace period setelah program berakhir: berapa hari referee yang sudah terdaftar masih bisa memenuhi syarat?
