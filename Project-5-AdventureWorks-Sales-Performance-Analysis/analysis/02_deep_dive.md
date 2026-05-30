# Analisis Deep Dive: Root Cause & What-If Simulation – AdventureWorks

**Tanggal:** 2025-04-09  
**Tools:** Microsoft Excel / LibreOffice Calc (Pivot, Formula, Goal Seek, Chart) + DuckDB (SQL)  
**Data:** `bikes_subcategory_analysis.csv` (SQL output), `03_WhatIf_Simulation.csv`

---

## 1. Tujuan & Konteks

**Dasar Analisis:**  
Hasil EDA (`01_eda.md`) menunjukkan **Bikes** sebagai driver utama penurunan profit (-Rp169K) dan **Pacific** sebagai wilayah dengan returns tertinggi (253/unit).  
**Tujuan Deep Dive:**  
1. **Root Cause:** Identifikasi produk spesifik (model/warna) dalam kategori Bikes yang menyebabkan penurunan.  
2. **What-If Simulation:** Simulasikan dampak perbaikan margin Bikes dan penurunan returns Pacific terhadap profit 2023.

---

## 2. Data & Sumber

### 2.1. Root Cause – Data Sub-Product Bikes

| File | Sumber | Isi |
|------|--------|-----|
| `bikes_subcategory_analysis.csv` | SQL (DuckDB) | ModelName, ProductColor, ProductStyle, SubcategoryName, year, qty, revenue, cost, profit |
| `Bikes_sorted.csv` | Pivot (Calc) | ModelName, ProductColor, 2021, 2022, Total Result, Profit_Delta (sorted descending) |

**SQL Query:**
```sql
-- sql/bikes_subcategory_analysis.sql
WITH bike_products AS (
    SELECT 
        p.ProductKey,
        p.ModelName,
        p.ProductColor,
        p.ProductStyle,
        p.ProductSubcategoryKey,
        sub.SubcategoryName
    FROM product p
    JOIN product_subcategory sub ON p.ProductSubcategoryKey = sub.ProductSubcategoryKey
    WHERE sub.SubcategoryName LIKE '%Bike%'
),
yearly_sales AS (
    SELECT 
        EXTRACT(YEAR FROM s.OrderDate) AS year,
        s.ProductKey,
        SUM(s.OrderQuantity) AS qty,
        SUM(s.OrderQuantity * p.ProductPrice) AS revenue,
        SUM(s.OrderQuantity * p.ProductCost) AS cost,
        SUM(s.OrderQuantity * (p.ProductPrice - p.ProductCost)) AS profit
    FROM sales_2021 s
    JOIN product p ON s.ProductKey = p.ProductKey
    GROUP BY year, s.ProductKey
    UNION ALL
    SELECT 
        EXTRACT(YEAR FROM s.OrderDate) AS year,
        s.ProductKey,
        SUM(s.OrderQuantity) AS qty,
        SUM(s.OrderQuantity * p.ProductPrice) AS revenue,
        SUM(s.OrderQuantity * p.ProductCost) AS cost,
        SUM(s.OrderQuantity * (p.ProductPrice - p.ProductCost)) AS profit
    FROM sales_2022 s
    JOIN product p ON s.ProductKey = p.ProductKey
    GROUP BY year, s.ProductKey
)
SELECT 
    bp.ModelName,
    bp.ProductColor,
    bp.ProductStyle,
    bp.SubcategoryName,
    ys.year,
    ys.qty,
    ys.revenue,
    ys.cost,
    ys.profit
FROM bike_products bp
JOIN yearly_sales ys ON bp.ProductKey = ys.ProductKey
ORDER BY bp.ModelName, bp.ProductColor, bp.ProductStyle, ys.year;
```

### 2.2. What-If Simulation – Data Baseline

| Item | Value | Source |
|------|-------|--------|
| Total Profit 2022 | 3,611,731 | `overall_metrics.csv` |
| Bikes Profit 2022 | 3,477,544 | `top_drivers_dec.csv` |
| Non-Bikes Profit 2022 | 134,187 | = Total - Bikes |
| Pacific Returns Value 2022 | 277,297 | `overall_metrics.csv` |

---

## 3. Root Cause Analysis (Sub-Product Bikes)

### 3.1. Proses di LibreOffice Calc

1. **Import CSV** `bikes_subcategory_analysis.csv` ke sheet `Raw_Bikes`.
2. **Buat PivotTable** (Data > Pivot Table):
   - **Row Fields:** `ModelName`, `ProductColor`
   - **Column Fields:** `year`
   - **Data Fields:** Sum of `profit`
3. **Hitung Delta** di kolom baru:
   ```
   = profit_2022 - profit_2021
   ```
