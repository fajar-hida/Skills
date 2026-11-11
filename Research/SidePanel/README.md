# Shopee Consumer Research — Gemini in Chrome Skills

**Dokumentasi pipeline:** SidePanel Skills v2.0  
**Kompatibilitas riset lanjutan:** Google Search AI Mode Master Prompt 4.0  
**Bahasa operasional:** Indonesia

Paket ini berisi empat Custom Skill untuk membangun dataset penelitian konsumen dari satu listing Shopee. Pipeline memisahkan profiling listing, ekstraksi ulasan, audit integritas data, dan ekspor agar setiap tahap memiliki tanggung jawab yang jelas serta data tetap dapat ditelusuri.

---

## 1. Komponen pipeline

| Skill | Versi | Fungsi utama | Output inti |
|---|---:|---|---|
| 01 — Shopee Product Profiler | 2.0 | Memprofilkan satu listing Shopee | Product Manifest |
| 02 — Shopee Review Extractor | 2.0 | Mengekstrak kartu ulasan yang terlihat | Review CSV + metadata eksekusi |
| 03 — Shopee Review Auditor | 2.0 | Mengaudit identitas, struktur, provenance, duplikasi, dan coverage | Audit Report |
| 04 — Shopee Review Exporter | 2.0 | Mengonsolidasikan dan mengemas data yang diberikan | Master CSV + NotebookLM Raw Text + Export Metadata |

### Pembagian tanggung jawab

- **Product Profiler** menangkap informasi listing dan snapshot marketplace. Ia tidak memverifikasi klaim penjual secara independen.
- **Review Extractor** menyalin informasi dari kartu ulasan yang benar-benar terlihat. Ia tidak menilai kualitas produk.
- **Review Auditor** memeriksa integritas dan keterlacakan dataset. Audit dataset bukan audit kualitas produk.
- **Review Exporter** menggabungkan data yang tersedia tanpa mengubah teks asli atau menghilangkan temuan audit.
- **Google Search AI Mode Master Prompt 4.0** menjalankan riset independen: verifikasi identitas kanonis, spesifikasi, klaim, pengalaman penggunaan, failure modes, DIY fixes, kompetitor, dan buyer fit.
- **NotebookLM** digunakan untuk membaca serta merekonsiliasi sumber SidePanel dan temuan riset independen. NotebookLM bukan pengganti audit data mentah.

---

## 2. Prinsip data yang wajib dipertahankan

### 2.1 Satu Product ID = satu listing Shopee

- product_id mengidentifikasi satu listing, bukan identitas universal produk.
- Listing berbeda harus memiliki product_id berbeda meskipun nama, foto, brand, atau model tampak sama.
- Jangan menggabungkan dua listing hanya berdasarkan kemiripan.
- research_key membantu menghubungkan konteks penelitian, tetapi tidak menggantikan product_id.
- Product URL menjadi referensi listing yang harus dipertahankan persis sesuai sumber.

### 2.2 Product Manifest bukan fakta terverifikasi

Product Manifest adalah snapshot informasi yang ditampilkan satu listing.

Field seperti product_name, brand, seller_claims, listing_specifications, listing_description, displayed_price, displayed_rating, displayed_review_count, dan displayed_sold_count harus tetap diperlakukan sebagai informasi listing, klaim penjual, atau snapshot marketplace. Field tersebut tidak otomatis menjadi fakta independen.

Verifikasi identitas kanonis, manufacturer, model resmi, spesifikasi teknis, dan klaim dilakukan melalui riset independen, terutama dengan Master Prompt 4.0.

### 2.3 Data ulasan harus tetap mentah dan dapat ditelusuri

- original_text harus dipertahankan verbatim.
- Jangan memperbaiki ejaan, menerjemahkan, meringkas, atau memotong teks ulasan.
- Jangan menghapus ulasan GENERIC, LOGISTICS_ONLY, MEDIA_ONLY, MIXED, EMPTY, PARTIAL, atau kandidat duplikat.
- Jangan menyimpulkan ulasan palsu tanpa bukti.
- Jangan memasukkan username atau informasi pribadi reviewer yang tidak diperlukan.
- Jangan menyamakan sampel yang diekstrak dengan seluruh ulasan listing.

### 2.4 Audit dataset bukan penilaian kualitas produk

Status PASS dari Review Auditor hanya menunjukkan bahwa pemeriksaan dataset memenuhi kriteria audit yang berlaku pada input yang diberikan. Status tersebut tidak membuktikan:
- produk berkualitas baik;
- klaim penjual benar;
- pengalaman reviewer autentik;
- seluruh ulasan listing telah dikumpulkan;
- produk aman, tahan lama, atau layak dibeli.

