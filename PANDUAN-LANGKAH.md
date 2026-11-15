# PANDUAN LANGKAH DEMI LANGKAH (versi paling mudah)

Ikuti dari atas ke bawah. Jangan lompat. Total waktu ±30 menit.
Kalau ada yang macet, lihat bagian **"Kalau Nyangkut"** di paling bawah.

---

## SIAPKAN DULU (5 menit)

Anda butuh 3 akun gratis. Semuanya bisa daftar pakai tombol **"Continue with Google"**
(pakai akun Gmail yang sama biar tidak bingung).

1. https://github.com  → Sign up
2. https://supabase.com → Start your project (gratis, tidak minta kartu)
3. https://vercel.com → Sign up → pilih **Continue with GitHub**

Siapkan juga:
- Chrome
- VS Code (atau Notepad)
- Folder `jadwal-kuliah` yang sudah saya buat

---

## LANGKAH 1 — Buat "gudang data" di Supabase (10 menit)

Anggap Supabase itu lemari arsip di internet. Kita bikin lemarinya dulu.

1. Buka https://supabase.com/dashboard → klik **New project**
   - Kalau diminta bikin "organization", isi nama bebas, klik Create.
2. Isi form:
   - **Name:** `jadwal-kuliah`
   - **Database Password:** ketik password, **CATAT di HP/nota**. (Ini password
     database, bukan password login. Tidak dipakai di aplikasi, tapi jangan hilang.)
   - **Region:** pilih **Southeast Asia (Singapore)** — paling dekat Indonesia.
3. Klik **Create new project**. Tunggu 1–3 menit sampai loading selesai.
4. Di menu kiri, klik ikon **`</>` SQL Editor**.
5. Buka file `jadwal-kuliah/supabase/schema.sql` di komputer Anda:
   - Klik kanan → Open with → Notepad / VS Code
   - Tekan **Ctrl + A** (pilih semua) lalu **Ctrl + C** (copy)
6. Kembali ke SQL Editor Supabase, klik kotak putih besar, tekan **Ctrl + V** (paste).
7. Klik tombol **Run** (kanan bawah) atau tekan **Ctrl + Enter**.
8. Kalau muncul tulisan hijau **"Success. No rows returned"** → BERHASIL.
   Kalau muncul merah, copy tulisannya dan kirim ke saya.

9. Cek hasilnya: menu kiri → **Table Editor** → harus ada tabel `jadwal_kuliah`
   dengan 4 baris contoh.

### Ambil 2 kunci rumah

10. Menu kiri paling bawah → **Project Settings** (ikon gerigi) → **API**.
11. Di halaman itu ada dua barang yang kita butuh:
    - **Project URL** — bentuknya `https://abcdefgh.supabase.co` → klik ikon copy
    - **anon public** (di bagian Project API keys) — hurufnya panjang sekali,
      diawali `eyJ...` → klik copy

> PENTING: yang dipakai hanya **anon public**. Jangan pernah pakai
> `service_role` / `secret`. Yang anon memang boleh dipublikasikan, yang
> service_role itu kunci induk — kalau bocor, orang bisa hapus seluruh data.

---

## LANGKAH 2 — Tempel 2 kunci itu ke aplikasi (2 menit)

Ini supaya website Anda tahu harus menyimpan data ke lemari yang mana.

1. Buka `jadwal-kuliah/js/config.js` di VS Code / Notepad.
2. Isinya dua baris seperti ini:

   ```js
   var SUPABASE_URL = "https://xxxxxxxxxxxx.supabase.co";
   var SUPABASE_ANON_KEY = "isi-dengan-anon-key-panjang-anda-di-sini";
   ```

3. Ganti bagian dalam tanda kutip (JANGAN hapus tanda kutip `"` nya):
   - Ganti `https://xxxxxxxxxxxx.supabase.co` dengan Project URL tadi.
   - Ganti `isi-dengan-anon-key-panjang-anda-di-sini` dengan anon key tadi.
