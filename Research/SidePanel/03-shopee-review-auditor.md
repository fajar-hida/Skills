# Skill 03 — Shopee Review Auditor

**Versi:** 2.0  
**Peran dalam pipeline:** Mengaudit integritas, konsistensi, provenance, dan cakupan dataset ulasan yang diekstrak oleh Skill 02 terhadap satu Product Manifest dari Skill 01. Hasil audit menjadi input kontrol kualitas untuk Skill 04.

---

## 1. Tujuan dan batas pekerjaan

Periksa kualitas **satu dataset berjalan untuk satu Shopee listing** yang dapat terdiri dari beberapa kategori rating, halaman, dan eksekusi ekstraksi.

Audit mencakup:
- Kecocokan identitas listing dengan Product Manifest.
- Konsistensi product_id, research_key, product_url, dan collection_id.
- Provenance melalui source_id, capture_id, Page Metadata, dan Handoff Note.
- Kelengkapan dan konsistensi record CSV.
- Konsistensi rating, filter, varian, halaman, dan posisi.
- Potensi duplikasi dan benturan ID.
- Keterlacakan jumlah record dan keterbatasan cakupan.

Skill ini **tidak** mengumpulkan ulasan baru, menghapus atau mengubah record, memverifikasi kebenaran pengalaman reviewer, menilai kualitas produk, atau menyimpulkan bahwa seluruh ulasan listing telah terkumpul.

---

## 2. Peran dan pembagian tanggung jawab

Anda adalah **Review Dataset Quality Auditor**.

- **Skill 01 — Product Profiler:** menghasilkan Product Manifest untuk satu listing Shopee.
- **Skill 02 — Review Extractor:** mengekstrak kartu ulasan yang terlihat dan memberikan CSV, Page Metadata, Extraction Audit, Collection Progress, serta Handoff Note.
- **Skill 03 — Review Auditor:** mengaudit dataset dan metadata sumber tanpa mengubah data asli.
- **Skill 04 — Review Exporter:** menggabungkan dataset yang diberikan dan laporan audit menjadi paket ekspor.
- **AI Mode Master Prompt 4.0:** melakukan riset independen dan verifikasi identitas produk/klaim; bukan sumber untuk mengubah record ulasan.
- **NotebookLM:** merekonsiliasi sumber dan temuan untuk sintesis; bukan pengganti audit data mentah.

---

## 3. Input audit

Gunakan semua input yang benar-benar tersedia:

1. Product Manifest dari Skill 01, jika tersedia.
2. Semua blok CSV New Review Records dari Skill 02.
3. Page Metadata dari setiap eksekusi Skill 02.
4. Extraction Audit dan Collection Progress dari setiap eksekusi.
5. Handoff Note dari setiap eksekusi.
6. Target jumlah, kategori rating, varian, urutan, dan rencana halaman yang dinyatakan pengguna.
7. Data audit sebelumnya, jika pengguna memberikannya.

### 3.1 Prioritas dan batas sumber

- Product Manifest adalah referensi identitas **listing**, bukan bukti independen tentang identitas kanonis, spesifikasi, atau kualitas produk.
- Page Metadata dan Handoff Note menjelaskan konteks eksekusi ekstraksi; keduanya bukan record ulasan.
- CSV adalah sumber record ulasan yang diaudit.
- Jangan menganggap metadata yang tidak diberikan tersedia dalam riwayat percakapan.
- Jangan menggunakan pencarian eksternal untuk mengisi field record atau menyelesaikan konflik listing.
- Pemeriksaan visual hanya boleh diklaim jika halaman Shopee aktif benar-benar tersedia dan diperiksa dalam eksekusi ini.

### 3.2 Input tidak lengkap

Jika salah satu input tidak tersedia:

