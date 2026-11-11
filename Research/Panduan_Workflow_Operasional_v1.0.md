# PANDUAN WORKFLOW OPERASIONAL
## Shopee Consumer Intelligence Research Pipeline

**Versi:** 1.0  
**Bahasa:** Indonesia  
**Status:** Panduan operasional  
**Berlaku untuk:** SidePanel Skills 01–04, Google AI Mode Master Prompt 4.0, NotebookLM Synthesis Prompt 1.1, dan Canonical Data Contract 1.0  
**Repository:** https://github.com/fajar-hida/Skills/tree/main/Research

---

## 1. Tujuan dokumen

Dokumen ini menjelaskan cara menjalankan pipeline penelitian produk dari awal sampai menghasilkan paket sintesis terstruktur yang siap diperiksa sebelum masuk ke Web App.

Workflow ini menggabungkan dua jalur pengumpulan bukti:

1. **SidePanel Shopee:** mengambil snapshot listing dan review yang terlihat, mengaudit integritas dataset, lalu mengekspor paket review.
2. **Google AI Mode:** melakukan riset independen untuk memeriksa identitas produk, klaim, spesifikasi, pengalaman penggunaan, failure mode, solusi, alternatif, dan buyer fit.

Kedua jalur digabungkan di **NotebookLM**. Hasil NotebookLM adalah draft sintesis, bukan publikasi otomatis.

> Prinsip utama: satu proses penelitian hanya untuk satu listing Shopee (product_id). Jangan mencampur listing atau varian yang belum terbukti identik.

## 2. Dokumen dan komponen yang digunakan

| Komponen | File repository | Fungsi |
|---|---|---|
| Kontrak data | [Canonical_Data_Contract_v1.0.md](Canonical_Data_Contract_v1.0.md) | Aturan identitas, ID, evidence, enum, dan kompatibilitas |
| Skill 01 | [01-shopee-product-profiler.md](SidePanel/01-shopee-product-profiler.md) | Membuat Product Manifest |
| Skill 02 | [02-shopee-review-extractor.md](SidePanel/02-shopee-review-extractor.md) | Mengekstrak review yang terlihat |
| Skill 03 | [03-shopee-review-auditor.md](SidePanel/03-shopee-review-auditor.md) | Mengaudit dataset review |
| Skill 04 | [04-shopee-review-exporter.md](SidePanel/04-shopee-review-exporter.md) | Membuat export package dan Evidence Mapping |
| Panduan SidePanel | [SidePanel README](SidePanel/README.md) | Petunjuk ringkas penggunaan empat skill |
| Riset AI Mode | [Master_Prompt.md](ModeAI/Master_Prompt.md) | Riset independen dan AI Mode handoff |
| Prompt NotebookLM | [NotebookLM_Synthesis_Prompt_v1.1.md](NotebookLM/NotebookLM_Synthesis_Prompt_v1.1.md) | Mengubah seluruh input menjadi JSON sintesis |

**Urutan otoritas:** Canonical Data Contract mengatur kontrak lintas tahap. Skill dan prompt mengatur pelaksanaan masing-masing tahap. Jika ada perbedaan instruksi, jangan menebak atau mengubah data mentah; catat konflik dan minta pemeriksaan.

## 3. Gambaran alur kerja

~~~text
                SATU LISTING SHOPEE
                        |
                        v
             Skill 01 — Product Profiler
                        |
                        v
                Product Manifest
                   /        \
                  /          \
                 v            v
       Skill 02 — Extractor   Google AI Mode
                 |            Master Prompt 4.0
                 v            |
          Review CSV +        v
          Metadata       AI Mode Research Dossier
                 |
                 v
       Skill 03 — Review Auditor
                 |
                 v
             Audit Report
                 |
                 v
       Skill 04 — Review Exporter
                 |
                 v
          SidePanel Export Package
                 |
                 +----------------------+
                                        |
                   DUA JALUR INPUT      |
                                        v
                              NotebookLM
                         Synthesis Prompt v1.1
                                        |
                                        v
                          JSON Synthesis v1.1
                                        |
                                        v
                         Human / Schema Review
                                        |
                                        v
                          Web App Import Gate
~~~

