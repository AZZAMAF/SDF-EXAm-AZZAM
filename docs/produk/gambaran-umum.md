# Gambaran Umum Sistem PerpusKu

Dokumen ini merangkum produk PerpusKu secara menyeluruh: deskripsi, cara kerja teknis, alur aplikasi, daftar fitur, tujuan, serta temuan dan keputusan yang diambil saat penyusunan dokumentasi.

## Daftar Isi

1. [Deskripsi Produk](#1-deskripsi-produk)
2. [Cara Kerja Teknis](#2-cara-kerja-teknis)
3. [Alur Aplikasi](#3-alur-aplikasi)
4. [Daftar Fitur](#4-daftar-fitur)
5. [Tujuan Produk](#5-tujuan-produk)
6. [Catatan Temuan dan Keputusan](#6-catatan-temuan-dan-keputusan)

---

## 1. Deskripsi Produk

### 1.1 Tentang PerpusKu

PerpusKu adalah sistem peminjaman buku untuk perpustakaan kampus. Sebelum PerpusKu ada, semua peminjaman dicatat tangan di buku besar. Akibatnya denda sering salah hitung dan petugas sulit melacak buku mana yang sedang dipinjam oleh siapa. Sekarang seluruh pencatatan dilakukan di satu sistem, dan PerpusKu sudah berjalan penuh di perpustakaan pusat.

### 1.2 Pengguna Sistem

Sistem ini hanya mengenal tiga aktor baku: **Anggota**, **Pustakawan**, dan **Admin**.

| Aktor | Siapa | Tugas Utama |
|---|---|---|
| **Anggota** | Mahasiswa dan dosen | Mencari buku, meminjam, memperpanjang, memesan (reservasi) buku yang sedang habis, dan melihat daftar pinjamannya sendiri. |
| **Pustakawan** | Petugas di meja layanan | Memproses pengembalian buku, mencatat pembayaran denda, mengelola data katalog buku, dan menangani reservasi yang tidak diambil. |
| **Admin** | Pengelola sistem | Melihat laporan bulanan dan dapat membekukan atau mengaktifkan kembali akun anggota. |

### 1.3 Cara Pemakaian Secara Umum

1. Anggota mendaftar satu kali memakai NIM (mahasiswa) atau NIP (dosen).
2. Anggota login, lalu mencari buku di katalog.
3. Anggota meminjam secara mandiri di kios layanan mandiri atau lewat ponsel saat berada di perpustakaan, dengan memindai kode yang ada pada buku.
4. Pengembalian buku dilakukan di meja layanan dan diproses oleh Pustakawan.

### 1.4 Aturan Peminjaman

| Aturan | Ketentuan |
|---|---|
| **Masa pinjam** | **7 hari** sejak tanggal pinjam. |
| **Kuota pinjam** | **3 buku** sekaligus untuk satu Anggota. |
| **Perpanjangan** | Maksimal 1 kali per peminjaman, menambah 7 hari. Hanya diizinkan jika buku tidak sedang ditunggu Anggota lain dan Anggota tidak memiliki denda. |
| **Denda keterlambatan** | Rp1.000 per hari per buku, dihitung mulai hari setelah jatuh tempo sampai buku dikembalikan. |
| **Pembayaran denda** | **Wajib tunai di meja Pustakawan.** Tidak ada pembayaran online. |
| **Reservasi** | Hanya untuk buku yang semua eksemplarnya sedang dipinjam. Antrean mengikuti urutan reservasi. Setelah buku dikembalikan, buku ditahan selama 2 hari untuk Anggota pertama di antrean. |
| **Keanggotaan** | Anggota harus berstatus aktif untuk meminjam, memperpanjang, dan reservasi. Keanggotaan aktif berarti masa berlakunya belum habis dan akunnya tidak sedang dibekukan Admin. |

> **Catatan inkonsistensi:** dokumen mentah `03-flow-aplikasi.txt` menyebut kuota 5 buku dan masa pinjam 14 hari. Nilai baku yang dipakai dokumentasi ini adalah **3 buku** dan **7 hari**, sesuai aturan bisnis produk. Lihat [Catatan Temuan](#6-catatan-temuan-dan-keputusan).

---

## 2. Cara Kerja Teknis

### 2.1 Bentuk Sistem

PerpusKu adalah aplikasi web. Anggota membukanya lewat browser di komputer perpustakaan, kios layanan mandiri, atau ponsel. Sistem terdiri dari tiga bagian yang bekerja berurutan:

```
[Browser]  -->  [Server aplikasi]  -->  [Basis data]
```

- **Browser** menampilkan halaman dan mengirim permintaan ke server.
- **Server aplikasi** menjalankan semua aturan (cek kuota, hitung denda, dan sebagainya) lalu membaca atau menulis ke basis data.
- **Basis data** menyimpan semua data secara permanen.

Server juga membaca (tanpa mengubah) data akademik kampus untuk mencocokkan NIM/NIP saat registrasi.

### 2.2 Data yang Disimpan

Basis data berisi lima tabel.

#### AKUN

| Kolom | Keterangan |
|---|---|
| `id_user` | Kunci utama. |
| `nim_nip` | Identitas unik Anggota/Pustakawan/Admin. |
| `nama` | Nama lengkap. |
| `kata_sandi` | Disimpan terenkripsi, tidak pernah berupa teks asli. |
| `peran` | `anggota` / `pustakawan` / `admin`. |
| `status` | `aktif` / `dibekukan`. |
| `masa_berlaku` | Tanggal akhir keanggotaan dari data akademik (hanya terisi untuk Anggota). |

#### BUKU

| Kolom | Keterangan |
|---|---|
| `id_buku` | Kunci utama. |
| `isbn` | Nomor ISBN. |
| `judul` | Judul buku. |
| `penulis` | Nama penulis. |
| `stok_total` | Jumlah eksemplar yang dimiliki perpustakaan. |
| `stok_tersedia` | Jumlah eksemplar di rak yang boleh dipinjam. Buku yang ditahan untuk pemesan reservasi tidak dihitung. |

#### PEMINJAMAN

| Kolom | Keterangan |
|---|---|
| `id_peminjaman` | Kunci utama. |
| `id_member` | Merujuk ke AKUN. |
| `id_buku` | Merujuk ke BUKU. |
| `tanggal_pinjam` | Tanggal peminjaman. |
| `tanggal_jatuh_tempo` | Batas akhir pengembalian. |
| `jumlah_perpanjangan` | Dimulai dari 0. |
| `tanggal_kembali` | Kosong selama buku belum dikembalikan. |

#### DENDA

| Kolom | Keterangan |
|---|---|
| `id_denda` | Kunci utama. |
| `id_peminjaman` | Merujuk ke PEMINJAMAN. |
| `jumlah` | Nominal dalam rupiah. |
| `status` | `belum_lunas` / `lunas`. |

#### RESERVASI

| Kolom | Keterangan |
|---|---|
| `id_reservasi` | Kunci utama. |
| `id_member` | Merujuk ke AKUN. |
| `id_buku` | Merujuk ke BUKU. |
| `waktu_reservasi` | Waktu pemesanan, dipakai untuk urutan antrean. |
| `status` | `menunggu` / `siap_diambil` / `selesai` / `batal`. |
| `batas_ambil` | Hanya terisi saat status `siap_diambil`. |

### 2.3 Login dan Hak Akses

- Login memakai NIM/NIP dan kata sandi.
- Setelah login, menu yang tampil ditentukan oleh peran. Anggota hanya bisa melihat data miliknya sendiri. Pustakawan dan Admin mendapat menu tambahan sesuai tugasnya.
- Akun Pustakawan dan Admin dibuat langsung di basis data oleh pengelola infrastruktur kampus, bukan lewat aplikasi.
- **Status kedaluwarsa tidak disimpan.** Server menentukannya setiap kali status keanggotaan diperiksa, dengan membandingkan `masa_berlaku` dengan tanggal hari ini.

### 2.4 Pencarian

Server mencocokkan kata kunci dengan judul, penulis, atau ISBN. Hasil pencarian menampilkan `stok_tersedia` untuk setiap buku.

### 2.5 Perubahan Stok

| Kejadian | Efek pada `stok_tersedia` |
|---|---|
| Buku dipinjam | Berkurang 1. |
| Buku dikembalikan, tidak ada antrean reservasi | Bertambah 1. |
| Buku dikembalikan, ada antrean reservasi | Tidak berubah — buku langsung ditahan untuk pemesan pertama. |
| Pemesan mengambil buku yang ditahan | Tidak berubah — buku itu memang sudah tidak dihitung. |
| Reservasi tidak diambil, dibatalkan | Bertambah 1 jika tidak ada antrean berikutnya; jika ada, buku ditahan untuk pemesan berikutnya. |

### 2.6 Denda

Data denda dicatat di tabel DENDA saat buku dikembalikan terlambat. Besarnya mengikuti tarif harian per buku. Status denda diubah menjadi `lunas` oleh Pustakawan setelah Anggota membayar **tunai**.

### 2.7 Laporan Bulanan

Sebuah pekerjaan terjadwal berjalan setiap tanggal 1 pukul 01:00. Pekerjaan ini menyusun rekap bulan sebelumnya: jumlah peminjaman, 10 buku paling sering dipinjam, dan total denda tercatat. Hasilnya disimpan di basis data. Admin membuka halaman Laporan untuk melihat dan mengunduhnya dalam bentuk PDF.

### 2.8 Cadangan Data

Basis data dicadangkan otomatis setiap malam oleh pengelola infrastruktur kampus.

---

## 3. Alur Aplikasi

Alur ditulis sebagai pseudocode tingkat tinggi, bukan kode sebenarnya.

**Kata kunci:** `MULAI`, `SELESAI`, `JIKA`, `SELAIN ITU`, `UNTUK SETIAP`, `BERHENTI`. Baris berawalan `#` adalah komentar.

| Flow | Nama | Aktor | Modul Terkait |
|---|---|---|---|
| 1 | Registrasi, Login, dan Pembekuan Akun | Semua | Modul 1 (Akun) |
| 2 | Pencarian Buku | Semua | Modul 2 (Buku) |
| 3 | Peminjaman Buku | Anggota | Modul 1, 2, 3 |
| 4 | Perpanjangan | Anggota | Modul 1, 3 |
| 5 | Pengembalian Buku | Pustakawan | Modul 2, 3 |
| 6 | Reservasi | Anggota, Pustakawan | Modul 1, 2, 3 |
| 7 | Pengelolaan Katalog | Pustakawan | Modul 2 |
| 8 | Laporan Bulanan | Admin | Modul 2, 3, 4 |

### Flow 1 — Registrasi, Login, dan Pembekuan Akun

#### 1A. Registrasi (calon Anggota)

```
MULAI
  calon Anggota membuka halaman Daftar
  calon Anggota mengisi NIM/NIP, nama, dan kata sandi
  cocokkan NIM/NIP dengan data akademik kampus
  JIKA tidak ditemukan di data akademik
      tampilkan "NIM/NIP tidak terdaftar", BERHENTI
  JIKA NIM/NIP sudah punya akun
      tampilkan "Akun sudah ada", BERHENTI
  simpan akun baru:
      peran = anggota, status = aktif,
      masa_berlaku = tanggal akhir studi/kontrak dari data akademik
  arahkan ke halaman login
SELESAI
```

#### 1B. Login (semua aktor)

```
MULAI
  aktor mengisi NIM/NIP dan kata sandi
  JIKA cocok dengan akun tersimpan
      masuk ke beranda, menu mengikuti peran
  SELAIN ITU
      tampilkan "NIM/NIP atau kata sandi salah"
SELESAI
```

#### 1C. Pembekuan Akun (Admin)

```
MULAI
  Admin mencari akun Anggota
  Admin memilih "Bekukan" atau "Aktifkan kembali"
  ubah status akun sesuai pilihan Admin
SELESAI
```

### Flow 2 — Pencarian Buku

```
MULAI
  aktor mengetik kata kunci
  cari buku yang judul, penulis, atau ISBN-nya cocok
  tampilkan daftar hasil beserta stok_tersedia
  JIKA aktor memilih satu buku
      tampilkan detail buku
SELESAI
```

### Flow 3 — Peminjaman Buku

**Aktor:** Anggota

```
MULAI
  Anggota memindai kode buku di kios layanan mandiri atau ponsel

  # cek keanggotaan
  Periksa status keanggotaan: harus aktif (belum kedaluwarsa dan tidak dibekukan)
  JIKA tidak aktif
      tampilkan "Keanggotaan tidak aktif", BERHENTI

  # cek kuota
  hitung jumlah buku yang sedang dipinjam Anggota
  JIKA jumlah sudah 3
      tampilkan "Kuota 3 buku tercapai", BERHENTI

  # cek stok
  Cek stok: stok_tersedia buku harus lebih dari 0,
            ATAU Anggota memiliki reservasi berstatus siap_diambil untuk buku ini
  JIKA tidak memenuhi
      tampilkan "Buku sedang habis", tawarkan reservasi (lihat Flow 6), BERHENTI

  # proses peminjaman
  catat peminjaman:
      tanggal_pinjam = hari ini
      tanggal_jatuh_tempo = hari ini + 7 hari
      jumlah_perpanjangan = 0
  JIKA peminjaman berasal dari reservasi siap_diambil
      ubah status reservasi menjadi selesai
      # stok_tersedia tidak dikurangi karena buku sudah ditahan
  SELAIN ITU
      kurangi stok_tersedia sebanyak 1
  tampilkan bukti peminjaman beserta tanggal jatuh tempo
SELESAI
```

> **Catatan:** nilai kuota (3) dan masa pinjam (7 hari) sudah diselaraskan dengan aturan bisnis. Dokumen mentah menyebut 5 buku dan 14 hari.

### Flow 4 — Perpanjangan

#### 4A. Perpanjangan Manual

**Aktor:** Anggota

```
MULAI
  Anggota membuka halaman Peminjaman Saya
  Anggota memilih satu peminjaman yang sedang berjalan, tekan "Perpanjang"

  Pastikan akun Anggota masih berlaku dan tidak sedang dibekukan
  JIKA tidak
      tampilkan "Akun tidak dapat memperpanjang", BERHENTI

  JIKA jumlah_perpanjangan peminjaman ini sudah 1
      tampilkan "Batas perpanjangan tercapai", BERHENTI

  JIKA ada reservasi berstatus menunggu untuk buku ini
      tampilkan "Buku sedang ditunggu Anggota lain", BERHENTI

  # periksa denda
  JIKA Anggota punya denda berstatus belum_lunas
      tampilkan "Selesaikan denda terlebih dahulu", BERHENTI
  UNTUK SETIAP buku lain milik Anggota yang sudah lewat jatuh tempo
      denda_berjalan = (jumlah hari terlambat x Rp1.000)
  JIKA total denda_berjalan lebih dari 0
      tampilkan "Selesaikan denda terlebih dahulu", BERHENTI

  tanggal_jatuh_tempo baru = tanggal_jatuh_tempo lama + 7 hari
  tambah jumlah_perpanjangan sebanyak 1
  tampilkan tanggal jatuh tempo baru
SELESAI
```

#### 4B. Perpanjangan Otomatis

```
MULAI
  Menjelang tanggal jatuh tempo, peminjaman diperpanjang otomatis.
  Perpanjangan otomatis dilakukan satu kali per peminjaman.
  Tanggal jatuh tempo baru ditampilkan di halaman Peminjaman Saya.
SELESAI
```

> **Catatan:** dokumen deskripsi produk hanya menyebut perpanjangan manual. Flow 4B (otomatis) dipertahankan sebagai fitur karena tertulis di alur, tetapi ditandai butuh klarifikasi. Lihat [Catatan Temuan](#6-catatan-temuan-dan-keputusan).

### Flow 5 — Pengembalian Buku

**Aktor:** Pustakawan

```
MULAI
  Pustakawan memindai kode buku yang dikembalikan
  cari peminjaman berjalan untuk buku tersebut
  JIKA tidak ditemukan
      tampilkan "Peminjaman tidak ditemukan", BERHENTI

  catat tanggal_kembali = hari ini

  # hitung denda
  hari_terlambat = tanggal_kembali - tanggal_jatuh_tempo
  JIKA hari_terlambat lebih dari 0
      denda = hari_terlambat x Rp1.000
      catat denda dengan status belum_lunas
      tampilkan jumlah denda kepada Pustakawan
      Anggota membayar tunai di meja
      Pustakawan mengubah status denda menjadi lunas

  # perbarui stok
  JIKA ada reservasi berstatus menunggu untuk buku ini
      ambil reservasi dengan waktu_reservasi paling awal
      ubah statusnya menjadi siap_diambil
      batas_ambil = hari ini + 2 hari
      # stok_tersedia tidak bertambah, buku ditahan
  SELAIN ITU
      tambah stok_tersedia sebanyak 1
SELESAI
```

### Flow 6 — Reservasi

#### 6A. Membuat Reservasi (Anggota)

```
MULAI
  Anggota membuka detail buku, tekan "Reservasi"

  Cek apakah Anggota valid: masa keanggotaan belum habis dan akun tidak dibekukan
  JIKA tidak valid
      tampilkan "Keanggotaan tidak aktif", BERHENTI

  Periksa ketersediaan buku
  JIKA masih ada eksemplar di rak (stok_tersedia lebih dari 0)
      tampilkan "Buku tersedia, silakan pinjam langsung", BERHENTI

  JIKA Anggota sudah punya reservasi menunggu untuk buku yang sama
      tampilkan "Sudah ada di antrean", BERHENTI

  simpan reservasi: waktu_reservasi = sekarang, status = menunggu
  tampilkan posisi antrean Anggota
SELESAI
```

#### 6B. Membatalkan Reservasi yang Tidak Diambil (Pustakawan)

```
MULAI
  Pustakawan membuka daftar reservasi siap_diambil yang batas_ambil-nya sudah lewat
  Pustakawan memilih reservasi, tekan "Batalkan"
  ubah status reservasi menjadi batal
  JIKA ada reservasi menunggu berikutnya untuk buku yang sama
      ubah status reservasi berikutnya menjadi siap_diambil
      batas_ambil = hari ini + 2 hari
  SELAIN ITU
      tambah stok_tersedia sebanyak 1
SELESAI
```

### Flow 7 — Pengelolaan Katalog

**Aktor:** Pustakawan

#### 7A. Tambah Buku

```
MULAI
  Pustakawan mengisi ISBN, judul, penulis, dan stok_total
  simpan buku baru dengan stok_tersedia = stok_total
SELESAI
```

#### 7B. Ubah Buku

```
MULAI
  Pustakawan memilih buku, mengubah data yang perlu diubah
  JIKA stok_total diubah
      sesuaikan stok_tersedia dengan selisih perubahan stok_total
  simpan perubahan
SELESAI
```

#### 7C. Hapus Buku

```
MULAI
  Pustakawan memilih buku, tekan "Hapus"
  JIKA ada peminjaman berjalan atau reservasi aktif untuk buku ini
      tampilkan "Buku masih dipakai, tidak dapat dihapus", BERHENTI
  hapus buku dari katalog
SELESAI
```

### Flow 8 — Laporan Bulanan

#### 8A. Penyusunan (terjadwal)

```
MULAI
  setiap tanggal 1 pukul 01:00
  hitung jumlah peminjaman bulan sebelumnya
  tentukan 10 buku yang paling sering dipinjam
  jumlahkan denda yang tercatat pada bulan tersebut
  simpan hasil rekap
SELESAI
```

#### 8B. Melihat Laporan (Admin)

```
MULAI
  Admin membuka halaman Laporan
  Admin memilih bulan
  tampilkan rekap bulan tersebut
  JIKA Admin menekan "Unduh"
      berikan file PDF
SELESAI
```

---

## 4. Daftar Fitur

Semua fitur di bawah ini sudah selesai dibangun dan berjalan di produk.

| Kode | Fitur | Aktor | Status |
|---|---|---|---|
| F-01 | Registrasi, Login, dan Pembekuan Akun | Anggota, Admin, Pustakawan | SELESAI |
| F-02 | Pencarian Katalog | Semua aktor login | SELESAI |
| F-03 | Peminjaman Buku | Anggota | SELESAI |
| F-04 | Perpanjangan Peminjaman | Anggota | SELESAI |
| F-05 | Pengembalian Buku dan Perhitungan Denda | Pustakawan | SELESAI |
| F-06 | Reservasi Buku | Anggota, Pustakawan | SELESAI |
| F-07 | Pengelolaan Katalog | Pustakawan | SELESAI |
| F-08 | Laporan Bulanan | Admin | SELESAI |

### Rincian Fitur

#### F-01 Registrasi, Login, dan Pembekuan Akun

- **Aktor:** Anggota (registrasi, login), Admin (pembekuan akun), Pustakawan (login)
- **Deskripsi:** Anggota mendaftar memakai NIM/NIP yang dicocokkan dengan data akademik. Semua aktor login dengan NIM/NIP dan kata sandi. Admin dapat membekukan dan mengaktifkan kembali akun Anggota.

#### F-02 Pencarian Katalog

- **Aktor:** Semua aktor yang sudah login
- **Deskripsi:** Mencari buku berdasarkan judul, penulis, atau ISBN. Hasil menampilkan jumlah eksemplar yang tersedia.

#### F-03 Peminjaman Buku

- **Aktor:** Anggota
- **Deskripsi:** Peminjaman mandiri dengan memindai kode buku. Sistem memeriksa keanggotaan, kuota, dan ketersediaan buku, lalu mencatat peminjaman beserta tanggal jatuh tempo.

#### F-04 Perpanjangan Peminjaman

- **Aktor:** Anggota
- **Deskripsi:** Peminjaman dapat diperpanjang secara manual oleh Anggota maupun secara otomatis menjelang jatuh tempo. Tanggal jatuh tempo baru tampil di halaman Peminjaman Saya.

#### F-05 Pengembalian Buku dan Perhitungan Denda

- **Aktor:** Pustakawan
- **Deskripsi:** Pustakawan memproses pengembalian dengan memindai kode buku. Denda keterlambatan dihitung otomatis dan dicatat. Pembayaran denda dilakukan tunai di meja Pustakawan, lalu Pustakawan menandai denda sebagai lunas.

#### F-06 Reservasi Buku

- **Aktor:** Anggota, Pustakawan
- **Deskripsi:** Anggota dapat memesan buku yang semua eksemplarnya sedang dipinjam dan melihat posisi antreannya. Buku yang kembali ditahan 2 hari untuk pemesan pertama. Pustakawan dapat membatalkan reservasi yang tidak diambil sampai batas waktunya.

#### F-07 Pengelolaan Katalog

- **Aktor:** Pustakawan
- **Deskripsi:** Menambah, mengubah, dan menghapus data buku serta jumlah stok. Buku yang masih dipinjam atau direservasi tidak dapat dihapus.

#### F-08 Laporan Bulanan

- **Aktor:** Admin
- **Deskripsi:** Rekap bulanan berisi jumlah peminjaman, buku yang paling sering dipinjam, dan total denda. Admin dapat melihat dan mengunduhnya sebagai PDF.

---

## 5. Tujuan Produk

Tanda ✅ berarti goal sudah tercapai dan berjalan di produk.

| Kode | Tujuan | Status | Bukti |
|---|---|---|---|
| G-01 | Anggota dapat mencari buku dan melihat ketersediaannya tanpa bantuan petugas. | ✅ Tercapai | Fitur F-02 |
| G-02 | Anggota dapat meminjam buku secara mandiri tanpa antre di meja layanan. | ✅ Tercapai | Fitur F-03 |
| G-03 | Anggota dapat memperpanjang pinjaman tanpa harus datang ke perpustakaan. | ✅ Tercapai | Fitur F-04 |
| G-04 | Denda keterlambatan terhitung otomatis dan akurat. | ✅ Tercapai | Fitur F-05 |
| G-05 | Antrean reservasi berjalan adil sesuai urutan pemesanan. | ✅ Tercapai | Fitur F-06 |
| G-06 | Pustakawan dapat mengelola katalog buku tanpa bantuan tim teknis. | ✅ Tercapai | Fitur F-07 |
| G-07 | Admin menerima laporan bulanan tanpa perlu menyusunnya manual. | ✅ Tercapai | Fitur F-08 |
| ~~G-08~~ | ~~Anggota menerima notifikasi email sebelum buku jatuh tempo.~~ | ❌ **DIANULIR** | Fitur tidak pernah dibangun |
| ~~G-09~~ | ~~Anggota dapat membayar denda secara online.~~ | ❌ **DIANULIR** | Bertentangan dengan aturan bisnis: denda wajib tunai |

> **Catatan:** G-08 dan G-09 ditandai ✅ di dokumen mentah `05-goal.txt`, tetapi keduanya **tidak muncul** di daftar fitur dan **tidak ada** di alur aplikasi. G-09 juga bertentangan langsung dengan aturan bisnis yang berlaku (pembayaran denda wajib tunai di meja Pustakawan). Keduanya **dianulir**.

---

## 6. Catatan Temuan dan Keputusan

Dokumen ini mencatat inkonsistensi yang ditemukan di `raw/` beserta keputusan yang diambil saat menyusun dokumentasi. Temuan ini juga dibuka sebagai **issue** di repo GitHub.

### Ringkasan Temuan

| # | Temuan | File Sumber | Dampak | Keputusan |
|---|---|---|---|---|
| T-01 | Kuota pinjam berbeda: 3 buku vs 5 buku | `01-deskripsi-produk.txt` vs `03-flow-aplikasi.txt` | Dokumentasi menyesatkan soal batas pinjam | Pakai **3 buku** (deskripsi produk = sumber aturan bisnis) |
| T-02 | Masa pinjam berbeda: 7 hari vs 14 hari | `01-deskripsi-produk.txt` vs `03-flow-aplikasi.txt` | Jatuh tempo salah dihitung | Pakai **7 hari** (deskripsi produk = sumber aturan bisnis) |
| T-03 | Perpanjangan otomatis tidak disebut di dokumen lain | `03-flow-aplikasi.txt` (Flow 4B) | Fitur tak terdokumentasi | Dipertahankan sebagai fitur, ditandai perlu klarifikasi |
| T-04 | G-08 (notifikasi email) & G-09 (bayar denda online) ditandai tercapai | `05-goal.txt` | Goal palsu, tidak ada fiturnya | **Dianulir** — lihat [Tujuan Produk](#5-tujuan-produk) |
| T-05 | Aturan denda & stok didefinisikan berulang di beberapa flow | `03-flow-aplikasi.txt` | Duplikasi logika | Didefinisikan sekali di modul pemiliknya, dirujuk dari tempat lain |

### Detail Temuan

#### T-01 — Kuota Pinjam: 3 vs 5 Buku

- `01-deskripsi-produk.txt` bagian "Aturan Peminjaman": **3 buku sekaligus**.
- `03-flow-aplikasi.txt` Flow 3: `JIKA jumlah sudah 5 → "Kuota 5 buku tercapai"`.
- **Keputusan:** memakai **3 buku**, karena deskripsi produk adalah dokumen aturan bisnis, sedangkan flow adalah pseudocode yang bisa mengandung kesalahan penulisan. Nilai ini dipakai konsisten di [Deskripsi Produk](#14-aturan-peminjaman), [Modul Transaksi](../modules/transaksi-modul-3.md), dan [Alur Aplikasi](#flow-3--peminjaman-buku).

#### T-02 — Masa Pinjam: 7 vs 14 Hari

- `01-deskripsi-produk.txt`: **7 hari sejak tanggal pinjam**.
- `03-flow-aplikasi.txt` Flow 3: `tanggal_jatuh_tempo = hari ini + 14 hari`.
- **Keputusan:** memakai **7 hari**, alasan sama dengan T-01.

#### T-03 — Perpanjangan Otomatis

- `03-flow-aplikasi.txt` punya Flow 4B (perpanjangan otomatis), tetapi `01-deskripsi-produk.txt` dan `04-daftar-fitur.txt` hanya menyebut perpanjangan manual.
- **Keputusan:** fitur otomatis dipertahankan di dokumentasi karena tertulis di flow, tetapi ditandai sebagai butuh klarifikasi. Nilai sementara: perpanjangan otomatis dilakukan **satu kali** per peminjaman.

#### T-04 — Goal yang Dianulir

- `05-goal.txt` menandai G-08 (notifikasi email) dan G-09 (bayar denda online) dengan ✅.
- Keduanya **tidak muncul** di `04-daftar-fitur.txt` dan **tidak ada** di `03-flow-aplikasi.txt`. G-09 juga bertentangan dengan aturan bisnis (denda wajib tunai).
- **Keputusan:** keduanya **dianulir** di [Tujuan Produk](#5-tujuan-produk) sampai ada klarifikasi dari pembuat dokumen.

#### T-05 — Duplikasi Aturan

- Aturan perhitungan denda (Rp1.000/hari) muncul di Flow 4A dan Flow 5.
- Aturan perubahan `stok_tersedia` muncul di Flow 3, Flow 5, dan Flow 6B.
- **Keputusan:** masing-masing didefinisikan **satu kali** — denda di [Modul 3](../modules/transaksi-modul-3.md), perubahan stok di [Modul 2](../modules/buku-modul-2.md) — lalu dirujuk dari tempat lain.