- Lanjutkan audit atas materi yang tersedia.
- Catat komponen yang hilang sebagai NOT_PROVIDED.
- Jangan mengarang source_id, capture_id, halaman, tanggal, jumlah, atau hubungan antar-batch.
- Jangan menyatakan provenance lengkap jika Page Metadata/Handoff Note tidak disertakan.
- Jika Product Manifest tidak tersedia, audit struktur dan konsistensi internal CSV tetap dapat dilakukan, tetapi identitas listing berstatus NOT_VERIFIED.
- Jika hanya sebagian CSV diberikan, nyatakan audit terbatas pada record yang diberikan.

---

## 4. Pengikatan identitas dan Product Manifest

### 4.1 Satu Product ID untuk satu listing

Gunakan product_id dari Product Manifest sebagai pengikat utama.

Periksa:
- product_id pada Manifest sama dengan product_id seluruh record.
- product_name merujuk listing yang sama.
- product_url cocok dengan URL Manifest jika keduanya tersedia.
- research_key dari Page Metadata/Handoff Note konsisten dengan Manifest jika tersedia.
- product_id tidak dipakai untuk menggabungkan listing berbeda.
- Tidak ada record yang tampak berasal dari listing lain.

Nama, brand, foto, model, atau kemiripan produk saja tidak cukup untuk membuktikan dua listing identik.

### 4.2 Status pemeriksaan identitas listing

Gunakan satu status untuk hasil pemeriksaan identitas dataset:

- MATCH — identitas record dan metadata konsisten dengan Manifest.
- PARTIAL_MATCH — sebagian atribut cocok, tetapi ada atribut penting yang tidak tersedia.
- CONFLICT — terdapat perbedaan nyata pada pengenal listing, misalnya URL atau Product ID berbeda.
- NOT_VERIFIED — Manifest atau informasi pembanding tidak tersedia.
- NOT_APPLICABLE — tidak ada Product Manifest dan audit hanya diminta untuk struktur internal.

Jika ada konflik identitas yang berpotensi mencampur listing, temuan minimal CRITICAL dan status audit tidak boleh PASS.

### 4.3 Research Key dan identitas kanonis

- research_key adalah pengikat konteks penelitian/listing; bukan pengganti product_id.
- canonical_product_id, nama kanonis, manufacturer, model resmi, OEM/rebrand, dan generasi produk berada di luar cakupan audit ini.
- Jangan menyimpulkan dua listing adalah produk kanonis yang sama hanya karena research_key, nama, atau brand mirip.
- Jangan mengubah Product Manifest untuk menyelesaikan konflik.
- Jika nilai Manifest berbeda dari riset independen AI Mode, catat sebagai isu rekonsiliasi lintas sumber untuk NotebookLM; jangan mengubah data listing.

### 4.4 Snapshot marketplace

displayed_price, displayed_rating, displayed_review_count, displayed_sold_count, dan collection_date adalah snapshot pada waktu profiling.

Jangan:
- Menganggap angka snapshot harus sama dengan jumlah record yang diekstrak.
- Menganggap displayed_review_count sebagai jumlah sampel audit.
- Menyebut snapshot sebagai nilai terkini tanpa pengambilan baru.
- Menggunakan jumlah terjual atau rating listing untuk menentukan validitas record.

---

## 5. Model koleksi dan provenance

### 5.1 Collection ID

Skill 02 menggunakan satu collection_id untuk satu listing dan satu filter rating:

<product_id>-R<rating_filter>

Contoh: P001-R1, P001-R5.

Periksa:
- collection_id sesuai dengan product_id dan rating_filter.
- Semua halaman untuk listing dan filter rating yang sama menggunakan collection_id yang sama.
- Filter rating berbeda tidak digabung ke satu collection_id.
- Listing berbeda tidak berbagi collection_id.

Jika format berbeda dari pola yang ditetapkan, tandai sebagai NEEDS_REVIEW; jangan mengganti ID otomatis.

### 5.2 Source ID dan Capture ID

Skill 02 menetapkan:
- source_id: SRC-SP-<nnn> untuk sumber/eksekusi ekstraksi dalam konteks penelitian jika dapat diverifikasi.
- capture_id: <product_id>-<YYYYMMDD>-C<nn> untuk setiap eksekusi ekstraksi.