Tahap AI Mode dapat berjalan setelah Product Manifest tersedia, bersamaan dengan pengumpulan review SidePanel. Namun, kedua jalur baru digabungkan setelah hasilnya disimpan dan diperiksa.

## 4. Persiapan sebelum memulai

### 4.1 Siapkan alat

- Google Chrome dengan akses ke listing Shopee yang akan diteliti.
- Gemini in Chrome / Side Panel yang dapat menjalankan Custom Skills.
- Google Search AI Mode untuk menjalankan Master Prompt 4.0.
- Google NotebookLM untuk menggabungkan dokumen sumber.
- Penyimpanan lokal atau folder proyek untuk menyimpan seluruh output.

Nama menu dan cara memasang Custom Skills dapat berubah mengikuti versi produk. Gunakan file skill pada repository sebagai instruksi masing-masing; jangan menganggap skill otomatis terpasang hanya karena file Markdown tersedia di GitHub.

### 4.2 Pasang atau siapkan empat Custom Skills

Gunakan file berikut sebagai instruksi skill sesuai mekanisme Custom Skills yang tersedia di Gemini in Chrome:

1. 01-shopee-product-profiler.md
2. 02-shopee-review-extractor.md
3. 03-shopee-review-auditor.md
4. 04-shopee-review-exporter.md

Pastikan setiap skill dipanggil sesuai tugasnya. Jangan meminta Extractor melakukan audit atau meminta Auditor mengubah review.

### 4.3 Buat satu folder kerja per listing

Gunakan struktur yang disarankan berikut. Nama file dapat mengikuti nama keluaran Skill 04 selama isi dan identitasnya tetap jelas.

~~~text
Research_Output/
└── P001/
    ├── 01_product_manifest.yaml
    ├── 02_extraction/
    │   ├── P001-R1-page-01.csv
    │   ├── P001-R1-page-02.csv
    │   ├── P001-R5-page-01.csv
    │   └── extraction_metadata/
    ├── 03_audit/
    │   └── P001_review_audit.md
    ├── 04_export/
    │   ├── P001_product_manifest.yaml
    │   ├── P001_shopee_reviews.csv
    │   ├── P001_notebooklm_reviews.txt
    │   ├── export_package_summary.md
    │   ├── provenance_register.md
    │   ├── evidence_mapping.yaml
    │   ├── audit_handoff_status.md
    │   └── coverage_limitations.md
    ├── 05_ai_mode/
    │   └── P001_ai_mode_research.md
    └── 06_notebooklm/
        └── P001_notebooklm_synthesis.json
~~~

Jika Skill 04 mengeluarkan Evidence Mapping di dalam Export Package Summary dan NotebookLM Raw Text, simpan kedua output tersebut. File evidence_mapping.yaml di atas hanya salinan terpisah untuk memudahkan pengelolaan, bukan syarat untuk menambah file baru.

### 4.4 Tetapkan identitas listing

Sebelum memulai, pastikan satu listing memiliki product_id unik, misalnya P001. Jangan memakai product_id yang sama untuk URL listing berbeda.

- **product_id:** satu listing Shopee.
- **research_key:** penghubung proses riset.
- **canonical_product_id:** biarkan null sampai identitas produk kanonis diverifikasi secara independen.
- **Varian:** jangan digabung jika kesetaraannya belum terbukti.

## 5. Jalur A — Pengumpulan data SidePanel

### Tahap 1 — Profilkan listing (Skill 01)

**Tujuan:** menghasilkan Product Manifest sebagai snapshot satu listing.

Langkah:

1. Buka halaman produk Shopee yang akan diteliti di Chrome.
2. Pastikan URL, nama produk, toko, harga, rating, jumlah ulasan, varian, dan informasi lain yang terlihat sesuai dengan listing aktif.
3. Jalankan Skill 01 — Shopee Product Profiler.
4. Berikan perintah berikut:

~~~text
Buat Product Manifest v2.0 untuk listing pada tab ini.
Pertahankan semua field wajib. Catat hanya informasi yang benar-benar terlihat.
Tandai informasi yang tidak tersedia sebagai UNKNOWN.
Jangan melakukan analisis kualitas produk atau memverifikasi klaim penjual.
~~~

