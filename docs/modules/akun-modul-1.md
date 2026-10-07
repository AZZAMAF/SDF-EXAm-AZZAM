# Modul 1 — Akun

## Tanggung Jawab

Modul ini bertanggung jawab mengurus **validasi otentikasi dan peran**: login, registrasi, penentuan peran (Admin, Pustakawan, Anggota), status akun (dibekukan atau aktif) yang diatur oleh Admin, serta masa berlaku keanggotaan.

## Isi

### Data

| Data | Keterangan |
|---|---|
| `NIM/NIP` | Identitas unik Anggota/Pustakawan/Admin. |
| `Peran` | `Admin`, `Anggota`, atau `Pustakawan`. |
| `Status` | `aktif` atau `dibekukan`. |
| `Masa_Berlaku` | Tanggal akhir keanggotaan (hanya untuk Anggota). |
| `kata_sandi` | Disimpan terenkripsi. |

### Aturan

- Mengecek apakah aktor adalah Admin, Pustakawan, atau Anggota.
- Mengecek status akun (aktif / dibekukan).
- Mengecek masa berlaku keanggotaan.
- **Status kedaluwarsa tidak disimpan.** Server menentukannya saat pemeriksaan dengan membandingkan `masa_berlaku` dengan tanggal hari ini.

## Hubungan

| Modul Lain | Arah | Keterangan |
|---|---|---|
| Modul 3 (Transaksi) | Dipanggil oleh | Memastikan Anggota yang mau pinjam statusnya aktif atau dibekukan. |

## Alasan Pengelompokan

Semua hal yang berhubungan dengan akun — login, registrasi, validasi peran, status, dan masa berlaku — dikelompokkan dalam satu modul agar logika identitas dan otorisasi tidak tersebar di tempat lain.
