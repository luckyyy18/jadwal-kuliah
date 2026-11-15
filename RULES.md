# RULES — Pagar Keamanan Proyek

Aturan ini **mengikat** setiap perubahan kode. Bila sebuah permintaan bertentangan
dengan berkas ini, permintaan ditolak.

## 1. Anti-XSS Policy (Wajib)

- **Dilarang** memakai `.innerHTML`, `.outerHTML`, `insertAdjacentHTML`, atau
  `document.write` untuk menampilkan data dari database/pengguna.
- **Wajib** memakai `.textContent` (atau `createElement` + `appendChild`).

```js
// BENAR
td.textContent = jadwal.mata_kuliah;

// SALAH — jalur masuk XSS
td.innerHTML = jadwal.mata_kuliah;
```

## 2. Input Validation (TC-01)

Validasi klien **sebelum** payload dikirim, minimal:

```js
const sks = Number(form.sks);
if (!Number.isInteger(sks) || sks <= 0) return "SKS harus lebih dari 0";
if (form.jam_selesai <= form.jam_mulai) return "Jam selesai harus lebih besar dari jam mulai";
```

Validasi klien bukan pengganti validasi server: constraint `CHECK` di PostgreSQL
tetap wajib ada (defense in depth).

## 3. Isolasi Kredensial

- Hanya **anon/publishable key** yang boleh ada di frontend (`js/config.js`).
- **Dilarang** menaruh `service_role` key, password database, atau connection string.
- `js/config.js` adalah satu-satunya berkas yang berisi kredensial publik.

## 4. Git Guardrail

- `.gitignore` wajib memuat `.env`, `.env.*`, `node_modules/`, `.vscode/mcp.json`.
- Sebelum `git push`, jalankan `git status` dan pastikan tidak ada berkas rahasia.
- Bila sebuah token pernah ter-commit: **rotate** token itu, jangan sekadar hapus file.

## 5. Gaya Kode

- Vanilla JS, tanpa framework, tanpa build step (harus jalan sebagai file statis).
- Satu tanggung jawab per fungsi; nama fungsi & kolom berbahasa Indonesia agar
  selaras dengan dokumen SKPL.
- Tidak ada `console.log` sisa debugging di rilis.
