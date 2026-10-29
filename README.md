# Analisis Kualitas Udara di Indonesia

Tugas besar Mahadata (IF25-40402), Program Studi Teknik Informatika, Institut Teknologi Sumatera.
Repositori ini berisi eksplorasi dan pembersihan data (EDA) kualitas udara dari tiga lokasi: Jakarta, Bali, dan Medan.

## Kelompok 10 (Kelas R)

| Nama | NIM |
|---|---|
| Muhammad Royhan Alfitra | 123140146 |
| M. Fadhil Hawari | 123140147 |
| Dzaky Pramadhani | 123140208 |
| M. Farisi Suyitno | 123140152 |

## Data

Sumber: [OpenAQ](https://openaq.org), diunduh sebagai CSV per lokasi dan disimpan di folder `data/`.

| Lokasi | location_id | Zona waktu asli | Periode (UTC) | Jam tersedia |
|---|---|---|---|---|
| Jakarta | 6551345 | +07:00 | 2026-10-01 06:00 s.d. 2026-10-20 11:00 | 427 |
| Bali | 6494341 | +08:00 | 2026-10-01 00:00 s.d. 2026-10-20 11:00 | 458 |
| Medan | 5586536 | +07:00 | 2026-10-11 11:00 s.d. 2026-10-20 11:00 | 217 |

Parameter yang dipakai (ada di ketiga lokasi):

| Parameter | Keterangan |
|---|---|
| `pm1` | Partikulat PM1 |
| `pm25` | Partikulat PM2.5 |
| `um003` | Jumlah partikel (particle count) |

Suhu (`temperature`) dan kelembapan (`relativehumidity`) hanya tersedia di Jakarta dan Bali, sehingga tidak dimasukkan ke dataset gabungan.

Data mentah berformat long (5076 baris untuk ketiga lokasi, satu baris per parameter per jam, 3 parameter × 1102 + 2 parameter × 885).

## Alur Pembersihan

1. Membaca seluruh CSV OpenAQ di folder `data/` dan menggabungkannya.
2. Menyamakan waktu ke UTC (`datetime_from_utc`), karena Bali memakai +08:00 sedangkan Jakarta dan Medan +07:00.
3. Memilih parameter yang ada di ketiga lokasi (`pm1`, `pm25`, `um003`).
4. Pivot dari format long ke wide: satu baris mewakili satu jam di satu lokasi.
5. Menambahkan kolom `lokasi` sebagai pembeda dan kolom `sensor_*` untuk menelusuri asal pengukuran.
6. Menyimpan hasil ke `gabungan.csv`.

## Keluaran: `gabungan.csv`

1102 baris, 8 kolom, sekitar 89.5 KB.

| Kolom | Tipe | Keterangan |
|---|---|---|
| `lokasi` | string | `jakarta`, `bali`, atau `medan` |
| `waktu` | datetime (UTC) | Awal jam pengukuran |
| `pm1` | float | Konsentrasi PM1 |
| `pm25` | float | Konsentrasi PM2.5 |
| `um003` | float | Jumlah partikel |
| `sensor_pm1`, `sensor_pm25`, `sensor_um003` | float | ID sensor OpenAQ |

Tidak ada nilai kosong pada `gabungan.csv`.

## Cara Menjalankan

```bash
pip install pandas jupyter
jupyter notebook tubes.ipynb
```

Jalankan sel secara berurutan. Notebook mencari CSV di `data/`, atau `tubes/data/` jika dijalankan dari folder induk.

> **Catatan:** sel pertama membaca semua `*.csv` di folder data, dan `gabungan.csv` ditulis ke folder yang sama. Jika notebook dijalankan lebih dari sekali, file keluaran ikut terbaca sebagai masukan dan jumlah baris menjadi 6178, bukan 5076. Hapus `gabungan.csv` sebelum menjalankan ulang, atau pindahkan keluaran ke folder lain.

## Struktur Repositori

```
.
├── data/
│   ├── *.csv            # data mentah OpenAQ per lokasi
│   └── gabungan.csv     # hasil gabungan (dibuat oleh notebook)
├── tubes.ipynb          # notebook EDA dan pembersihan
└── README.md
```

## Keterbatasan

- Periode hanya 20 hari di Oktober 2026, sehingga pola musiman tidak dapat ditangkap.
- Cakupan Medan lebih pendek (9 hari) daripada Bali dan Jakarta. Perbandingan antar kota perlu dibatasi pada rentang 11-20 Oktober.
- Terdapat jam yang hilang di tiap lokasi dan tidak diinterpolasi.
- Pencilan (misalnya PM2.5 maksimum 188 dan um003 maksimum 17.200) belum ditangani.
- Tanpa suhu dan kelembapan, pengaruh cuaca terhadap partikulat tidak dapat dianalisis.

## Status

- [x] Penggabungan dan pembersihan data
- [ ] Visualisasi eksplorasi
- [ ] Penanganan pencilan
- [ ] Pemodelan
