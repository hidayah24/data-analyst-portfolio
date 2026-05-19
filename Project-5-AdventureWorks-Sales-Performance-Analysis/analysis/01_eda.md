# Analisis EDA: Profit Decline 2022 – AdventureWorks

**Tanggal:** 2025-04-09  
**Tools:** Microsoft Excel / LibreOffice Calc + DuckDB (SQL)  
**Data:** `01_decline_profit.csv` (gabungan `overall_metrics.csv`, `top_drivers_dec.csv`, `dec_by_territory.csv`)

---

## 1. Tujuan & Konteks

**Problem Statement**  
Volume penjualan naik 25% (36.230 → 45.314 unit) dari 2021 ke 2022, namun **profit bersih turun 2,14%** (Rp 3.690.735 → Rp 3.611.732).  
**Tujuan EDA:** Mengidentifikasi **driver utama** penurunan profit (margin erosi, returns tinggi, atau wilayah tertentu) menggunakan SQL dan Excel/Calc.

---

## 2. Data & Sumber

Dataset terdiri dari 3 file CSV yang digabung:

| File Sumber | Isi Utama |
|------------|-----------|
| `overall_metrics.csv` | Ringkasan tahunan: total qty, gross revenue/cost, net profit, returns, margin % |
| `top_drivers_dec.csv` | Per kategori (Bikes, Clothing, Accessories): qty, revenue, cost, margin ratio, delta profit |
| `dec_by_territory.csv` | Per benua (Europe, North America, Pacific): net sales, returns per qty |

**Struktur data gabungan (`01_decline_profit.csv`)** – 26 kolom, 14 baris (2 year → Overall, 4 kategori, 6 wilayah).  
Contoh kolom kunci: `year`, `total_qty`, `net_profit`, `profit_yoy_pct`, `categoryname`, `continent`, `decline_rank`, `returns_per_qty`.

---

## 3. Proses EDA di LibreOffice Calc

### 3.1. Import & Pembersihan Data
1. **Buka CSV** sebagai sheet baru (File > Open, pilih separator koma).
2. **Hapus NaN / blank** – ganti `null` dan sel kosong dengan `0` menggunakan Find & Replace (Ctrl+H, *regular expressions*).
3. **Data type**: Kolom numerik diubah ke angka (Number), kolom teks tetap teks.

### 3.2. Pembuatan Pivot Table (DataPilot)
- **Pilih sel A1:AA22** → `Data > Pivot Table > Layout`:
  - **Row Fields:** `year`
  - **Page Fields:** `categoryname`, `continent`, `source`, `decline_rank`
  - **Data Fields:** `total_qty` (Sum), `net_profit` (Sum), `profit_yoy_pct` (Average), `gross_margin_pct` (Average), `returns_per_qty` (Average)
- Hasil: Pivot dinamis siap difilter.

### 3.3. Eksplorasi dengan Filter
Dilakukan 6 filter combo untuk menggali insight:

| # | Filter | Temuan Utama | Sheet Hasil |
|---|--------|-------------|-------------|
| 1 | `source=Categories` + `decline_rank≤3` | **Bikes rank 1**: profit delta -Rp169K, margin ratio 1.70→1.69 | `01_Top3_Decliners` |
| 2 | `source=Territory` + `continent=Pacific` | **Pacific net_sales 2022 hanya Rp34K** (vs Rp4,27jt 2021), returns 253/unit | `02_Pacific_Crisis` |
| 3 | `year=2022` + `profit_yoy_pct<0` | Profit -2,14% konfirmasi hypothesis | `03_YoY_Negative` |
| 4 | `categoryname=Bikes` + `source=Categories` | Margin erosion spesifik Bikes (volume +79 unit) | `04_Bikes_Drivers` |
| 5 | `returns_per_qty>200` + `source=Territory` | Pacific selalu >250, tertinggi | `05_High_Returns` |
| 6 | Kombinasi Categories+2022+rank≤2 | Bikes vs Clothing: Bikes turun, Clothing naik +23K | `06_Top2_2022_Fix` |

