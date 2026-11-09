# BLUEPRINT & DOKUMENTASI LENGKAP ARSITEKTUR WEB APP

## Indonesia-First Consumer Intelligence & Reality Check Affiliate System

**Versi:** 4.0 (Updated with Utility & Problem-Solving Framework)

## 1. IKHTISAR SISTEM & PRINSIP UTAMA

Sistem ini dirancang untuk membangun **Web App Penasihat Belanja Independen berbasis AI** yang menyajikan ulasan produk e-commerce secara jujur, objektif, dan berfokus pada bukti (*evidence-led*).

Web App ini tidak bertujuan untuk melakukan *hard-selling*, melainkan memberikan **Pre-Checkout Reality Check** (peringatan risiko sebelum membeli) yang dipadukan dengan **Problem-Solving & Functional Utility** (penjelasan kegunaan nyata), **Verified Wins** (bukti kepuasan pembeli teliti), serta **Solusi DIY/Anti-Zonk**.

### **Prinsip Utama:**

1. **The Honest Balance (Keseimbangan Kejujuran):** Reality Check bertindak sebagai *rem* (membangun kepercayaan dan mencegah salah beli), sementara Kelebihan & Problem-Solving bertindak sebagai *gas* (meningkatkan keyakinan membeli).

2. **Problem-Solving First:** Menjelaskan produk bukan dari aspek spesifikasi semata, melainkan dari masalah harian (*pain points*) yang berhasil dituntaskan produk tersebut.

3. **Zero Coding Scraper:** Memanfaatkan kombinasi Google AI Mode, Master Prompt 3.0, dan Chrome Side Panel secara semi-manual untuk mendapatkan data kualitatif paling murni tanpa risiko diblokir sistem marketplace.

4. **Monetisasi Etis:** Mengarahkan transaksi via Link Afiliasi Shopee dengan menyajikan rekomendasi *bundle cross-sell* yang rasional dan relevan.

## 2. DIAGRAM ALUR DATA (DATA PIPELINE)

```
[ STEP 1: DATA MINING ]
  ├── Google AI Mode (Master Prompt 3.0) ──> Riset Lintas Platform (TikTok, YT, Forum, Reddit)
  └── Chrome Side Panel (Shopee)       ──> Ekstraksi Semi-Manual (10-20 Ulasan B1 & B5 Terbaru)
                                              │
                                              ▼
[ STEP 2: SYNTHESIS & EXTRACTION ]
  └── NotebookLM (Pabrik Sintesis)      ──> Mengolah Data Mentah Menggunakan Prompt Khusus
                                              │
                                              ▼
[ STEP 3: DATABASE STRUCTURE ]
  └── reality_check_db.json            ──> File Penyimpanan JSON Terstruktur & Terpusat
                                              │
                                              ▼
[ STEP 4: INTERFACE & CONVERSION ]
  └── Gradio Web App (Hugging Face)     ──> UI Interaktif: Search, Reality Check, Utility, Traffic Loop

```

## 3. METODOLOGI PENGUMPULAN DATA MENTAH (DATA MINING)

### **A. Google AI Mode (Master Prompt 3.0)**

Menggunakan Master Prompt 3.0 (*Indonesia-First Cross-Platform Consumer Intelligence Research*) untuk melakukan pencarian kualitatif independen lintas platform:

* **Purchase Intelligence:** Marketplace (Shopee, Tokopedia, Blibli, Lazada, TikTok Shop).

* **Social Intelligence:** TikTok, YouTube, Instagram Reels, Facebook Groups, X/Twitter, Reddit, blog dan website review.

* **Technical Intelligence:** Spesifikasi produsen, panduan material, dan standar perawatan.

### **B. Chrome Side Panel (Shopee Extraction)**

Melakukan ekstraksi cepat ulasan ekstrem langsung dari halaman produk Shopee via Gemini Side Panel di Google Chrome:

* **Ulasan Bintang 1 (10-20 sampel):** Fokus pada keluhan fatal, kerusakan material, aus jangka pendek, atau salah ekspektasi.

* **Ulasan Bintang 5 Terbaru (10-20 sampel):** Fokus pada ulasan berpola *Before-After*, tips pemakaian yang berhasil, dan kegunaan nyata (*real utility*).

