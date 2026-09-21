# Desain Analisis (分析（ぶんせき）デザイン) — Olist Repeat Purchase

Dokumen ini versi bahasa Indonesia dari `analysis_design_ja.md`. Istilah kunci Jepang gue kasih furigana + arti, biar sekalian jadi bahan hafalan buat 面接（めんせつ / wawancara).

> Cara baca tabel istilah: **漢字（ふりがな）** = arti Indonesia.

---

## Kamus istilah (面接用)

| Jepang | Furigana | Arti / cara pakai saat wawancara |
|--------|----------|----------------------------------|
| 分析デザイン | ぶんせきデザイン | Desain analisis — rancangan sebelum sentuh data |
| ビジネス課題 | ビジネスかだい | Masalah bisnis |
| ビジネス目的 | ビジネスもくてき | Tujuan bisnis |
| 分析目的 | ぶんせきもくてき | Tujuan analisis |
| 仮説 | かせつ | Hipotesis |
| 検証 | けんしょう | Verifikasi / pengujian |
| 施策 | せさく | Langkah/aksi perbaikan |
| 優先順位 | ゆうせんじゅんい | Urutan prioritas |
| 新規顧客 | しんきこきゃく | Pelanggan baru |
| 既存顧客 | きぞんこきゃく | Pelanggan lama/eksisting |
| 客単価 | きゃくたんか | Nilai transaksi rata-rata (AOV) |
| 配送遅延 | はいそうちえん | Keterlambatan pengiriman |
| 再購入率 | さいこうにゅうりつ | Tingkat pembelian ulang |
| 構造問題 | こうぞうもんだい | Masalah struktural |
| 棄却 | ききゃく | Ditolak (hipotesis) |
| 意思決定 | いしけってい | Pengambilan keputusan |

---

## 1. Menemukan masalah bisnis — ビジネス課題（かだい）の発見（はっけん）

Olist itu marketplace Brazil yang menghubungkan seller kecil ke mall besar. Waktu gue lihat data secara garis besar, jumlah order tumbuh **~3x dalam setahun** (rata-rata 2.435 order/bulan di semester 1 2017 → 6.864 di semester 1 2018).

Tapi pertumbuhan ini **hampir seluruhnya datang dari pelanggan baru (新規顧客 / しんきこきゃく)**. Biaya akuisisi pelanggan baru (CAC) biasanya beberapa kali lipat biaya mempertahankan pelanggan lama, jadi "tumbuh cuma dari akuisisi" itu struktur yang rapuh — berhenti begitu budget iklan habis.

Dari riset awal, tingkat pembelian ulang (repeat rate) cuma **sekitar 3%** (EC normal 20–30%). Di sinilah ruang perbaikan terbesar. → Ini yang gue pilih jadi tema.

## 2. Tujuan bisnis — ビジネス目的（もくてき）

Menaikkan penjualan **tanpa menambah biaya akuisisi**. Caranya: **menaikkan repeat rate pelanggan lama (現状 sekarang ~3%).**

## 3. Tujuan analisis — 分析目的（ぶんせきもくてき）

Mengidentifikasi faktor yang menghambat pembelian ulang, lalu **menentukan langkah perbaikan (施策 / せさく) mana yang harus diprioritaskan, dengan bukti angka.**

> Poin penting: bukan sekadar "cari cara naikin repeat", tapi sampai **menentukan prioritas keputusan (意思決定 / いしけってい)**. Analisis baru bernilai kalau berujung ke keputusan.

## 4. Pertanyaan & hipotesis — 問い・仮説（かせつ）

| # | Hipotesis | Alasan (kenapa gue mikir begitu) |
|---|-----------|----------------------------------|
| 1 | Keterlambatan pengiriman menurunkan skor review, dan pelanggan yang kecewa nggak balik lagi | Brazil luas, logistik jadi tantangan. "Pengalaman buruk → churn" itu pola klasik EC |
| 2 | Repeat rate beda jauh antar kategori produk | Barang habis pakai dibeli ulang, tapi elektronik/barang tahan lama cukup sekali |
| 3 | Pelanggan bernilai tinggi (RFM atas) berperilaku beda — ada segmen yang layak ditarget | Sebagian besar revenue terkonsentrasi di sedikit pelanggan (hukum Pareto) |

## 5. Cara verifikasi — 検証（けんしょう）方法（ほうほう / logika analisis）

### Definisi metrik (dikunci di awal)

- **Pelanggan repeat** = pakai `customer_unique_id`, punya ≥2 order yang selesai dikirim (delivered)
  - ⚠️ `customer_id` berubah tiap order, jadi **jangan** dipakai (kalau salah, repeat rate jadi hampir 0%). Ini jebakan terkenal dataset ini