5. Periksa hasil dan simpan Product Manifest sebagai 01_product_manifest.yaml.
6. Catat product_id, research_key, product_url, collection_date, dan source_access_status.

**Quality gate tahap 1:**

- [ ] Satu Manifest hanya mewakili satu listing.
- [ ] product_id dan product_url benar.
- [ ] Varian yang terlihat dicatat tanpa asumsi.
- [ ] Klaim seller tetap dianggap klaim seller.
- [ ] Data yang tidak terlihat tidak diisi dengan tebakan.

Jangan lanjut jika identitas listing yang dibuka tidak cocok dengan Manifest.

### Tahap 2 — Ekstrak review (Skill 02)

**Tujuan:** mengumpulkan review yang benar-benar terlihat, bukan membuat penilaian kualitas produk.

Langkah:

1. Tetap gunakan listing dan Product Manifest yang sama.
2. Buka bagian ulasan.
3. Pilih filter rating dan urutan review yang ingin dikumpulkan.
4. Pastikan halaman review sudah selesai dimuat.
5. Jalankan Skill 02 dengan perintah awal:

~~~text
Mulai koleksi ulasan untuk listing ini menggunakan Product Manifest berikut.
Ekstrak hanya kartu ulasan yang terlihat dan dapat dibaca pada halaman aktif.
Jangan membuat analisis kualitas produk atau rekomendasi.
~~~

6. Simpan CSV dan seluruh metadata yang dihasilkan.
7. Pindah halaman secara manual, tunggu konten selesai dimuat, lalu jalankan perintah lanjutan:

~~~text
Lanjutkan collection yang sama untuk product_id ini.
Ekstrak hanya record baru yang terlihat pada halaman aktif.
Pertahankan collection_id untuk filter rating yang sama.
Jangan mengulang record yang sudah diekstrak.
~~~

8. Ulangi sampai target halaman atau target jumlah sampel tercapai.
9. Ulangi untuk filter rating lain yang memang ingin diteliti.

**Aturan penting:**

- Satu collection_id untuk satu listing dan satu filter rating.
- Filter rating berbeda menggunakan collection_id berbeda.
- Jangan mengganti collection_id ketika melanjutkan halaman pada filter yang sama.
- Jangan mengarang nomor halaman atau posisi.
- Pertahankan original_text verbatim.
- Jangan menghapus review kosong, parsial, logistik, atau kandidat duplikat secara diam-diam.
- Simpan Page Metadata, Extraction Audit, Collection Progress, dan Handoff Note dari setiap eksekusi.
- Perpindahan halaman/filter dilakukan manual kecuali fitur otomasi memang tersedia.

**Quality gate tahap 2:**

- [ ] Semua record berasal dari kartu yang benar-benar terlihat.
- [ ] product_id, collection_id, rating_filter, source_page, dan source_position konsisten.
- [ ] Semua CSV memakai header kanonis 23 kolom.
- [ ] Metadata dari setiap eksekusi disimpan.
- [ ] Tidak ada teks asli yang diringkas atau ditulis ulang.

### Tahap 3 — Audit dataset (Skill 03)

**Tujuan:** memeriksa integritas dan keterlacakan dataset. Ini bukan audit kualitas produk atau verifikasi keaslian review.

Siapkan seluruh input:

1. Product Manifest.
2. Semua blok/file CSV hasil Skill 02.
3. Page Metadata setiap eksekusi.
4. Extraction Audit.
5. Collection Progress.
6. Handoff Note.
7. Target pengumpulan, bila ditentukan.

Jalankan Skill 03 dan berikan perintah:

~~~text
Audit seluruh record ulasan yang tersedia untuk product_id ini.
Gunakan Product Manifest dan seluruh metadata ekstraksi yang saya berikan.
Periksa identitas, konsistensi record, provenance, duplikasi, dan coverage.
Jangan menghapus atau mengubah record maupun original_text.
Laporkan input yang tidak tersedia sebagai NOT_PROVIDED atau status yang sesuai.
~~~

Periksa bagian-bagian output Auditor:

