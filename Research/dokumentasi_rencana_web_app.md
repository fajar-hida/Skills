# BLUEPRINT & DOKUMENTASI ARSITEKTUR WEB APP

## Indonesia-First Consumer Intelligence & Evidence-Based Reality Check Affiliate System

**Versi:** 5.0 — Evidence-Based Consumer Intelligence  
**Status:** Blueprint arsitektur dan kontrak produk  
**Bahasa utama:** Bahasa Indonesia  
**Target implementasi awal:** Python + Gradio, dengan arsitektur data yang siap dipindahkan ke database relasional.

---

## 1. IKHTISAR SISTEM

Web App ini adalah **alat bantu keputusan belanja independen** yang membantu konsumen Indonesia memahami kecocokan produk, manfaat nyata, risiko, keterbatasan, bukti pengalaman pengguna, dan opsi penyelesaian masalah sebelum melakukan pembelian.

Fokus utamanya adalah **Pre-Checkout Reality Check** yang dipadukan dengan **Problem-Solving & Functional Utility**. Aplikasi tidak boleh berubah menjadi halaman hard-selling atau mesin pemberi skor tanpa dasar.

### 1.1 Tujuan produk

1. Membantu pengguna menilai apakah suatu produk sesuai dengan kebutuhan dan kondisi penggunaannya.
2. Menjelaskan manfaat fungsional berdasarkan bukti, bukan hanya menyalin klaim penjual.
3. Menampilkan keluhan, keterbatasan, dan potensi ketidakcocokan secara proporsional.
4. Menyediakan solusi, perawatan, atau workaround hanya jika dasar bukti dan batas keselamatannya jelas.
5. Menghubungkan setiap temuan penting ke sumber yang dapat diperiksa.
6. Menghasilkan rekomendasi yang transparan, termasuk ketika bukti belum cukup untuk menyimpulkan.

### 1.2 Prinsip utama

- **Evidence before persuasion:** klaim dan rekomendasi harus ditopang sumber yang dapat ditelusuri.
- **The Honest Balance:** tampilkan manfaat dan risiko secara seimbang. Banyaknya ulasan positif tidak menghapus risiko yang relevan, dan satu keluhan tidak otomatis membuktikan cacat umum.
- **Problem-Solving First:** jelaskan masalah pengguna yang mungkin terbantu, kondisi penggunaan, dan batas manfaat produk.
- **Identity before synthesis:** pastikan produk dan listing yang dibahas benar sebelum menggabungkan data.
- **Traceability by design:** setiap temuan penting memiliki Evidence ID dan Source ID.
- **Uncertainty is valid output:** gunakan status belum diketahui atau bukti tidak cukup; jangan mengisi kekosongan dengan dugaan.
- **No fabricated authority:** jangan menampilkan expert score, match percentage, durability forecast, atau angka keyakinan tanpa metodologi yang terdokumentasi dan dapat diuji.
- **Ethical affiliate:** komisi afiliasi tidak boleh menentukan urutan, skor, atau kesimpulan rekomendasi.
- **Privacy by minimization:** kumpulkan data minimum yang diperlukan dan jangan menyimpan identitas pribadi reviewer.

### 1.3 Batas sistem

Aplikasi bukan laboratorium pengujian produk, lembaga sertifikasi, layanan diagnosis keselamatan, atau jaminan kualitas. Aplikasi menyintesis sumber yang tersedia. Temuan harus dibedakan dari hasil pengujian fisik independen.

---

## 2. ARSITEKTUR DAN ALUR DATA

### 2.1 Pipeline riset

1. **Google AI Mode — Master Prompt 4.0:** riset independen lintas platform, identitas produk, spesifikasi produsen, pengalaman penggunaan, kegagalan, solusi, pesaing, dan kecocokan pembeli.
2. **Chrome Side Panel — Skill 01 Product Profiler:** menangkap snapshot satu listing Shopee ke Product Manifest.
3. **Chrome Side Panel — Skill 02 Review Extractor:** mengekstrak data ulasan mentah sesuai kontrak CSV, tanpa membuat analisis kualitas atau rekomendasi.
4. **Chrome Side Panel — Skill 03 Review Auditor:** memeriksa konsistensi identitas, provenance, record, duplikasi, dan cakupan dataset.
5. **Chrome Side Panel — Skill 04 Review Exporter:** membuat paket ekspor dan handoff untuk sintesis.
6. **NotebookLM:** menyintesis sumber dan data ekspor menjadi temuan terstruktur dengan rujukan Source ID dan Evidence ID.
7. **Validation & Publication Gate:** validasi skema, identitas, bukti, konflik, keamanan, dan kelayakan publikasi.
8. **Database:** menyimpan entitas, snapshot, dataset, sumber, bukti, temuan, solusi, dan versi publikasi.
9. **Gradio Web App:** menyediakan pencarian, Reality Check, utility, buyer fit, request queue, dan shareable card.

### 2.2 Diagram alur

    [Google AI Mode / Master Prompt 4.0] ----> [Research Sources]
                                                    |
    [Skill 01 Product Profiler] ----------------> [Product Manifest]
                                                    |
    [Skill 02 Review Extractor] ----------------> [Raw Review CSV]
                                                    |
    [Skill 03 Review Auditor] ------------------> [Audit Report]
                                                    |
    [Skill 04 Review Exporter] ------------------> [Export Package]
                                                    |
                                                    v
    [NotebookLM Synthesis + Evidence Mapping]
                         |
                         v
    [Schema / Identity / Evidence / Safety Validation]
                         |
                         v
    [Versioned Database] <----> [Admin Review & Publication]
                         |
                         v
    [Gradio Web App: Search / Reality Check / Utility / Request]

