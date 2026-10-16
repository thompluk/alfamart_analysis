# 🛒 Evaluasi Efektivitas Promo JSM & Deteksi Kebocoran Margin akibat Reseller di Alfamart

## 1. Project Overview
Proyek ini menganalisis data historis transaksi kasir (*Point of Sale*) Alfamart untuk mengevaluasi efektivitas strategi **Promo JSM (Jumat, Sabtu, Minggu)** serta mengidentifikasi risiko **kebocoran margin (*margin leakage*)** akibat aktivitas pembelian borongan oleh *reseller*/tengkulak.

**Problem Statement:**  
Manajemen Alfamart mengindikasikan terjadinya erosi profitabilitas pada Promo JSM karena alokasi barang diskon bersubsidi disapu bersih oleh pembeli non-rumah-tangga (*reseller/B2B*). Kondisi ini memicu kelangkaan stok (*out-of-stock*) bagi target pelanggan rumah tangga (B2C) dan membebankan biaya subsidi diskon yang tidak tepat sasaran.

**Key Objectives:**
- **Objective 1 (Evaluasi Efektivitas & Halo Effect):** Menguji secara statistik apakah Promo JSM berhasil meningkatkan ukuran keranjang belanja (*basket size*) dan pendorong penjualan barang reguler (*loss leader/cross-selling*) pada konsumen B2C.
- **Objective 2 (Kuantifikasi Dampak Finansial & Stok):** Mengukur total kerugian subsidi diskon yang salah sasaran (*misdirected subsidy*) dan persentase stok nasional yang tergerus (*inventory drain*).
- **Objective 3 (Profil & Episentrum Reseller):** Mengidentifikasi karakteristik pembeli borongan serta memetakan lokasi gerai yang menjadi pusat kebocoran stok.
- **Objective 4 (Perumusan Kebijakan POS Limit):** Menerjemahkan batas kewajaran konsumsi (*IQR thresholding*) menjadi rekomendasi pembatasan kuantitas otomatis (*dynamic hard limit*) pada sistem kasir POS.

---

## 2. Data Sources
Analisis ini menggunakan dataset historis transaksi kasir periode **Januari 2021 – Agustus 2025** yang terorganisir dalam skema bintang (*Star Schema*) dengan total **455.148 baris transaksi** dari lebih dari **1.000 gerai** dan **500 SKU produk**:

- **[Fact Table] `alfa_pos_transactions.csv`**: Mencatat 455.148 baris detail transaksi kasir (`receipt_id`, `store_id`, `sku_id`, `member_id`, `trx_time`, `qty`, `total_price`, `discount_amount`, `transaction_status`).
- **[Dimension] `alfa_stores.csv`**: Data master 1.003 gerai Alfamart (`store_id`, `city`, `province`, `store_type`, `store_format`, `has_self_checkout`).
- **[Dimension] `alfa_products.csv`**: Katalog master 507 SKU produk (`sku_id`, `product_name`, `category`, `brand`, `unit_price`, `is_jsm_promo`, `jsm_discount_pct`).
- **[Dimension] `alfa_members.csv`**: Profil keanggotaan 48.214 member terdaftar (`member_id`, `membership_tier`, `points_balance`, `registration_date`).
- **[Satellite] `alfa_complaint_tickets.csv`**: Data 10.759 tiket komplain pelanggan (`ticket_id`, `receipt_id`, `complaint_category`, `csat_score`, `disputed_amount`).

---

## 3. Technologies Used
- **Programming Language:** Python 3.12 (Pandas, NumPy, SciPy, Statsmodels)
- **Data Analytics & Statistics:** 
  - Non-Parametric Testing: *Mann-Whitney U Test* (Uji pergeseran *basket size* mayoritas)
  - Parametric & Effect Size: *Welch's T-Test*, *Cohen's d*, 95% *Confidence Interval*
  - Proportion Testing: *Two-Proportion Z-Test* (Uji lonjakan kepadatan anomali)
  - Outlier & Anomaly Detection: *Interquartile Range (IQR) Contextual Thresholding*