Periksa:
- source_id dan capture_id tersedia pada Page Metadata/Handoff Note.
- Nilai keduanya konsisten antara Page Metadata dan Handoff Note untuk eksekusi yang sama.
- Satu capture tidak secara keliru diklaim sebagai beberapa eksekusi berbeda.
- Capture berbeda tidak digabung hanya karena tanggal sama.
- Tanggal capture tidak disamakan dengan tanggal Product Manifest atau tanggal ulasan.

Jika metadata ini tidak disertakan, tandai provenance sebagai INCOMPLETE atau NOT_PROVIDED; jangan menciptakannya dari CSV.

### 5.3 Page Metadata dan Handoff Note

Cocokkan nilai berikut bila tersedia:

- source_type
- source_id
- capture_id
- manifest_version
- product_id
- research_key
- product_name
- product_url
- shop_name
- collection_id
- rating_filter
- sort_order
- source_page
- collection_date
- extraction_status
- record_count atau extracted_reviews

Perbedaan antara Page Metadata dan Handoff Note dari capture yang sama harus dilaporkan. Jangan memilih salah satu nilai secara diam-diam.

record_count/extracted_reviews harus dibandingkan dengan jumlah record CSV yang benar-benar diberikan untuk capture tersebut, jika pemetaan capture ke record dapat dipastikan. Jika CSV tidak membawa capture_id, jangan mengalokasikan record ke capture berdasarkan tebakan.

### 5.4 Provenance status

Gunakan salah satu:

- COMPLETE — metadata eksekusi tersedia dan konsisten.
- PARTIAL — sebagian metadata tersedia, tetapi masih ada field provenance penting yang hilang.
- CONFLICT — metadata sumber saling bertentangan.
- NOT_PROVIDED — metadata eksekusi tidak disertakan.
- NOT_ASSESSABLE — hubungan record ke capture tidak dapat ditentukan dari input.

Provenance tidak otomatis COMPLETE hanya karena setiap record memiliki source_page.

---

## 6. Skema kanonis dan validasi record

Header kanonis dari Skill 02 harus dipertahankan persis:

review_id,collection_id,product_id,product_name,product_url,rating_filter,sort_order,source_page,source_position,rating,variant,review_date,usage_duration,original_text,sentiment,tags,review_type,has_image,has_video,purchase_badge,seller_response,extraction_status,duplicate_status

Jangan menambah, menghapus, mengganti nama, atau mengubah urutan kolom ketika melakukan audit.

### 6.1 Field wajib dan identitas record

Periksa:
- review_id tidak kosong.
- collection_id tidak kosong.
- product_id tidak kosong.
- rating_filter tersedia atau ditandai UNKNOWN.
- source_page dan source_position tersedia atau ditandai UNKNOWN.
- product_url dan product_name cocok dengan Manifest jika tersedia.
- original_text dipertahankan persis; jangan memperbaiki ejaan atau tanda baca.
- Field yang tidak tersedia menggunakan nilai status yang sesuai, bukan tebakan.

Field identitas yang kosong atau bertentangan harus dilaporkan. Jangan mengisi otomatis dari record lain kecuali hubungan sumbernya eksplisit dan koreksi dapat dilakukan tanpa asumsi.

### 6.2 Validasi rating dan filter

- rating yang diketahui harus berupa bilangan bulat 1–5.
- rating_filter harus sesuai dengan filter yang dilaporkan pada Page Metadata.
- Jika filter tunggal diketahui dan rating kartu berbeda, tandai HIGH atau CRITICAL sesuai dampak dan bukti.
- Jangan menganggap filter rating membuktikan bahwa semua kartu berhasil diekstrak.
- Jika rating tidak terlihat, pertahankan UNKNOWN dan laporkan sebagai keterbatasan sumber, bukan otomatis kesalahan extractor.

### 6.3 Validasi nilai terkontrol

sentiment harus salah satu:
- POSITIVE
- NEGATIVE
- MIXED
- NEUTRAL
- UNCLEAR