### 2.5 Snapshot memiliki batas waktu

Harga, rating, jumlah ulasan, dan jumlah terjual adalah snapshot pada collection_date Product Manifest. Jangan menyebutnya sebagai nilai terkini tanpa pemeriksaan ulang.

---

## 3. Urutan penggunaan

Gunakan satu percakapan Gemini untuk satu listing jika memungkinkan. Pastikan output tahap sebelumnya tetap tersedia atau disimpan agar dapat diberikan kembali pada tahap berikutnya.

### Tahap 1 — Profilkan listing

1. Buka satu listing produk Shopee pada tab Chrome.
2. Jalankan Skill 01 — Shopee Product Profiler.
3. Minta pembuatan Product Manifest.
4. Periksa product_id, research_key, product_url, shop_name, varian, dan collection_date.
5. Simpan Product Manifest sebagai referensi tetap untuk seluruh proses listing tersebut.

Perintah:

~~~text
Buat Product Manifest v2.0 untuk listing pada tab ini. Pertahankan semua field wajib dan tandai informasi yang tidak terlihat sebagai UNKNOWN.
~~~

Output utama:
- Product Identity Summary
- Product Manifest YAML
- Collection Setup
- Handoff Note

### Tahap 2 — Ekstrak ulasan

1. Buka bagian ulasan pada listing yang sama.
2. Pilih filter rating dan urutan ulasan.
3. Jalankan Skill 02 pada halaman aktif.
4. Pindah halaman secara manual, tunggu kartu ulasan tampil, lalu jalankan Extractor kembali.
5. Ulangi sampai target pengumpulan pengguna tercapai atau halaman yang direncanakan selesai.
6. Ulangi proses untuk kategori rating lain yang ingin diteliti.

Perintah awal:

~~~text
Mulai koleksi ulasan untuk listing ini menggunakan Product Manifest yang diberikan. Ekstrak hanya kartu yang terlihat pada halaman aktif.
~~~

Perintah lanjutan:

~~~text
Lanjutkan collection yang sama. Ekstrak record baru dari halaman aktif saja dan pertahankan collection_id yang sama.
~~~

Catatan:
- Extractor hanya boleh mengeluarkan record yang benar-benar terlihat dan dapat dibaca.
- Satu collection_id digunakan untuk satu product_id dan satu filter rating.
- Jangan mengganti collection_id saat berpindah halaman untuk filter rating yang sama.
- Filter rating berbeda menggunakan collection_id berbeda.
- Pindah halaman, mengganti filter, dan menunggu halaman termuat dilakukan manual kecuali otomasi yang sesuai memang tersedia.

### Tahap 3 — Audit dataset

Setelah record dari halaman dan kategori rating yang diperlukan terkumpul:

1. Berikan Product Manifest.
2. Berikan seluruh blok CSV dari Skill 02.
3. Berikan Page Metadata, Extraction Audit, Collection Progress, dan Handoff Note dari setiap eksekusi yang tersedia.
4. Jalankan Skill 03.
5. Baca temuan, status identitas, provenance, coverage, duplikasi, dan Handoff to Exporter.
6. Jangan mengubah record sumber untuk membuat hasil audit terlihat lebih baik.

Perintah:

~~~text
Audit seluruh record ulasan yang tersedia untuk product_id ini. Gunakan Product Manifest dan seluruh metadata ekstraksi yang saya berikan. Jangan menghapus atau mengubah record maupun original_text.
~~~

Status audit Skill 03:
- PASS
- PASS_WITH_WARNINGS
- NEEDS_REVIEW
- FAIL

Hasil audit terbatas pada input yang benar-benar diberikan. Jika metadata atau halaman tidak tersedia, auditor harus menyatakannya.

### Tahap 4 — Ekspor dataset

Berikan kepada Skill 04:

1. Product Manifest.
2. Seluruh record CSV dari Skill 02.
3. Metadata ekstraksi yang tersedia.
4. Laporan Skill 03.
5. Target atau konteks tambahan yang relevan.

Perintah:

~~~text
Konsolidasikan seluruh input yang saya berikan untuk product_id ini. Pertahankan Product Manifest, header CSV kanonis, provenance, hasil audit, dan original_text. Buat Master CSV serta NotebookLM Raw Text dan laporkan semua keterbatasan.
~~~

Output Skill 04:
- Export Package Summary
- Product Manifest
- Export Inventory
- Consolidation Audit
- Provenance Register
- Audit and Handoff Status
- Master CSV
- NotebookLM Raw Text
- Coverage and Limitations
- Handoff Note

