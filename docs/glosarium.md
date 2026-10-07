# Glosarium Istilah PerpusKu

Daftar istilah yang dipakai konsisten di seluruh dokumentasi.

## Aktor Baku

Sistem ini hanya mengenal **tiga aktor**. Istilah lain seperti "user", "member", atau "peminjam" tidak dipakai.

| Istilah | Arti |
|---|---|
| **Anggota** | Mahasiswa atau dosen yang memakai layanan perpustakaan. |
| **Pustakawan** | Petugas meja layanan yang memproses pengembalian, denda, katalog, dan reservasi. |
| **Admin** | Pengelola sistem yang melihat laporan bulanan dan mengatur status akun. |

## Istilah Domain

| Istilah | Arti |
|---|---|
| **Kios layanan mandiri** | Perangkat di perpustakaan tempat Anggota meminjam buku sendiri dengan memindai kode. |
| **Eksemplar** | Satu salinan fisik dari sebuah judul buku. |
| **Masa berlaku** | Tanggal akhir keanggotaan, diambil dari data akademik kampus. |
| **Denda** | Biaya keterlambatan pengembalian buku. |
| **Reservasi** | Pemesanan buku yang seluruh eksemplarnya sedang dipinjam. |
| **Antrean reservasi** | Urutan Anggota yang menunggu buku yang sama, berdasarkan waktu pemesanan. |

## Istilah Teknis

| Istilah | Arti |
|---|---|
| **`stok_total`** | Jumlah seluruh eksemplar yang dimiliki perpustakaan. |
| **`stok_tersedia`** | Jumlah eksemplar yang ada di rak dan boleh dipinjam. Buku yang ditahan untuk pemesan tidak dihitung. |
| **`jumlah_perpanjangan`** | Berapa kali sebuah peminjaman sudah diperpanjang (mulai dari 0). |
| **`masa_berlaku`** | Atribut akun berisi tanggal akhir keanggotaan (hanya untuk Anggota). |
| **`batas_ambil`** | Tenggat pengambilan buku yang sudah berstatus `siap_diambil`. |
| **Status akun** | `aktif` atau `dibekukan`. |
| **Status denda** | `belum_lunas` atau `lunas`. |
| **Status reservasi** | `menunggu`, `siap_diambil`, `selesai`, atau `batal`. |
| **Cron job** | Pekerjaan terjadwal yang berjalan otomatis tanpa campur tangan manusia. |
| **PlantUML** | Alat untuk membuat diagram dari kode teks. |

## Istilah Proses

| Istilah | Arti |
|---|---|
| **Dekomposisi** | Memecah produk menjadi bagian-bagian kecil (modul). |
| **Modul** | Kumpulan tanggung jawab, data, dan aturan yang fokus pada satu hal. |
| **Swimlane / partition** | Kolom pada activity diagram yang menunjukkan modul penanggung jawab tiap langkah. |
| **`<<include>>`** | Relasi use case yang berarti use case tujuan selalu dijalankan. |
| **`<<extend>>`** | Relasi use case yang berarti use case tujuan dijalankan secara opsional. |