4. Hasil akhirnya kira-kira:

   ```js
   var SUPABASE_URL = "https://abcdefgh.supabase.co";
   var SUPABASE_ANON_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.panjang....";
   ```

5. Simpan: **Ctrl + S**.

---

## LANGKAH 3 — Coba di komputer sendiri (3 menit)

Sebelum online, pastikan sudah jalan.

1. Buka folder `jadwal-kuliah`, klik dua kali **index.html** → terbuka di Chrome.
2. Tanda berhasil:
   - Banner kuning "Mode demo" **tidak muncul** (kalau masih muncul, berarti
     config.js salah — ulangi Langkah 2)
   - Di pojok kanan atas ada badge hijau "Tersambung"
   - Tabel sudah berisi 4 jadwal contoh
3. Tes cepat: isi form (misal Mata Kuliah `Jaringan Komputer`, SKS `2`,
   jam 08:00–09:40) → klik **Simpan Jadwal**.
4. Buka Supabase → Table Editor → `jadwal_kuliah` → klik ikon **refresh**.
   Baris baru harus muncul. Kalau muncul → aplikasi Anda SUDAH HIDUP. 🎉

---

## LANGKAH 4 — Taruh kode di GitHub (5 menit, tanpa terminal)

GitHub itu seperti Google Drive untuk kode. Panitia minta link repo publik.

1. Buka https://github.com → klik **+** kanan atas → **New repository**.
2. Isi:
   - **Repository name:** `jadwal-kuliah`
   - Pilih **Public** (WAJIB, kalau private panitia tidak bisa buka)
   - Jangan centang apa pun (tanpa README, tanpa .gitignore)
3. Klik **Create repository**.
4. Di halaman berikutnya klik tulisan **"uploading an existing file"**.
5. Buka folder `jadwal-kuliah` di komputer Anda, lalu **masuk ke dalamnya** dan
   pilih **ISI** foldernya (bukan foldernya):
   - `index.html`
   - `PRD.md`
   - `RULES.md`
   - `README.md`
   - `LAPORAN-DAY4.md`
   - `.gitignore`
   - folder `js`
   - folder `supabase`
6. Seret semuanya ke kotak upload di browser. Tunggu semua nama file muncul.
7. Klik tombol hijau **Commit changes**.
8. **CEK PENTING:** `index.html` harus berada di HALAMAN DEPAN repo, sejajar
   dengan folder `js`. Kalau yang muncul malah satu folder `jadwal-kuliah`,
   klik folder itu → masuk → lalu ulangi upload dari dalam (atau pakai tombol
   `..` untuk naik). Kalau `index.html` terkubur satu folder, Vercel akan error 404.
9. Copy link repo-nya, bentuknya:
   `https://github.com/USERNAME/jadwal-kuliah`

---

## LANGKAH 5 — Onlinekan di Vercel (3 menit)

1. Buka https://vercel.com → **Continue with GitHub** (biar repo tadi otomatis terlihat).
2. Klik **Add New…** → **Project**.
3. Di daftar repo, cari `jadwal-kuliah` → klik **Import**.
   (Kalau tidak muncul, klik "Adjust GitHub App Permissions" lalu izinkan repo itu.)
4. Di halaman konfigurasi **ubah 3 hal saja**:
   - **Framework Preset:** pilih **Other**
   - **Build Command:** kosongkan (hapus isinya)
   - **Output Directory:** isi `.` (titik satu)
   - Root Directory: biarkan kosong
5. Klik **Deploy**. Tunggu ±30 detik sampai muncul confetti.
6. Klik **Visit** → website Anda terbuka di internet.
   Link-nya bentuknya `https://jadwal-kuliah-xxxx.vercel.app`
7. Copy link itu.

Kalau di halaman web muncul **404: NOT_FOUND**, artinya `index.html` bukan di
halaman depan repo → ulangi poin 8 Langkah 4, lalu di Vercel klik
**Deployments → …(titik tiga) → Redeploy**.

---

## LANGKAH 6 — Ambil 6 screenshot bukti (5 menit)