- Audit Summary.
- Findings Table.
- Coverage Report.
- Provenance Register.
- Duplicate Register.
- Annotation / Fix List.
- Handoff to Exporter.

**Interpretasi status:**

| Audit Status | Arti operasional |
|---|---|
| PASS | Pemeriksaan yang dapat dilakukan lulus |
| PASS_WITH_WARNINGS | Dapat diteruskan dengan catatan |
| NEEDS_REVIEW | Perlu pemeriksaan manusia sebelum digunakan |
| FAIL | Ada masalah yang membuat dataset tidak aman diproses sebagai satu kesatuan |

Handoff Skill 03 hanya memakai READY_FOR_EXPORT, READY_WITH_WARNINGS, atau NOT_READY. Auditor tidak mengeluarkan PARTIAL_EXPORT.

**Quality gate tahap 3:**

- [ ] Audit menggunakan semua input yang tersedia.
- [ ] Temuan konflik identitas diperiksa.
- [ ] Missing metadata tidak ditutupi.
- [ ] Duplikasi tidak dihapus tanpa aturan.
- [ ] Coverage dinyatakan sebagai cakupan sampel, bukan sensus semua review.
- [ ] Handoff Status dan daftar temuan disimpan.

Jika status FAIL atau ada konflik identitas kritis, selesaikan masalah atau pisahkan data konflik sebelum mengekspor. Jangan memaksa semua data menjadi satu dataset.

### Tahap 4 — Ekspor dataset (Skill 04)

**Tujuan:** menghasilkan paket ekspor yang mempertahankan data mentah, audit, provenance, dan Evidence Mapping.

Berikan kepada Skill 04:

1. Product Manifest.
2. Seluruh CSV hasil ekstraksi.
3. Semua metadata ekstraksi.
4. Laporan lengkap Skill 03.
5. Target dan catatan keterbatasan yang relevan.

Perintah:

~~~text
Konsolidasikan seluruh input yang saya berikan untuk product_id ini.
Pertahankan Product Manifest, header CSV kanonis, original_text,
provenance, hasil audit, coverage, dan seluruh keterbatasan.
Buat Master CSV, NotebookLM Raw Text, Export Package Summary,
Provenance Register, dan evidence_mapping sesuai Canonical Data Contract v1.0.
Jangan mengubah atau mengarang ID.
~~~

Pastikan output berisi:

- Export Package Summary.
- Product Manifest.
- Export Inventory.
- Consolidation Audit.
- Provenance Register.
- Audit and Handoff Status.
- Master CSV.
- NotebookLM Raw Text.
- Coverage and Limitations.
- Handoff Note.
- evidence_mapping pada Export Package Summary dan NotebookLM Raw Text.

**Evidence Mapping wajib:**

- Evidence ID SidePanel menggunakan pola EV-SP-000001, EV-SP-000002, dan seterusnya.
- Setiap mapping menunjuk ke source_id, product_id, collection_id, review_id, dan locator.
- Urutan alokasi mengikuti Canonical Data Contract §3.3.
- Simpan mapping persis. Jangan mengalokasikan ulang ID ketika paket yang sama diekspor ulang.
- Record tanpa review_id stabil masuk ke mapping exceptions dan tidak diberi Evidence ID.
- Evidence ID menunjuk ke record mentah; bukan pernyataan bahwa review autentik.

**Perbedaan status:**

- Audit Status adalah hasil Skill 03.
- Handoff Status adalah keputusan Auditor untuk meneruskan dataset.
- Export Readiness adalah hasil Skill 04.
- PARTIAL_EXPORT hanya digunakan Exporter ketika sebagian data yang aman saja yang dapat diekspor.

**Quality gate tahap 4:**

- [ ] product_id dan Manifest cocok.
- [ ] Jumlah record input dan output dapat direkonsiliasi.
- [ ] Master CSV memakai header kanonis 23 kolom tanpa tambahan.
- [ ] original_text tetap verbatim.
- [ ] Evidence Mapping tersedia dan tidak duplikat.
- [ ] Provenance Register dan audit dibawa ke paket.
- [ ] Coverage dan limitations jelas.
- [ ] Export Readiness tidak disamakan dengan Audit Status.

## 6. Jalur B — Riset independen Google AI Mode

