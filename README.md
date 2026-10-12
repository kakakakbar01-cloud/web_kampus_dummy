# SIAKAD Lembah Cahaya

Portal kampus **fiktif** untuk latihan keamanan web. Seluruh akun dan data bersifat sintetis dan disimpan di `localStorage` browser (tanpa backend), sehingga bisa dihosting di GitHub Pages.

> Aplikasi ini sengaja dibuat rentan. Jangan dipakai dengan data asli.

## Fitur

| Peran | Kemampuan |
|---|---|
| Mahasiswa | Masuk, melihat nilai, mengunggah tugas, berdiskusi |
| Dosen | Memberi nilai, melihat tugas masuk, berdiskusi |
| Admin | Mengelola akun (tambah, reset sandi, hapus), berdiskusi |

## Akun demo (sintetis)

| Username | Sandi | Peran |
|---|---|---|
| admin.cahaya | Admin#Cahaya2026 | admin |
| dosen.laras | Dosen#Laras26 | dosen |
| dosen.hendra | Dosen#Hendra26 | dosen |
| mhs.nadira | Mhs#Nadira26 | mahasiswa |
| mhs.farel | Mhs#Farel26 | mahasiswa |
| mhs.salsa | Mhs#Salsa26 | mahasiswa |

## Cara deploy ke GitHub Pages

1. Upload `index.html` dan `README.md` ke repo baru (Public).
2. Buka **Settings → Pages**.
3. Pada *Build and deployment*, pilih **Deploy from a branch**, branch `main`, folder `/ (root)`, lalu **Save**.
4. Tunggu 1-2 menit. URL situs: `https://<username>.github.io/<nama-repo>/`

## Kerentanan yang sengaja dibuat

### 1. Percobaan sandi tanpa pembatasan
- **Lokasi:** fungsi `doLogin()` di `index.html` (komentar `[KERENTANAN #1]`).
- **Masalah:** tidak ada batas percobaan gagal, penundaan, CAPTCHA, atau penguncian akun.
- **Cara menguji:** coba login dengan sandi salah berkali-kali (misalnya 50 kali) pada username yang sama. Tidak ada pemblokiran, dan percobaan berikutnya dengan sandi benar tetap berhasil.
- **Perbaikan:** hitung percobaan gagal per akun/IP, beri jeda yang makin lama (backoff), kunci sementara akun, dan tambahkan CAPTCHA.

### 2. Sesi tetap aktif setelah logout atau pergantian sandi
- **Lokasi:** fungsi `logout()` dan `forms.chpass()` (komentar `[KERENTANAN #2]`).
- **Masalah:** saat logout, token hanya dihapus dari browser, tetapi sesi di `db.sessions` tidak dibatalkan. Mengganti sandi juga tidak mencabut sesi yang sudah ada.
- **Cara menguji:**
  1. Login, buka tab **Profil**, salin token sesi.
  2. Klik **Keluar**.
  3. Buka DevTools → Application → Local Storage, buat key `kp_token` berisi token tadi.
  4. Muat ulang halaman. Anda langsung masuk lagi tanpa sandi.
  5. Ulangi dengan mengganti sandi di tab Profil: token lama tetap valid.
- **Perbaikan:** hapus sesi di sisi server saat logout, cabut semua sesi saat sandi diganti, beri masa kedaluwarsa pada token.

## Catatan

- Karena statis, data tersimpan per browser. Untuk mengulang dari awal, hapus key `kp_db_v1` dan `kp_token` di Local Storage.
- Teks yang tampil (komentar diskusi, nama, judul tugas) sudah di-escape, sehingga **tidak** ada XSS. Hanya dua kerentanan di atas yang diklaim.
- Unggah tugas hanya menyimpan metadata berkas (nama dan ukuran), bukan isinya.