### 2.3 Pemisahan lapisan data

Data harus dipisahkan menjadi tiga lapisan:

- **Raw layer:** salinan sumber, Product Manifest, review CSV, metadata ekstraksi, audit, dan dokumen riset. Data mentah tidak boleh ditimpa oleh kesimpulan AI.
- **Derived layer:** entitas identitas, evidence register, temuan, konflik, solusi, buyer fit, dan hasil sintesis.
- **Published layer:** konten yang telah melewati validasi dan persetujuan publikasi. UI publik hanya membaca versi yang berstatus published.

Jangan menjadikan satu objek JSON hasil NotebookLM sebagai satu-satunya sumber kebenaran. JSON dapat dipakai sebagai format pertukaran atau prototipe, tetapi data harus dapat dinormalisasi dan diverifikasi.

---

## 3. KONTRAK IDENTITAS PRODUK

### 3.1 Identitas listing dan identitas produk

Istilah berikut tidak boleh dipertukarkan:

- **product_id / listing_id:** ID untuk satu listing Shopee yang diteliti. Mengikuti identitas listing pada Product Manifest.
- **canonical_product_id:** ID produk kanonis lintas listing, hanya boleh diberikan jika kesamaan produk telah diverifikasi secara independen.
- **research_key:** kunci penghubung pekerjaan riset. Bukan pengganti product_id atau canonical_product_id.
- **variant_id:** identitas varian jika dapat dibuktikan dari sumber. Jika varian tidak dapat ditentukan, statusnya harus dinyatakan tidak diketahui.

Satu produk kanonis dapat memiliki beberapa listing. Sebaliknya, dua listing dengan nama, foto, atau kata kunci mirip tidak otomatis merupakan produk yang sama.

### 3.2 Aturan penggabungan

1. Jangan menggabungkan review dari listing berbeda hanya karena nama atau mereknya mirip.
2. Jangan menyatukan varian berbeda jika kesetaraan varian belum terbukti.
3. Jangan menetapkan canonical_product_id berdasarkan kemiripan judul saja.
4. Jika identitas bertentangan, hentikan sintesis lintas listing dan tandai konflik.
5. Product Manifest adalah snapshot listing pada waktu pengambilan, bukan verifikasi independen atas klaim penjual.
6. seller_claims, listing_specifications, dan listing_description harus tetap diberi label sebagai klaim/listing-originated.

### 3.3 Snapshot marketplace

Nilai displayed_price, displayed_rating, displayed_review_count, displayed_sold_count, listed_variants, dan collection_date bersifat temporal. Simpan waktu pengambilan dan jangan menampilkannya seolah-olah data real-time jika belum diperbarui.

---

## 4. KONTRAK INTEGRASI SIDE PANEL

Aplikasi harus kompatibel dengan empat skill pada folder Research/SidePanel dan tidak mengubah kontrak data mereka secara sepihak.

### 4.1 Tanggung jawab skill

| Skill | Tanggung jawab | Bukan tanggung jawab |
|---|---|---|
| 01 Product Profiler | Mengambil snapshot satu listing dan membuat Product Manifest | Menetapkan kebenaran spesifikasi atau identitas kanonis |
| 02 Review Extractor | Mengekstrak record ulasan mentah dan metadata | Menilai kualitas produk, akar masalah, atau rekomendasi |
| 03 Review Auditor | Mengaudit identitas, provenance, konsistensi, duplikasi, dan cakupan | Menghapus data atau menilai kualitas produk |
| 04 Review Exporter | Membuat paket ekspor dan handoff | Mengubah data mentah atau membuat kesimpulan produk |

### 4.2 Kontrak CSV review

Header kanonis berikut harus dipertahankan persis dan berurutan:

    review_id,collection_id,product_id,product_name,product_url,rating_filter,sort_order,source_page,source_position,rating,variant,review_date,usage_duration,original_text,sentiment,tags,review_type,has_image,has_video,purchase_badge,seller_response,extraction_status,duplicate_status

Ketentuan:

- Jangan menambah, menghapus, mengganti nama, atau mengubah urutan kolom tanpa revisi kontrak seluruh skill dan migrasi downstream.
- original_text harus dipertahankan verbatim.
- Jangan menyimpan nama akun, nomor telepon, alamat, atau identitas pribadi reviewer.
- ID yang tidak diketahui tidak boleh ditebak. Penanda PUNK atau UNK adalah keterbatasan data, bukan izin untuk mengarang.
- Data review harus tetap terkait dengan product_id listing yang tepat.
- Status audit dataset tidak sama dengan penilaian kualitas produk.
- Dataset parsial tidak boleh disebut sebagai seluruh ulasan listing.

### 4.3 Paket ekspor Skill 04

Paket minimum yang diharapkan:

- <product_id>_product_manifest.yaml
- <product_id>_shopee_reviews.csv
- <product_id>_notebooklm_reviews.txt

Sertakan audit report, collection progress, provenance register, dan handoff note apabila tersedia pada output skill. Simpan file asli tanpa modifikasi dan catat versi skill/format yang digunakan.

