# newyorktaxi
Demand Hotspots & Flow Imbalance (Origin–Destination)

# Latar Belakang

Layanan taksi menghadapi pola permintaan yang tidak merata: ada jam dan zona yang selalu ramai, sementara drop-off dapat “membawa” mobil ke wilayah yang sepi sehingga pengemudi lama menunggu order berikutnya. Akibatnya waktu tunggu pelanggan naik, mobil “nyangkut” di zona tidak produktif, dan produktivitas pengemudi turun. Analisis ini membongkar pola permintaan dan ketidakseimbangan arus perjalanan (origin–destination) agar alokasi supply taksi berbasis data—bukan sekadar intuisi.

# Rumusan Masalah

1. Pada Hari dan Jam berapa saja rush hour (panggilan permintaan taksi terpadat) terjadi?
2. Daerah mana saja yang paling banyak punya permintaan pick up taksi?
3. Apakah permintaan pick up taksi pada daerah yang punya banyak permintaan pick up tersebut konsisten per harinya?
4. Bagaimana laju kendaraan taksi pada daerah hotspot (daerah banyak permintaan pick up taxi)?
5. Apakah Rush Hour memiliki hubungan dengan panjang perjalanan (short / long trip)? Bagaimana hubungannya?
6. Apakah ada hubungan dengan panjang perjalanan dengan kemungkinan drop off ke daerah low demand pick up taxi?
7. Apakah ada perbedaan signifikan antara jarak tempuh trip taksi dengan jam berangkatnya?
8. Bagaimana distribusi revenue/pendapatan taxi antar zona?
9. Bagaimana distribusi jumlah penumpang antar jam?
10. Apakah ada hubungan antara jarak perjalanan dengan kelompok jumlah penumpang?

# Data Preparation

1. Drop Kolom
Ditemukan ada kolom dengan semua baris bernilai null sehingga di drop
2. Handling Missing Values
Ditemukan beberapa baris dengan missing values yang jumlahnya sama. Baris-baris tersebut diputuskan untuk diberi placeholder atau diisi dengan nilai tertentu
3. Outliers
Ditemukan banyak outliers seperti panjang perjalanan sampai ribuan miles dan waktu naik taksi bisa lebih dari 10 jam.
4. Feature Engineering
Dibuat feature engineering untuk memahami data lebih baik seperti kategori waktu pickup taxi dan kategori perjalanan taksi, jumlah pendapatan per miles, dan sebagainya
5. Handling Data Anomali
Kolom total amount fee yang harus dibayar penumpang tidak cocok dengan jumlah seharusnya. Juga ada beberapa nilai di kolom tersebut bernilai minus, sehingga di drop.

# Data Analisa

## Waktu
1. Diketahui dari statistika deskriptif bahwa tipe panggilan street-hail paling banyak, dan bahwa ada beberapa tempat dan terlihat waktu tertentu dimana cenderung padat panggilan taxi
2. Dalam waktu sebulan pada Januari 2023, terbukti dengan visualisasi dan uji hipotesis kalau hari Rabu Kamis Jumat lebih sering panggilan pickup Taxi dan waktu Sore Hari jam 15.00 - 18.00 adalah waktu paling padat.
   
## Antar Zona

1. Dari grafik barplot mengenai zona jumlah tertinggi pickup dan dropoff, jumlah maksimal dropoff adalah di bawah 3500 pada daerah id 75 dan 74, sedangkan jumlah pickup maksimal ada pada daerah dengan ID 74 dan 75, dengan nilai lebih dari 12000 pickup dan 8000 pickup. Tidak banyak daerah id yang sama pada pickup dan dropoff terbanyak. Sehingga disimpulkan bahwa terdapat banyak pickup khusus di beberapa daerah tertentu, dan penumpang melakukan dropoff ke zona id yang berbeda, baik itu jarak dekat maupun jauh.
2. Dengan mengambil top 10% zona, yaitu 23 zona dari 226 zona pada data dengan jumlah pickup tertinggi, kita dapati kalau sebanyak 85% panggilan pickup taxi terjadi pada 23 zona tersebut. Dilakukan uji hipotesis mann whitney u terhadap kesimpulan tersebut pun terbukti dengan p value di bawah 0.05 bahwa zona dalam top 23 tersebut memiliki jumlah pickup lebih besar dari zona di luar top 23.
3. Selanjutnya ingin dibuktikan apakah kesimpulan tersebut konsisten tiap hari, dengan melihat apakah median persentase pickup di daerah top 23 zona tersebut lebih besar dari 50% tiap harinya? Setelah dilakukan uji hipotesis didapati bahwa p value di bawah 0.05 sehingga kita menerima H1 yang mengatakan bahwa benar kalau secara konsisten top 23 zona tersebut menyumbang lebih dari separuh jumlah pickup per harinya.
4. Kemudian ingin diketahui apakah top 23 zona tersebut (hotspots) mempunyai kecenderungan punya jumlah pickup lebih tinggi di jam berikutnya di banding zona non hotspots. Setelah diuji, didapati bahwa benar kalau daerah hotspot cenderung berlanjut ramai di jam berikutnya.
5. Kemudian ingin dilihat apakah taksi pada daerah hotspot mempunyai waktu perjalanan per miles yang efisien. Ternyata didapati bahwa cenderung tidak lebih efisien. Taxi cenderung lebih lama waktu perjalanannya di daerah hotspoit dibanding daerah lainnya (indikasi kemacetan/kepadatan)

## Trip Length (Short/Long)

Ingin dicari tahu apakah perjalanan panjang lebih sering berakhir dropoff di zona low demand serta distribusi distance antar jam 

1. Setelah dilakukan uji chi square, didapati bahwa dari jam 7 sampai jam 6 sore tidak ada hubungan signifikan dengan jarak perjalanan taxi yang pendek. Tetapi pada waktu rush hour (15.00-18.00), Terdapat hubungan signifikan antara jam rush hour dengan jarak trip taxi yang pendek dan panjang. Dengan melihat nilai crosstab , disimpulkan bahwa pada jam berangkat di rush hour, perjalanan taxi yang panjang cenderung makin jarang (orang lebih sering melakukan short trip di dalam kota saja).
2. Melalui uji chi square juga disimpulkan bahwa walaupun ada hubungan signifikan antara long trip dan taksi pergi dropoff ke daerah low demand, perbedaan proporsinya relatif kecil. Tapi bisa disimpulkan bahwa perjalanan long trips cenderung drop off ke zona sepi.

## Earnings

1. Terdapat bukti sangat signifikan melalui uji kruskal walilis bahwa minimal ada 1 zona dengan distribusi pendapatan per minutenya berbeda signifikan.
2. Kemudian didapat kesimpulan bahwa minimal ada 1 kategori waktu (pagi/siang/sore/malam) yang distribusi pendapatan per minutenya berbeda signifikan.

## Passenger Count

1. Diketahui bahwa jumlah penumpang mempunyai distribusi perbedaan cukup signifikan antar jam.
2. Dan didapat kesimpulan bahwa tidak ada perbedaan yang berarti dalam distribusi jarak perjalanan antar kelompok jumlah penumpang

# Kesimpulan dan Saran
1. Pada waktu rush hour alokasikan banyak taxi pada daerah2 dengan demand permintaan tinggi.
2. Untuk taxi yang melakukan dropoff di zona low demand, beri insentif agar kembali ke daerah hotspot.
3. Beri bonus agar driver taxi pergi ke zona yang banyak demand pickup tinggi.
4. Jika waktu tunggu pickup taxi makin lama segera pindah ke zona lainnya.