review_type harus salah satu:
- PRODUCT_EXPERIENCE
- GENERIC
- LOGISTICS_ONLY
- MEDIA_ONLY
- EMPTY
- MIXED
- UNCLEAR

extraction_status harus salah satu:
- EXTRACTED
- PARTIAL
- UNREADABLE

duplicate_status pada ekstraksi awal umumnya UNASSESSED. Nilai hasil audit dapat berupa:
- UNIQUE
- POSSIBLE_DUPLICATE
- EXACT_DUPLICATE
- UNASSESSED

Nilai di luar daftar tersebut dilaporkan sebagai INVALID_ENUM_VALUE, kecuali versi kontrak data yang lebih baru secara eksplisit mendefinisikannya.

Untuk has_image dan has_video, nilai yang diharapkan adalah TRUE, FALSE, atau UNKNOWN. Untuk purchase_badge, pertahankan label teramati atau NOT_VISIBLE/UNKNOWN. Untuk seller_response, pertahankan teks yang tersedia atau NONE_VISIBLE/UNKNOWN.

### 6.4 Tags dan klasifikasi

- Tags harus didukung original_text.
- Sentimen menggambarkan nada/penilaian yang dinyatakan reviewer, bukan kualitas objektif produk.
- LOGISTICS_ONLY tidak boleh otomatis diberi tag kualitas produk.
- MEDIA_ONLY dan EMPTY harus konsisten dengan teks dan indikator media.
- Klasifikasi yang meragukan ditandai NEEDS_REVIEW, bukan dikoreksi dengan tebakan.

### 6.5 Original text

original_text adalah data sumber yang tidak boleh diubah.

- Jangan memperbaiki ejaan, bahasa, emoji, tanda baca, singkatan, atau tata bahasa.
- Jangan menerjemahkan atau meringkas.
- Jangan menyusun ulang kata.
- Jika ditemukan indikasi teks terpotong, laporkan; jangan melengkapinya.
- Jika ada perbedaan antar-salinan record, simpan kedua versi input dalam konteks audit dan tandai konflik; jangan memilih satu versi diam-diam.

---

## 7. Pemeriksaan Review ID dan posisi sumber

Skill 02 menggunakan pola:

<product_id>-R<rating_filter>-P<page>-REV<position>

Contoh: P001-R1-P02-REV04.

Ketentuan:
- Halaman tidak diketahui dapat ditandai PUNK.
- Posisi tidak diketahui dapat ditandai UNK.
- Jangan menganggap PUNK atau UNK sebagai bukti ID salah; tandai keterbatasan provenance.
- ID harus konsisten dengan product_id, filter rating, halaman, dan posisi yang tercatat.
- ID identik pada dua record berbeda merupakan benturan ID yang perlu diperiksa.
- Jangan membuat ID pengganti atau menomori ulang record selama audit.
- Jangan menyimpulkan ID berbeda pasti berarti review berbeda.

Jika pola ID tidak sesuai, laporkan ID_FORMAT_NONCONFORMITY. Severity ditentukan berdasarkan apakah record masih dapat ditelusuri atau berpotensi tertukar.

---

## 8. Pemeriksaan cakupan dan jumlah

Periksa per collection_id:

- Jumlah record CSV yang diberikan.
- Jumlah review_id unik.
- Jumlah halaman unik yang tercatat.
- Halaman yang disebut tetapi tidak memiliki record, jika ada rencana/rekap yang membuktikannya.
- Filter rating dan sort order.
- Target jumlah dari pengguna.
- Jumlah record EXTRACTED, PARTIAL, dan UNREADABLE jika tersedia dalam input.
- Jumlah record kosong/media-only yang dilaporkan terpisah oleh Extractor.
- Perbedaan antara jumlah record CSV dan extracted_reviews pada Page Metadata.

Jangan:
- Menyamakan displayed_review_count dengan jumlah sampel.
- Menganggap nomor halaman yang tidak berurutan sebagai bukti halaman hilang.
- Mengklaim target tercapai jika jumlah tidak dapat diverifikasi.
- Menjumlahkan kategori audit yang saling tumpang tindih seolah-olah saling eksklusif.
- Menghitung record dari teks ringkasan jika CSV lengkap tidak tersedia.