Jalur ini dapat dimulai setelah Product Manifest tersedia. Master Prompt 4.0 menggunakan Manifest sebagai referensi listing, bukan sebagai fakta yang telah diverifikasi.

### Tahap 5 — Jalankan Master Prompt 4.0

1. Buka Google Search AI Mode.
2. Ambil isi file [Master_Prompt.md](ModeAI/Master_Prompt.md) dari repository.
3. Salin seluruh prompt yang berlaku, termasuk bagian output dan Canonical Data Contract handoff.
4. Berikan Product Manifest v2.0 yang berasal dari Skill 01.
5. Tambahkan konteks pengguna bila tersedia: kebutuhan, skenario pemakaian, batas anggaran, lokasi penggunaan, dan prioritas.
6. Jalankan riset sesuai instruksi prompt. Jangan hanya meminta ringkasan umum; prompt harus menjalankan verifikasi identitas, riset sumber, pencarian kontradiksi, dan analisis bukti.
7. Simpan seluruh hasil akhir sebagai P001_ai_mode_research.md.

Gunakan perintah awal berikut setelah prompt dan Manifest diberikan:

~~~text
Lakukan penelitian lengkap untuk satu listing ini berdasarkan Master Prompt 4.0.
Gunakan Product Manifest sebagai referensi awal, bukan sebagai fakta terverifikasi.
Verifikasi identitas dan klaim melalui sumber independen.
Pertahankan Source ID SRC-AI dan Evidence ID EV-AI yang stabil.
Pastikan laporan memiliki bagian NOTEBOOKLM HANDOFF DATA.
Jangan mengarang sumber, URL, kutipan, locator, pengujian, atau hasil yang tidak ditemukan.
~~~

Periksa bahwa dossier memiliki:

- Metadata penelitian / research_run_id.
- Kesimpulan listing identity dan canonical identity secara terpisah.
- Source Registry dengan Source ID.
- Evidence Ledger dengan Evidence ID dan Source ID.
- Findings yang mengacu ke Evidence.
- Claim vs Reality.
- Pengalaman pengguna dan batas representativitas.
- Failure modes dan root cause yang diberi label sesuai tingkat pembuktiannya.
- Problem-solution intelligence dengan batas keselamatan.
- Alternatives / competitors.
- Buyer fit dan recommendation.
- Conflicts, limitations, dan open questions.
- Bagian NOTEBOOKLM HANDOFF DATA.

**Jangan lakukan hal berikut:**

- Jangan menyalin klaim seller sebagai fakta.
- Jangan menyamakan nama produk mirip sebagai identitas sama.
- Jangan menggabungkan bukti varian berbeda.
- Jangan menganggap keluhan tunggal sebagai failure umum.
- Jangan membuat Source ID atau Evidence ID untuk sumber yang tidak ditemukan.
- Jangan mengubah Product Manifest untuk memasukkan hasil riset independen.

**Quality gate tahap 5:**

- [ ] Research run memiliki ID.
- [ ] Setiap sumber memiliki Source ID.
- [ ] Setiap bukti atomik memiliki Evidence ID dan menunjuk ke Source ID.
- [ ] Locator atau URL dapat ditelusuri.
- [ ] Klaim, pengalaman pengguna, hipotesis, dan fakta terkonfirmasi dibedakan.
- [ ] Hasil riset disimpan sebagai Markdown dossier lengkap.

Jika bagian handoff tidak tersedia, jangan meminta NotebookLM mengarangnya. Perbaiki dossier AI Mode terlebih dahulu atau tandai data yang tidak tersedia.

## 7. Jalur C — Sintesis NotebookLM

### Tahap 6 — Siapkan satu NotebookLM untuk satu listing

Buat satu NotebookLM khusus untuk satu product_id. Jangan menggunakan satu notebook untuk menggabungkan listing yang tidak berkaitan.

Tambahkan sumber berikut:

**Wajib:**

1. Product Manifest final dari Skill 01.
2. Export Package Summary Skill 04.
3. Master CSV Skill 04.
4. NotebookLM Raw Text Skill 04.
5. Provenance Register dan Evidence Mapping.
6. Audit and Handoff Status.
7. Coverage and Limitations.
8. AI Mode Research Dossier dari Master Prompt 4.0.

