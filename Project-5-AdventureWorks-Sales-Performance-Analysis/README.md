# AdventureWorks Sales Performance Analysis

## Deskripsi Proyek
Sebagai Analis Data untuk AdventureWorks Corp., saya menangani tantangan bisnis yang kritis: **mengidentifikasi penyebab akar penurunan keuntungan yang signifikan pada tahun 2022 meskipun volume penjualan meningkat**, dan memberikan rekomendasi aksi untuk pemulihan.

### Pernyataan Masalah
Profitabilitas menurun tajam dari tahun 2021 ke tahun 2022. Menganalisis faktor-faktor multidimensi termasuk tren penjualan, segmen pelanggan, kinerja produk, pengembalian, dan variasi wilayah untuk menemukan pendorong seperti erosi margin, kategori dengan pengembalian tinggi, dan wilayah dengan kinerja buruk.

## Objectives
1. ...

## Dataset
- ...

## EDA 

> **Analisis EDA:** Profit Decline 2022 – Volume Naik 25% Tapi Profit Turun 2,14%  
> **Tools:** DuckDB (SQL) + Micrososft Excel / LibreOffice Calc

---

## 📌 Daftar Isi

<details>
<summary><b>1. Pendahuluan & Problem Statement</b></summary>

### 1.1. Latar Belakang
Sebagai Data Analyst AdventureWorks Corp., ditemukan anomali: **penjualan meningkat drastis** namun **profit justru menurun**.

### 1.2. Problem Statement
| Metrik | 2021 | 2022 | Perubahan |
|--------|------|------|-----------|
| **Volume (Qty)** | 36.230 | 45.314 | **+25,07% ↗️** |
| **Profit Bersih** | Rp 3.690.735 | Rp 3.611.732 | **-2,14% ↘️** |
| **Margin %** | 42,55% | 42,34% | -0,21 pp |
| **Returns (unit)** | 770 | 972 | +26,2% |

**Pertanyaan:**  
*"Mengapa profit turun meski penjualan naik? Driver utamanya apa?"*

### 1.3. Pendekatan Analisis
- **SQL (DuckDB):** Ekstrak data agregat per tahun, kategori, dan wilayah
- **Microsoft Excel / LibreOffice Calc:** DataPilot (PivotTable), filter interaktif, dashboard visual

</details>

<details>
<summary><b>2. SQL EDA Queries (DuckDB)</b></summary>

### 2.1. Data Sources
```sql
-- Dataset terdiri dari 3 file CSV:
-- 1) overall_metrics.csv      → Ringkasan tahunan
-- 2) top_drivers_dec.csv      → Breakdown per kategori produk
-- 3) dec_by_territory.csv     → Breakdown per benua
```

### 2.2. SQL untuk Overall Metrics
- **Fungsi:** Aggregate qty, revenue, cost, profit, returns per tahun
- **Advanced SQL:** CTE (`WITH`) chaining, `LAG()` untuk YoY, `COALESCE()` untuk null returns
- **Fokus:** Membuktikan volume naik + profit turun

### 2.3. SQL untuk Top Drivers (per Kategori)
- **Fungsi:** Breakdown per kategori (Bikes, Clothing, Accessories)
- **Advanced SQL:** `ROW_NUMBER()` untuk ranking penurunan profit
- **Temuan:** Bikes profit delta **-Rp169K** (rank 1)

### 2.4. SQL untuk Wilayah (Territory)
- **Fungsi:** `LEFT JOIN returns` untuk menghitung net sales setelah return
- **Temuan:** Pacific net sales 2022 = **Rp34K** (sebelumnya Rp4,27jt)

### 2.5. Export to CSV
```sql
COPY (SELECT ...) TO 'datasets/outputs/prof_dec/overall_metrics.csv' (FORMAT CSV, HEADER);
COPY (SELECT ...) TO 'datasets/outputs/prof_dec/top_drivers_dec.csv' (FORMAT CSV, HEADER);
COPY (SELECT ...) TO 'datasets/outputs/prof_dec/dec_by_territory.csv' (FORMAT CSV, HEADER);
```

</details>

<details>
<summary><b>3. EDA di Excekl / Calc</b></summary>

### 3.1. Import & Clean Data
- File → Open → pilih CSV (separator koma)
- **Handle NaN:** Find & Replace (Ctrl+H) → ganti `null` → `0`)
- Gabung 3 CSV jadi 1 sheet: `01_decline_profit.csv` (26 kolom, 14 baris)

