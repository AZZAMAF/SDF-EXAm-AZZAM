# Modul 3 — Transaksi

## Tanggung Jawab

Menjalankan **operasional sirkulasi dan denda**: peminjaman, pengembalian, perpanjangan, denda, dan reservasi.

## Isi

### Data

| Data | Keterangan |
|---|---|
| `Jumlah_perpanjangan` | Berapa kali peminjaman sudah diperpanjang (mulai dari 0). |
| `tanggal_kembali` | Tanggal buku dikembalikan (kosong selama belum kembali). |
| `jumlah_denda` | Nominal denda, Rp1.000/hari. |
| `status_denda` | `lunas` atau `belum lunas`. |
| `waktu_reservasi` | Waktu pemesanan, dipakai untuk urutan antrean. |
| `status_reservasi` | `menunggu`, `siap_diambil`, `selesai`, atau `batal`. |
| `tanggal_pinjam` | Tanggal peminjaman. |

> **Modul ini tidak memiliki data stok.** Untuk ketersediaan buku, Modul 3 hanya **memanggil** data dari [Modul 2](buku-modul-2.md).

### Aturan

- Kuota **3 buku** sekaligus per Anggota.
- Masa pinjam **7 hari** sejak tanggal pinjam.
- Denda terhitung mulai hari keterlambatan (Rp1.000/hari per buku).
- Harus Anggota, bukan akun yang dibekukan.
- Buku yang kembali **ditahan 2 hari** untuk pemesan pertama.
- Pustakawan dapat membatalkan reservasi yang tidak diambil sampai batas waktunya.
- Perpanjangan maksimal **1 kali** (manual).
- Pembayaran denda wajib **tunai** di meja Pustakawan saat itu juga.

> **Catatan:** aturan denda dan mutasi stok didefinisikan di sini dan di [Modul 2](buku-modul-2.md) secara terpisah sesuai kepemilikan datanya — lihat [Gambaran Umum](../produk/gambaran-umum.md#6-catatan-temuan-dan-keputusan) T-05.

## Hubungan

| Modul Lain | Arah | Keterangan |
|---|---|---|
| Modul 1 (Akun) | Memanggil | Mengambil data akun dan status keanggotaan. |
| Modul 2 (Buku) | Memanggil | Mengambil data buku dan `stok_tersedia`. |

## Alasan Pengelompokan

Modul ini menggabungkan segala transaksi menjadi satu agar lebih mudah dibaca dan efisien.
