# Produk Apa yang Sering Dibeli Bersamaan? Market Basket Analysis Toko Online (Online Retail UK)

Proyek portofolio analisis data dengan Python. Memakai algoritma **Apriori** untuk menemukan produk yang sering muncul di keranjang belanja yang sama, sebagai dasar keputusan penataan produk, paket bundling, dan pengelolaan stok.

---

## Ringkasan eksekutif

**Pertanyaan:** Produk apa yang sering dibeli bersamaan, dan apa yang bisa dilakukan toko dengan informasi itu?

**Jawaban:** Dari **18,273 transaksi** yang berisi lebih dari satu produk, ditemukan **467 aturan** (282 kombinasi produk unik). Hampir semuanya berasal dari **produk satu seri atau varian**: pembeli yang mengambil satu varian cenderung melengkapi serinya.

- **Dampak terbesar:** pembeli cangkir dan lepek Regency hijau cenderung membeli yang motif mawar juga. Dari 100 transaksi yang memuat yang hijau, sekitar **76** memuat yang mawar. Kombinasi ini muncul di sekitar **4.2%** transaksi multi-produk.
- **Hubungan terkuat:** satu set *herb marker* (thyme, mint, rosemary, parsley, chives, basil) hampir selalu dibeli bersamaan. Dari 100 transaksi yang memuat thyme dan mint, sekitar **90** juga memuat rosemary dan parsley.

**Rekomendasi:** jual produk satu seri sebagai set lengkap, tampilkan varian seri yang sama berdampingan, dan jaga stok antar varian tetap seimbang. Efeknya diuji dulu sebelum diterapkan penuh.

---

## Daftar isi