---

## 4. Kontrak ID dan provenance

Gunakan pola yang ditetapkan Skill 02 dan diperiksa Skill 03 serta Skill 04.

| Elemen | Pola | Contoh | Fungsi |
|---|---|---|---|
| product_id | ID satu listing | P001 | Mengikat seluruh data ke satu listing |
| research_key | product_id + listing slug | P001-rak-kamar-mandi-model-x | Menghubungkan konteks penelitian |
| collection_id | product_id-R-rating_filter | P001-R1 | Satu filter rating untuk satu listing |
| review_id | product_id-R-rating_filter-P-page-REV-position | P001-R1-P02-REV04 | Mengidentifikasi record ulasan |
| source_id | SRC-SP-nnn | SRC-SP-001 | Mengidentifikasi sumber/eksekusi SidePanel |
| capture_id | product_id-YYYYMMDD-C-nn | P001-20261111-C01 | Mengidentifikasi satu capture ekstraksi |

Aturan:
- Jangan membuat ulang ID yang sudah diberikan.
- Jangan menebak nomor halaman atau posisi. Gunakan PUNK atau UNK sesuai kontrak Extractor jika tidak diketahui.
- Jangan mengalokasikan record ke capture tertentu jika pemetaannya tidak dapat dibuktikan.
- CSV kanonis tidak memiliki capture_id; hubungan record ke capture harus disimpan melalui metadata provenance jika tersedia.
- Tanggal capture, tanggal snapshot Manifest, dan tanggal ulasan adalah hal yang berbeda.

---

## 5. Standar Master CSV

Skill 02, Skill 03, dan Skill 04 harus menggunakan header yang sama persis dan urutan kolom yang sama:

~~~csv
review_id,collection_id,product_id,product_name,product_url,rating_filter,sort_order,source_page,source_position,rating,variant,review_date,usage_duration,original_text,sentiment,tags,review_type,has_image,has_video,purchase_badge,seller_response,extraction_status,duplicate_status
~~~

Ketentuan:
- Satu record ulasan satu baris.
- Encoding UTF-8 dan delimiter koma.
- Field yang mengandung koma, tanda kutip, atau baris baru harus di-escape sesuai standar CSV.
- Tanda kutip internal digandakan.
- Pertahankan emoji dan karakter bahasa Indonesia.
- Jangan menambahkan kolom provenance atau audit ke CSV kanonis tanpa memperbarui kontrak data dan seluruh skill downstream.
- Simpan metadata tambahan dalam bagian metadata ekspor, bukan dengan mengubah header CSV.

---

## 6. Struktur output per listing

Nama file yang disarankan:

~~~text
<product_id>/
├── <product_id>_product_manifest.yaml
├── <product_id>_shopee_reviews.csv
└── <product_id>_notebooklm_reviews.txt
~~~

### 6.1 Product Manifest YAML

Berisi snapshot satu listing dan klaim/informasi yang ditampilkan listing. Jangan mencampurkan hasil verifikasi AI Mode ke dalam Manifest.

### 6.2 Master CSV

Berisi record ulasan terstruktur dari seluruh halaman dan collection yang benar-benar diberikan. Kandidat duplikat dan record dengan keterbatasan tetap dipertahankan.

### 6.3 NotebookLM Raw Text

Berisi konteks identitas listing, collection, provenance, status audit, coverage, keterbatasan, dan seluruh record ulasan dengan original_text verbatim.

NotebookLM Raw Text adalah dokumen sumber mentah yang disiapkan untuk membantu analisis. Ia tidak boleh menyisipkan kesimpulan atau rekomendasi di antara record.

### 6.4 Export metadata

Skill 04 juga menghasilkan ringkasan dalam output ekspor berupa Export Inventory, Consolidation Audit, Provenance Register, Audit and Handoff Status, Coverage and Limitations, serta Handoff Note. Bagian-bagian ini menjaga keterlacakan tanpa menambah kolom pada CSV kanonis.

---

## 7. Memahami status dan keterbatasan

### 7.1 Status identitas

- MATCH — atribut identitas yang tersedia konsisten dengan Manifest.
- PARTIAL_MATCH — sebagian atribut cocok, sebagian informasi penting tidak tersedia.
- CONFLICT — terdapat konflik identitas.
- NOT_VERIFIED — informasi pembanding tidak tersedia.

### 7.2 Status provenance

- COMPLETE — metadata eksekusi tersedia dan konsisten.
- PARTIAL — sebagian metadata penting tidak tersedia.
- CONFLICT — metadata sumber bertentangan.
- NOT_PROVIDED — metadata tidak disertakan.
- NOT_ASSESSABLE — hubungan record dan capture tidak dapat dinilai.