### 4.4 Status yang harus dipertahankan

Status audit dan ekspor tidak boleh disederhanakan menjadi satu boolean.

Audit status:
- PASS
- PASS_WITH_WARNINGS
- NEEDS_REVIEW
- FAIL

Export readiness:
- READY_FOR_EXPORT
- READY_WITH_WARNINGS
- NOT_READY

Coverage status:
- TARGET_REACHED
- IN_PROGRESS
- NOT_SPECIFIED
- PRIOR_DATA_NOT_AVAILABLE
- POTENTIAL_COVERAGE_GAP
- NOT_ASSESSABLE

Status tersebut menjelaskan kondisi dataset dan proses, bukan kualitas produk.

---

## 5. METODOLOGI RISET DAN EVIDENCE MANAGEMENT

### 5.1 Kategori sumber

Sumber dapat berasal dari:

- Dokumentasi resmi produsen, manual, datasheet, sertifikasi, dan kebijakan garansi.
- Listing marketplace dan klaim penjual.
- Review pembeli marketplace.
- Forum, komunitas, Reddit, blog, dan media sosial.
- Video demonstrasi atau pengalaman penggunaan.
- Pengujian independen atau pengukuran yang metodologinya tersedia.
- Sumber regulasi atau standar yang relevan.

Sumber harus diberi tipe dan konteks. Klaim penjual tidak boleh dinaikkan menjadi fakta produsen. Testimoni pengguna tidak otomatis membuktikan performa umum.

### 5.2 ID dan provenance

Gunakan namespace yang konsisten:

- Source ID AI Mode: SRC-AI-001
- Evidence ID AI Mode: EV-AI-001
- Source ID SidePanel: SRC-SP-001
- Evidence ID SidePanel: EV-SP-001

ID harus unik dalam ruang lingkup sistem. Jangan menggunakan dua pola ID berbeda untuk jenis sumber yang sama. Simpan URL atau referensi sumber, judul, platform, tanggal akses, tanggal publikasi jika diketahui, tipe sumber, dan catatan akses.

Untuk review, pertahankan review_id, collection_id, product_id, source_page, source_position, capture_id, dan source_id sesuai informasi yang benar-benar tersedia. Jangan menebak relasi record-ke-capture jika tidak tercatat.

### 5.3 Evidence register

Setiap evidence minimal memiliki:

- evidence_id
- source_id
- product_id dan/atau canonical_product_id jika relevan
- variant_id atau status varian
- evidence_type
- claim_or_observation
- verbatim_excerpt bila kutipan diperlukan
- source_locator (URL, halaman, timestamp video, review_id, atau lokasi lain)
- observed_at / accessed_at
- source_date bila tersedia
- verification_status
- limitations
- related_finding_ids

Kutipan harus benar-benar berasal dari sumber, singkat, dan tidak boleh direkayasa atau digabung seolah-olah satu kutipan.

### 5.4 Status verifikasi

Gunakan status yang jelas, misalnya:

- **MANUFACTURER_CONFIRMED:** didukung dokumen resmi produsen.
- **INDEPENDENTLY_TESTED:** didukung pengujian independen dengan metode yang dapat ditinjau.
- **MULTI_SOURCE_CORROBORATED:** didukung beberapa sumber independen yang konsisten.
- **USER_REPORTED:** pengalaman atau laporan pengguna.
- **SELLER_CLAIM:** klaim pada listing atau materi penjual.
- **HYPOTHESIS:** dugaan yang masuk akal tetapi belum terverifikasi.
- **CONFLICTING:** terdapat bukti yang saling bertentangan.
- **UNVERIFIED:** belum ada dasar yang cukup.
- **NOT_AVAILABLE:** informasi tidak ditemukan atau tidak dapat diakses.

Status ini bukan ranking kualitas. Jangan menaikkan status hanya karena jumlah pengulangan klaim.

### 5.5 Claim-to-evidence mapping

Setiap finding yang ditampilkan sebagai fakta atau dasar rekomendasi harus mempunyai satu atau lebih evidence_ids. Temuan tanpa evidence harus ditampilkan sebagai pertanyaan terbuka, hipotesis, atau informasi yang belum tersedia—bukan fakta.

Jika sumber bertentangan:
1. Simpan kedua bukti.
2. Jelaskan aspek yang bertentangan.
3. Bandingkan otoritas, kedekatan sumber dengan produk, tanggal, varian, dan metode.
4. Jangan menghapus bukti yang tidak mendukung narasi.
5. Publikasikan konflik atau tahan klaim jika belum dapat diselesaikan.

### 5.6 Kekuatan bukti dan confidence

Jangan menggunakan angka confidence 0–100 atau skor pakar tanpa definisi, kalibrasi, dan pengujian. Untuk versi awal, gunakan label kualitatif seperti HIGH, MODERATE, LIMITED, atau INSUFFICIENT hanya jika kriteria operasionalnya ditetapkan.

Minimal pertimbangkan:
- otoritas sumber;
- independensi sumber;
- relevansi terhadap model/varian;
- keterkinian;
- metode pengamatan/pengujian;
- konsistensi lintas sumber;
- keterbatasan sampel.

Banyak review yang menyalin pola kalimat atau berasal dari konteks yang sama tidak otomatis menjadi bukti independen.

---