* **Instruksi Ekstraksi Side Panel:**

  > *"Tolong kelompokkan ulasan-ulasan ini menjadi dua: (1) Keluhan fatal dan akar masalah dari bintang 1, serta (2) Alasan kepuasan spesifik, kegunaan nyata, dan trik pemakaian dari bintang 5. Abaikan ulasan klise seperti 'barang bagus pengiriman cepat'."*

## 4. PROMPT LENGKAP UNTUK NOTEBOOKLM (SINTESIS DATA)

Salin dan tempel prompt di bawah ini ke dalam chat box **NotebookLM** setelah memasukkan hasil riset Google AI Mode dan ulasan dari Chrome Side Panel sebagai sumber (*Sources*):

```
# ROLE & INSTRUCTION FOR NOTEBOOKLM
Anda adalah Senior Consumer Intelligence Analyst & JSON Data Extractor.
Tugas Anda adalah menganalisis seluruh sumber data yang ada di notebook ini (hasil riset Google AI Mode, dokumen teknis, serta ulasan Shopee Bintang 1 dan Bintang 5). 

Ekstrak seluruh temuan menjadi SATU OBJEK JSON TUNGGAL yang valid dan bersih, tanpa kalimat pembuka atau penutup.

ATURAN EKSTRAKSI KETAT:
1. STRICT ANTI-HALUSINASI: Jangan mengarang data. Jika data tidak ditemukan di sumber, isi dengan "Tidak ada data terverifikasi".
2. PEMISAHAN MASALAH: Pisahkan masalah kualitas/fungsi produk (Product Defect) dari masalah pengiriman/layanan toko (Delivery Issue).
3. KUTIPAN ASLI: Kutipan (quotes) harus verbatim (persis seperti yang tertulis di sumber, maks 25 kata).
4. UTILITY & PROBLEM SOLVING FOCUS: Identifikasi dengan jelas masalah utama harian yang diselesaikan oleh produk ini dan trik kreatif pemakaian dari komunitas.

Hasilkan output persis mengikuti skema JSON di bawah ini:

{
  "product_id": "GANTI_DENGAN_SLUG_PRODUK",
  "product_name": "Nama Produk Resmi",
  "shopee_link": "URL_AFILIASI_SHOPEE",
  "pinterest_image_url": "URL_GAMBAR_ESTETIS_PINTEREST",
  "match_tags": ["tag1", "tag2", "tag3"],
  "expert_score": 8.5,
  "product_utility_and_solving": {
    "core_pain_point_solved": "Masalah utama pengguna/rumah tangga yang paling tuntas diselesaikan oleh produk ini.",
    "functional_efficiency": "Bagaimana mekanisme fungsional produk mempermudah hidup, menghemat waktu, atau menghemat ruang.",
    "smart_workaround": "Tips, trik kreatif, atau modifikasi kecil dari komunitas agar produk berfungsi 100% optimal."
  },
  "strengths": {
    "headline": "Kelebihan Utama Paling Menonjol",
    "detail": "Ringkasan manfaat nyata berdasarkan bukti bintang 5 dan riset sosial."
  },
  "reality_check": {
    "skenario_buruk": "Masalah terburuk atau keluhan fatal yang dilaporkan pembeli bintang 1 setelah beberapa waktu pemakaian.",
    "root_cause": "Akar penyebab kegagalan fungsi atau kelemahan material.",
    "mismatched_buyer": "Tipe pembeli atau kondisi penggunaan yang DILARANG membeli produk ini."
  },
  "durability_forecast": {
    "bulan_1": "Kondisi fisik dan performa fungsi di minggu 1 hingga minggu ke-4.",
    "bulan_3": "Titik kritis ketahanan atau penurunan fungsi di bulan ke-3."
  },
  "solutions": {
    "maintenance_tip": "Langkah pemeliharaan/pemasangan resmi yang disarankan produsen.",
    "diy_fix": "Solusi perbaikan mandiri jika terjadi masalah umum pada produk."
  },
  "verified_wins": {
    "buyer_quote": "Kutipan asli ulasan bintang 5 dari pembeli yang sangat puas karena menggunakan cara pakai yang benar.",
    "buyer_persona": "Profil pembeli yang cocok (misal: Anak Kos, Pemilik Dapur Minimalis).",
    "success_factor": "Faktor kunci mengapa mereka berhasil menggunakan produk tanpa kendala."
  },
  "bundle_cross_sell": {
    "item_name": "Nama Aksesori atau Barang Pelengkap",
    "reason": "Alasan teknis/fungsional mengapa barang ini wajib dibeli bersamaan untuk mengoptimalkan produk utama."
  }
}

HANYA HASILKAN KODE JSON DI ATAS.

```