**Tambahan jika digunakan dalam riset:**

- Manual/datasheet produsen.
- Dokumen garansi atau dukungan resmi.
- Hasil pengujian independen.
- Dokumen regulasi atau recall.
- Sumber komunitas yang benar-benar dipakai.

Jangan menambahkan file instruksi sebagai bukti produk. Prompt NotebookLM adalah instruksi pemrosesan, bukan sumber fakta produk.

### Tahap 7 — Jalankan NotebookLM Synthesis Prompt v1.1

1. Buka file [NotebookLM_Synthesis_Prompt_v1.1.md](NotebookLM/NotebookLM_Synthesis_Prompt_v1.1.md).
2. Salin bagian prompt utama dan addendum Canonical Data Contract secara lengkap.
3. Masukkan prompt ke NotebookLM sebagai instruksi untuk sintesis.
4. Pastikan seluruh sumber listing yang benar sudah tersedia.
5. Minta NotebookLM menghasilkan satu objek JSON sesuai schema v1.1.

Perintah eksekusi:

~~~text
Lakukan sintesis untuk product_id ini saja.
Gunakan seluruh sumber yang tersedia dan ikuti NotebookLM Synthesis Prompt v1.1
serta Canonical Data Contract v1.0.
Pertahankan semua upstream Source ID dan Evidence ID.
Gunakan evidence_mapping SidePanel yang disediakan oleh Exporter.
Jangan mengarang bukti atau ID sumber.
Keluarkan tepat satu objek JSON valid sesuai schema_version 1.1,
tanpa Markdown fence atau teks di luar JSON.
~~~

Simpan hasil sebagai P001_notebooklm_synthesis.json.

### Tahap 8 — Periksa hasil sintesis

NotebookLM menghasilkan draft. Sebelum digunakan lebih lanjut, lakukan pemeriksaan:

- [ ] Output dapat diparse sebagai JSON standar.
- [ ] schema_version bernilai 1.1.
- [ ] synthesis_prompt_version bernilai 1.1.
- [ ] product_identity.product_id sama dengan Manifest dan export package.
- [ ] Listing identity dan canonical identity tidak tercampur.
- [ ] Semua Source ID dan Evidence ID merujuk ke input yang tersedia.
- [ ] Evidence SidePanel mengarah ke mapping yang benar.
- [ ] Setiap finding penting memiliki evidence_ids.
- [ ] Counter evidence dan konflik tidak dihilangkan.
- [ ] Solusi memiliki dimensi type, source, verification, implementation, dan safety.
- [ ] DIY yang belum dinilai keselamatannya tidak ditampilkan sebagai instruksi aman.
- [ ] Recommendation memiliki alasan dan evidence.
- [ ] Open questions dan limitations tetap disertakan.
- [ ] Publication status tidak menjadi APPROVED atau PUBLISHED dari NotebookLM.

Jika NotebookLM menghasilkan ID yang tidak sesuai aturan, jangan memperbaiki dengan mengganti upstream ID. Ulangi sintesis dengan koreksi yang spesifik.

## 8. Handoff dan penyimpanan hasil

Simpan output final per listing dengan struktur berikut:

| Artefak | Format | Fungsi |
|---|---|---|
| Product Manifest | YAML | Snapshot listing |
| Review dataset | CSV | Record review terstruktur |
| Review raw text | TXT | Konteks dan teks review asli |
| Audit Report | Markdown | Integritas dan coverage dataset |
| Export Package Summary | Markdown/YAML | Ringkasan ekspor |
| Provenance Register | Markdown/CSV | Jejak sumber dan capture |
| Evidence Mapping | YAML/JSON | Penghubung Evidence ID ke review |
| AI Mode Dossier | Markdown | Riset independen |
| NotebookLM Synthesis | JSON | Draft sintesis terstruktur |

Jangan menimpa raw input dengan hasil sintesis. Simpan setiap versi output dan catat waktu pembuatannya.

