# Modul 4 — Rekap

## Tanggung Jawab

Mengeksekusi pekerjaan di balik layar (*cron job*) untuk menyusun rekapitulasi data bulanan, serta menyediakan halaman khusus bagi Admin untuk melihat dan mengunduh laporan tersebut dalam format PDF.

## Isi

### Data

| Data | Keterangan |
|---|---|
| Bulan/Tahun Laporan | Periode rekap. |
| Total Peminjaman | Jumlah peminjaman pada bulan tersebut. |
| Daftar 10 Buku Terlaris | Buku dengan frekuensi pinjam tertinggi. |
| Total Denda (Rupiah) | Akumulasi denda tercatat. |
| File PDF laporan | Hasil yang bisa diunduh Admin. |

### Aturan

- Sistem "bangun" dan berjalan otomatis setiap **tanggal 1 pukul 01:00** untuk menghitung dan merangkum seluruh aktivitas bulan sebelumnya.
- Hanya **Admin** yang berwenang membuka halaman ini dan mengunduh PDF-nya.

## Hubungan

| Modul Lain | Arah | Keterangan |
|---|---|---|
| Modul 2 (Buku) | Memanggil | Mengambil data judul buku untuk diurutkan dan dicari 10 buku dengan frekuensi pinjam tertinggi. |
| Modul 3 (Transaksi) | Memanggil | Mengambil data untuk menghitung berapa kali buku dipinjam dan menjumlahkan nominal denda yang terkumpul sebulan terakhir. |

## Alasan Pengelompokan

Fungsi rekap bulanan tidak ada hubungannya dengan operasional harian pengunjung. Memisahkannya menjadi modul sendiri memastikan proses rekap otomatis di tengah malam tidak mengganggu atau memperberat kinerja fitur peminjaman buku harian.