1. [Pertanyaan bisnis](#1-pertanyaan-bisnis)
2. [Data dan istilah](#2-data-dan-istilah)
3. [Cara kerja singkat](#3-cara-kerja-singkat)
4. [Membersihkan data](#4-membersihkan-data)
5. [Temuan](#5-temuan)
6. [Rekomendasi](#6-rekomendasi)
7. [Cara mengukur keberhasilan](#7-cara-mengukur-keberhasilan)
8. [Keterbatasan](#8-keterbatasan)
9. [Catatan teknis](#9-catatan-teknis)
10. [Cara menjalankan](#10-cara-menjalankan)
11. [Isi repositori](#11-isi-repositori)
12. [Alat dan kemampuan yang dipakai](#12-alat-dan-kemampuan-yang-dipakai)

---

## 1. Pertanyaan bisnis

Sebuah toko online ingin tahu **produk apa yang cenderung dibeli satu paket**, untuk tiga keputusan:

| Keputusan | Contoh pemakaian hasil |
|-----------|------------------------|
| Penataan produk | Menampilkan produk "sering dibeli bersamaan" di halaman produk, atau menaruhnya berdekatan di rak |
| Bundling | Membuat paket dari produk yang biasanya dibeli bersama |
| Stok | Memastikan produk pasangan tidak habis saat produk utamanya dipromosikan |

---

## 2. Data dan istilah

**Sumber:** dataset Online Retail UK, transaksi toko online selama setahun. Data yang sama dengan proyek [Analisis Retensi Pelanggan](tautan-proyek-retensi). [tautan sumber data]

| Hal | Isi |
|-----|-----|
| Periode data | 1 Desember 2010 - 9 Desember 2011 (seluruh periode dipakai, karena yang dilihat isi keranjang, bukan tren bulanan) |
| Ukuran data mentah | 541,909 baris, 8 kolom: `InvoiceNo`, `StockCode`, `Description`, `Quantity`, `InvoiceDate`, `UnitPrice`, `CustomerID`, `Country` |
| Satu baris berarti | satu produk dalam satu transaksi |

**Istilah yang dipakai** (dijelaskan dengan bahasa sehari-hari):

| Istilah | Arti sederhana |
|---------|----------------|
| Transaksi | Satu nomor invoice, ibarat satu struk belanja |
| Transaksi multi-produk | Struk yang berisi **lebih dari satu** jenis produk. Struk berisi satu produk tidak punya "pasangan", jadi tidak dianalisis |
| Aturan (A → B) | Pola "yang membeli A cenderung membeli B juga" |
| **Support** | Dari 100 transaksi multi-produk, berapa yang memuat kombinasi ini. Semakin besar, semakin banyak transaksi yang terdampak |
| **Confidence** | Dari transaksi yang memuat A, berapa persen yang juga memuat B. Semakin tinggi, semakin bisa diandalkan |
| **Lift** | Berapa kali lebih sering A dan B dibeli bersamaan dibanding kalau pelanggan memilih produk secara acak. Di atas 1 berarti ada hubungan |

---

## 3. Cara kerja singkat

1. **Bersihkan** data transaksi (buang retur, biaya ongkir, data tidak valid).
2. **Bentuk keranjang:** untuk tiap transaksi, catat produk apa saja yang ada di dalamnya.
3. **Cari kombinasi yang cukup sering** (Apriori): hanya kombinasi yang muncul di minimal **1%** transaksi multi-produk yang dipertimbangkan.
4. **Pilih aturan yang bisa diandalkan:** hanya aturan dengan confidence minimal **70%** yang dipertahankan.

---

## 4. Membersihkan data

![Log pembersihan data](images/01_log_pembersihan.png)
<!-- Gambar: tangkapan layar tabel log_df (output cell 6 di notebook) -->

Setiap langkah dicatat jumlah baris dan invoice-nya, supaya semua angka bisa ditelusuri.

| Langkah | Alasan | Baris dibuang | Invoice dibuang |
|---------|--------|---------------|-----------------|
| Hapus baris duplikat persis | Dugaan salah catat (asumsi, bukan bukti) | 5,268 | 0 |
| Hapus baris dengan nilai tidak valid | Quantity atau UnitPrice tidak bisa dibaca sebagai angka | 0 | 0 |
| Hapus transaksi pembatalan (invoice diawali "C") | Itu retur, bukan pembelian | 9,251 | 3,836 |
| Hapus Quantity atau UnitPrice bernilai 0 atau negatif | Koreksi stok atau salah input | 2,512 | 2,104 |
| Hapus kode non-produk (ongkir, biaya bank, data uji) | Kalau dibiarkan, "ongkir" akan tampak seperti produk yang sering dibeli bersama | 2,374 | 187 |
| Hapus baris tanpa nama produk | Tidak bisa dikenali | 0 | 0 |

Beberapa keputusan lain:

- **Satu kode produk dengan beberapa nama** diseragamkan ke nama yang paling sering dipakai, supaya produk yang sama tidak terhitung sebagai produk berbeda.
- **Baris tanpa nomor pelanggan tetap dipertahankan.** Analisis keranjang memakai nomor invoice, bukan pelanggan, jadi nomor pelanggan tidak dibutuhkan.
- **Outlier jumlah beli tidak dibuang.** Apriori hanya melihat produk itu ada atau tidak di sebuah transaksi, bukan berapa banyak yang dibeli. Membuang baris berjumlah besar justru bisa menghilangkan produk dari sebuah transaksi dan menggeser hasil tanpa alasan yang jelas.

**Hasil:** dari 541,909 baris (25,900 invoice) tersisa **522,504 baris dan 19,773 transaksi** yang siap dianalisis, dengan **18,273 transaksi multi-produk** (92.4% dari seluruh transaksi).

**Pengecekan silang:** setelah tiga langkah pertama (duplikat, pembatalan, jumlah atau harga 0 atau negatif), jumlah barisnya 524,878, sama dengan tabel kerja di proyek retensi. Kedua proyek memakai aturan pembersihan yang konsisten.

---

## 5. Temuan

### Temuan 1: Pasangan dengan dampak terbesar adalah cangkir dan lepek Regency

![Aturan dengan support tertinggi](images/02_aturan_support_tertinggi.png)
<!-- Gambar: tangkapan layar tabel product_association (beberapa baris teratas, urut support tertinggi) -->

| | Nilai | Artinya |
|---|-------|---------|
| Aturan | cangkir-lepek Regency hijau → cangkir-lepek Regency mawar | |
| Support | 4.2% | Sekitar 770 dari 18,273 transaksi multi-produk memuat keduanya |
| Confidence | 75.7% | Dari 100 transaksi yang memuat yang hijau, sekitar 76 juga memuat yang mawar |
| Lift | 13.0 | Dibeli bersamaan 13 kali lebih sering daripada kalau acak |

**Artinya:** ini aturan yang menyentuh paling banyak transaksi, jadi paling berpotensi berdampak kalau dijadikan paket atau ditampilkan berdampingan.

### Temuan 2: Hubungan paling rapat ada di satu set herb marker

![Aturan dengan lift tertinggi](images/03_aturan_lift_tertinggi.png)
<!-- Gambar: tangkapan layar tabel top_lift (10 aturan lift tertinggi) -->

Kesepuluh aturan dengan lift tertinggi seluruhnya berasal dari satu set: *herb marker* thyme, mint, rosemary, parsley, chives, dan basil.

| | Nilai | Artinya |
|---|-------|---------|
| Aturan teratas | thyme + mint → rosemary + parsley | |
| Support | 1.0% | Sekitar 186 transaksi |
| Confidence | 90.3% | Dari 100 transaksi yang memuat thyme dan mint, sekitar 90 juga memuat rosemary dan parsley |
| Lift | 76.7 | Hampir selalu dibeli sebagai satu set |

**Artinya:** hubungan ini sangat kuat, tetapi sempit. Sepuluh baris itu satu cerita (satu set produk dalam variasi yang berbeda), bukan sepuluh temuan terpisah.

### Temuan 3: Di luar herb marker, polanya sama: varian satu seri

![Aturan lift tertinggi di luar herb marker](images/04_aturan_selain_herb_marker.png)
<!-- Gambar: tangkapan layar tabel bukan_herb (aturan lift tertinggi setelah herb marker dikeluarkan) -->

Seri lain yang muncul: set teh Regency (pink, hijau, mawar), seri mainan Poppy's Playhouse (kitchen, bathroom, livingroom), dekorasi kayu (pohon dan stocking), lilin polkadot (biru dan pink), serta toples selai (tutup hijau dan pink). Contohnya, tea plate Regency pink → hijau: confidence 91.1%, lift 43.8.

**Artinya:** pembeli cenderung **melengkapi seri atau varian**. Itu temuan yang sah, tetapi mudah ditebak. Pasangan lintas kategori (misalnya produk dapur dengan produk dekorasi) tidak muncul di batas support 1%.

### Temuan 4: Angka support terlihat kecil, dan itu wajar

Aturan dengan support tertinggi hanya 4.2%. Penyebabnya:

- Toko ini menjual **ribuan produk berbeda**, sehingga isi keranjang sangat beragam. Sebuah pasangan produk jarang muncul di banyak transaksi sekaligus.
- Lift yang sangat tinggi hampir selalu datang dari produk yang jarang dibeli, sehingga support-nya kecil.

**Cara membaca:** support menunjukkan seberapa besar dampaknya, confidence seberapa bisa diandalkan, dan lift seberapa kuat hubungannya. Ketiganya dibaca bersama, tidak cukup hanya satu.

---

## 6. Rekomendasi

Dua aturan teratas sama-sama berasal dari produk satu seri, jadi rekomendasinya diarahkan ke penjualan seri:

| # | Rekomendasi | Alasan |
|---|-------------|--------|
| 1 | **Jual set lengkap.** Tawarkan paket seri (misalnya set herb marker lengkap) di samping penjualan satuan | Pembeli satu varian cenderung membeli varian lain dari seri yang sama |
| 2 | **Tampilkan varian seri yang sama berdampingan** di halaman produk atau di rak | Memudahkan pelanggan melengkapi seri yang sudah dipilih |
| 3 | **Jaga stok seimbang antar varian.** Saat satu varian dipromosikan atau habis, pantau varian pasangannya | Varian dalam satu seri sering dibeli bersamaan |
| 4 | **Langkah berikutnya:** cari pasangan lintas seri atau kategori dengan menurunkan batas support | Aturan satu seri mudah ditebak, jadi nilai tambahnya kecil |

---

## 7. Cara mengukur keberhasilan

Aturan asosiasi hanya menunjukkan produk yang **sering dibeli bersamaan**, bukan bukti bahwa paket atau penataan baru akan menaikkan penjualan. Karena itu diuji dulu:

1. Pilih beberapa produk dengan aturan kuat (misalnya cangkir-lepek Regency).
2. Tampilkan fitur "sering dibeli bersamaan" atau paket seri di **sebagian** halaman produk, dan biarkan sebagian lainnya seperti biasa (kelompok pembanding).
3. Bandingkan, misalnya, persentase transaksi yang memuat dua produk atau lebih di kedua kelompok. Selisihnya adalah efek perubahan tersebut.

---

## 8. Keterbatasan

- **Hubungan bukan sebab-akibat.** Produk yang sering dibeli bersamaan tidak berarti satu produk menyebabkan pembelian produk lain.
- **Hanya transaksi multi-produk** yang dianalisis. Hasilnya tidak menggambarkan pembeli yang hanya membeli satu jenis produk.
- **Batas support 1%** membuang pasangan yang lebih jarang. Pasangan lintas kategori kemungkinan ada di bawah batas itu dan tidak terlihat.
- **Lift tertinggi berasal dari produk satu seri**, jadi aturannya mudah ditebak.
- **Produk dengan nama sama tetapi kode berbeda** dianggap satu produk, karena keranjang dibentuk dari nama produk.
- **Kode non-produk dibuang berdasarkan pola kode** (lima angka, kadang diikuti huruf). Kode produk yang tidak berpola itu ikut terbuang, jumlahnya kecil.
- **Baris kembar dihapus dengan asumsi salah catat.** Kalau sebagian ternyata pembelian sah, hasilnya sedikit berbeda.
- **Tidak ada data biaya atau margin.** Aturan tidak bisa diurutkan berdasarkan keuntungan.
- **Satu toko, satu tahun data.** Pola musiman tidak bisa dipastikan.

---

## 9. Catatan teknis

Pada percobaan awal, Apriori gagal dengan error kehabisan memori (butuh sekitar 13.7 GiB). Dua perbaikan yang dipakai:

1. **Membuang produk yang terlalu jarang** (muncul di kurang dari 1% transaksi multi-produk) sebelum Apriori. Produk seperti itu tidak mungkin ada di kombinasi yang lolos batas 1%, jadi **hasilnya tidak berubah**, tetapi tabel keranjang jauh lebih kecil. Dari ribuan produk, sekitar 898 yang tersisa.
2. **Mode hemat memori** (`low_memory=True`) pada fungsi Apriori.

Semua angka batas (`MIN_SUPPORT`, `MIN_CONFIDENCE`) diatur di satu tempat di bagian atas notebook.

---

## 10. Cara menjalankan

1. Unduh data dari [tautan sumber data], simpan sebagai `data/Online Retail.csv`.
2. Pasang library: `pip install -r requirements.txt`.
3. Buka `project_Market_Basket_Analysis.ipynb` di Jupyter, lalu pilih **Kernel → Restart & Run All**.

---

## 11. Isi repositori

```
.
├── README.md
├── project_Market_Basket_Analysis_fix.ipynb   # notebook analisis
└── Online Retail.xlsx                         # dataset                    
```

---

## 12. Alat dan kemampuan yang dipakai

- **Python:** pandas untuk pembersihan dan penyiapan data, mlxtend untuk algoritma Apriori dan aturan asosiasi.
- **Jupyter Notebook:** analisis yang bisa diulang dan terdokumentasi.
- **Kemampuan:** menyusun keputusan pembersihan data, menerjemahkan metrik (support, confidence, lift) ke bahasa bisnis, menangani kendala memori, dan menulis batasan analisis dengan jelas.

---

**Dibuat oleh:** Agi Agustian Davi | [AgiAgustianDavi](https://www.linkedin.com/in/agi-agustian-davi/) | [Email](mailto:agidavi6@gmail.com)