## 5. SKEMA DATABASE JSON FINAL (`reality_check_db.json`)

Berikut adalah struktur file JSON terpusat yang akan dibaca langsung oleh aplikasi Python Gradio:

```
[
  {
    "product_id": "rak_kamar_mandi_01",
    "product_name": "Rak Dinding Kamar Mandi Tempel Alumunium Tanpa Bor",
    "shopee_link": "https://shopee.co.id/universal-link-affiliate-anda",
    "pinterest_image_url": "https://i.pinimg.com/originals/ab/cd/ef/rak_kamar_mandi_bersih.jpg",
    "match_tags": ["kamar mandi", "rak tempel", "tanpa bor", "anti karat", "kos minimalis"],
    "expert_score": 8.2,
    "product_utility_and_solving": {
      "core_pain_point_solved": "Mengatasi kamar mandi sempit & berantakan tanpa perlu merusak dinding keramik dengan bor paku.",
      "functional_efficiency": "Sistem tiris air otomatis di dasar rak mencegah sabun menjadi lembek dan membusuk akibat tergenang air.",
      "smart_workaround": "Bersihkan dinding dengan alkohol 70% sebelum stiker ditempel, lalu sangga stiker dengan masking tape selama 24 jam sebelum diberi beban."
    },
    "strengths": {
      "headline": "Desain Estetis Ala Hotel & Material Bebas Karat",
      "detail": "Material alumunium anoda ringan yang tahan air garam/sabun, dipadu dengan estetika minimalis modern."
    },
    "reality_check": {
      "skenario_buruk": "Rak tiba-tiba jatuh ambruk di malam hari dan stiker perekat terkelupas penuh.",
      "root_cause": "Daya rekat stiker melemah akibat kelembapan tinggi dan permukaan tembok keramik masih berkerak sisa sabun/minyak saat ditempel.",
      "mismatched_buyer": "Pemilik kamar mandi dengan dinding tembok cat biasa, keramik berpori kasar, atau keramik bertekstur kulit jeruk."
    },
    "durability_forecast": {
      "bulan_1": "Berfungsi sempurna, tampilan ruangan rapi seketika.",
      "bulan_3": "Titik kritis perekat jika sering terkena uap air panas (hot shower) secara langsung tanpa ventilasi memadai."
    },
    "solutions": {
      "maintenance_tip": "Lap bersih area tempel, diamkan 24 jam tanpa beban setelah perekat dipasang.",
      "diy_fix": "Jika perekat bawaan mulai longgar, perkuat area pinggir stiker menggunakan silikon sealant transparan."
    },
    "verified_wins": {
      "buyer_quote": "Awalnya was-was takut copot. Tapi pas ikutan tips bersihin pake alkohol dan didiemin 24 jam, kokoh banget gak geser sebulan lebih!",
      "buyer_persona": "Anak Kos / Pengguna Teliti",
      "success_factor": "Disiplin mengikuti protokol pembersihan permukaan dinding sebelum pemasangan."
    },
    "bundle_cross_sell": {
      "item_name": "Stiker Perekat Cadangan Heavy Duty 3M + Cairan Alkohol Medis 70%",
      "reason": "Menjamin daya rekat 2x lebih kuat dan memastikan permukaan dinding bebas minyak sebelum ditempel."
    }
  }
]

```

## 6. RANCANGAN DESAIN UI/UX WEB APP (GRADIO LAYOUT)

Antarmuka Web App dirancang menggunakan kartu produk yang terstruktur (*The Yin & Yang Reality Check Layout*):

