# JadwalKuliah — Sistem Data Jadwal Kuliah (SKPL_05)

Aplikasi web full-stack statis untuk mencatat, memfilter, mengubah, dan menghapus
jadwal kuliah. Dibangun mengikuti alur **Spec-Driven Vibe Coding** workshop Day 4:
`PRD.md` → UI → `RULES.md` → Supabase → uji test case → Vercel.

| Bagian | Teknologi |
|---|---|
| UI | HTML5 + Tailwind CSS (CDN) |
| Logika | Vanilla JavaScript (ES6) |
| Database | Supabase — tabel `jadwal_kuliah` |
| Hosting | Vercel (tanpa build step) |

## Struktur Berkas

```
jadwal-kuliah/
├── index.html          # UI dashboard (form + statistik + riwayat)
├── js/
│   ├── config.js       # SUPABASE_URL + anon key (satu-satunya kredensial)
│   └── app.js          # CRUD, validasi, anti-XSS render
├── supabase/
│   └── schema.sql      # tabel, index, RLS policy, trigger anti-bentrok
├── PRD.md              # spesifikasi fitur REQ-01..REQ-04 + test case
├── RULES.md            # pagar keamanan (anti-XSS, validasi, git)
└── .gitignore
```

## 1. Siapkan Supabase (5 menit)

1. Buka <https://supabase.com> → **New project** (region Singapore, gratis).
2. Setelah project siap, masuk **SQL Editor** → tempel seluruh isi
   `supabase/schema.sql` → **Run**. Harus muncul `Success. No rows returned`.
3. Buka **Project Settings → API**, salin:
   - **Project URL** → `https://xxxx.supabase.co`
   - **anon public** key (JANGAN `service_role`)
4. Buka `js/config.js`, ganti dua nilai itu.

> Tanpa konfigurasi, aplikasi tetap jalan dalam **mode demo** (localStorage) dan
> menampilkan banner kuning. Data belum masuk ke Supabase.

## 2. Jalankan Lokal

Karena memakai CDN, cukup buka `index.html` di browser. Jika ingin server lokal:

```bash
# pilih salah satu
python -m http.server 5500
npx serve .
```

Lalu buka <http://localhost:5500>.

## 3. Uji Test Case (DevTools F12)

| ID | Aksi | Ekspektasi |
|---|---|---|
| TC-01 | Isi SKS `-2` lalu simpan | Merah: “SKS harus lebih dari 0”, **tidak ada** request POST di Network |
| TC-02 | Simpan 08:00–09:40, SKS 2 | `POST /rest/v1/jadwal_kuliah 201`, Total Jadwal +1, Total SKS +2 |
| TC-03 | Jam mulai 10:00, selesai 09:00 | “Jam selesai harus lebih besar dari jam mulai” |
| TC-04 | Ruang & hari sama, jam beririsan | “Jadwal bentrok di ruang …” |
| TC-05 | Mata kuliah `<img src=x onerror=alert('HACKED')>` | Tampil sebagai teks, **tidak ada** alert |
| TC-06 | Edit SKS lalu hapus | Statistik berubah, baris hilang |

## 4. Deploy ke Vercel

```bash
git init
git add .
git status          # pastikan tidak ada .env / token
git commit -m "feat: sistem data jadwal kuliah (SKPL_05)"
git branch -M main
git remote add origin https://github.com/<username>/jadwal-kuliah.git
git push -u origin main
```

1. Buka <https://vercel.com> → **Add New → Project** → import repo tadi.
2. Framework Preset: **Other**, Build Command: kosong, Output Directory: `.`
3. **Deploy** → dapat URL `https://jadwal-kuliah-xxxx.vercel.app`.

Setiap `git push` berikutnya otomatis ter-deploy ulang.

## Catatan Keamanan

- Hanya **anon key** yang dipublikasikan; `service_role` dilarang di frontend.
- RLS aktif dengan policy `CRUD Publik` **plus** `CHECK` di PostgreSQL, jadi
  `curl` langsung ke API tetap ditolak bila SKS ≤ 0 atau jam terbalik.
- Semua render data memakai `textContent` (`RULES.md` §1) sehingga XSS tidak eksekusi.
