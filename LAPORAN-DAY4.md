# LAPORAN PRAKTIKUM — PUNCAK WORKSHOP DAY 4
## Spec-Driven Vibe Coding & Secure Engineering

**Nama:** `[isi nama]`
**NIM:** `[isi NIM]`
**Kelas / Prodi:** `[isi]`
**Politeknik Negeri Cilacap**

**Judul Proyek:** Sistem Data Jadwal Kuliah (SKPL_05)
**Repositori GitHub:** `[tempel link repo publik]`
**Live Vercel:** `[tempel link *.vercel.app]`
**Tanggal Pengerjaan:** `[isi tanggal]`

> Catatan pemakaian: bagian bertanda `[...]` dan tabel bukti di Bab 8 wajib Anda isi
> sendiri setelah deploy. Teks lain boleh dipakai langsung.

---

## BAB 1 — Latar Belakang

Pengelolaan jadwal kuliah di tingkat jurusan umumnya masih memakai spreadsheet yang
tersebar di beberapa berkas, sehingga rawan data ganda dan sulit dicek bentroknya.
Dokumen SKPL_05 mendefinisikan kebutuhan sistem pencatatan jadwal kuliah berbasis web
dengan empat kebutuhan fungsional utama.

Praktikum Day 4 menguji pendekatan **Spec-Driven Vibe Coding**: dokumen SKPL diubah
menjadi berkas spesifikasi (`PRD.md` dan `RULES.md`) yang menjadi pagar kerja AI, bukan
sekadar prompt bebas. Hasil yang diukur bukan hanya "aplikasi jadi", tetapi aplikasi
yang **lolos test case panitia**, **tahan XSS**, dan **live di Vercel**.

## BAB 2 — Tujuan

1. Menghasilkan aplikasi CRUD jadwal kuliah sesuai REQ-01 s.d. REQ-04 pada SKPL_05.
2. Menerapkan tiga lapis pertahanan keamanan (frontend, API/RLS, database engine).
3. Membuktikan hasil uji TC-01 s.d. TC-06 melalui DevTools.
4. Mendeploy aplikasi ke Vercel dan menyimpan kode di repositori GitHub publik.

## BAB 3 — Alat dan Bahan

| Kategori | Alat |
|---|---|
| Editor | Visual Studio Code |
| UI | HTML5, Tailwind CSS (CDN) |
| Logika | Vanilla JavaScript (ES6) |
| Database | Supabase (PostgreSQL) |
| Versi kode | Git + GitHub |
| Hosting | Vercel |
| Pengujian | Google Chrome DevTools (F12) |
| Desain | Google Stitch / Excalidraw (wireframe) |

## BAB 4 — Metode Pengerjaan

Alur enam langkah sesuai roadmap workshop (120 menit):

| Tahap | Menit | Keluaran |
|---|---|---|
| 1. Ekstraksi SKPL | 00–15 | `PRD.md` |
| 2. Integrasi UI | 15–35 | `index.html` |
| 3. Pagar keamanan | 35–45 | `RULES.md` |
| 4. Database Supabase | 45–75 | `supabase/schema.sql`, CRUD aktif |
| 5. DevTools & uji XSS | 75–105 | bukti TC-01…TC-06 |
| 6. Deploy Vercel | 105–120 | URL `*.vercel.app` |

## BAB 5 — Perancangan

### 5.1 Kebutuhan Fungsional

| Kode | Kebutuhan | Implementasi |
|---|---|---|
| REQ-01 | Catat jadwal baru | Form 7 kolom + validasi + `INSERT` |
| REQ-02 | Ringkasan statistik | 3 kartu: total jadwal, total SKS, hari terpadat |
| REQ-03 | Filter & pencarian | Dropdown hari + kotak cari (matkul/dosen/ruang) |
| REQ-04 | Edit & hapus | Tombol Edit (mode ubah) + Hapus (konfirmasi) |

### 5.2 Rancangan Database

Tabel `jadwal_kuliah`:

| Kolom | Tipe | Aturan |
|---|---|---|
| id | BIGINT identity | Primary key |
| mata_kuliah | TEXT | NOT NULL, tidak boleh kosong |
| dosen | TEXT | NOT NULL |
| hari | TEXT | CHECK salah satu dari 6 hari |
| jam_mulai | TIME | NOT NULL |
| jam_selesai | TIME | NOT NULL, `> jam_mulai` |
| ruang | TEXT | NOT NULL |
| sks | SMALLINT | `CHECK (sks > 0 AND sks <= 6)` |
| created_at | TIMESTAMPTZ | default `NOW()` |

### 5.3 Arsitektur Pertahanan (Defense in Depth)

1. **Layer 1 — Frontend:** JavaScript menolak SKS ≤ 0 dan jam terbalik sebelum request dikirim.
2. **Layer 2 — RLS Policy:** anon key hanya boleh operasi CRUD dengan syarat `sks > 0`.
3. **Layer 3 — Database:** `CHECK` constraint + trigger `cegah_bentrok_jadwal()` menolak
   data ilegal walau API ditembak langsung memakai `curl`.

## BAB 6 — Implementasi

### 6.1 Validasi Klien (kunci TC-01)

