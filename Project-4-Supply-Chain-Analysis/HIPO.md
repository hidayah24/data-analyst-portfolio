# Masalah
1. Negatif Profit

# Insight
1. Tidak ada hubungan negatif profit dengan status order, karena status `COMPLETE` malah menyumbah negatif profit paling besar
2. Mean loss seragam ~-100/order di semua bin → Masalah sistemik per order, bukan spesifik `discount level`. Puncak Loss di Diskon Moderat (6-10%: -25k; 16-20%: -21k) Diskon ini dorong volume sales tinggi, tapi margin habis. Strategi promo "aman" malah rugi besar secara absolut. Dan order_item_discount_rate `berkorelasi rendah` dengan profit_per_order


# Strategi sementara
2. Pricing Floor: Naikkan min product price 10-20% untuk order 0% diskon → cegah loss dasar. Discount Tiered: Hindari 6-20% untuk low-margin category; batasi <5% saja.