# Telkomsel Device Bundling — Data Quality & IMEI Validation

**Capstone Project Modul 3 — Business Data Analytics**  
**Purwadhika Job Connector**  
**Author: Axellia Verda Qania**

---

## 📌 Project Overview

Telkomsel menjalankan program **Device Bundling**, yaitu pelanggan membeli perangkat (iPhone atau Samsung) dan memperoleh paket kuota dengan harga lebih murah. Program ini memberikan subsidi di awal dengan harapan pelanggan bertahan dalam jangka panjang.

Salah satu kontrol yang digunakan adalah pencocokan **IMEI**, yaitu membandingkan:

- `registered_imei` → IMEI perangkat yang didaftarkan saat pembelian paket bundling
- `used_imei` → IMEI perangkat yang benar-benar digunakan saat sesi internet

Jika kedua IMEI berbeda, kondisi tersebut dapat menjadi indikasi penyalahgunaan.

Namun, sebelum mismatch IMEI digunakan sebagai dasar penindakan pelanggan, perlu dipastikan terlebih dahulu apakah data tersebut cukup valid dan konsisten.

### Business Question

> **Apakah data IMEI Telkomsel cukup valid untuk dijadikan dasar menindak penyalahgunaan program Device Bundling?**

---

## 🎯 Analytical Questions

Analisis menjawab empat pertanyaan utama:

1. Berapa besar indikasi penyalahgunaan jika data mentah dipercaya apa adanya?
2. Apakah indikasi tersebut benar-benar menunjukkan perpindahan perangkat oleh pelanggan?
3. Apakah pola yang ditemukan lebih mungkin menunjukkan error sistem?
4. Kejanggalan pencatatan apa lagi yang terdapat di luar IMEI dan berapa pelanggan yang terdampak?

---

## 📊 Dataset

Analisis menggunakan tiga tabel yang saling berhubungan:

### 1. `tsel_subscribers.csv`

Data pelanggan. Satu baris mewakili satu nomor HP.

| Column | Description |
|---|---|
| `msisdn` | Nomor HP pelanggan |
| `activation_date` | Tanggal kartu SIM diaktifkan |
| `nik_kk_status` | Status verifikasi NIK/KK |
| `registered_imei` | IMEI perangkat yang didaftarkan |

### 2. `tsel_packages.csv`

Data paket. Satu baris mewakili satu jenis paket.

| Column | Description |
|---|---|
| `package_code` | Kode paket |
| `package_name` | Nama paket |
| `price` | Harga paket dalam Rupiah |

### 3. `tsel_data_usage.csv`

Data pemakaian internet. Satu baris mewakili satu sesi penggunaan.

| Column | Description |
|---|---|
| `session_id` | ID sesi |
| `msisdn` | Nomor HP pelanggan |
| `package_code` | Kode paket |
| `used_imei` | IMEI perangkat yang digunakan |
| `session_time` | Waktu sesi |
| `network_type` | Jenis jaringan |
| `payload_mb` | Jumlah data yang digunakan dalam MB |

### Relationship

```text
tsel_subscribers          tsel_packages
      │ msisdn                  │ package_code
      │                         │
      └──────► tsel_data_usage ◄┘
```

---

## 🔍 Methodology

Analisis dilakukan menggunakan tahapan:

1. **Data Loading**
   - Membaca ketiga dataset menggunakan Pandas.
   - Kolom identitas seperti `msisdn` dan IMEI dibaca sebagai string untuk menjaga format dan presisi.

2. **Exploratory Data Analysis**
   - Mengecek ukuran dan struktur data.
   - Mengecek missing values.
   - Mengecek duplikasi.
   - Mengecek statistik deskriptif.
   - Mengeksplorasi `network_type`, `used_imei`, `nik_kk_status`, dan tanggal.
   - Mengecek hubungan antar tabel.

3. **Data Cleaning & Validation**
   - Menangani format tanggal.
   - Mengidentifikasi nilai IMEI anomali.
   - Mengidentifikasi payload yang tidak wajar.
   - Memeriksa konsistensi tipe jaringan.
   - Memeriksa sesi yang terjadi sebelum tanggal aktivasi.
   - Memeriksa kualitas data NIK/KK.

4. **IMEI Mismatch Analysis**
   - Membandingkan `registered_imei` dan `used_imei`.
   - Mengukur jumlah pelanggan dan sesi yang terindikasi mismatch.
   - Menelusuri pola mismatch berdasarkan pelanggan, waktu, dan paket.

5. **Statistical Validation**
   - Menggunakan **two-proportion z-test** untuk menguji apakah tingkat kejanggalan berbeda secara signifikan antar kelompok.

6. **Business Recommendation**
   - Menilai apakah data IMEI sudah layak digunakan sebagai dasar penindakan.
   - Menyusun rekomendasi untuk perlindungan subsidi dan peningkatan kualitas data.

---

## 📈 Key Findings

### Data Quality

Dari **300.000 sesi penggunaan internet**:

- **3.013 sesi (1%)** harus dikeluarkan karena memiliki payload yang sangat tidak wajar:
  - 1.469 sesi memiliki payload negatif sebesar -150,5 MB.
  - 1.544 sesi tercatat menggunakan hampir 1 TB dalam satu sesi.