### 3.2. Membuat DataPilot (PivotTable)
- Data → Pivot Table → Layout:
  - **Row:** `year`
  - **Page (filter):** `categoryname`, `continent`, `source`, `decline_rank`
  - **Data:** `total_qty` (Sum), `net_profit` (Sum), `profit_yoy_pct` (Avg), `returns_per_qty` (Avg)

### 3.3. 6 Filter Combo untuk Eksplorasi
| # | Filter | Insight Kunci |
|---|--------|---------------|
| 1 | `source=Categories` + `decline_rank≤3` | Bikes rank 1: -Rp169K |
| 2 | `source=Territory` + `continent=Pacific` | Pacific net_sales anjlok 99% |
| 3 | `year=2022` + `profit_yoy_pct<0` | Profit -2,14% → Hypothesis confirmed |
| 4 | `categoryname=Bikes` + `source=Categories` | Margin erosi spesifik Bikes |
| 5 | `returns_per_qty>200` + `Territory` | Pacific returns tertinggi |
| 6 | Kombinasi Categories+2022+rank≤2 | Bikes turun vs Clothing naik |

### 3.4. Dashboard Index (Hyperlinked TOC)
1. Sheet `Dashboard_Index` berisi tabel dengan hyperlink ke 6 filter sheet
2. Kolom: `No | Sheet Name | Insight | Impact | Action | Status`
3. Klik → lompat ke sheet terkait

</details>

<details>
<summary><b>4. Visualisasi Dashboard</b></summary>

5 grafik utama di sheet `Viz_Dashboard`:

### 4.1. Volume vs Profit YoY (Combo Chart)
- Volume bar hijau naik, profit line merah turun → **Anomali jelas**

### 4.2. Profit Delta per Kategori (Horizontal Bar)
- Bikes bar merah ke kiri (-Rp169K), Clothing/Apparel hijau ke kanan

### 4.3. Net Sales per Wilayah 2022 (Column)
- Pacific bar merah pendek, EU/NA tinggi → **Wilayah krisis**

### 4.4. Returns Heatmap (Table + Conditional Color)
- Pacific 253/unit → merah terang

### 4.5. Margin Trend (Line)
- Margin tipis turun (42,55% → 42,34%)

</details>

<details>
<summary><b>5. Temuan Utama & Rekomendasi (Sementara)</b></summary>

### 5.1. Insight Kunci
| Dimensi | Driver | Dampak |
|---|--------|--------|
| 🥇 | **Bikes** – margin erosion | -Rp169K (~60% decline) |
| 🥇 | **Pacific** – returns tinggi | net_sales Rp34K, returns 253/unit |
| 🟢 | Clothing positif | +Rp23K mitigator |
| 🟡 | Margin overall | Turun tipis → bukan driver utama |

### 5.2. Root Cause Summary
> **Volume bukan masalah.** Profit turun karena:
> 1. **Erosi margin Bikes** (harga cost naik/price tidak naik)
> 2. **Returns tinggi di Pacific** (produk cacat atau logistik)

### 5.3. Rekomendasi Aksi
| Prioritas | Aksi | Target |
|-----------|------|--------|
| 🔴 High | Audit harga & cost Bikes, naikkan 2-3% | Pulihkan -Rp | 
| 🔴 High | QC & kebijakan no-return di Pacific | Turunkan returns >250/unit |
| 🟡 Medium | Scale Clothing & Accessories | Kompensasi profit |
| 🟢 Low | Monitoring margin overall | Jaga >42% |

</details>

<details>
<summary><b>6. Struktur File (Output)</b></summary>

```
adventureworks-eda/
├── analysis/
│   └── 01_EDA.md
├── datasets/
│   ├── metadata.md
│   └── outputs/prof_dec/
│       ├── overall_metrics.csv
│       ├── top_drivers_dec.csv
│       └── dec_by_territory.csv
├── excel/01_eda/
│   ├── 01_decline_profit.csv
│   ├── 01_decline_profit.xlsx
│   └── sheets:
│       ├── Dashboard_Index
│       ├── Viz_Dashboard
│       ├── Data_For_Viz
│       ├── 01_Top3_Decliners
│       ├── 02_Pacific_Crisis
│       ├── 03_YoY_Negative
│       ├── 04_Bikes_Drivers
│       ├── 05_High_Returns
│       ├── 06_Top2_2022_Fix
├── sql/
│   └── eda_queries.sql
├── README.md