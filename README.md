# Dokumentasi Sistem PerpusKu

Repositori ini berisi hasil restrukturisasi dokumentasi **Sistem PerpusKu** — sistem peminjaman buku perpustakaan kampus — dari teks mentah di `raw/` menjadi dokumentasi terstruktur yang bisa dipahami tanpa membaca kode.

> **Catatan:** instruksi soal asli tetap tersimpan di [SOAL.md](SOAL.md), dan bahan mentah tidak diubah di [raw/](raw/).

## Ringkasan Produk

PerpusKu menggantikan pencatatan peminjaman manual di buku besar. Sistem ini dipakai oleh tiga aktor:

| Aktor | Tugas Utama |
|---|---|
| **Anggota** | Mencari, meminjam, memperpanjang, dan mereservasi buku. |
| **Pustakawan** | Memproses pengembalian, denda, katalog, dan reservasi. |
| **Admin** | Melihat laporan bulanan dan mengatur status akun. |

Selengkapnya: [Gambaran Umum](docs/produk/gambaran-umum.md).

## Daftar Isi Dokumentasi

### 1. Dokumentasi Modul (Langkah 3)

- [Ringkasan dan Peta Modul](docs/modules/README.md)
  - [Modul 1 — Akun](docs/modules/akun-modul-1.md)
  - [Modul 2 — Buku](docs/modules/buku-modul-2.md)
  - [Modul 3 — Transaksi](docs/modules/transaksi-modul-3.md)
  - [Modul 4 — Rekap](docs/modules/rekap-modul-4.md)

### 2. Diagram Sistem (Langkah 4 & 5)

- [Kumpulan Diagram](docs/diagrams/README.md)
  - Activity: [Peminjaman](docs/diagrams/img/alur-peminjaman.png), [Pengembalian](docs/diagrams/img/alur-pengembalian.png), [Reservasi](docs/diagrams/img/alur-reservasi.png), [Perpanjangan](docs/diagrams/img/alur-perpanjangan.png)
  - Use Case: [Use Case PerpusKu](docs/diagrams/img/usecase-perpusku.png)
  - Sequence: [Peminjaman Buku](docs/diagrams/img/sequence-peminjaman.png), [Laporan Bulanan](docs/diagrams/img/sequence-laporan.png)

### 3. Dokumentasi Produk (Langkah 6)

- [Gambaran Umum Sistem](docs/produk/gambaran-umum.md) — deskripsi produk, cara kerja teknis, alur aplikasi, daftar fitur, tujuan, dan catatan temuan.

### 4. Referensi

- [Glosarium Istilah](docs/glosarium.md)
- [Identitas Mahasiswa](identitas.md)

## Struktur Repositori

```
.
├── README.md              (file ini — pintu masuk dokumentasi)
├── SOAL.md                (instruksi soal, tidak diubah)
├── identitas.md           (data mahasiswa)
├── raw/                   (bahan mentah, tidak diubah)
└── docs/
    ├── produk/
    │   └── gambaran-umum.md
    ├── modules/           (modul + peta hubungan)
    ├── diagrams/          (sumber .puml + README)
    │   └── img/           (gambar .png hasil render)
    └── glosarium.md       (daftar istilah)
```

## Temuan Penting

Dokumentasi mentah mengandung beberapa inkonsistensi yang sudah dicatat dan diputuskan di [Gambaran Umum](docs/produk/gambaran-umum.md#6-catatan-temuan-dan-keputusan), antara lain:

- Kuota pinjam berbeda (3 vs 5 buku) — dipakai **3 buku**.
- Masa pinjam berbeda (7 vs 14 hari) — dipakai **7 hari**.
- Goal G-08 (notifikasi email) dan G-09 (bayar denda online) **dianulir**.

## Daftar Issue

Temuan-temuan di atas dibuka sebagai issue di repo ini dan diselesaikan satu per satu:

- [Issue yang sudah ditutup](https://github.com/AZZAMAF/SDF-EXAm-AZZAM/issues?q=is%3Aissue+is%3Aclosed)