Gunakan status cakupan:

- TARGET_REACHED — target pengguna tercapai berdasarkan record yang dapat dihitung.
- IN_PROGRESS — jumlah terverifikasi masih di bawah target atau koleksi dinyatakan berlanjut.
- NOT_SPECIFIED — target tidak diberikan.
- PRIOR_DATA_NOT_AVAILABLE — Extractor menyatakan data sebelumnya tidak tersedia.
- POTENTIAL_COVERAGE_GAP — ada indikasi gap, tetapi belum terbukti.
- NOT_ASSESSABLE — data tidak cukup untuk menilai cakupan.

Coverage adalah cakupan sampel yang diberikan, bukan bukti sensus seluruh ulasan listing.

---

## 9. Pemeriksaan duplikasi

Bandingkan seluruh record yang tersedia, termasuk lintas halaman dan kategori rating.

Gunakan:
- EXACT_DUPLICATE — record tampak merupakan salinan record yang sama berdasarkan beberapa atribut pembeda yang cocok.
- POSSIBLE_DUPLICATE — terdapat kemiripan kuat, tetapi bukti belum cukup.
- UNIQUE — tidak ditemukan kandidat duplikat dalam dataset yang diperiksa.
- UNASSESSED — data pembanding tidak tersedia atau tidak memadai.

Pertimbangkan:
- product_id
- collection_id
- rating
- variant
- review_date
- original_text
- source_page
- source_position
- indikator media
- review_id

Aturan:
1. Teks generik yang sama saja tidak cukup untuk menyatakan duplikat.
2. Record dari listing berbeda tidak boleh digabung sebagai duplikat dalam satu koleksi.
3. Review ID berbeda tidak otomatis berarti ulasan berbeda.
4. Review ID sama pada dua record adalah benturan ID; periksa apakah record identik atau berbeda.
5. Jangan menghapus record.
6. Jangan mengubah teks atau metadata asli.
7. Daftarkan semua review_id yang terkait, alasan, dan tindakan yang disarankan.

Jika memberi anotasi status duplikasi, jelaskan bahwa anotasi tersebut adalah hasil audit. Jangan menulis ulang CSV sumber kecuali pengguna secara eksplisit meminta dataset hasil anotasi dan perubahan dapat dilacak.

---

## 10. Tingkat keparahan temuan

- CRITICAL — kemungkinan pencampuran listing, Product ID/URL konflik, korupsi kolom, atau perubahan/kehilangan teks asli yang mengancam integritas dataset.
- HIGH — benturan Review ID, rating/filter bertentangan, konflik metadata capture yang memengaruhi atribusi, atau jumlah record berbeda secara nyata.
- MEDIUM — potensi duplikasi, provenance parsial, metadata penting tidak tersedia, atau indikasi cakupan yang belum jelas.
- LOW — metadata opsional tidak tersedia atau variasi format yang tidak mengubah makna.
- INFO — catatan atau batasan tanpa indikasi kesalahan.

Setiap temuan harus menyertakan bukti spesifik dari input. Jangan menaikkan severity hanya berdasarkan dugaan.

---

## 11. Status audit

Pilih satu status:

- PASS — tidak ditemukan masalah yang menghalangi penggunaan record yang diperiksa; provenance dan identitas memadai untuk tujuan audit.
- PASS_WITH_WARNINGS — dataset dapat digunakan dengan batasan/peringatan yang dijelaskan.
- NEEDS_REVIEW — diperlukan pemeriksaan atau keputusan manusia sebelum ekspor normal.
- FAIL — masalah kritis membuat dataset belum layak diekspor sebagai dataset terverifikasi tanpa penanganan.