- `network_type` memiliki **10 variasi penulisan**, padahal hanya terdapat 3 jenis jaringan.
- **11.875 sesi (4%)** tercatat terjadi sebelum kartu diaktifkan, dengan selisih 1–9 hari, dan berdampak pada **10.083 pelanggan**.
- **5.202 nomor (14,86%)** memiliki status NIK/KK kosong.
- Secara keseluruhan, **18.967 dari 34.999 pelanggan (54,19%)** memiliki minimal satu penanda kualitas data yang memerlukan validasi.

### IMEI Analysis

Dari 34.999 pelanggan:

- **10.530 pelanggan (30%)** memiliki IMEI terdaftar.
- Secara mentah, **8.641 pelanggan (82,06%)** terlihat mengalami IMEI mismatch.
- Namun, seluruh **17.928 sesi bermasalah** berasal dari satu nilai IMEI yang sama:

```text
350000009999999
```

- **99,86% pelanggan** yang terkena mismatch masih memiliki sesi dengan IMEI yang benar.
- Tingkat kejanggalan relatif konsisten:
  - Per bulan: **19,67%–22,58%**
  - Per jam: **18,62%–21,84%**
  - Per hari: **19,58%–20,57%**
  - Per paket: **19,46%–20,87%**
- Paket mahal memiliki tingkat kejanggalan **19,86%**, sedangkan paket murah **20,29%**.
- Two-proportion z-test menghasilkan **p-value = 0,3196**, sehingga tidak terdapat perbedaan yang signifikan secara statistik.

---

## 💡 Conclusion

Berdasarkan pola data, **data IMEI belum cukup valid untuk dijadikan dasar langsung untuk menindak penyalahgunaan Device Bundling**.

Angka awal sebesar 82,06% mismatch terlihat sangat tinggi. Namun, setelah ditelusuri, seluruh sesi bermasalah berasal dari satu nilai IMEI yang sama dan pola mismatch muncul secara selang-seling, bukan sebagai perpindahan perangkat permanen.

Pola tersebut lebih menyerupai **proses otomatis atau masalah pencatatan sistem** daripada perilaku pelanggan. Namun, analisis ini **tidak dapat memastikan penyebab teknisnya** tanpa validasi dari tim terkait.

---

## 🚀 Recommendations

### 1. Terapkan perlindungan subsidi secara bertahap

Jangan langsung menghentikan benefit hanya karena ditemukan IMEI mismatch. Gunakan mismatch sebagai **early warning**, kemudian lakukan verifikasi pelanggan sebelum memberikan tindakan lebih lanjut.

### 2. Implementasikan IMEI Locking setelah kualitas data terverifikasi

IMEI Locking dapat digunakan untuk memastikan benefit bundling hanya digunakan pada perangkat yang sesuai. Namun, implementasinya sebaiknya dilakukan setelah akurasi data IMEI dipastikan.

### 3. Jadikan data quality sebagai syarat sebelum automated enforcement

Sebelum menerapkan blacklist, blocking, atau penghentian benefit otomatis:

- Validasi IMEI.
- Bersihkan payload yang tidak wajar.
- Standarisasi kategori jaringan.
- Pastikan status registrasi dapat ditelusuri.

### 4. Hitung ulang potensi kebocoran subsidi

Angka **82,06% tidak dapat langsung dianggap sebagai fraud rate**. Setelah anomali IMEI diperbaiki, perusahaan perlu menghitung kembali:

- jumlah pelanggan yang benar-benar menggunakan perangkat berbeda,
- benefit yang digunakan tidak sesuai ketentuan,
- potensi subsidi yang dapat diselamatkan,
- biaya monitoring dan verifikasi.

---

## 🛠️ Tools & Technologies

- **Python**
- **Pandas** — data manipulation
- **NumPy** — numerical analysis
- **Matplotlib** — visualization
- **Seaborn** — visualization
- **Statsmodels** — statistical testing
- **Jupyter Notebook** — analysis environment

---

## 📁 Repository Structure

```text
telkomsel-device-bundling-analysis/
│
├── data/
│   ├── tsel_data_usage.csv
│   ├── tsel_subscribers.csv
│   └── tsel_packages.csv
│
├── notebook/
│   └── Capstone_Telkomsel_Final.ipynb
│
├── README.md
└── .gitignore
```

> Jika dataset bersifat confidential atau tidak boleh dipublikasikan, jangan upload file CSV ke repository public. Simpan hanya notebook dan gunakan data dictionary/sample data sebagai pengganti.

---

## ⚠️ Limitations

- Penyebab teknis kejanggalan IMEI belum dapat dipastikan hanya dari dataset.
- Tidak tersedia data lokasi sehingga masalah tidak dapat dipetakan berdasarkan wilayah.
- `is_bundling_customer` dalam analisis berarti **memiliki IMEI terdaftar**, bukan bukti bahwa pelanggan pasti membeli paket bundling.
- Data hanya mencakup periode **2023**.
- Tidak tersedia data pembanding berupa kasus penyalahgunaan yang sudah terbukti.

---

## 👤 Author

**Axellia Verda Qania**  
Communication Science Graduate — Universitas Diponegoro  
Business Data Analytics — Purwadhika Job Connector