**Penting:** workflow ini saat ini merupakan proses manual/semi-manual lintas Gemini in Chrome, Google Search AI Mode, dan NotebookLM. Membuat file di GitHub tidak otomatis memasang skill, memindahkan data antar-aplikasi, menjalankan riset, atau mengimpor JSON ke Web App.

## 9. Aturan keputusan jika terjadi masalah

| Kondisi | Tindakan |
|---|---|
| Listing berbeda dari Manifest | Hentikan tahap tersebut; periksa product_id dan URL |
| Varian tidak dapat dipastikan | Pisahkan atau tandai NOT_VERIFIED; jangan gabungkan |
| Review tidak memiliki locator yang stabil | Pertahankan sebagai keterbatasan; jangan membuat Evidence ID |
| Audit FAIL | Jangan lakukan full export; perbaiki atau pisahkan konflik |
| Audit tidak disertakan | Exporter mencatat AUDIT_NOT_PROVIDED; jangan menyebut dataset telah diaudit |
| Evidence Mapping tidak ada | Traceability SidePanel PARTIAL; perbaiki paket sebelum sintesis final |
| AI Mode tidak menemukan sumber | Catat NO_RELEVANT_FINDINGS atau status akses yang sesuai; jangan membuat sumber |
| Source/Evidence ID tidak cocok | Catat conflict dan lakukan human review |
| Root cause belum terbukti | Pertahankan sebagai HYPOTHESIS atau USER_REPORTED sesuai sumber |
| Solusi DIY tidak jelas keselamatannya | NEEDS_REVIEW / NOT_ASSESSED; jangan menyarankan sebagai tindakan aman |
| JSON tidak valid | Perbaiki dengan prompt ulang; jangan mengubah isi bukti secara manual tanpa catatan |
| Identitas kanonis konflik | BLOCKED_IDENTITY atau NEEDS_HUMAN_REVIEW; jangan lanjut publikasi |

## 10. Checklist akhir sebelum data dipakai

### Identitas dan listing
- [ ] Satu workflow hanya untuk satu product_id.
- [ ] URL listing dan Manifest cocok.
- [ ] Canonical product identity diverifikasi terpisah dari listing identity.
- [ ] Varian tidak tercampur.

### Data dan provenance
- [ ] Review CSV menggunakan header kanonis 23 kolom.
- [ ] original_text dipertahankan verbatim.
- [ ] Metadata ekstraksi dan audit disimpan.
- [ ] Source ID dan Evidence ID tidak dibuat-buat.
- [ ] Evidence Mapping SidePanel lengkap untuk record yang diekspor.
- [ ] Setiap Evidence menunjuk ke Source yang benar.

### Sintesis dan keselamatan
- [ ] AI Mode dossier memiliki handoff data.
- [ ] NotebookLM output JSON schema 1.1 valid.
- [ ] Conflicts, limitations, dan open questions tidak hilang.
- [ ] Solution taxonomy lengkap.
- [ ] Rekomendasi tidak melampaui kekuatan bukti.
- [ ] Affiliate disclosure tidak memengaruhi kesimpulan.
- [ ] Publication gate tetap dilakukan oleh manusia/backend.

## 11. Versi dan perubahan kontrak

Jika terjadi perubahan pada:

- field atau key JSON;
- enum atau makna enum;
- pola ID;
- aturan mapping;
- header CSV;
- aturan identity resolution;

maka periksa Canonical Data Contract terlebih dahulu. Perubahan struktur atau makna kontrak memerlukan versi baru dan catatan migrasi. Perubahan redaksi instruksi yang tidak mengubah bentuk data dapat menaikkan versi prompt tanpa menaikkan schema.

Jangan memperbarui satu skill secara terpisah jika perubahan tersebut memengaruhi tahap downstream.

## 12. Referensi repository

- [Research folder](https://github.com/fajar-hida/Skills/tree/main/Research)
- [Canonical Data Contract v1.0](Canonical_Data_Contract_v1.0.md)
- [SidePanel Skills](SidePanel/README.md)
- [Master Prompt 4.0](ModeAI/Master_Prompt.md)
- [NotebookLM Synthesis Prompt v1.1](NotebookLM/NotebookLM_Synthesis_Prompt_v1.1.md)
- [Web App Blueprint v5.0](dokumentasi_rencana_web_app.md)
