# newyorktaxi
Demand Hotspots & Flow Imbalance (Origin–Destination)

Analisis ini membedah:
1. Permintaan (pickup) per waktu & zona,
2. Ketidakseimbangan arus (PU vs DO) → net flow & severity,
3. Kecepatan / kemacetan (speed),
4. Campuran panjang trip (trip length mix),
5. Perilaku penumpang,
6. Utilization & earnings proxy.
Tujuan akhirnya: alokasi supply lebih cerdas (staging & reposisi), pengalaman pelanggan naik, pendapatan driver stabil.

**Latar Belakang**

Kota besar seperti New York selalu punya dinamika permintaan taksi yang tidak merata sepanjang waktu dan ruang: ada jam-jam yang melonjak, ada zona yang selalu ramai, dan ada pula tujuan yang “menguras” supply karena mengirim mobil ke wilayah yang tidak produktif. Dalam konteks ini, alokasi supply taksi yang hanya mengandalkan intuisi berisiko menciptakan ketidakseimbangan arus (origin–destination) yang membuat waktu tunggu penumpang naik, pendapatan per menit/mil turun, dan pengalaman pelanggan memburuk. Proyek ini membangun kerangka berbasis data untuk mengenali hotspot permintaan yang stabil, memahami ketidakseimbangan arus OD, dan merancang strategi staging serta reposisi armada sehingga mobil tidak “nyangkut” di zona sepi namun tetap siap di tempat & waktu yang tepat.

**Pernyataan Masalah**

Masalah inti yang dijawab: bagaimana menurunkan waktu tunggu dan mengurangi macetnya mobil di zona sepi dengan mengalokasikan supply berbasis pola permintaan dan arus OD yang teramati? Secara khusus, kita ingin memastikan bahwa zona dengan pickup tinggi tidak men-drop mobil ke “lubang hitam” (zona low-demand) secara sistematis, sekaligus menjaga kecepatan/efisiensi perjalanan dan memaksimalkan metrik proxy pendapatan (revenue per minute/mile) tanpa mengorbankan ketersediaan untuk pelanggan di jam puncak.

Data Preparation

Load & tipe data: parsing lpep_pickup_datetime/lpep_dropoff_datetime, konversi numerik (trip_distance, passenger_count, total_amount bila ada).
Fitur waktu: pickup_date, pickup_hour (0–23), pickup_ts (dibulatkan jam), pickup_tod4 (pagi/siang/sore/malam).
Durasi & efisiensi: trip_duration_min = dropoff - pickup, speed_mph = distance / hours, dur_per_mile = menit/mil (untuk efisiensi rute).
Imputasi passenger_count: isi NA berbasis kategori trip_distance_cat × pickup_tod4 (median per kategori) supaya sesuai konteks panjang trip & waktu.

Penanganan outlier:

trip_distance: flag IQR; analisis sensitif (speed/rev/min) pakai winsorize ringan (mis. P1–P99).
Durasi ekstrem: >500 menit dianggap anomali operasional → dikecualikan dari metrik speed/efisiensi, tetap dihitung untuk permintaan (pickup count).
revenue_per_min cenderung bias ke atas pada trip sangat pendek → gunakan median/IQR & winsorize saat perlu.
Panel zona×jam & OD: agregasi pu_count (pickup) dan do_count (dropoff) per PULocationID × pickup_ts; turunkan next_pu (pickup jam berikutnya di origin = wait proxy), net_flow = PU − DO
trip_len_cat (short/mid/long via kuartil), hotspot = top-decile zona berdasarkan total pickup.

Data Analysis

1. Temporal Demand Intensity
Across hours (0–23): pakai share per jam per hari; Kruskal–Wallis ⇒ distribusi antar jam berbeda signifikan.
Peak 15–18: di dalam 15–18 relatif homogen; Wilcoxon paired (per-hari) ⇒ peak > jam lain.
Weekday vs weekend: total pickup harian beda (Mann–Whitney U), sesuai pola aktivitas.

2. Zone Hotspot — Strength & Stability
Konsentrasi: top-decile zona menyumbang >50% pickup (Chi-square vs baseline 10%; per-hari Wilcoxon > 0.5).
Stabilitas: ranking zona ramai stabil antar hari (Spearman tinggi terhadap rata-rata lintas hari).
Kualitas hotspot: hotspot punya wait proxy (next_pu) lebih tinggi (MWU, greater), net_flow > 0 (Wilcoxon), dan dur_per_mile lebih rendah (MWU, less) → ramai, cepat “hidup lagi”, dan efisien.

3. Travel Speed 
Speed beda antar jam (Kruskal–Wallis); jam teramai cenderung lebih lambat.
Sehingga disimpulkan ada infikasi keramaian atau kemacetan

4. Trip Length Mix
Rush menaikkan jumlah short-trip 
Long-trip lebih sering berakhir di zona low-demand (Chi-square).

5. Passenger
passenger_count beda antar jam (KW).
Share multi (≥2) lebih tinggi sore (15–18) (Wilcoxon paired per-hari) dan weekend (MWU).
pc ↑ ↔ distance ↑ (KW/Spearman); speed trip solo vs multi berbeda (MWU).
8) Utilization & Earnings Proxy
revenue_per_min beda antar zona dan beda antar ToD (KW).
Severity berasosiasi negatif dengan revenue_per_min (Spearman).
Hotspot memiliki rev/min lebih tinggi (MWU), dan rush menaikkan rev/min di hotspot (MWU).
Catatan: interpretasi rev/min hati-hati pada trip pendek; gunakan median & winsorize.
Rekomendasi & Kesimpulan Insight
Staging & penempatan: fokuskan supply ke hotspot stabil menjelang 15:00, pertahankan hingga 18:00; lakukan reposisi terencana setelah puncak menuju origin dengan next_pu tinggi dan net_flow ≥ 0.
Mitigasi flow imbalance: pantau severity zona×jam; hindari pola OD yang “menguras” supply (A→B) tanpa aliran balik. Setelah long-trip ke low-demand, berikan nudge reposisi ke sumber demand.
Efisiensi rute: prioritaskan OD efisien (dur_per_mile rendah) saat rush; awareness bahwa speed menurun ketika jam teramai.
Kapasitas penumpang: sore & weekend cenderung multi-passenger; siapkan jenis kendaraan/operasi yang sesuai.
Produktivitas driver: pilih zona dengan kombinasi rev/min median tinggi + next_pu tinggi − severity rendah (skor komposit sederhana).