### 7.3 Status kesiapan ekspor

- READY_FOR_EXPORT — input utama tersedia, identitas tidak bermasalah, struktur dapat diproses, dan audit PASS.
- READY_WITH_WARNINGS — ekspor dapat dibuat dengan keterbatasan atau temuan yang harus dibawa ke tahap berikutnya.
- NOT_READY — konflik atau kerusakan membuat penggabungan tidak aman.
- PARTIAL_EXPORT — hanya sebagian input atau record dapat diproses dengan aman.

Export readiness berbeda dari audit status. File yang berhasil dibuat tidak otomatis berarti dataset lolos audit.

### 7.4 Istilah keterbatasan

| Status | Makna |
|---|---|
| NOT_PROVIDED | Komponen tidak disertakan dalam input |
| NOT_VISIBLE | Informasi tidak terlihat pada sumber |
| UNKNOWN | Nilai tidak diketahui |
| NOT_ASSESSABLE | Bukti tidak cukup untuk menilai |
| PARTIAL | Hanya sebagian data tersedia atau dapat diproses |

---

## 8. Kontinuitas percakapan dan penyimpanan

Custom Skill menyimpan instruksi, bukan database permanen.

Ketika berpindah halaman atau menjalankan skill berikutnya:
- Output sebelumnya hanya dapat digunakan jika masih tersedia dalam konteks percakapan atau diberikan kembali.
- Jangan mengandalkan model untuk mengingat semua record dari eksekusi lama.
- Simpan Product Manifest, setiap output Extractor, laporan Auditor, dan paket ekspor di penyimpanan pengguna.
- Jika konteks percakapan hilang, berikan kembali file/data sebelumnya.
- Jika hanya ringkasan atau jumlah yang tersisa tanpa record mentah, jangan merekonstruksi record dari ringkasan tersebut.

Praktik yang disarankan:
- Satu percakapan untuk satu listing.
- Simpan semua output Extractor selama proses pengumpulan.
- Jalankan Auditor setelah data yang diperlukan tersedia.
- Jalankan Exporter menggunakan input mentah dan laporan audit.
- Simpan paket ekspor final sebelum memulai penelitian listing lain.

---

## 9. Hubungan dengan Master Prompt 4.0 dan NotebookLM

Pipeline SidePanel mengumpulkan dan mengaudit bukti marketplace. Riset AI Mode memiliki fungsi berbeda: memeriksa klaim dan identitas produk menggunakan sumber independen.

Alur integrasi yang disarankan:

1. Buat Product Manifest dari listing Shopee.
2. Kumpulkan review dan jalankan audit SidePanel.
3. Ekspor Product Manifest, Master CSV, NotebookLM Raw Text, dan metadata ekspor.
4. Gunakan Product Manifest sebagai referensi listing ketika menjalankan Google Search AI Mode Master Prompt 4.0.
5. Jangan perlakukan informasi Manifest sebagai fakta terverifikasi; Master Prompt harus menguji identitas dan klaim melalui bukti independen.
6. Masukkan hasil riset AI Mode dan paket ekspor SidePanel ke NotebookLM.
7. Rekonsiliasi bukti dengan mempertahankan asal sumber dan mencatat konflik.
8. Lakukan sintesis konsumen hanya setelah batas data, identitas, provenance, dan konflik bukti dipahami.

### Pemisahan namespace sumber

Pertahankan namespace sumber sesuai asal data. Contoh:
- SRC-SP-001 untuk sumber SidePanel.
- SRC-AI-001 untuk sumber riset AI Mode.
- EV-SP-001 untuk evidence yang berasal dari SidePanel.
- EV-AI-001 untuk evidence yang berasal dari AI Mode.

Jangan menggunakan ulang ID yang sama untuk sumber berbeda. Jangan mengubah asal bukti hanya karena bukti tersebut dipindahkan ke NotebookLM.

---

## 10. Perintah singkat

| Skill | Perintah |
|---|---|
| 01 — Product Profiler | Buat Product Manifest v2.0 untuk listing pada tab ini. |
| 02 — Review Extractor (awal) | Mulai koleksi ulasan untuk halaman aktif berdasarkan Product Manifest ini. |
| 02 — Review Extractor (lanjutan) | Lanjutkan collection yang sama; ekstrak record baru dari halaman aktif saja. |
| 03 — Review Auditor | Audit seluruh record ulasan yang tersedia untuk product_id ini. Jangan hapus atau ubah teks asli. |
| 04 — Review Exporter | Konsolidasikan seluruh input yang saya berikan dan buat Master CSV serta NotebookLM Raw Text untuk product_id ini. |

