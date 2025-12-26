---
marp: true
theme: default
paginate: true
header: "Supply Chain Analysis"
footer: "Taufik Hidayah"
---

# Analisis Kerugian Profit Rantai Pasok
## Penyebab Akar, Dampak & Strategi Revenue Kuartal Depan

**Nama:** Taufik Hidayah  
**Tanggal:** 24-12-2025  

![bg right:40% w:500px](https://img.icons8.com/color/96/000000/truck.png)

---

# Daftar Isi
1. Ringkasan Eksekutif  
2. Pendahuluan  
3. Metodologi  
4-8. Temuan Utama & Dampak  
9. Dampak Finansial  
10. 5 Strategi  
11. Diskusi  
12. Kesimpulan  

![bg right w:300px](https://img.icons8.com/color/96/000000/presentation.png)

---

# Ringkasan Eksekutif
**Masalah:** Kerugian -$326k (18% order negatif).  

**Penyebab Utama:** Pengiriman Same Day qty 4-5 (80% kerugian besar).  

**Solusi Cepat:** Bukan diskon! Optimasi pengiriman → **+10-15% revenue kuartal depan**.  

**Rekomendasi #1:** Larang Same Day qty>3 → **simpan $9.7k**.  

![w:300px](Rincian_Kerugian.png)

---

# Pendahuluan
**Tujuan Proyek:**  
- Naikkan revenue Q via 3-5 strategi.  
- Identifikasi driver, optimasi harga, segmentasi.  

**Dataset:** 15.5k baris, 41 kolom.  

**Pertanyaan:** Mengapa profit negatif?  

---

# Metodologi
**Langkah EDA:**  
1. Data Cleaning.  
2. Explolatory Data Analysis (EDA).  
3. Root Cause Analysis. 
4. Visualizing and Reporting  

![w:900px](Metodologi.png)

---

# Temuan 1: Pengiriman Dominan
**Same Day qty4-5: -$9.7k**  

| Mode | Rata Loss | Total $ |
|------|-----------|---------|
| Same Day Tail | -442 | -9.7k |
| Standard | -105 | -174k |

**Alasan:** Biaya express tinggi di bundle.  

![bg right w:700px](fig1.png)

---

# Temuan 2: Hotspot Kategori
**Sport/Apparel 20%+ Kerugian (Women's Apparel terburuk)**  

- Same Day qty 4-5: -$227 rata.  
- Sport urgent → jebakan express.  

![w:450px](Hotspot.png)

---

# Temuan 3: Risiko Geo
**LATAM/USCA Same Day Parah**  

| Market | Qty4-5 Loss $ |
|--------|---------------|
| USCA | -4,992 |
| LATAM | -3,736 |

**Alasan:** Jarak + premium.

![bg right w:750](fig2.png)

---

# Temuan 4: Anomali Delay
**Same Day: -7 hari rata (anomali!)**  
**Standard: 15 hari lambat.**  

- Outlier: -1400 hari qty 4-5.  
**Alasan:** Error data/penalti ops.  

![w:700](scatter_delay.png)

---

# Temuan 5: Bukan Diskon
**Rata Diskon 8-12%, 0% >25%.**  

**Alasan:** Biaya pengiriman tetap utama.  

![w:500](dsc_rate.png)

---

# Dampak Finansial Keseluruhan
**Total: -$326k**  
- Same Day: -$26k (8% bagian).  
- Fix Tail → **+$9.7k (+3%)**.  

![w:400px](Total_Kerugian.png)

![bg right w:600](bar_total_Kerugian.png)

---

# 5 Strategi Actionable
1. **Larang Same Day qty>3** → +$9.7k (3%).  
2. **Tier pengiriman per kategori** → Apparel Standard.  
3. **Optimasi geo LATAM/USCA** → +5%.  
4. **Batas bundle qty express** → Stabil.  
5. **SLA delay <7 hari** → +6%.  

---

# Diskusi & Risiko
**Kelebihan:** Data-driven, menang cepat.  
**Risiko:**  
- Delay negatif → bersihkan data.  
- Resistensi vendor.    

---

# Dashboard
![w:700](Supply_Chain_Profit_Loss_Dashboard.jpg)

---

# Kesimpulan
**Inti:** **Identifikasi kerugian $326k & strategi +15% revenue via shipping optimization.**  

**Achievements:**  
1. Temukan root cause (Same Day tail qty4-5).  
2. Simulasi save $9.7k+ dari data real.  
3. 5 strategi actionable siap implementasi.  

**Learnings/Skills:**  
- EDA advance (Pandas, visual insights).  
- Business impact dari data supply chain.  

**Terima Kasih!**  