4. **Sort** descending berdasarkan `Profit_Delta`.
5. **Conditional Formatting:**
   - Delta < -50,000 → Background Merah
   - Delta antara -50,000 dan -10,000 → Background Oranye
   - Delta > 0 → Background Hijau
6. **Export** sebagai `Bikes_sorted.csv`.

### 3.2. Hasil Root Cause

| Model | Warna | Profit 2021 | Profit 2022 | Profit Delta |
|-------|-------|-------------|-------------|--------------|
| **Road-250** | Black | 492,423 | 201,446 | **-290,977** |
| **Road-250** | Red | 290,100 | 54,235 | **-235,865** |
| **Mountain-200** | Black | 798,965 | 682,941 | **-116,024** |
| **Mountain-200** | Silver | 771,433 | 683,705 | **-87,728** |
| Road-650 | Red | 48,612 | 0 | -48,612 |
| Road-650 | Black | 46,610 | 0 | -46,610 |
| Touring-1000 | Blue | 202,078 | 364,461 | **+162,384** |
| Touring-1000 | Yellow | 193,056 | 352,734 | **+159,677** |
| Road-350-W | Yellow | 192,347 | 371,088 | **+178,741** |
| *Total Bikes* | | *3,668,907* | *3,501,636* | *-167,271* |

### 3.3. Agregasi per Model

| Model | Total Delta | Kontribusi thd Penurunan |
|-------|-------------|--------------------------|
| **Road-250** | **-526,842** | **315%** (turun jauh melebihi total, ditutupi model lain) |
| **Mountain-200** | **-203,752** | **122%** |
| Road-650 | -95,223 | 57% |
| Touring-1000 | +322,061 | -193% (menutupi kerugian) |
| Road-350-W | +178,741 | -107% (menutupi kerugian) |

### 3.4. Evaluasi: Apakah Warna Signifikan?

**Tidak.** Perbedaan delta antar warna dalam model yang sama relatif kecil:

| Model | Selisih Delta Warna | % dari Delta Model |
|-------|---------------------|--------------------|
| Road-250 (Black vs Red) | ~55,000 | 10% |
| Mountain-200 (Black vs Silver) | ~28,000 | 14% |
| Touring-1000 (Yellow vs Blue) | ~2,700 | 0.8% |

**Kesimpulan:** Warna bukan faktor penyebab. Pola penurunan terkait **model**, bukan warna. Analisis cukup dilakukan pada level model.

### 3.5. Temuan Root Cause

| Temuan | Detail |
|--------|--------|
| **Penyebab utama** | **Road-250** turun -526,842 (315% dari total penurunan Bikes) |
| **Penyebab kedua** | **Mountain-200** turun -203,752 (122%) |
| **Mitigator** | Touring-1000 (+322,061) dan Road-350-W (+178,741) menutupi sebagian kerugian |
| **Warna** | Tidak signifikan, cukup analisis level model |

---

## 4. What-If Simulation (Profit Recovery 2023)

### 4.1. Asumsi Skenario

| Variable | Skenario 1 | Skenario 2 | Skenario 3 |
|----------|------------|------------|------------|
| Bikes Margin Improvement | +1% | +5% | +10% |
| Pacific Returns Reduction | -10% | -20% | -30% |
| Non-Bikes Volume Growth | 10% | 10% | 10% |

**Alasan skenario:**
- Margin improvement kecil karena perbaikan fokus pada Road-250 dan Mountain-200.
- Returns reduction karena quality control di Pacific.
- Volume growth konservatif (10%) karena asumsi pertumbuhan tidak seagresif 2022 (25%).

### 4.2. Model Perhitungan di LibreOffice Calc

Dibuat di sheet `WhatIf_Calculation` dengan struktur:

```
Row  | Component                  | Formula
-----|----------------------------|--------
3    | Bikes Current Profit       | =Baseline!B5 (3,477,544)
4    | Margin Improvement %       | =Input!C18
5    | Bikes New Profit           | =B3 * (1 + B4)
6    | Bikes Profit Gain          | =B5 - B3
9    | Pacific Returns Current    | =Baseline!B11 (277,297)
10   | Returns Reduction %        | =Input!C19
11   | Pacific Returns New        | =B9 * (1 + B10)
12   | Returns Savings            | =B9 - B11
15   | Non-Bikes Profit           | =Baseline!B14 (134,187)
16   | Volume Growth %            | =Input!C20
17   | New Profit from Volume     | =B15 * (1 + B16)
18   | Volume Gain                | =B17 - B15
24   | TOTAL PROFIT 2023          | =B5 + B17 + B12
25   | Improvement vs 2022        | =B24 - Baseline!B4
26   | Improvement %              | =B25 / Baseline!B4
```

### 4.3. Hasil Simulasi

