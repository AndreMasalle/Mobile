# Automobile Data Preprocessing Pipeline

Proyek ini berisi pipeline ETL untuk membersihkan dan mentransformasi dataset mobil bekas. Pipeline membaca data mentah dari folder `data/raw/`, melakukan serangkaian pembersihan dan transformasi, lalu menyimpan dataset yang sudah diproses ke `data/processed/automobileEDA_processed.csv`.

---

## Deskripsi Singkat Dataset

Dataset mentah memiliki **30 kolom** yang terdiri dari fitur kategorikal dan numerik. Kolom-kolom tersebut antara lain:

- **transaction_date**: tanggal transaksi (tidak relevan, akan dihapus)
- **symboling**: tingkat risiko asuransi (kategorikal)
- **normalized-losses**: kerugian normalisasi (numerik)
- **make**: merek mobil (kategorikal)
- **aspiration**: tipe aspirasi mesin, `std` atau `turbo` (kategorikal)
- **num-of-doors**: jumlah pintu (kategorikal)
- **body-style**: gaya bodi (kategorikal)
- **drive-wheels**: penggerak roda (kategorikal)
- **engine-location**: lokasi mesin (kategorikal)
- **wheel-base**, **length**, **width**, **height**, **curb-weight**: dimensi dan berat (numerik)
- **engine-type**, **num-of-cylinders**, **engine-size**, **fuel-system**: spesifikasi mesin (campuran kategorikal/numerik)
- **bore**, **stroke**, **compression-ratio**, **horsepower**, **peak-rpm**: parameter teknis (numerik)
- **city-mpg**, **highway-mpg**, **city-L/100km**: efisiensi bahan bakar (numerik)
- **price**: harga mobil (numerik, target potensial)
- **horsepower-binned**: kategori tenaga (ordinal: low, medium, high)
- **diesel**, **gas**: indikator bahan bakar (kategorikal biner)

Tujuan dari pipeline ini adalah menghasilkan dataset yang bersih, konsisten, dan siap digunakan untuk analisis atau pemodelan machine learning.

---

## Sumber Dataset

Dataset merupakan versi "kotor" dari **Automobile Dataset** yang umum digunakan dalam pembelajaran data science. Data mentah disediakan dalam folder `data/raw/` dengan nama `automobileEDA_dirty_training.csv`. Dataset ini sengaja diberi noise berupa missing values, duplikat, inkonsistensi format teks, dan tipe data yang tidak tepat untuk melatih keterampilan preprocessing.



## Struktur Folder Project

---
├── data/
│ ├── raw/
│ │ └── automobileEDA_dirty_training.csv # dataset mentah
│ └── processed/
│ └── automobileEDA_processed.csv # dataset hasil pipeline (otomatis dibuat)
├── pipeline.py # script utama ETL
├── requirements.txt # daftar dependency
└── README.md # dokumentasi ini


---

## Kondisi Awal Dataset

Setelah proses ekstraksi, dataset mentah memiliki karakteristik sebagai berikut:

- **Jumlah baris**: 205 (termasuk duplikat)
- **Jumlah kolom**: 26
- **Missing values**: tersebar di beberapa kolom numerik dan kategorikal
- **Duplikat**: terdapat beberapa baris duplikat
- **Tipe data**: beberapa kolom kategorikal terbaca sebagai numerik (mis. `symboling`, `diesel`, `gas`), sementara kolom teks memiliki inkonsistensi kapitalisasi dan spasi
- **Kolom tidak relevan**: terdapat kolom `transaction_date` yang tidak digunakan dalam analisis

---

## Permasalahan yang Ditemukan

1. **Data duplikat** – dapat menyebabkan bias pada model.
2. **Missing values** – perlu diimputasi agar tidak mengganggu algoritma.
3. **Format teks tidak konsisten** – nilai seperti `" TOYOTA "` dan `"toyota"` harus disamakan.
4. **Tipe data tidak sesuai** – kolom kategorikal seperti `num-of-doors`, `num-of-cylinders` seharusnya diperlakukan sebagai kategori, bukan numerik.
5. **Skala fitur numerik berbeda** – beberapa fitur memiliki rentang nilai jauh berbeda, sehingga perlu diskalakan.
6. **Kolom kategorikal perlu encoding** – model machine learning hanya menerima input numerik.
7. **Kolom ordinal** – `horsepower-binned` memiliki urutan bermakna (low < medium < high) dan harus di-encode secara ordinal.