### 3.4. Dashboard Index Terintegrasi
Buat sheet `Dashboard_Index` dengan tabel hyperlink ke 6 sheet filter.  
**Format**:
- Kolom: `No | Sheet Name | Key Filter | Insight | Impact | Action | Status`
- Hyperlink (Ctrl+K) → klik lompat ke sheet terkait.

---

## 4. Visualisasi (Sheet `Viz_Dashboard`)

Untuk kemudahan pembacaan, dibuat **5 grafik** dari data helper `Data_For_Viz` (ekstrak manual dari Pivot unfiltered):

### 4.1. Volume vs Profit YoY
- **Chart:** Column + Line (combo)
- **Insight:** Volume +25% (bar hijau naik), Profit -2,14% (garis merah turun) → **Konfirmasi problem statement**.

### 4.2. Profit Delta per Kategori
- **Chart:** Horizontal Bar (positif/negatif)
- **Insight:** Bikes (bar merah ke kiri, -Rp169K) dominan; Clothing & Accessories naik.

### 4.3. Net Sales per Wilayah 2022
- **Chart:** Column
- **Insight:** Pacific (bar merah pendek) – net sales hampir nol, sementara EU dan NA stabil.

### 4.4. Returns Heatmap
- **Table** + Conditional Color Scale (merah >200)
- **Insight:** Pacific 253 → zona bahaya returns.

### 4.5. Margin Trend
- **Chart:** Line
- **Insight:** Margin overall turun tipis (42,55% → 42,34%), didorong Bikes yang lebih rendah.

---

## 5. Temuan Utama (Insight)

| Kategori | Temuan | Dampak ke Profit |
|----------|--------|------------------|
| **Bikes** | Profit -Rp169K meski volume +79 | ~60% penyebab decline |
| **Pacific** | Net sales anjlok 99%, returns 253/unit | ~30% kerugian regional |
| **Returns** | Pacific returns tertinggi; returns_value 2022 = Rp277K | Erosi profit ~7-8% |
| **Clothing** | Positif (+23K) | Bisa jadi mitigator |
| **Margin** | Margin tipis turun (0.21 pp) | Bukan driver utama, tapi akumulasi |

**Kesimpulan Akhir:**  
Penurunan profit bukan karena volume, melainkan **margin erosi pada Bikes** dan **returns tinggi di Pacific**.

---

## 6. Rekomendasi Aksi

| Prioritas | Rekomendasi | Target |
|-----------|-------------|--------|
| **High** | Audit pricing & cost Bikes, naikkan harga 2-3% | Pulihkan -Rp169K |
| **High** | Quality control & kebijakan no-return di Pacific | Kurangi returns >250/unit |
| **Medium** | Scale up Clothing (+23K) | Kompensasi profit |
| **Low** | Monitoring margin overall (perbaikan kecil) | Jaga >42% |

---

## 7. Lampiran (File Struktur)

```
excel/01_eda/
├── 01_decline_profit.csv          # Data gabungan
├── 01_decline_profit.xlsx         # File Excel (Pivot, Dashboard)
├── sheets:
│   ├── Dashboard_Index            # TOC dengan hyperlink
│   ├── Viz_Dashboard              # 5 grafik
│   ├── Data_For_Viz               # Data helper untuk grafik
│   ├── 01_Top3_Decliners
│   ├── 02_Pacific_Crisis
│   ├── 03_YoY_Negative
│   ├── 04_Bikes_Drivers
│   ├── 05_High_Returns
│   ├── 06_Top2_2022_Fix
└── analysis/
    └── 01_EDA.md                  # File ini
```

---

**Catatan:**  
- File `.xlsx` berisi semua pivot, filter, dan chart. Buka dengan Excel atau LibreOffice Calc ≥7.6.  
- Data dapat diperbarui dengan mengganti CSV dan me-refresh pivot (kanan > Refresh).  
- Visualisasi siap diekspor ke PDF untuk presentasi.