- **Keterlambatan (配送遅延)** = tanggal kirim aktual − tanggal estimasi > 0 hari
- **Repeat rate untuk lihat sebab-akibat** = dihitung dari **order pertama** tiap pelanggan (atribut delay/review order pertama), lalu lihat apakah dia beli lagi
  - Kalau order ke-2 dst dicampur, sebab dan akibat bisa kebalik → makanya dibatasi ke order pertama

### Langkah

1. **Memahami data** — cek jumlah baris, primary key, periode, missing value dari 9 tabel; pahami relasi lewat ER diagram
2. **Potret kondisi sekarang** — hitung repeat rate, distribusi jumlah order, AOV (客単価)
3. **Hipotesis 1** — grup delay (lebih cepat / tepat waktu / telat 1-7 hari / telat 8+ hari) × rata-rata review; skor review order pertama × repeat rate. Nggak berhenti di grafik — **uji chi-square (カイ二乗検定)** buat pastikan signifikansi
4. **Hipotesis 2** — repeat rate per kategori (dibatasi kategori dengan ≥500 pelanggan biar nggak bias sampel kecil)
5. **Hipotesis 3** — segmentasi RFM. Tapi 97% pelanggan punya F=1, jadi F diubah dari kuartil ke biner "repeater atau bukan" (sesuaikan metode dengan data)
6. **Analisis cohort** — repeat rate per bulan pembelian pertama. Buat memisahkan "belum sempat repeat" vs "memang nggak repeat"
7. **Usulan 施策** — hitung dampak tiap langkah = jumlah pelanggan target × besar perbaikan × AOV, lalu buat prioritas

## 6. Hasil — 結果（けっか）

| Hipotesis | Hasil | Isinya |
|-----------|-------|--------|
| 1 bagian awal (delay → review turun) | ✅ Terbukti | 4,28 → 1,69 bintang. Telat 8+ hari: 70% kasih bintang 1 |
| 1 bagian akhir (review turun → nggak beli lagi) | ❌ **Ditolak (棄却 / ききゃく)** | Bintang 1 maupun 5, repeat rate sama ~3% (p=0,875). Efek langsung delay cuma 0,6 poin |
| 2 (beda antar kategori) | ✅ Terbukti | Home appliance 10,5% vs elektronik 2,9% (beda 3,5x) |
| 3 (ada segmen yang layak ditarget) | ✅ Terbukti | "Promising one-timers" = 24% pelanggan / 38% revenue. Total repeater cuma 5,6% revenue |

**Temuan terbesar: hipotesis 1 runtuh.** Cerita klasik "perbaiki pengiriman → repeat naik" **nggak berlaku** di Olist. Pelanggan yang puas pun nggak balik. Artinya repeat rate 3% bukan soal keluhan individual, tapi **masalah struktural (構造問題 / こうぞうもんだい)** yang berakar di model bisnisnya (kumpulan banyak seller kecil; pelanggan beli "produk", bukan "toko", secara sekali-jalan). Cohort juga konsisten: pelanggan bulan apa pun, repeat bulan berikutnya <1%.

## 7. Usulan langkah perbaikan — 施策提案（せさくていあん / urut prioritas）

1. **CRM pasca-pembelian ke ~22.500 "promising one-timers"** — kirim kupon berikutnya + rekomendasi produk terkait ke pembeli di kategori repeat tinggi (home appliance, fashion accessories, furniture, bed & bath). Naik +3 poin ≈ **R$110rb/tahun**
2. **Perbaikan delay dijalankan sebagai "proteksi brand", bukan retention** — 32% review bintang 1-2 berasal dari order telat. Tapi efek langsung ke repeat cuma ~R$3rb, jadi keputusan investasinya dinilai dari sisi menjaga review & kepercayaan
3. **Kalau repeat memang rendah secara struktural, geser ke maksimalkan LTV one-timer** — cross-sell & naikin nilai keranjang lebih cocok dengan struktur Olist

## 8. Keterbatasan & langkah berikutnya

- Data berhenti Oktober 2018, jadi pelanggan yang pertama beli di akhir periode punya waktu observasi pendek (repeat rate sedikit di-underestimate)
- Hubungan review × repeat diuji berbasis korelasi. Kalau mau bilang sebab-akibat secara ketat, langkah berikutnya propensity score matching
- Data geolocation belum dipakai; analisis waktu kirim per negara bagian bisa mempertajam usulan #2

## 9. Yang gue pelajari (poin buat 面接)

- **Nulis desain analisis dulu** bikin gue bisa memotong "menarik tapi nggak nyambung ke tujuan" di tengah jalan
- **Hipotesis yang ditolak (棄却) justru bernilai.** Gue membantah cerita klasik pakai data, dan prioritas 施策 berubah (dari investasi pengiriman → investasi CRM). Tugas analisis bukan mengonfirmasi hipotesis, tapi mengubah keputusan
- **Definisi metrik menentukan kesimpulan.** Jebakan `customer_unique_id`, pembatasan ke order pertama, penanganan review ganda — salah asumsi agregasi, kesimpulannya bisa terbalik 180°
