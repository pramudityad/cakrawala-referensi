# Tambahan (tidak dinilai) — empat stasiun gallery walk dan cara mengklasifikasi satu keputusan

Dokumen aslinya adalah dokumentasi frontend admin produksi (React 18 + TypeScript + Vite + Tailwind,
27 KB). Yang diproyeksikan di kelas hanya empat halaman — itu yang diringkas di bawah. **Bacalah sebagai
pseudo-code**: yang dicari adalah *klasifikasi keputusan*, bukan sintaks React.

## Stasiun 1 — Service Overview

Satu halaman ikhtisar: aplikasi apa ini, dan tumpukan teknologinya.

| Keputusan | Isi |
| :--- | :--- |
| Bahasa | TypeScript, strict mode, path alias |
| Build | Vite |
| Styling | Tailwind CSS |
| State | Jotai (primer) + React Query (server state) + Redux (auth saja) |
| Form | React Hook Form + Zod |
| Testing | Vitest + Cypress |
| Dokumentasi | satu `CLAUDE.md` di root |

**Yang bisa diklasifikasi** — TypeScript strict sejak awal · Vite vs tooling lain · Tailwind vs CSS
biasa · Vitest **dan** Cypress sekaligus · dokumentasi arsitektur ditulis sebagai file di repo.

## Stasiun 2 — State Management

Arsitektur "triple-hybrid" — tiga pendekatan state dalam satu aplikasi, dengan pembagian tugas jelas:

| Lapisan | Untuk apa | Catatan dokumen |
| :--- | :--- | :--- |
| Redux Toolkit | auth saja (JWT) | **legacy — hindari untuk fitur baru**, sedang dimigrasi keluar |
| Jotai (+ bunshi) | state aplikasi primer | pola *molecule* per halaman, mis. `atoms/dish.ts` bersama `pages/dishes/` |
| React Query | server state | `atomWithQuery` untuk baca, `atomWithMutation` untuk tulis |

Aturan tulis dokumen: **jangan** pakai Redux untuk fitur baru · **jangan** menaruh state tak berhubungan
dalam satu molecule · **jangan** buat atom global tanpa molecule.

**Yang bisa diklasifikasi** — tiga solusi state dalam satu app (dokumen menyebut ini normal) · pola
molecule (state dikelompokkan per halaman, bukan per tipe) · `atomWithQuery` sebagai jembatan Jotai↔React
Query · keputusan memigrasi keluar dari Redux secara bertahap · satu atom tanpa molecule dilarang.

## Stasiun 3 — Forms & Validation

Satu pola: **React Hook Form + Zod**, dengan resolver dari skema. Validasi didefinisikan sekali sebagai
skema, dipakai sebagai resolver dan sebagai sumber tipe. Ada juga *custom form fields* — komponen field
yang sudah terintegrasi RHF.

**Yang bisa diklasifikasi** — validasi sebagai skema (bukan `if` di handler) · tipe diturunkan dari skema,
bukan ditulis dua kali · field dibungkus komponen sendiri · pesan error per-field otomatis.

## Stasiun 4 — Component System

Dua generasi komponen hidup berdampingan:

| | V3 — dipakai | Legacy — dihindari |
| :--- | :--- | :--- |
| Lokasi | `/src/ui/new/` (alias `@v3/components`) | `/src/components/`, `@v2/components` |
| Isi | 40+ elemen: `FormTextInput`, `FormSelect`, `DataTable`, `Modal`, `Drawer`, `Toast`, `Badge`, `Tag`, `SROnly`, … | versi lama |
| Status | migrasi ke sini | sedang ditinggalkan |

**Yang bisa diklasifikasi** — membangun pustaka komponen internal (bukan memakai UI kit publik) ·
memisahkan komponen form (terikat RHF) dari komponen tampilan · `DataTable` dengan sort/pagination
bawaan · aksesibilitas disiapkan sebagai komponen (`SROnly`) · membiarkan dua generasi komponen
berdampingan selama migrasi.

---

# Cara memberi sticky

Ambil **satu** keputusan dari **satu** stasiun. Lalu tulis:

```
Stasiun: ....
Keputusan: ....
Sticky: 🟢 kita lakukan | 🟡 kita tunda | 🔴 kita tolak
Alasan: .... (ikat ke kendala timmu)
```

Alasanmu harus menyebut kendala nyata tim: **satu sesi per pekan**, lab **35 menit**, **tidak ada PR**,
dan kamu memakai **Vue, bukan React**. Contoh bentuk alasan:

- 🟢 **kita lakukan — Sesi 12.** State dikelompokkan per halaman seperti pola *molecule*. Di Sesi 12 kita
  sudah pakai composable per fitur; memindahkan `useItems()` ke `composables/` adalah langkah kecil dan
  tidak menambah pustaka baru.
- 🟡 **kita tunda.** Validasi berbasis skema (satu definisi → resolver + tipe). Butuh pustaka validasi;
  ditunda sampai ada form yang benar-benar dikerjakan bareng, bukan form demo satu field.
- 🔴 **kita tolak.** Tiga pustaka state sekaligus. Dengan scope satu semester dan lab 35 menit, memilih
  satu cara sudah cukup; risiko yang diterima: saat app membesar nanti, pemisahan server-state vs
  app-state harus dipikirkan dari awal — dan itu bisa jadi utang.

Perhatikan contoh di atas **tidak** menilai keputusan itu bagus atau buruk untuk perusahaan asalnya.
Yang dinilai: apakah alasanmu masuk akal **untuk timmu sendiri**, sekarang, dengan waktu yang kamu punya.

## Yang kamu kumpulkan

Satu keputusan → satu sticky → alasan yang mengikat ke kendala tim. Sebut **stasiun nomor berapa** dan
**keputusan apa** (mis. "pattern molecule", "validasi sebagai skema", "40+ komponen V3"). Tidak perlu
menyalin isi dokumen.
