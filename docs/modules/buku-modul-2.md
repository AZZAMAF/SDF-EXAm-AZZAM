# Modul 2 — Buku

## Tanggung Jawab

Mengelola **identitas buku** dan **mutasi `stok_tersedia`** di rak perpustakaan, termasuk melayani pencarian buku.

## Isi

### Data

| Data | Keterangan |
|---|---|
| `Judul` | Judul buku. |
| `Penulis` | Nama penulis. |
| `ISBN` | Nomor ISBN. |
| `stok_total` | Jumlah seluruh eksemplar yang dimiliki perpustakaan. |
| `stok_tersedia` | Jumlah eksemplar di rak yang boleh dipinjam. |

> Modul ini **tidak memiliki** data transaksi, denda, atau reservasi. Semua itu milik [Modul 3](transaksi-modul-3.md).

### Aturan

**Hak akses eksklusif:** hanya **Pustakawan** yang berwenang menambah, mengedit, atau menghapus data buku. Anggota hanya bisa mencari; Admin tidak ikut campur urusan rak buku.

**Aturan hapus buku:** buku **tidak bisa dihapus** dari sistem jika sedang dipinjam atau sedang direservasi.

**Mutasi stok (`stok_tersedia`):**

| Kejadian | Efek |
|---|---|
| Buku sukses dipinjam | Berkurang 1 ➖ |
| Buku dikembalikan, **tidak ada** Anggota lain yang mengantre reservasi | Bertambah 1 ➕ |
| Buku dikembalikan, **ada** antrean reservasi | Statis (tetap) ⏸️ — buku langsung "ditahan" untuk pemesan urutan pertama, jadi angka stok di sistem tidak ditambah. |

## Hubungan

| Modul Lain | Arah | Keterangan |
|---|---|---|
| Modul 3 (Transaksi) | Dipanggil oleh | Mengurangi atau menambah stok saat ada peminjaman, pengembalian, atau reservasi. |

## Alasan Pengelompokan

Modul ini berisi data buku dan stok buku. Menggabungkannya dalam satu modul memudahkan programmer karena seluruh urusan katalog dan ketersediaan sudah terpusat di satu tempat.
