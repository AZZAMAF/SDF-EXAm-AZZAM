# Ringkasan dan Peta Modul

Sistem PerpusKu didekomposisi menjadi **4 modul utama** untuk mencegah tumpang tindih logika dan mempermudah pemeliharaan sistem.

## Daftar Modul

| # | Modul | Tanggung Jawab Utama | File |
|---|---|---|---|
| 1 | Akun | Validasi otentikasi, peran, status, dan masa berlaku | [akun-modul-1.md](akun-modul-1.md) |
| 2 | Buku | Identitas buku dan mutasi `stok_tersedia` | [buku-modul-2.md](buku-modul-2.md) |
| 3 | Transaksi | Operasional sirkulasi dan denda | [transaksi-modul-3.md](transaksi-modul-3.md) |
| 4 | Rekap | Cron job laporan bulanan PDF untuk Admin | [rekap-modul-4.md](rekap-modul-4.md) |

## Peta Hubungan Antar Modul

- **Modul 3 (Transaksi)** adalah pusat operasional harian. Modul ini bergantung pada **Modul 1 (Akun)** untuk validasi identitas dan status keanggotaan, serta pada **Modul 2 (Buku)** untuk ketersediaan fisik stok.
- **Modul 4 (Rekap)** berjalan di latar belakang (terjadwal) dan bergantung pada **Modul 3** dan **Modul 2** untuk menarik data historis.
- **Modul 1** dan **Modul 2** adalah modul dasar yang bersifat independen dan tidak saling mengetahui satu sama lain.

```
                 +-------------------+
                 |   Modul 4: Rekap  |
                 +---------+---------+
                           | menarik data
              +------------+------------+
              v                         v
   +-------------------+     +-------------------+
   | Modul 3: Transaksi|---->|  Modul 2: Buku    |
   +---------+---------+     +-------------------+
             |
             v
   +-------------------+
   |  Modul 1: Akun    |
   +-------------------+
```

## Aturan Anti-Duplikasi

Tidak boleh ada logika atau aturan yang didefinisikan berulang. Aturan berikut didefinisikan **satu kali** di modul pemiliknya, lalu dirujuk dari tempat lain:

| Aturan | Pemilik | Perujuk |
|---|---|---|
| Validasi keanggotaan aktif (masa berlaku + status) | Modul 1 | Modul 3 (peminjaman, perpanjangan, reservasi) |
| Kuota maksimal 3 buku per Anggota | Modul 3 | — |
| Perhitungan denda Rp1.000/hari | Modul 3 | — |
| Mutasi `stok_tersedia` | Modul 2 | Modul 3 (memanggil, bukan mendefinisikan ulang) |
| Penahanan buku 2 hari untuk pemesan pertama | Modul 3 | Modul 2 (stok tidak bertambah) |

## Contoh Dampak Perubahan

Jika perpustakaan memutuskan menambah atribut `denda_hilang` (buku hilang didenda lebih mahal), programmer hanya perlu memodifikasi **Modul 3 (Transaksi)**. Modul 1, Modul 2, dan Modul 4 tidak akan terdampak atau error, karena seluruh urusan denda sudah diabstraksi penuh di dalam Modul 3.