## 6. NOTEBOOKLM — PROMPT SINTESIS TERKONTROL

NotebookLM menghasilkan **draft sintesis JSON** yang mengikuti kontrak Web App. Prompt operasional lengkap dan schema kanonis dipisahkan ke dokumen khusus agar dapat disalin ke NotebookLM tanpa mengambil instruksi yang bersifat dokumentasi aplikasi.

**Prompt kanonis:** [NotebookLM Synthesis Prompt v1.1](NotebookLM/NotebookLM_Synthesis_Prompt_v1.1.md)

### 6.1 Input wajib

Untuk satu proses sintesis satu listing, masukkan:

1. Product Manifest v2.0 dari Skill 01.
2. Paket ekspor Skill 04: Master CSV, NotebookLM Raw Text, Export Package Summary, Provenance Register, Audit and Handoff Status, Coverage and Limitations, dan Handoff Note jika tersedia.
3. Hasil riset Google AI Mode dengan Master Prompt 4.0.
4. Dokumen resmi produsen, manual, datasheet, pengujian independen, atau sumber pendukung lain yang benar-benar digunakan.

Jangan mencampur listing berbeda. Jika sumber beberapa listing terlanjur berada dalam notebook yang sama, NotebookLM harus melaporkan konflik identitas dan tidak menyintesisnya sebagai satu produk.

### 6.2 Aturan proses

- Product Manifest menjadi referensi identitas listing dan snapshot, bukan bukti kebenaran klaim.
- Skill 03/04 menyediakan status integritas dataset dan provenance; status tersebut tidak boleh diubah menjadi penilaian kualitas produk.
- AI Mode menyediakan riset independen. Klaim seller, laporan pengguna, dokumen produsen, dan hasil uji harus tetap dibedakan.
- NotebookLM mempertahankan upstream Source ID dan Evidence ID; ID upstream yang hilang tidak boleh dibuat-buat. ID untuk entitas sintesis baru (finding, solution, conflict, question) dibuat deterministik sesuai Canonical Data Contract §3.5.
- Setiap finding dan solution wajib memiliki evidence_ids yang valid.
- Bukti yang bertentangan harus dipertahankan sebagai conflict, bukan dipilih diam-diam.
- Review asli tetap berada pada raw package. NotebookLM tidak boleh menulis ulang original_text atau membuat kutipan seolah berasal dari reviewer.
- Output tidak boleh memuat expert_score, match_score, persentase risiko, durability forecast, atau klaim kuantitatif tanpa metodologi yang disetujui.
- Solusi DIY harus memiliki bukti, kondisi penerapan, batas, dan catatan keselamatan. Jika tidak aman/tidak cukup bukti, jangan merekomendasikannya.
- NotebookLM hanya boleh menghasilkan publication_status draft/review/blocking. Ia tidak boleh menetapkan APPROVED atau PUBLISHED.

### 6.3 Output yang diharapkan

NotebookLM mengeluarkan satu objek JSON dengan schema_version 1.0. Objek mencakup:

- research_metadata dan input_inventory;
- product_identity dan listing_snapshot;
- dataset_audit;
- sources dan evidence;
- findings (product_utility, strengths, risks_and_limitations, buyer_fit, root_cause_assessments, verified_wins);
- solutions;
- recommendation;
- conflicts dan open_questions;
- affiliate disclosure metadata.

Jika data tidak tersedia, gunakan null atau array kosong. Jangan mengisi field agar tampak lengkap. Prompt kanonis mendefinisikan struktur lengkap setiap objek, enum status, aturan referensi, dan quality gate.

### 6.4 File output dan handoff

Simpan hasil sebagai file draft bernama `PRODUCT_ID_notebooklm_synthesis.json`. Nama tersebut adalah pola penamaan, bukan ID baru. Simpan semua input mentah secara terpisah.

Sebelum import:
1. Parse JSON.
2. Validasi schema_version dan seluruh field.
3. Cocokkan product_id/research_key/product_url dengan Manifest.
4. Pastikan setiap source_id/evidence_id yang dirujuk tersedia.
5. Pastikan finding/solution tidak memiliki referensi bukti yang putus.
6. Periksa konflik identitas, varian, dan bukti.
7. Jalankan pemeriksaan keselamatan solusi.
8. Masukkan ke status DRAFT atau NEEDS_HUMAN_REVIEW; jangan publish otomatis.

## 7. MODEL DATA DAN SKEMA PERTUKARAN

### 7.1 Entitas konseptual

| Entitas | Fungsi | Relasi utama |
|---|---|---|
| Canonical Product | Identitas produk lintas listing yang telah diverifikasi | Dapat memiliki banyak Listing |
| Listing | Satu listing marketplace | Memiliki banyak Snapshot dan Review Collection |
| Listing Snapshot | Kondisi listing pada waktu pengambilan | Terhubung ke Manifest |
| Variant | Varian yang dapat dibuktikan | Terikat ke listing/produk sesuai bukti |
| Review Collection | Satu rangkaian pengambilan review | Memiliki banyak Review Record |
| Review Record | Data review mentah | Terikat pada satu listing dan collection |
| Source | Asal informasi | Memiliki banyak Evidence |
| Evidence | Potongan bukti yang terlokalisasi | Mendukung atau membantah banyak Finding |
| Finding | Kesimpulan/observasi terstruktur | Memiliki banyak Evidence |
| Solution | Perawatan, workaround, atau solusi resmi | Terhubung ke Finding dan Evidence |
| Research Run | Satu pelaksanaan riset/sintesis | Menyimpan versi prompt dan input |
| Publication | Versi konten yang disetujui untuk publik | Mengacu pada temuan dan data versi tertentu |
| Product Request | Permintaan riset dari pengguna | Memiliki status proses dan hasil |