```js
var sks = Number(data.sks);
if (!Number.isFinite(sks) || !Number.isInteger(sks) || sks <= 0) {
  return { ok: false, pesan: "SKS harus lebih dari 0" };
}
if (data.jam_selesai <= data.jam_mulai) {
  return { ok: false, pesan: "Jam selesai harus lebih besar dari jam mulai" };
}
```

### 6.2 Anti-XSS (kunci TC-05)

```js
function sel(text, kelas) {
  var td = document.createElement("td");
  td.className = kelas;
  td.textContent = text;   // WAJIB; .innerHTML dilarang oleh RULES.md
  return td;
}
```

### 6.3 Kunci Aturan di Database (kunci TC-01 s.d. TC-04)

```sql
sks SMALLINT NOT NULL CHECK (sks > 0 AND sks <= 6),
CONSTRAINT jam_valid CHECK (jam_selesai > jam_mulai)
```

## BAB 7 — Hasil Pengujian

| ID | Skenario | Input | Hasil yang Diharapkan | Hasil |
|---|---|---|---|---|
| TC-01 | Tolak SKS tidak valid | `sks = -2` | Pesan "SKS harus lebih dari 0", tanpa request POST | `[ ] PASS` |
| TC-02 | Tambah jadwal valid | 08:00–09:40, SKS 2 | `POST … 201 Created`, Total Jadwal +1, SKS +2 | `[ ] PASS` |
| TC-03 | Tolak jam terbalik | 10:00 → 09:00 | "Jam selesai harus lebih besar dari jam mulai" | `[ ] PASS` |
| TC-04 | Tolak bentrok ruang | Ruang & hari sama | "Jadwal bentrok di ruang …" | `[ ] PASS` |
| TC-05 | Anti-XSS | `<img src=x onerror=alert('HACKED')>` | Tampil sebagai teks, tanpa alert | `[ ] PASS` |
| TC-06 | Edit & hapus | SKS 2 → 3, lalu hapus | Statistik berubah, baris hilang | `[ ] PASS` |

## BAB 8 — Bukti Screenshot (WAJIB DIISI)

Ambil tangkapan layar berikut, simpan di folder `bukti/`, lalu tempel di laporan:

| Gambar | Yang harus terlihat | Cara mengambil |
|---|---|---|
| Gambar 1 | Dashboard kosong/berisi, 3 kartu statistik + tabel | Buka URL Vercel, screenshot penuh |
| Gambar 2 | TC-01 gagal validasi | Isi SKS `-2` → Simpan → screenshot pesan merah |
| Gambar 3 | TC-02 sukses + Network 201 | F12 → Network → Simpan jadwal valid → screenshot baris `POST /rest/v1/jadwal_kuliah 201` |
| Gambar 4 | TC-05 anti-XSS | Simpan mata kuliah `<img src=x onerror=alert('HACKED')>` → screenshot tabel (teks biasa, tanpa alert) + Console bersih |
| Gambar 5 | Konsol bersih | F12 → Console → screenshot tanpa error merah |
| Gambar 6 | Bukti tabel Supabase | Supabase Studio → Table Editor → `jadwal_kuliah` → screenshot isi baris |

## BAB 9 — Deployment

```bash
git init
git add .
git status            # pastikan tidak ada .env atau token
git commit -m "feat: sistem data jadwal kuliah (SKPL_05)"
git branch -M main
git remote add origin https://github.com/<username>/jadwal-kuliah.git
git push -u origin main
```

Vercel: **Add New → Project → import repo** → Framework Preset *Other*, Build Command
kosong, Output Directory `.` → **Deploy**. Hasil: `https://[nama]-[hash].vercel.app`.

## BAB 10 — Kendala dan Solusi

| Kendala | Solusi |
|---|---|
| AI cenderung memakai `.innerHTML` | Dilarang eksplisit di `RULES.md`; render memakai `createElement` + `textContent` |
| Validasi frontend bisa dilewati | Ditambah `CHECK` constraint + RLS policy di PostgreSQL |
| Jadwal ganda di ruang yang sama | Trigger `cegah_bentrok_jadwal()` menolak rentang jam yang beririsan |
| Kredensial bocor ke repo | `.gitignore` memuat `.env*`, hanya anon key yang dipublikasikan |
| Tailwind via CDN butuh internet | Diterima: aplikasi demo tugas, tanpa build step |

## BAB 11 — Kesimpulan

1. SKPL_05 berhasil diimplementasikan menjadi aplikasi CRUD jadwal kuliah dengan
   empat kebutuhan fungsional dan enam test case.
2. Pola *spec-driven* (`PRD.md` + `RULES.md`) membuat keluaran AI dapat diverifikasi,
   bukan sekadar "jalan".
3. Keamanan tidak bergantung pada satu lapis: validasi klien, RLS, dan constraint
   database saling menutup celah.
4. Aplikasi live di Vercel dan kode terbuka di GitHub sebagai portofolio.

## Lampiran

- `PRD.md` — spesifikasi kebutuhan
- `RULES.md` — pagar keamanan
- `supabase/schema.sql` — skema database
- `js/app.js` — logika aplikasi
- Link repo: `[isi]` · Link live: `[isi]`