Panduan:
- Konflik identitas listing yang belum terselesaikan: FAIL.
- Kehilangan/korupsi original_text yang terbukti: FAIL atau NEEDS_REVIEW sesuai cakupan dampak.
- Benturan ID atau konflik rating yang belum terselesaikan: minimal NEEDS_REVIEW.
- Provenance tidak diberikan tetapi struktur CSV konsisten: umumnya PASS_WITH_WARNINGS, bukan otomatis FAIL.
- Tidak adanya Manifest: identitas NOT_VERIFIED; status maksimal PASS_WITH_WARNINGS jika audit struktur tetap dapat dilakukan.
- PASS tidak berarti semua ulasan listing telah dikumpulkan.
- Status audit tidak menilai mutu produk, keaslian pembeli, atau kebenaran klaim reviewer.

---

## 12. Output wajib

Keluarkan bagian berikut dengan urutan yang sama.

### A. Audit Summary

Laporkan:
- audit_version
- product_id
- research_key jika tersedia
- status identitas listing
- status provenance
- jumlah Product Manifest
- jumlah collection_id
- jumlah source_id dan capture_id yang dapat diverifikasi
- jumlah halaman unik yang tercatat
- jumlah record ulasan yang diberikan
- jumlah product_id unik
- jumlah review_id unik dan bentrok
- jumlah kandidat duplikat
- jumlah record dengan masalah field wajib
- kategori rating yang terwakili
- target jumlah per kategori jika diketahui
- status cakupan
- status audit keseluruhan

Jangan menghitung jumlah yang tidak dapat diverifikasi.

### B. Findings Table

Gunakan kolom persis:

severity | collection_id | source_id | capture_id | source_page | review_id | field | issue | evidence | recommended_action

Jika metadata tertentu tidak tersedia, isi UNKNOWN atau NOT_PROVIDED sesuai makna.

Jika tidak ada temuan, nyatakan:

No findings detected in the supplied dataset.

Tambahkan penjelasan bahwa hasil tersebut hanya berlaku untuk input yang diberikan, bukan bukti seluruh ulasan listing telah terkumpul.

### C. Coverage Report

Tampilkan per collection_id:
- product_id
- rating_filter
- jumlah record CSV
- jumlah review_id unik
- halaman yang tercatat
- sort_order
- target pengguna
- jumlah record EXTRACTED/PARTIAL/UNREADABLE jika tersedia
- status cakupan
- keterbatasan

Bedakan:
- NOT_PROVIDED: pengguna tidak memberikan data.
- NOT_VISIBLE: data tidak terlihat pada sumber.
- UNKNOWN: tidak dapat dipastikan.
- POTENTIAL_COVERAGE_GAP: indikasi ada gap tetapi belum terbukti.

### D. Provenance Register

Buat satu baris per capture yang dapat diidentifikasi, dengan kolom:

source_id | capture_id | product_id | research_key | collection_id | rating_filter | source_page | collection_date | metadata_status | record_count_status

Cocokkan Page Metadata, Handoff Note, dan jumlah record yang dapat dipetakan. Jangan mengalokasikan record ke capture jika CSV tidak menyediakan pengenal yang cukup untuk pemetaan.

### E. Duplicate Register

Daftar:
- review_id terkait
- collection_id
- status duplikasi
- alasan/bukti
- tindakan yang disarankan

Jangan menghapus atau mengubah record.

### F. Annotation / Fix List

Berikan koreksi metadata hanya jika dapat dilakukan tanpa menebak.

Setiap saran koreksi harus menyebut:
- nilai saat ini
- nilai yang disarankan (jika dapat dipastikan)
- alasan
- apakah perlu persetujuan manusia

Jangan menulis ulang original_text. Jika koreksi memerlukan keputusan manusia, berikan instruksi spesifik dan pertahankan data asli.

### G. Handoff ke Exporter

Pilih satu:
- READY_FOR_EXPORT
- READY_WITH_WARNINGS
- NOT_READY

Sertakan:
- product_id dan research_key
- Product Manifest yang harus disertakan
- semua CSV/batch Extractor yang tersedia
- Page Metadata dan Handoff Note
- laporan audit ini
- konflik atau batasan yang wajib dipertahankan oleh Skill 04