### 7.2 Relasi utama

- Satu canonical product dapat terhubung ke banyak listing, tetapi hanya setelah verifikasi identitas.
- Satu listing dapat memiliki banyak snapshot dan review collection.
- Satu review record hanya boleh terikat pada listing yang benar-benar menjadi sumbernya.
- Satu source dapat menghasilkan banyak evidence.
- Satu finding dapat didukung atau dibantah oleh banyak evidence.
- Satu evidence dapat terkait ke beberapa finding.
- Satu publication harus menunjuk ke versi data dan riset yang digunakan.

### 7.3 Kontrak JSON pertukaran — versi 1.0

Schema JSON kanonis untuk output NotebookLM ditetapkan dalam dokumen [NotebookLM Synthesis Prompt v1.1](NotebookLM/NotebookLM_Synthesis_Prompt_v1.1.md), bukan pada contoh JSON parsial di blueprint ini.

Kontrak tersebut adalah sumber kebenaran untuk:
- nama dan struktur seluruh key;
- tipe data dan penggunaan null/array kosong;
- enum status;
- struktur sources, evidence, findings, solutions, conflicts, dan open_questions;
- pemetaan input SidePanel/AI Mode;
- publication status dan validasi referensi.

Jangan membuat implementasi backend dari contoh lama atau menebak field yang tidak dijelaskan. Jika schema berubah, naikkan schema_version, perbarui prompt NotebookLM, validator backend, dokumentasi, dan migration secara bersamaan.

### 7.4 Relasi dan validasi referensi

- `product_identity.product_id` harus sama dengan Product Manifest.
- `listing_identity_status` memeriksa kecocokan listing terhadap Manifest; `canonical_identity_status` memeriksa verifikasi identitas produk kanonis. Kedua status tidak boleh digabung.
- `canonical_product_id` hanya diisi setelah verifikasi independen.
- Setiap `evidence.source_id` harus merujuk ke salah satu objek pada `sources`.
- Setiap `finding.evidence_ids`, `solution.evidence_ids`, dan referensi bukti pada `conflicts` harus merujuk ke objek pada `evidence`.
- Referensi yang tidak ditemukan harus menyebabkan validation failure atau human review; jangan diperbaiki dengan membuat ID baru.
- Dataset audit harus mempertahankan status Skill 03/04 dan tidak boleh dipetakan ke product quality score.

### 7.5 Versioning dan kompatibilitas

Simpan secara terpisah:
- `schema_version`;
- `synthesis_prompt_version`;
- Master Prompt version;
- SidePanel skill versions;
- research_run_id;
- synthesized_at;
- input inventory.

Setiap perubahan schema harus memiliki migration dan pengujian backward compatibility. Data mentah tidak boleh ditimpa saat migrasi.

---

## 8. VALIDASI, MODERASI, DAN GATE PUBLIKASI

Tidak ada hasil sintesis yang otomatis tampil ke publik.

### 8.1 Validasi otomatis

Sebelum publikasi, sistem memeriksa:

1. Validitas JSON dan schema_version.
2. Kesesuaian product_id dengan Product Manifest.
3. Status canonical identity dan pemetaan varian.
4. Keberadaan setiap source_id dan evidence_id yang dirujuk.
5. Tidak ada finding faktual tanpa evidence.
6. Kutipan memiliki sumber dan locator.
7. Status audit dan export readiness tidak saling disamakan.
8. Tidak ada review dari listing lain yang tercampur.
9. Tidak ada field wajib yang diisi dengan placeholder seolah fakta.
10. Affiliate URL dan URL sumber lolos validasi format serta kebijakan keamanan.
11. Solusi DIY memiliki safety_notes dan kondisi penerapan.
12. Tidak ada klaim kuantitatif yang tidak memiliki metode/sumber.

### 8.2 Pemeriksaan manusia

Admin/editor harus meninjau:
- konflik identitas dan varian;
- klaim risiko tinggi atau keselamatan;
- root cause dan klaim ketahanan;
- rekomendasi DIY;
- kutipan dan konteks sumber;
- buyer mismatch;
- rekomendasi bundle;
- disclosure afiliasi;
- bahasa yang terlalu absolut atau menyesatkan.

### 8.3 Status publikasi

- DRAFT
- VALIDATION_FAILED
- NEEDS_HUMAN_REVIEW
- APPROVED
- PUBLISHED
- ARCHIVED

Setiap perubahan konten publik membuat publication version baru. Simpan siapa/apa yang menyetujui, waktu persetujuan, versi dataset, dan versi prompt yang digunakan.

---

## 9. DESAIN UI/UX WEB APP

### 9.1 Halaman utama

Komponen:
- Search bar berdasarkan nama, kategori, brand, model, atau masalah pengguna.
- Kategori produk dan kebutuhan.
- Buyer Quiz opsional.
- Product Request Box.
- Informasi cara kerja, batas data, dan disclosure afiliasi.