| Component | Skenario 1 | Skenario 2 | Skenario 3 |
|-----------|------------|------------|------------|
| **Bikes New Profit** | 3,515,400 | 3,651,421 | 3,825,298 |
| Bikes Profit Gain | 37,856 | 173,877 | 347,754 |
| Pacific Returns Savings | 27,730 | 55,459 | 83,189 |
| Non-Bikes New Profit | 147,606 | 147,606 | 147,606 |
| **Total Profit 2023** | **3,690,735** | **3,854,486** | **4,056,093** |
| **Improvement vs 2022** | **+79,004** | **+242,755** | **+444,362** |
| **Improvement %** | **+2.19%** | **+6.72%** | **+12.30%** |

### 4.4. Insight Utama

| Temuan | Detail |
|--------|--------|
| **Skenario 1 = Level 2021** | Profit 3,690,735 **sama persis** dengan profit 2021. Perbaikan kecil (margin +1%, returns -10%) sudah cukup. |
| **Skenario 2 = Rekomendasi** | Margin +5% + returns -20% → profit naik 6.72% di atas 2022, dan 4.4% di atas 2021. **Realistis dan achievable.** |
| **Skenario 3 = Terlalu Agresif** | Butuh margin +10% dan returns -30%. Mungkin tidak realistis dalam 1 tahun. |

### 4.5. Goal Seek – Target Profit 2021

**Pertanyaan:** Berapa % margin Bikes diperlukan untuk mencapai profit 2021 (3,690,735) dengan asumsi returns -20% dan volume +10%?

**Tools → Goal Seek:**
- Formula cell: Total Profit 2023
- Target value: 3,690,735
- Variable cell: Margin Improvement %

**Hasil:** Margin Bikes perlu naik **~1%** (sudah tercapai di Skenario 1).

---

## 5. Kesimpulan & Rekomendasi Final

### 5.1. Root Cause Summary

```
Profit Decline Bikes 2022: -167,271
├── Road-250: -526,842 (315%) ← PENYEBAB UTAMA
├── Mountain-200: -203,752 (122%) ← PENYEBAB KEDUA
├── Road-650: -95,223 (57%)
├── Touring-1000: +322,061 ← MITIGATOR
└── Road-350-W: +178,741 ← MITIGATOR
```

### 5.2. Recovery Strategy

| Prioritas | Aksi | Target | Dampak |
|-----------|------|--------|--------|
| **#1 (High)** | Audit pricing & cost **Road-250** | Margin +5% | +173,877 profit |
| **#2 (High)** | Quality control **Pacific** | Returns -20% | +55,459 profit |
| **#3 (Medium)** | Scale up **Touring-1000** | Volume +10% | Kompensasi kerugian |
| **#4 (Low)** | Evaluasi **Mountain-200** | Margin +2-3% | +40,000-60,000 profit |

### 5.3. Target 2023

| Target | Profit | vs 2022 | vs 2021 |
|--------|--------|---------|---------|
| **Minimal** (Skenario 1) | 3,690,735 | +2.19% | = Level 2021 |
| **Rekomendasi** (Skenario 2) | 3,854,486 | +6.72% | +4.4% |
| **Stretch** (Skenario 3) | 4,056,093 | +12.30% | +9.9% |

---

## 6. Lampiran (File Struktur)

```
outputs/deep_dive/
├── bikes_model_and_colour.csv   # SQL output (raw sub-product)
├── bikes_subcategory.csv                 # Pivot result (sorted by delta)
├── bikes_subcategory.csv

excel/02_deep_dive/
├── 02_deep_dive.xlsx:            
│   ├── bikes_subcategory            # Import CSV
│   ├── Model vs Profit              # Compare model and profit include the vizualitation
│   ├── WhatIf                       # Make simple hypotesys for next analysis

excel/02_deep_dive/
├── 02_root_cause_analysis.xlsx:            
│   ├── bikes_subcategory                 # Import CSV
│   ├── Pivot Table_bikes_subcategory_1
│   ├── Pivot Table_bikes_subcategory_2

excel/02_deep_dive/
├── 02_bikes_model_and_colour.xlsx:         
│   ├── Raw_Bikes                    # Import CSV sub-product
│   ├── Pivot_Bikes                  # Pivot Reslut
│   ├── Bikes_Sorted                 # Make sort for bikes model

excel/03_WhatIf_Simulation/
├── 03_WhatIf.xlsx:         
│   ├── Summary_WhatIf               
│   ├── WhatIf_Baseline              # Baseline for next calculation
│   ├── WhatIf_Input                 # The input for the calculation
│   ├── WhatIf_Calculation           # Make WhatIf calculation for simulation
│   ├── WhatIf_Viz                   # Make vizualitation for the results

sql/
├── query_SQL.ipynb   # Query untuk data sub-product