- **Data Visualization:** Matplotlib, Seaborn
- **Development Environment:** Jupyter Notebook (`.ipynb`)
- **Version Control:** Git

---

## 4. Project Structure

```
├── README.md          <- The top-level README for developers using this project.
│
├── data
│   ├── raw            <- Data from third party sources (Star Schema CSVs).
│   └── cleaned        <- The data that has been cleaned (Filtered VOID & normalized).
│
└── notebooks          <- Jupyter notebooks for exploratory analysis & hypothesis testing.
                         e.g., `Alfamart_Analysis_GammaGroup.ipynb`.
```

---

## 5. Summary of Findings

### 5.1 Business Insights
1. **Keberhasilan Strategi *Loss Leader* & *Halo Effect*:**
   - Melalui pengujian statistik (*Welch's T-Test* & *Cohen's d*) pada transaksi normal tanpa diskon (`discount_amount == 0`), terbukti bahwa Promo JSM sukses berfungsi sebagai pemancing trafik. Pelanggan B2C terbukti meningkatkan pembelian produk reguler bermargin penuh saat akhir pekan.
2. **Lonjakan Pembelian Anomali (*Promo Hunters / Resellers*):**
   - Hasil *Two-Proportion Z-Test* membuktikan adanya lonjakan transaksi di atas batas wajar konsumsi rumah tangga (\\(Q3 + 1.5 \times IQR\\)), meningkat dari **8,14% pada hari biasa** menjadi **39,92% saat periode JSM** (\\(p\text{-value} = 0.00\\)).
3. **Eksploitasi Fasilitas Keanggotaan (*Super Member*):**
   - Sebanyak **72,0% dari total transaksi borongan JSM** dilakukan menggunakan kartu *Member*. Hal ini mengindikasikan bahwa *reseller* beroperasi secara teredukasi dari dalam sistem dan memicu beban ganda (*double margin hit*: diskon harga + poin loyalti).
4. **Episentrum Kebocoran Stok (*Geotargeting Analysis*):**
   - Sebanyak **90,3% transaksi borongan JSM** (19.763 dari 21.886 struk anomali) terpusat pada gerai bertipe **"Dekat Pasar/Grosir"**.
   - Sebaliknya, proporsi borongan di gerai tipe **"Residential/Perumahan"** justru menyusut di bawah 10%, membuktikan bahwa stok subsidi yang seharusnya dinikmati ibu rumah tangga tersedot habis oleh pedagang pasar.

### 5.2 Actionable Recommendations
1. **Implementasi *Dynamic Hard Limit* pada Kasir POS:**
   - Terapkan pembatasan kuantitas maksimal otomatis (*max qty per receipt/member*) pada sistem kasir POS dan aplikasi Alfagift khusus untuk SKU promo rawan borongan (misal: Minyak Goreng, Indomie, Aqua) berdasarkan parameter \\(Q3 + 1.5 \times IQR\\).
2. **Penerapan Kebijakan *Geotargeting Restrictions*:**
   - Terapkan aturan batas kuantitas ketat difokuskan pada klaster gerai bertipe **"Dekat Pasar/Grosir"**, sementara gerai tipe **"Residential/Perumahan"** tetap diberikan kuota lebih fleksibel agar kenyamanan belanja pelanggan B2C tidak terganggu.
3. **Restrukturisasi Mekanik Promosi (*Cross-Selling Requirement*):**
   - Ubah skema diskon langsung menjadi promosi bersyarat, misalnya: *"Harga Diskon JSM Minyak Goreng berlaku jika disertai pembelian non promo minimal Rp50.000"* untuk menyaring *reseller* murni sekaligus mengamankan margin silang.
4. **Audit *Super Member* & Pengawasan Internal:**
   - Lakukan pemantauan dan audit operasional untuk mengantisipasi praktik "tembak struk" oleh oknum kasir di area pasar.

---

## 6. Contact
- **Name:** Thomas Dany Haryanto
- **Email:** thomasdany38@gmail.com
- **LinkedIn:** [Thomas Dany Haryanto](www.linkedin.com/in/thomas-dany-haryanto)