Perintah singkat tidak menggantikan input yang dibutuhkan. Sertakan Manifest, record, metadata, dan laporan audit ketika skill memerlukannya.

---

## 11. Quality checklist pipeline

Sebelum menggunakan hasil untuk analisis atau keputusan pembelian, pastikan:

- [ ] Satu Product Manifest mewakili satu listing.
- [ ] product_id dan research_key konsisten di seluruh tahap.
- [ ] Klaim penjual dan spesifikasi listing belum dianggap fakta independen.
- [ ] Record berasal dari kartu ulasan yang benar-benar terlihat dan dapat dibaca.
- [ ] Filter rating, halaman, dan posisi tidak tertukar.
- [ ] original_text dipertahankan verbatim.
- [ ] Record generik, logistik, campuran, kosong, partial, atau kandidat duplikat tidak dihapus diam-diam.
- [ ] Seluruh metadata ekstraksi yang tersedia diberikan kepada Auditor.
- [ ] Hasil audit dan temuan yang belum terselesaikan dibawa ke Exporter.
- [ ] Header Master CSV sama persis dengan kontrak kanonis.
- [ ] Jumlah input dan output dapat direkonsiliasi.
- [ ] Provenance tidak dibuat berdasarkan tebakan.
- [ ] Status audit dibedakan dari export readiness.
- [ ] Coverage dan limitations dinyatakan secara eksplisit.
- [ ] Dataset tidak disebut sebagai sensus seluruh ulasan tanpa bukti yang memadai.
- [ ] Hasil AI Mode tetap terpisah dari Product Manifest dan record review mentah.

---

## 12. Batas akses dan larangan umum

- Ekstraksi hanya mencakup kartu ulasan yang benar-benar terlihat dan dapat dibaca dari tab aktif.
- Skill tidak menjamin akses ke seluruh halaman, seluruh ulasan, atau seluruh varian listing.
- Perpindahan halaman, penggantian filter, dan pemuatan konten dilakukan manual kecuali fitur otomasi tersedia secara eksplisit.
- Jangan mengklaim data telah disimpan permanen jika hanya muncul di percakapan.
- Jangan mengklaim file dibuat atau dapat diunduh sebelum pembuatan file benar-benar berhasil.
- Jangan menyebut dataset sebagai seluruh ulasan listing jika hanya sampel yang tersedia.
- Jangan menyimpulkan manipulasi, ulasan palsu, atau kualitas produk tanpa bukti independen yang memadai.
- Jangan membuat root cause analysis, diagnosis, DIY fix, prediksi daya tahan, atau rekomendasi pembelian pada tahap ekstraksi, audit, atau ekspor.
- Jangan mengubah kontrak data satu skill secara sepihak tanpa memeriksa kompatibilitas skill lainnya.

---

## 13. Isi paket

- 01-shopee-product-profiler.md — Product Manifest v2.0.
- 02-shopee-review-extractor.md — Review Extractor v2.0.
- 03-shopee-review-auditor.md — Review Auditor v2.0.
- 04-shopee-review-exporter.md — Review Exporter v2.0.
- README.md — dokumentasi operasional pipeline.

Folder:
https://github.com/fajar-hida/Skills/tree/main/Research/SidePanel

Master Prompt 4.0:
https://raw.githubusercontent.com/fajar-hida/Skills/main/Research/AI%20Mode/Master_Prompt.md

---

## Canonical Data Contract v1.0

Kontrak lintas pipeline sekarang ditetapkan dalam [Canonical Data Contract v1.0](../Canonical_Data_Contract_v1.0.md). Dokumen tersebut menjadi acuan normatif untuk identitas, namespace Source/Evidence ID, status, enum, pemetaan AI Mode–NotebookLM, dan kompatibilitas.

Perubahan penting:
- Header Master CSV tetap 23 kolom dan tidak berubah.
- Skill 02 hanya mengekstrak review; Skill 04 membuat `evidence_mapping` di metadata ekspor.
- Evidence ID SidePanel menggunakan `EV-SP-000001` dan seterusnya, dengan mapping stabil ke `source_id`, `review_id`, `collection_id`, `product_id`, serta locator halaman/posisi.
- Handoff Skill 03 tetap tiga status. `PARTIAL_EXPORT` hanya status Exporter.
- Audit Status, Export Readiness, dan Publication Status adalah dimensi yang berbeda.
- NotebookLM menerima AI Mode dossier dan SidePanel export package melalui adapter prompt v1.1.