```
+-----------------------------------------------------------------------------------+
|  🔍 SEARCH BAR: [ Ketik nama produk atau masalah rumah tangga... ]               |
+-----------------------------------------------------------------------------------+
|  📊 MATCH SCORE: 94%  |  SKOR KEPAKARAN: 8.2/10                                   |
|  PRODUK: Rak Dinding Kamar Mandi Tempel Alumunium Tanpa Bor                       |
+-----------------------------------------------------------------------------------+
|  🎯 MASALAH UTAMA YANG DITUNTASKAN (Core Problem Solver)                          |
|  • Mengatasi kamar mandi sempit & berantakan tanpa merusak dinding keramik.      |
|  ⚙️ EFISIENSI FUNGSIONAL (Product Utility)                                        |
|  • Lubang drainase bawah mencegah sabun lembek & tempat berkembang biak bakteri.  |
+-----------------------------------------------------------------------------------+
|  💚 APA YANG BIKIN JATUH CINTA                 |  ⚠️ PRE-CHECKOUT REALITY CHECK   |
|  • Desain estetis ala hotel modern.           |  • Rak rawan ambruk jika dipasang |
|  • Material alumunium anti karat.             |    di dinding berpori/berminyak.  |
|                                               |  • Akar Masalah: Kelembapan &     |
|                                               |    kerak sabun di permukaan.      |
+-----------------------------------------------------------------------------------+
|  ⭐ KISAH SUKSES PEMBELI (Verified Win)                                            |
|  "Awalnya was-was takut copot. Tapi pas ikutan tips bersihin pake alkohol dan     |
|  didiemin 24 jam, kokoh banget gak geser sebulan lebih!"                          |
+-----------------------------------------------------------------------------------+
|  🛠️ TIPS PEMECAHAN MASALAH (Smart Workaround & Solution)                          |
|  • Lap dinding dengan alkohol 70% & wajib diamkan 24 jam sebelum diberi beban.   |
|  • Trik DIY: Tambahkan silikon sealant transparan jika dinding agak lembap.       |
+-----------------------------------------------------------------------------------+
|  📦 RECOMMENDED BUNDLE (Cross-Sell)                                               |
|  • Disarankan beli bersama: Stiker Heavy Duty 3M + Alkohol Medis 70%              |
+-----------------------------------------------------------------------------------+
|  [ 🛑 Batalkan & Cari Lain ]     [ 🛒 Amankan Solusi Ini via Shopee 🚀 ]          |
+-----------------------------------------------------------------------------------+
|  💌 REQUEST REVIEW: [ Tempel Link Shopee Produk Incaran Anda ] -> [ Kirim ]      |
+-----------------------------------------------------------------------------------+

```

## 7. STRATEGI RETENSI & PEMICU TRAFIK BERULANG (TRAFFIC LOOP)

Untuk memastikan Web App tidak hanya dikunjungi sekali, 3 fitur pengikat disuntikkan ke dalam aplikasi:

1. **Request Product Review Box (Pemicu Kembali):**

   * Pengguna dapat menempelkan URL produk Shopee yang ingin mereka ketahui risiko dan kegunaannya.

   * *Efek Retensi:* Pengguna akan kembali ke Web App 1-2 hari kemudian untuk mengecek apakah riset produk mereka sudah diproses oleh AI.

2. **Shareable Reality Check Card (Pemicu Viral Organik):**

   * Tombol *\[ Copy Summary for WhatsApp \]* yang merangkum hasil *Reality Check*, kelebihan, dan solusi ke dalam format teks rapi.

   * *Efek Trafik:* Pengguna membagikan peringatan ini ke grup keluarga/teman atau kolom komentar TikTok untuk menyelamatkan orang lain dari barang *zonk*.

3. **Interactive Buyer Quiz (Kuis Saring Kebutuhan):**

   * Kuis interaktif 3 pertanyaan di halaman utama (Kategori $\rightarrow$ Budget $\rightarrow$ Kondisi Rumah) untuk memberikan rekomendasi produk paling tepat secara instan.

## 8. ROADMAP IMPLEMENTASI EKSEKUSI

* \[x\] **Fase 1:** Finalisasi Dokumentasi Arsitektur & Master Prompt 3.0.

* \[ \] **Fase 2:** Riset 5-10 Produk Perdana menggunakan Google AI Mode + Chrome Side Panel.

* \[ \] **Fase 3:** Sintesis Data via NotebookLM & Penyusunan File `reality_check_db.json`.

* \[ \] **Fase 4:** Pembuatan Kode Python Gradio (`app.py`) & Pengujian UI/UX Lokal.

* \[ \] **Fase 5:** Deployment ke Hugging Face Spaces & Pengujian Link Afiliasi Shopee.