### 9.2 Halaman Product Reality Check

Urutan tampilan yang disarankan:

1. **Identitas produk:** nama listing, merek/model jika terverifikasi, varian, toko, dan tautan listing.
2. **Research status:** tanggal pembaruan, status identitas, status audit dataset, dan cakupan sampel.
3. **Ringkasan keputusan:** cocok, cocok dengan syarat, kurang cocok, atau bukti belum cukup. Keputusan harus diturunkan dari aturan yang transparan, bukan skor arbitrer.
4. **Masalah yang dapat dibantu:** kebutuhan dan kondisi penggunaan yang relevan.
5. **Manfaat yang didukung bukti:** setiap manfaat dapat dibuka untuk melihat sumber.
6. **Reality Check:** risiko, keluhan, keterbatasan, dan konteks frekuensinya tanpa menggeneralisasi sampel kecil.
7. **Root cause:** pisahkan terkonfirmasi, hipotesis, dan belum diketahui.
8. **Buyer fit / mismatch:** siapa yang mungkin cocok dan kondisi yang membuat produk kurang sesuai.
9. **Solutions & maintenance:** pisahkan petunjuk resmi, workaround pengguna, DIY yang ditinjau, dan kebutuhan profesional.
10. **Evidence drawer:** sumber, kutipan singkat, tanggal, locator, status verifikasi, dan keterbatasan.
11. **Related products / bundle:** hanya jika relevansi fungsional terbukti dan hubungan komersial diungkapkan.
12. **Affiliate CTA:** tombol pembelian dengan disclosure yang jelas.

### 9.3 Jangan tampilkan tanpa metodologi

- Expert Score 8.2/10.
- Match Score 94%.
- Persentase risiko atau durability prediction.
- Estimasi usia pakai bulan 1/bulan 3.
- Klaim "anti karat", "waterproof", "aman", "pasti awet", atau "wajib beli" yang hanya bersumber dari seller claim.
- Ringkasan jumlah review yang seolah mewakili semua pembeli jika dataset hanya sampel.

### 9.4 Bahasa dan visualisasi

Gunakan bahasa Indonesia yang mudah dipahami. Label teknis dapat disertai penjelasan singkat. Bedakan visual status bukti, bukan hanya warna positif/negatif. Hindari desain yang menakut-nakuti atau mendorong pembelian secara manipulatif.

---

## 10. FITUR UTAMA DAN TRAFFIC LOOP

### 10.1 Product Request Queue

Pengguna dapat mengirim URL listing Shopee untuk meminta riset.

Alur:
1. Validasi URL dan domain yang diizinkan.
2. Normalisasi URL untuk mengidentifikasi listing.
3. Cek apakah listing sudah ada atau sedang diproses.
4. Tampilkan status antrean.
5. Admin meninjau permintaan dan menentukan prioritas.
6. Riset mengikuti pipeline SidePanel dan AI Mode.
7. Setelah publikasi, pengguna dapat menemukan hasil melalui URL/ID produk.

Status request:
- RECEIVED
- URL_INVALID
- DUPLICATE
- QUEUED
- IN_RESEARCH
- WAITING_FOR_DATA
- IN_REVIEW
- PUBLISHED
- REJECTED
- CANCELLED

Jangan menjanjikan waktu penyelesaian tertentu jika kapasitas riset belum dapat dipastikan.

### 10.2 Shareable Reality Check Card

Sediakan tombol salin ringkasan yang memuat:
- identitas produk/listing;
- ringkasan manfaat;
- risiko utama;
- siapa yang cocok/tidak cocok;
- status keterbatasan bukti;
- tautan halaman sumber.

Jangan menghilangkan caveat atau status bukti ketika membuat ringkasan shareable.

### 10.3 Interactive Buyer Quiz

Kuis dapat menanyakan:
- kategori/kebutuhan;
- rentang anggaran;
- kondisi penggunaan;
- batasan penting pengguna.

Rekomendasi harus menjelaskan alasan dan bukti. Jangan menyebut hasil kuis sebagai jaminan kecocokan.

### 10.4 Related products dan bundle

Cross-sell hanya ditampilkan jika:
- fungsi pelengkapnya jelas;
- kompatibilitas didukung sumber;
- tidak ada klaim "wajib" tanpa dasar;
- potensi konflik kepentingan diungkapkan;
- rekomendasi tidak dipengaruhi besaran komisi.

---

## 11. MONETISASI DAN TRANSPARANSI AFILIASI

1. Beri disclosure yang mudah ditemukan bahwa sebagian tautan dapat menghasilkan komisi.
2. Disclosure harus muncul dekat CTA atau rekomendasi komersial, bukan hanya di halaman terpisah.
3. Komisi tidak boleh memengaruhi kesimpulan, status buyer fit, urutan hasil, atau penilaian risiko.
4. Jika produk tidak cocok atau bukti belum cukup, aplikasi harus berani menyatakan demikian.
5. Tautan afiliasi harus dipisahkan dari URL sumber riset.
6. Simpan affiliate_url secara terpisah dari product_url asli.
7. Jangan membuat klaim diskon, harga, stok, atau ketersediaan tanpa data terbaru.

---

## 12. KEAMANAN, PRIVASI, DAN OPERASIONAL

### 12.1 URL dan request safety