Buka link Vercel Anda di Chrome. Tekan **F12** untuk buka DevTools.
Ambil screenshot dengan **Windows + Shift + S**.
Simpan semua ke folder `bukti/` dengan nama `gambar1.png` … `gambar6.png`.

**Gambar 1 — Tampilan utama**
Tutup DevTools (F12 lagi) → screenshot seluruh halaman: kartu statistik + tabel.

**Gambar 2 — TC-01 (SKS tidak valid)**
Klik di kolom SKS → ketik `-2` → klik **Simpan Jadwal**.
Akan muncul kotak merah "SKS harus lebih dari 0". Screenshot pesannya.

**Gambar 3 — TC-02 (berhasil simpan, bukti ke server)**
Tekan F12 → klik tab **Network** → centang jangan, biarkan kosong.
Isi form lengkap: `Jaringan Komputer`, dosen bebas, SKS `2`, jam 08:00–09:40 →
klik **Simpan Jadwal**.
Di daftar Network akan muncul baris `jadwal_kuliah` → klik baris itu →
klik tab **Headers** → screenshot yang memperlihatkan **Status Code: 201**.
Ini bukti panitia bahwa data benar-benar masuk server, bukan cuma tampil di layar.

**Gambar 4 — TC-05 (anti-XSS)**
Isi Mata Kuliah dengan: `<img src=x onerror=alert('HACKED')>` → Simpan.
Kalau AMAN: di tabel tulisan itu tampil sebagai teks biasa dan **tidak ada
popup alert**. Screenshot tabelnya sekalian tab Console.
Kalau sampai muncul alert "(HACKED)" → langsung kabari saya, ada bug.

**Gambar 5 — Console bersih**
F12 → tab **Console** → screenshot. Harus tidak ada tulisan merah.

**Gambar 6 — Bukti di Supabase**
Buka Supabase → Table Editor → `jadwal_kuliah` → screenshot semua baris datanya.

---

## LANGKAH 7 — Isi laporan (3 menit)

1. Buka `LAPORAN-DAY4.md` (bisa dibuka di VS Code atau di GitHub).
2. Ganti semua tulisan dalam kurung siku:
   - `[isi nama]`, `[isi NIM]`, `[isi]`
   - `[tempel link repo publik]` → link GitHub dari Langkah 4
   - `[tempel link *.vercel.app]` → link Vercel dari Langkah 5
3. Di Bab 7, ganti `[ ] PASS` menjadi `PASS` setelah testing Anda berhasil.
4. Di Bab 8, tempel 6 gambar tadi di bawah tabelnya.
5. Copy semua isinya ke Word/Google Docs, rapikan, export PDF.

---

## YANG DISERAHKAN KE PANITIA

1. Link repo GitHub **publik** → `https://github.com/USERNAME/jadwal-kuliah`
2. Link live → `https://jadwal-kuliah-xxxx.vercel.app`
3. Screenshot TC-01 dan TC-02 (Gambar 2 dan Gambar 3)

---

## Kalau Nyangkut

| Gejala | Penyebab | Obat |
|---|---|---|
| Banner kuning masih muncul | config.js belum diisi / salah ketik | Ulangi Langkah 2, pastikan tanda kutip tidak hilang, simpan Ctrl+S, refresh dengan Ctrl+F5 |
| Muncul "Gagal memuat data" | schema.sql belum dijalankan | Ulangi Langkah 1 poin 5–8 |
| `new row violates row-level security policy` | policy RLS belum ada | Jalankan ulang seluruh isi schema.sql (aman diulang) |
| Tombol Simpan tidak bereaksi | ada error JS | F12 → Console → kirim tulisan merahnya ke saya |
| Vercel 404 NOT_FOUND | index.html terkubur di subfolder | Poin 8 Langkah 4 |
| Vercel build failed | Build Command terisi | Kosongkan Build Command, Deploy ulang |
| Data hilang setelah refresh | config.js salah / masih mode demo | Cek badge "Tersambung" |
| Lupa database password Supabase | — | Tidak masalah, aplikasi tidak memakainya. Bisa reset di Settings → Database |
