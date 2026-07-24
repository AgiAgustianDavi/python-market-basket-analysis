# Market Basket Analysis - Online Retail Customer Behaviour Analysis

![Python](https://img.shields.io/badge/Python-3.8%2B-blue?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-1.5%2B-150458?style=for-the-badge&logo=pandas&logoColor=white)
![Mlxtend](https://img.shields.io/badge/Mlxtend-0.21%2B-red?style=for-the-badge)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?style=for-the-badge&logo=jupyter&logoColor=white)

Proyek ini berfokus pada **Market Basket Analysis (MBA)** menggunakan algoritma **Apriori** untuk menganalisis perilaku pembelian pelanggan (*customer purchasing behavior*) dari dataset transaksi retail online. Tujuan utamanya adalah untuk menemukan asosiasi produk (*association rules*) yang dapat dimanfaatkan untuk strategi pemasaran, penataan barang (*product placement*), serta rekomendasi *cross-selling* dan *bundling*.

---

## Context & Problem Statement

Dalam industri e-commerce dan retail, memahami produk mana yang sering dibeli secara bersamaan sangat penting untuk menyusun strategi penjualan yang efektif. 
Tanpa analisis yang tepat:
- Kesempatan *cross-selling* dan *up-selling* terbuang sia-sia.
- Tata letak produk (baik di toko fisik maupun antarmuka e-commerce) kurang optimal.
- Penawaran promo/bundle tidak sesuai dengan preferensi riil pelanggan.

Proyek ini menyelesaikan masalah tersebut dengan mengestraksi *frequent itemsets* dan aturan asosiasi (*association rules*) menggunakan kriteria **Support**, **Confidence**, dan **Lift**.

---

## Dataset Overview

* **Source File:** `Online Retail Data.csv`
* **Raw Records:** ~461,773 baris & 7 kolom
* **Cleaned Records:** 350,092 baris transaksi & 15,004 *unique baskets* (transaksi dengan lebih dari 1 jenis produk)
* **Atribut Utama:**
  - `order_id`: ID Unik untuk setiap transaksi (Order/Invoice)
  - `product_code`: Kode produk
  - `product_name`: Nama produk (setelah pembersihan & normalisasi nama ter-frekuensi)
  - `quantity`: Jumlah produk yang dibeli
  - `order_date`: Tanggal dan waktu transaksi
  - `price`: Harga per unit produk
  - `customer_id`: ID Unik pelanggan
  - `amount`: Calculated column (`quantity` × `price`)

---

## Data Preprocessing & Cleaning Workflow

Prosedur pembersihan data mencakup beberapa tahap penting:
1. **Handling Missing Values:** Menghapus baris yang kehilangan `customer_id` atau `product_name`.
2. **Text Normalization:** Mengubah seluruh `product_name` menjadi huruf kecil (*lowercase*).
3. **Filtering Test Transactions:** Menghapus data uji coba yang memiliki kata `'test'` pada `product_code` atau `product_name`.
4. **Handling Cancellations:** Menghapus transaksi pembatalan (ID yang diawali dengan huruf `'C'`).
5. **Adjusting Quantities & Prices:** Mengubah nilai `quantity` negatif menjadi positif dan memfilter `price > 0`.
6. **Resolving Product Name Ambiguities:** Menyeragamkan nama produk berdasarkan nama yang paling sering muncul (*mode*) untuk tiap `product_code`.
7. **Outlier Removal:** Menghapus pencilan (*outliers*) pada kolom `quantity` dan `amount` menggunakan ambang batas Z-score ($|Z| < 3$).
8. **One-Hot Encoding (Basket Matrix):** Membuat pivot table matriks transaksi × produk, kemudian di-encode menjadi boolean/binary flag (*True/False*).
9. **Single-Item Filtering:** Memfilter transaksi yang hanya berisi 1 jenis produk untuk fokus pada hubungan antar-produk.

---

## Methodology & Analysis

Algoritma **Apriori** diterapkan dari *library* `mlxtend` dengan kriteria:
* **Minimum Support:** `0.01` (1%)
* **Minimum Confidence:** `0.70` (70%)

Metrics utama yang dievaluasi:
* **Support:** Mengukur seberapa sering gabungan produk muncul dalam seluruh transaksi.
* **Confidence:** Mengukur seberapa sering produk $B$ dibeli ketika produk $A$ dibeli.
* **Lift:** Mengukur sekuat apa hubungan asosiasi dibanding jika kedua produk dibeli secara independen (Nilai $> 1$ menunjukkan asosiasi positif kuat).

---

## Key Findings

Dari total **86 aturan asosiasi** yang dihasilkan, berikut beberapa temuan teratas:

| Antecedent (Produk A) | Consequent (Produk B) | Support | Confidence | Lift |
| :--- | :--- | :---: | :---: | :---: |
| `red hanging heart t-light holder` | `white hanging heart t-light holder` | **4.25%** | **72.22%** | **4.06** |
| `sweetheart ceramic trinket box` | `strawberry ceramic trinket box` | **3.75%** | **76.05%** | **10.14** |
| `toilet metal sign` | `bathroom metal sign` | **2.17%** | **80.49%** | **19.83** |
| `red retrospot sugar jam bowl` | `red retrospot small milk jug` | **1.68%** | **70.99%** | **19.12** |
| `painted metal pears assorted` | `assorted colour bird ornament` | **1.66%** | **75.68%** | **9.64** |

### Insight Penting:
1. **Hanging T-Light Holder Series:** 
   - Produk `red hanging heart t-light holder` muncul pada ~5.88% transaksi.
   - Ketika pelanggan membeli versi **Red**, sebesar **72.22%** dari mereka juga akan membeli versi **White**.
   - Keduanya memiliki kekuatan asosiasi **4.06x lebih besar** daripada dibeli secara terpisah.
2. **Trinket Box & Sign Series:** 
   - Pasangan barang dekoratif serupa (misal: `toilet metal sign` ➔ `bathroom metal sign`) memiliki kecenderungan dibeli bersama hingga Confidence **>80%** dan Lift mencapai **19.83x**.

---

## Business Recommendations & Actionable Insights

Berdasarkan hasil analisis aturan asosiasi di atas, berikut adalah rekomendasi strategis yang dapat diterapkan:

### 1. Product Placement & Merchandising Strategy
* **Toko Fisik:** Letakkan produk dengan nilai *Lift* tinggi (seperti versi Merah dan Putih dari `hanging heart t-light holder`, atau `toilet metal sign` & `bathroom metal sign`) di rak yang berdekatan atau bersebelahan.
* **Toko Online / E-Commerce:** Tampilkan rekomendasi *"Frequently Bought Together"* atau *"Customers Who Bought This Also Bought..."* secara otomatis pada halaman detail produk ketika pembeli menambahkan `red hanging heart t-light holder` ke keranjang.

### 2. Product Bundling & Promotional Packaging
* **Cross-Product Bundles:** Buat paket *bundle* promo hemat (misal: Paket Dekorasi Rumah/Kamar Mandi) yang menggabungkan produk *antecedent* dan *consequent* dengan penawaran diskon khusus.
* **Variasi Warna/Set:** Sediakan paket multi-warna (misal: Set *Trinket Box Strawberry & Sweetheart*) dalam satu kemasan penjualan.

### 3. Inventory & Supply Chain Management
* **Synchronized Stocking:** Karena pembeli cenderung membeli kedua produk secara bersamaan, stok persediaan untuk produk *consequent* (seperti `white hanging heart t-light holder`) harus disesuaikan dan dipantau agar tidak kehabisan stok saat produk *antecedent* dipromosikan.