- Batasi URL request pada domain marketplace yang diizinkan.
- Tolak skema selain HTTPS bila tidak diperlukan.
- Jangan melakukan arbitrary server-side fetch terhadap URL kiriman pengguna.
- Cegah SSRF, redirect ke alamat privat, dan akses metadata/cloud internal.
- Jangan mengeksekusi konten atau file yang diunggah pengguna.
- Batasi ukuran input, frekuensi request, dan panjang teks.
- Validasi URL afiliasi dan URL eksternal sebelum ditampilkan.

### 12.2 Secret dan akses

- API key, token, dan credential disimpan di environment secrets, bukan di source code atau database publik.
- Pisahkan peran admin/editor dan pengguna publik.
- Terapkan autentikasi untuk dashboard administrasi.
- Jangan menyimpan data pribadi reviewer atau data pengguna yang tidak diperlukan.
- Sediakan backup, logging, dan prosedur pemulihan.

### 12.3 Audit trail

Catat:
- perubahan schema;
- import/export data;
- perubahan identity mapping;
- perubahan evidence/finding;
- hasil validasi;
- persetujuan publikasi;
- perubahan affiliate link;
- waktu dan versi proses.

Log tidak boleh menyimpan credential atau data pribadi yang tidak diperlukan.

---

## 13. ARSITEKTUR TEKNIS DAN DEPLOYMENT

### 13.1 Tahap prototipe

- Python untuk backend dan validasi.
- Gradio untuk UI prototipe.
- Pydantic atau JSON Schema untuk validasi payload.
- JSON versioned dapat digunakan sebagai penyimpanan awal dengan backup dan validasi ketat.
- Hugging Face Spaces dapat digunakan untuk demo setelah secrets dan akses admin ditangani dengan benar.

### 13.2 Tahap pertumbuhan

Saat data dan pengguna bertambah, migrasikan ke PostgreSQL atau database relasional setara. Simpan file mentah di object/file storage dan simpan metadata serta relasi di database.

Struktur modul yang disarankan:
- ingestion
- identity_resolution
- review_data
- source_registry
- evidence_registry
- synthesis
- validation
- publication
- product_search
- request_queue
- affiliate
- admin
- audit_log

### 13.3 Prinsip implementasi

- UI tidak boleh menjadi tempat logika validasi utama.
- Validasi harus berada di backend dan dapat diuji.
- Simpan schema_version dan prompt/skill version pada setiap research run.
- Sediakan migration script untuk perubahan schema.
- Hindari satu file JSON besar yang ditulis bersamaan oleh banyak proses.
- Pisahkan data draft dari data published.
- Sediakan endpoint/service internal yang konsisten sebelum memperluas UI.

---

## 14. ROADMAP IMPLEMENTASI

### Fase 1 — Kontrak dan fondasi data
- [ ] Tetapkan schema_version dan data dictionary.
- [ ] Finalisasi pemetaan Product Manifest v2.0.
- [ ] Tetapkan kontrak CSV review yang tidak berubah.
- [ ] Tetapkan Source/Evidence ID dan claim-to-evidence mapping.
- [ ] Tetapkan identity resolution dan aturan varian.

### Fase 2 — Pipeline riset
- [ ] Gunakan Master Prompt 4.0.
- [ ] Jalankan SidePanel Skill 01–04.
- [ ] Simpan raw export dan audit report.
- [ ] Finalisasi prompt NotebookLM sesuai kontrak schema.
- [ ] Uji konflik identitas, bukti kosong, bukti bertentangan, dan dataset parsial.

### Fase 3 — Backend dan validasi
- [ ] Implementasi model Pydantic/JSON Schema.
- [ ] Implementasi import dan validasi Product Manifest/CSV/export package.
- [ ] Implementasi source/evidence/finding registry.
- [ ] Implementasi publication gate dan versioning.
- [ ] Implementasi request queue dan audit log.

### Fase 4 — UI Gradio
- [ ] Search dan halaman detail produk.
- [ ] Evidence drawer dan status bukti.
- [ ] Reality Check dan buyer fit.
- [ ] Solution & safety notes.
- [ ] Product Request Queue.
- [ ] Shareable Reality Check Card.
- [ ] Buyer Quiz.
- [ ] Affiliate disclosure dan CTA.

### Fase 5 — QA dan deployment
- [ ] Uji schema, identity, provenance, evidence links, dan migration.
- [ ] Uji input berbahaya, URL tidak valid, rate limit, dan secrets.
- [ ] Uji tampilan pada mobile dan desktop.
- [ ] Uji semua klaim publik terhadap evidence.
- [ ] Backup dan recovery test.
- [ ] Deploy prototipe dan monitor error.

### Fase 6 — Scale dan monetisasi
- [ ] Evaluasi migrasi PostgreSQL.
- [ ] Tambahkan pencarian terindeks.
- [ ] Tambahkan workflow editorial dan multi-admin.
- [ ] Tambahkan analytics yang menghormati privasi.
- [ ] Evaluasi affiliate conversion tanpa mengubah independensi rekomendasi.

---

## 15. ACCEPTANCE CRITERIA / DEFINISI SELESAI

Versi awal Web App belum dianggap siap publik sebelum seluruh kriteria kritis berikut terpenuhi:

1. Satu product_id hanya mewakili satu listing.
2. canonical_product_id tidak dibuat otomatis dari kemiripan nama.
3. Review dari listing berbeda tidak tercampur.
4. Header CSV kanonis dipertahankan persis.
5. Raw data dapat ditelusuri dan tidak ditimpa hasil sintesis.
6. Setiap finding publik memiliki evidence_ids yang valid.
7. Setiap evidence memiliki source_id dan locator yang dapat diperiksa.
8. Konflik dan keterbatasan tidak disembunyikan.
9. Audit dataset tidak dipresentasikan sebagai audit kualitas produk.
10. Sampel review tidak diklaim sebagai seluruh populasi review.
11. Root cause dibedakan antara terkonfirmasi dan hipotesis.
12. Solusi DIY memiliki bukti, batas penerapan, dan catatan keselamatan.
13. Tidak ada skor atau forecast tanpa metodologi yang terdokumentasi.
14. Hanya konten APPROVED yang dapat dipublikasikan.
15. Affiliate disclosure terlihat dan tautan afiliasi terpisah dari sumber.
16. URL request divalidasi dan tidak memicu arbitrary server-side fetch.
17. Perubahan schema dan konten memiliki versi serta audit trail.

---

## 16. KEPUTUSAN DESAIN YANG DIPERTAHANKAN DARI VERSI SEBELUMNYA

Konsep berikut tetap dipertahankan karena masih relevan:
- Pre-Checkout Reality Check.
- Problem-Solving First dan Product Utility.
- Keseimbangan manfaat dan risiko.
- Verified Wins jika kutipan dan sumbernya benar-benar tersedia.
- Maintenance, workaround, dan DIY dengan kontrol bukti serta keselamatan.
- Bundle/Cross-Sell yang relevan dan transparan.
- Product Request Box dan status antrean.
- Shareable Reality Check Card.
- Interactive Buyer Quiz.
- Gradio sebagai jalur prototipe.

Perubahan terpenting dari versi 4.0 adalah bahwa **bukti, identitas listing, provenance, audit dataset, dan validasi publikasi kini menjadi fondasi sistem**, bukan sekadar tambahan setelah sintesis. Dengan demikian, Web App tidak hanya menyajikan ringkasan AI, tetapi dapat menjelaskan dari mana setiap kesimpulan berasal, seberapa terbatas bukti yang tersedia, dan kapan sistem sebaiknya tidak memberikan rekomendasi.

---

## 17. Canonical Data Contract v1.0 — Normative Cross-Pipeline Standard

Dokumen normatif: [Canonical Data Contract v1.0](Canonical_Data_Contract_v1.0.md).

### 17.1 Source of truth
Kontrak kanonis menetapkan identity model, Source/Evidence ID namespace, deterministic SidePanel evidence mapping, source type, verification, audit, export readiness, coverage, solution taxonomy, recommendation, dan publication status. Prompt/skill tidak boleh mendefinisikan enum yang bertentangan. Bila ada konflik, pertahankan raw value dan tandai untuk review.

### 17.2 AI Mode adapter
Master Prompt 4.0 tetap menghasilkan Markdown dossier untuk riset manusia, disertai `NOTEBOOKLM HANDOFF DATA` yang memuat research metadata, source registry, evidence registry, findings index, dan unresolved items. NotebookLM Prompt v1.1 mengubah dossier tersebut menjadi JSON canonical. Jangan mengharuskan AI Mode menghasilkan schema NotebookLM secara langsung.

Pemetaan section Master Prompt ke JSON dan alias field diatur pada Canonical Data Contract §6 dan NotebookLM Prompt v1.1.

### 17.3 SidePanel Evidence Registry
Skill 04 wajib mengeluarkan `evidence_mapping` pada Export Package Summary dan NotebookLM Raw Text. Mapping menghubungkan `EV-SP-000001` ke source_id, product_id, collection_id, review_id, serta page/position locator. Header CSV tidak berubah. Package tanpa mapping belum memiliki full evidence traceability dan harus ditandai PARTIAL_EXPORT atau diregenerasi sesuai kontrak.

### 17.4 Status dimensions
Simpan terpisah:
- Dataset Audit Status: PASS, PASS_WITH_WARNINGS, NEEDS_REVIEW, FAIL; AUDIT_NOT_PROVIDED hanya untuk input yang tidak menyertakan audit.
- Skill 03 Handoff Status: READY_FOR_EXPORT, READY_WITH_WARNINGS, NOT_READY.
- Export Readiness: READY_FOR_EXPORT, READY_WITH_WARNINGS, NOT_READY, PARTIAL_EXPORT.
- Publication Status: DRAFT, VALIDATION_FAILED, NEEDS_HUMAN_REVIEW, BLOCKED_IDENTITY, APPROVED, PUBLISHED, ARCHIVED.

NotebookLM tidak memiliki kewenangan menyetujui atau memublikasikan. Hanya editorial workflow Web App dapat menetapkan APPROVED/PUBLISHED/ARCHIVED.

### 17.5 Solution model
Backend menyimpan solution_type, solution_source, solution_verification_status, dan safety_status secara terpisah sesuai Canonical Data Contract §5.10. Jangan memetakan workaround pengguna menjadi solusi resmi produsen.

### 17.6 Migration and validation
Validator wajib memeriksa enum membership, listing identity, canonical identity status, source/evidence referential integrity, deterministic mapping uniqueness, evidence locator, solution safety, dan publication gate. Jangan mengubah raw data atau renumber ID saat migrasi.