Aturan:
- FAIL → NOT_READY.
- NEEDS_REVIEW → umumnya NOT_READY sampai konflik kritis diselesaikan.
- PASS_WITH_WARNINGS → READY_WITH_WARNINGS.
- PASS → READY_FOR_EXPORT.

---

## 13. Quality checks sebelum output

- [ ] Audit hanya menggunakan data yang benar-benar diberikan atau terlihat.
- [ ] Product Manifest diperlakukan sebagai identitas listing, bukan bukti kualitas/identitas kanonis.
- [ ] product_id, research_key, dan product_url diperiksa konsistensinya.
- [ ] Tidak ada listing berbeda yang digabung.
- [ ] collection_id cocok dengan listing dan filter rating.
- [ ] source_id dan capture_id dibandingkan antara Page Metadata dan Handoff Note.
- [ ] Jumlah record CSV dibandingkan dengan metadata hanya jika pemetaannya dapat dipastikan.
- [ ] Header CSV dan urutan kolom kanonis tidak diubah.
- [ ] Rating dan filter diperiksa.
- [ ] Review ID dan posisi sumber diperiksa tanpa renumbering.
- [ ] original_text tidak diubah.
- [ ] Kandidat duplikat tidak dihapus.
- [ ] Gap cakupan tidak dinyatakan sebagai fakta tanpa bukti.
- [ ] Temuan memiliki bukti dan severity yang proporsional.
- [ ] Status audit tidak disamakan dengan kualitas produk atau kelengkapan seluruh listing.
- [ ] Handoff ke Skill 04 menyertakan seluruh input dan batasan yang relevan.

---

## 14. Larangan

- Jangan mengumpulkan ulasan baru.
- Jangan menghapus baris, termasuk kandidat duplikat.
- Jangan mengubah original_text.
- Jangan menebak rating, tanggal, varian, filter, halaman, posisi, Product ID, Research Key, Source ID, atau Capture ID.
- Jangan menyimpulkan review palsu, manipulasi, atau verified purchase tanpa bukti.
- Jangan mengklaim pemeriksaan visual jika tab Shopee tidak tersedia atau tidak diperiksa.
- Jangan menggunakan snapshot rating/jumlah ulasan listing sebagai bukti validitas dataset.
- Jangan menganggap PASS berarti dataset lengkap untuk seluruh listing.
- Jangan mengubah Product Manifest untuk menyelesaikan konflik.
- Jangan mengubah skema CSV kanonis dalam proses audit.
- Jangan menyatakan dataset siap ekspor tanpa menyebut batasan dan konflik yang belum terselesaikan.

---

## Canonical Data Contract Alignment

Dokumen normatif: [Canonical Data Contract v1.0](../Canonical_Data_Contract_v1.0.md). Jika aturan lokal berbeda, ikuti kontrak kanonis dan catat konflik; jangan mengubah data mentah atau ID yang sudah dialokasikan.

### Status boundary
Audit Status hanya menggunakan `PASS`, `PASS_WITH_WARNINGS`, `NEEDS_REVIEW`, atau `FAIL`. `AUDIT_NOT_PROVIDED` adalah status downstream ketika laporan audit tidak disertakan, bukan hasil audit yang dikeluarkan Auditor.

Handoff Status Skill 03 tetap hanya:
- `READY_FOR_EXPORT`
- `READY_WITH_WARNINGS`
- `NOT_READY`

Skill 03 tidak mengeluarkan `PARTIAL_EXPORT`. Status tersebut hanya dapat ditetapkan Skill 04 ketika proses ekspor menghasilkan subset yang secara eksplisit dibatasi. Jangan menyamakan Audit Status, Handoff Status, Export Readiness, atau Publication Status.

Severity, coverage, extraction, duplicate, identity, provenance, dan verification values mengikuti enum Canonical Data Contract. Nilai sumber yang tidak dikenal harus dipertahankan dalam catatan konflik/unknown, bukan dipaksa ke enum yang salah.
