# Shopee Consumer Research — Gemini in Chrome Skills (Final)

Paket ini berisi empat Custom Skill terpisah untuk memprofilkan listing, mengumpulkan ulasan, mengaudit data, dan mengekspor dataset.

## Prinsip desain final

**Satu produk = satu dataset penelitian.** Untuk pengumpulan sekitar 20–40 ulasan per kategori rating, tidak perlu menyimpan satu file untuk setiap halaman.

Halaman tetap dicatat sebagai metadata (`source_page` dan `source_position`) agar asal record dapat ditelusuri. `collection_id` membedakan kategori rating dalam produk yang sama, misalnya `P001-R1` dan `P001-R5`.

Empat Skill:
1. **Shopee Product Profiler** — membuat Product Manifest untuk satu listing.
2. **Shopee Review Extractor** — mengekstrak record dari halaman aktif dan mengeluarkan record baru saja.
3. **Shopee Review Auditor** — memeriksa konsistensi dan kualitas seluruh record yang tersedia.
4. **Shopee Review Exporter** — menggabungkan seluruh record yang diberikan menjadi satu Master CSV dan satu dokumen teks NotebookLM.

## Urutan penggunaan

1. Buka listing produk di Chrome desktop, lalu jalankan Product Profiler.
2. Simpan Product Manifest atau pertahankan dalam percakapan penelitian yang sama.
3. Buka ulasan, pilih kategori rating dan urutan yang diinginkan.
4. Jalankan Review Extractor pada halaman pertama.
5. Pindah halaman secara manual, tunggu kartu ulasan tampil, lalu jalankan Extractor lagi. Skill mengeluarkan record baru untuk halaman aktif dan memakai collection_id yang sama.
6. Ulangi sampai target sekitar 20–40 record untuk kategori tersebut tercapai.
7. Ganti kategori rating, misalnya dari 1 bintang ke 5 bintang. Jalankan Extractor lagi; kategori baru mendapat collection_id berbeda.
8. Jalankan Review Auditor pada seluruh output Extractor yang masih tersedia/diberikan.
9. Jalankan Review Exporter dengan Product Manifest, seluruh record Extractor, dan laporan Auditor.
10. Simpan satu Master CSV dan satu NotebookLM Raw Text per produk.

## Perintah singkat

- Product Profiler: `Buat Product Manifest untuk listing pada tab ini.`
- Review Extractor: `Mulai koleksi ulasan untuk halaman aktif.` atau `Lanjutkan koleksi yang sama; ekstrak record baru dari halaman aktif saja.`
- Review Auditor: `Audit seluruh record ulasan yang tersedia untuk product_id ini. Jangan hapus atau ubah teks asli.`
- Review Exporter: `Gabungkan seluruh record yang saya berikan dan ekspor satu Master CSV serta NotebookLM Raw Text untuk produk ini.`

## Standar data bersama

Header kanonis:

`review_id,collection_id,product_id,product_name,product_url,rating_filter,sort_order,source_page,source_position,rating,variant,review_date,usage_duration,original_text,sentiment,tags,review_type,has_image,has_video,purchase_badge,seller_response,extraction_status,duplicate_status`

Definisi:
- `collection_id`: satu kategori rating untuk satu produk.
- `source_page`: halaman asal kartu ulasan.
- `source_position`: posisi kartu pada halaman asal.
- `review_id`: ID record yang unik.
- `original_text`: teks ulasan verbatim.

## Hal penting tentang kontinuitas data

Custom Skill menyimpan instruksi, bukan database. Saat Extractor dijalankan pada halaman berikutnya, ia dapat merujuk pada record sebelumnya hanya jika output sebelumnya masih tersedia dalam konteks percakapan atau diberikan kembali oleh pengguna.

Karena targetnya relatif kecil (20–40 ulasan per kategori), pendekatan yang praktis adalah:
- gunakan satu percakapan Gemini untuk satu produk;
- jalankan Extractor pada setiap halaman;
- pertahankan output halaman dalam percakapan;
- setelah satu produk selesai, jalankan Auditor dan Exporter;
- simpan hasil final di luar percakapan.

Jika Gemini tidak lagi memiliki record sebelumnya dalam konteks, jangan mengandalkan total kumulatif yang ditebak. Berikan kembali hasil sebelumnya atau mulai sesi ekspor dengan dataset yang telah disatukan.

## Batas akses dan kualitas

- Ekstraksi hanya mencakup kartu yang benar-benar terlihat dan dapat dibaca Gemini dari tab aktif.
- Pindah halaman, mengganti filter, dan menunggu konten termuat dilakukan manual kecuali fitur otomasi yang sesuai tersedia secara eksplisit.
- Jangan menyebut hasil sebagai seluruh ulasan listing. Sebut sebagai sampel/record yang berhasil dikumpulkan.
- Jangan membuang ulasan generik atau logistik.
- Jangan menyimpulkan review palsu, manipulasi, atau verified purchase tanpa bukti.
- Hindari username reviewer karena tidak dibutuhkan untuk analisis produk.
- Jangan menganggap label `PASS` sebagai bukti kelengkapan seluruh listing.

## Isi paket

- `01-shopee-product-profiler.md`
- `02-shopee-review-extractor.md`
- `03-shopee-review-auditor.md`
- `04-shopee-review-exporter.md`
- `README.md`