---

## Cleaning yang Dilakukan Beserta Alasannya

| Tindakan | Alasan                                                                         |
|----------|--------------------------------------------------------------------------------|
| Menghapus baris duplikat | Menghindari data ganda yang dapat mempengaruhi statistik dan model             |
| Mengimputasi missing values pada kolom numerik dengan **median** | Median robust terhadap outlier, cocok untuk data dengan distribusi tidak normal |
| Mengimputasi missing values pada kolom kategorikal dengan **modus** | Modus untuk mempertahankan distribusi kategori                                 |
| Mengubah teks menjadi **lowercase** dan **strip** | Menyeragamkan format agar tidak ada perbedaan hanya karena kapitalisasi/spasi  |
| Mengubah tipe data kolom tertentu menjadi string | Memastikan kolom kategorikal tidak diperlakukan sebagai angka oleh transformer |
| Menghapus kolom `transaction_date` | Kolom tidak relevan untuk analisis atau modeling                               |

---

## Transformasi yang Dilakukan

1. **Scaling numerik** – `MinMaxScaler` untuk fitur numerik yang dipilih, agar rentang nilai menjadi 0–1.
2. **One‑Hot Encoding** – untuk kolom nominal: `aspiration`, `body-style`, `drive-wheels`, `engine-location`, `engine-type`, `fuel-system`. Menghasilkan kolom biner untuk setiap kategori.
3. **Ordinal Encoding** – untuk `horsepower-binned` dengan urutan `['low', 'medium', 'high']` → output `horsepower_ordinal` bernilai 0, 1, atau 2.
4. **Frequency Encoding** – untuk kolom `make`. Mengganti setiap merek dengan frekuensi relatif kemunculannya di data training → output `make_freq`.
5. **Passthrough** – kolom `num-of-doors`, `num-of-cylinders`, `symboling`, `diesel`, `gas` dipertahankan sebagai kolom tunggal setelah imputasi modus dan konversi ke string.

---

## Contoh Hasil Sebelum dan Sesudah Transformasi

### Sebelum (data mentah)

| horsepower-binned | make        | aspiration | body-style | drive-wheels | engine-type | fuel-system |
|-------------------|-------------|------------|------------|--------------|-------------|-------------|
| medium            | alfa-romero | std        | hatchback  | rwd          | ohcv        | mpfi        |
| medium            | audi        | std        | sedan      | fwd          | ohc         | mpfi        |
| medium            | audi        | std        | sedan      | 4wd          | ohc         | mpfi        |


### Sesudah (data processed)

| horsepower_ordinal | make_freq | aspiration_std | body-style_hatchback | body-style_sedan | drive-wheels_fwd | drive-wheels_4wd | engine-type_ohc | fuel-system_mpfi |
|--------------------|-----------|----------------|----------------------|------------------|------------------|------------------|-----------------|------------------|
| 1.0                | 0.014925  | 1.0            | 1.0                  | 0.0              | 0.0              | 0.0              | 0.0             | 1.0              |
| 1.0                | 0.034826  | 1.0            | 0.0                  | 1.0              | 1.0              | 0.0              | 1.0             | 1.0              |
| 1.0                | 0.034826  | 1.0            | 0.0                  | 1.0              | 0.0              | 1.0              | 1.0             | 1.0              |


---

## Jumlah Data Sebelum dan Sesudah Diproses

- **Sebelum diproses**: 205 baris, 30 kolom
- **Setelah drop duplikat**: 201 baris
- **Setelah seluruh transformasi**: 201 baris, 50 kolom

---

## Cara Menginstal Dependency

Pastikan Python 3.8+ terinstal, lalu jalankan:

```bash
pip install -r requirements.txt