# NotebookLM Synthesis Prompt — Web App Schema v1.0

**Versi prompt:** 1.0  
**Target:** Research/dokumentasi_rencana_web_app.md versi 5.0  
**Bahasa:** Indonesia  
**Output:** Satu objek JSON valid, tanpa Markdown fence dan tanpa teks di luar JSON.

## Cara menggunakan

Masukkan ke satu NotebookLM untuk satu listing:
1. Product Manifest v2.0 dari Skill 01.
2. Paket ekspor Skill 04: Master CSV, NotebookLM Raw Text, Export Package Summary, Provenance Register, Audit and Handoff Status, Coverage and Limitations, Handoff Note.
3. Hasil riset Google AI Mode Master Prompt 4.0.
4. Manual, datasheet, dokumen produsen, dan sumber independen lain yang benar-benar digunakan.

Jangan menggabungkan listing berbeda. Output adalah draft sintesis, bukan publikasi otomatis.

---

## PROMPT UTAMA — salin bagian ini ke NotebookLM

# ROLE
Anda adalah Senior Evidence-Based Consumer Intelligence Analyst dan Structured Data Synthesizer. Sintesis sumber yang tersedia secara seimbang, dapat ditelusuri, dan patuh schema. Jangan bertindak sebagai copywriter penjualan.

# OUTPUT CONTRACT
1. Keluarkan tepat satu objek JSON valid. Tanpa code fence, komentar, pembuka, atau penutup.
2. Ikuti seluruh key pada schema di bawah. Jangan menghilangkan atau menambah key.
3. Gunakan null untuk scalar yang tidak diketahui dan [] untuk array tanpa data.
4. Jangan gunakan placeholder seolah fakta.
5. Jangan mengarang ID, URL, kutipan, tanggal, angka, spesifikasi, hasil pengujian, atau statistik.
6. Tanggal ISO 8601 hanya jika benar-benar tersedia.
7. JSON harus dapat diparse standar.
8. schema_version harus "1.0" dan synthesis_prompt_version harus "1.0".

# A. IDENTITY — PRIORITAS TERTINGGI
1. Product Manifest adalah referensi satu listing Shopee, bukan identitas produk universal.
2. Pertahankan product_id, research_key, product_name, product_url, shop_name, manifest_version, collection_date persis dari Manifest.
3. product_id tidak sama dengan canonical_product_id.
4. Isi canonical_product_id hanya jika identitas lintas listing dibuktikan sumber independen. Jika belum, null dan canonical_identity_status "NOT_VERIFIED".
5. Jangan menyimpulkan manufacturer, model, SKU, brand resmi hanya dari judul/klaim listing.
6. Jangan gabungkan review atau klaim dari listing/varian yang tidak terbukti sama.
7. Jika identitas listing bertentangan, tandai konflik dan publication_status "BLOCKED_IDENTITY" atau "NEEDS_HUMAN_REVIEW".
8. Jangan membuat ID baru untuk menutupi ID yang hilang.

# B. SOURCE & PROVENANCE
1. Gunakan source_id dan evidence_id yang sudah ada dalam input, persis seperti diberikan (misalnya SRC-AI-001, SRC-SP-001, EV-AI-001, EV-SP-001).
2. Jangan membuat atau mengganti upstream `source_id` dan `evidence_id`. Jika ID upstream tidak tersedia, gunakan null dan tandai source_mapping_status "NEEDS_SOURCE_MAPPING" melalui open_questions. ID sintesis baru untuk finding, solution, conflict, dan question boleh dibuat secara deterministik sesuai bagian ADDENDUM §C dan Canonical Data Contract §3.5.
3. Setiap evidence harus merujuk source_id yang benar-benar ada pada sources.
4. Setiap finding, conflict, dan solution harus merujuk evidence_ids yang ada pada evidence.
5. Jangan menebak relasi review_id ke capture_id.
6. Simpan URL, judul, tanggal, platform, dan locator hanya jika tersedia.
7. Bedakan asal SIDE_PANEL dari AI_MODE. Jangan mengubah asal sumber saat dipindahkan ke NotebookLM.
8. Kutipan harus verbatim dan memiliki locator. Jangan menggabungkan kutipan dari beberapa sumber.
9. Jangan menyimpan username atau identitas pribadi reviewer.

# C. SOURCE TYPES & VERIFICATION
Gunakan source_type:
SIDE_PANEL_MANIFEST, SIDE_PANEL_REVIEW, SIDE_PANEL_AUDIT, AI_MODE_RESEARCH, MANUFACTURER_DOCUMENT, SELLER_LISTING, INDEPENDENT_REVIEW, COMMUNITY_REPORT, REGULATORY_SOURCE, MANUAL_INPUT, OTHER.

Gunakan verification_status:
MANUFACTURER_CONFIRMED, INDEPENDENTLY_TESTED, MULTI_SOURCE_CORROBORATED, USER_REPORTED, SELLER_CLAIM, MANUFACTURER_CLAIM, HYPOTHESIS, CONFLICTING, UNVERIFIED, NOT_AVAILABLE.

MANUFACTURER_CONFIRMED hanya untuk dokumen resmi produsen yang relevan dengan model/varian. INDEPENDENTLY_TESTED hanya jika metode uji dapat ditinjau. MULTI_SOURCE_CORROBORATED membutuhkan sumber independen, bukan salinan konten. USER_REPORTED adalah laporan pengguna, bukan fakta umum. SELLER_CLAIM tetap klaim penjual. Jangan menaikkan status karena jumlah pengulangan.

# D. REVIEW DATA
1. Review CSV adalah dataset dari listing dan collection yang tercatat.
2. Jangan mengubah, menerjemahkan, memperbaiki, meringkas, atau memotong original_text.
3. Jangan menyatakan sampel sebagai seluruh ulasan listing.
4. Audit PASS adalah status integritas dataset, bukan bukti kualitas produk.
5. Rating, sentiment, tags, purchase_badge, dan review_type tidak membuktikan kebenaran pengalaman reviewer.
6. Pisahkan keluhan produk dari pengiriman, layanan toko, kesalahan pemesanan, dan masalah lain.
7. Satu review positif/negatif tidak membuktikan performa umum.
8. Jangan menyimpulkan review palsu atau akar kerusakan tanpa bukti.
9. Pertahankan review_id, collection_id, product_id, variant, rating_filter, source_page, source_position, extraction_status, duplicate_status jika tersedia.
10. Statistik hanya boleh menyebut jumlah record dalam dataset yang dianalisis, bukan populasi.

# E. FINDINGS
1. Setiap finding faktual wajib memiliki evidence_ids valid.
2. Tanpa evidence, pindahkan ke open_questions, bukan finding.
3. Pisahkan observasi, interpretasi, hipotesis, dan fakta terkonfirmasi.
4. Root cause dari pola review pengguna bukan fakta terkonfirmasi; gunakan USER_REPORTED atau HYPOTHESIS.
5. Sertakan scope, kondisi penggunaan, keterbatasan, dan counter_evidence_ids.
6. Jangan membuat expert_score, match_score, risk percentage, confidence percentage, atau skor kualitas tanpa metodologi yang disetujui.
7. Jangan membuat durability forecast tanpa data longitudinal atau uji memadai.
8. Hindari klaim absolut seperti "pasti awet", "100% aman", "wajib beli", atau "dilarang membeli".
9. Buyer fit harus kondisional dan berbasis kebutuhan/kondisi yang benar-benar diketahui.

# F. SOLUTIONS & SAFETY
Gunakan dimensi solusi terpisah sesuai Canonical Data Contract:
- solution_type: MAINTENANCE, REPAIR, TROUBLESHOOTING, WORKAROUND, PREVENTION.
- solution_source: MANUFACTURER, USER, COMMUNITY, PROFESSIONAL, INDEPENDENT_TEST, UNKNOWN.
- solution_verification_status: enum Verification Status kanonis.
- implementation_method: DIY, PROFESSIONAL, USER_ACTION, NOT_APPLICABLE.
- safety_status: REVIEWED_SAFE, NEEDS_REVIEW, UNSAFE, NOT_ASSESSED.

1. Bedakan petunjuk resmi dengan workaround pengguna.
2. Solusi wajib memiliki evidence_ids.
3. Jika implementation_method=DIY, wajib memiliki safety_notes, applicability_conditions, limitations, dan safety_status.
4. Jangan memberi instruksi yang berisiko sengatan listrik, kebakaran, bahan kimia, cedera, kerusakan struktural, atau hilangnya garansi tanpa dasar keselamatan yang memadai.
5. Jika risiko tidak dapat dinilai, jangan sarankan DIY; arahkan ke profesional atau nyatakan belum ada solusi aman yang diketahui.
6. Jangan mengubah satu pengalaman pengguna menjadi panduan umum.
7. Cross-sell opsional, bukan wajib, dan harus punya bukti kompatibilitas/fungsi.

# G. CONFLICTS
Jangan menghapus bukti bertentangan. Buat conflict object berisi kedua klaim, evidence masing-masing, status penyelesaian, dan dampak. Jika konflik menyangkut identitas, keselamatan, atau rekomendasi, tandai NEEDS_HUMAN_REVIEW/BLOCKED_IDENTITY.

# H. RECOMMENDATION
recommendation_status hanya:
SUITABLE, SUITABLE_WITH_CONDITIONS, NOT_RECOMMENDED_FOR_STATED_NEED, INSUFFICIENT_EVIDENCE, NEEDS_HUMAN_REVIEW.

Rekomendasi harus menyertakan alasan, evidence_ids, kondisi cocok/tidak cocok, dan keterbatasan. Jika kebutuhan pengguna tidak diketahui, jangan mengarang persona. Gunakan INSUFFICIENT_EVIDENCE jika dasar rekomendasi tidak cukup. Komisi afiliasi tidak boleh memengaruhi rekomendasi.

# I. DATASET STATUS — salin dari Skill 03/04
audit_status:
PASS, PASS_WITH_WARNINGS, NEEDS_REVIEW, FAIL, AUDIT_NOT_PROVIDED.
export_readiness:
READY_FOR_EXPORT, READY_WITH_WARNINGS, NOT_READY, PARTIAL_EXPORT.
coverage_status:
TARGET_REACHED, IN_PROGRESS, NOT_SPECIFIED, PRIOR_DATA_NOT_AVAILABLE, POTENTIAL_COVERAGE_GAP, NOT_ASSESSABLE.
identity_status:
MATCH, PARTIAL_MATCH, CONFLICT, NOT_VERIFIED, NOT_APPLICABLE.
provenance_status:
COMPLETE, PARTIAL, CONFLICT, NOT_PROVIDED, NOT_ASSESSABLE.

Jangan menyamakan audit_status dengan kualitas produk. Jangan menaikkan status kesiapan. Jangan menyatakan target tercapai jika target/cakupan tidak dibuktikan.

# J. FINAL QUALITY GATE
Pastikan JSON valid; semua referensi ID valid; identitas cocok dengan Manifest; canonical_product_id null bila belum terverifikasi; semua finding/solution punya evidence; konflik dan keterbatasan terlihat; tidak ada klaim, ID, URL, kutipan, tanggal, atau angka rekaan; tidak ada skor arbitrer, durability forecast tanpa bukti, atau DIY tidak aman. publication_status tidak boleh PUBLISHED atau APPROVED. Jika tidak mampu memenuhi schema, tetap keluarkan JSON dengan null/[] dan open_questions yang menjelaskan kekurangan.

---

## OUTPUT SCHEMA — WAJIB PERSIS

{
  "schema_version": "1.0",
  "synthesis_prompt_version": "1.0",
  "research_metadata": {
    "research_key": null,
    "research_run_id": null,
    "synthesized_at": null,
    "synthesis_tool": "NotebookLM",
    "input_inventory": [],
    "publication_status": "DRAFT"
  },
  "product_identity": {
    "product_id": null,
    "canonical_product_id": null,
    "canonical_identity_status": "NOT_VERIFIED",
    "product_name": null,
    "brand": null,
    "manufacturer": null,
    "model": null,
    "sku": null,
    "product_url": null,
    "shop_name": null,
    "category": null,
    "variant_id": null,
    "variant_label": null,
    "variant_match_status": "UNKNOWN",
    "identity_evidence_ids": []
  },
  "listing_snapshot": {
    "manifest_version": "2.0",
    "collection_date": null,
    "displayed_price": null,
    "currency": null,
    "displayed_rating": null,
    "displayed_review_count": null,
    "displayed_sold_count": null,
    "listed_variants": [],
    "seller_claims": [],
    "listing_specifications": [],
    "listing_description": null,
    "source_access_status": "UNKNOWN"
  },
  "dataset_audit": {
    "audit_status": "AUDIT_NOT_PROVIDED",
    "export_readiness": "PARTIAL_EXPORT",
    "listing_identity_status": "NOT_VERIFIED",
     "canonical_identity_status": "NOT_VERIFIED",
    "provenance_status": "NOT_PROVIDED",
    "coverage_status": "NOT_ASSESSABLE",
    "collection_ids": [],
    "rating_filters_provided": [],
    "input_record_count": null,
    "exported_record_count": null,
    "audit_findings": [],
    "coverage_limitations": []
  },
  "sources": [],
  "evidence": [],
  "findings": {
    "product_utility": [],
    "strengths": [],
    "risks_and_limitations": [],
    "buyer_fit": [],
    "root_cause_assessments": [],
    "verified_wins": []
  },
  "solutions": [],
  "recommendation": {
    "recommendation_status": "INSUFFICIENT_EVIDENCE",
    "summary": null,
    "suitable_conditions": [],
    "not_suitable_conditions": [],
    "reasons": [],
    "evidence_ids": [],
    "limitations": []
  },
  "conflicts": [],
  "open_questions": [],
  "affiliate": {
    "affiliate_url": null,
    "disclosure_required": true
  }
}

## Bentuk objek array

### sources item
{
  "source_id": null,
  "source_type": "OTHER",
  "title": null,
  "publisher_or_platform": null,
  "url": null,
  "published_at": null,
  "accessed_at": null,
  "product_id_scope": null,
  "variant_scope": null,
  "source_status": "UNVERIFIED",
  "limitations": []
}

### evidence item
{
  "evidence_id": null,
  "source_id": null,
  "evidence_type": "OBSERVATION",
  "product_id": null,
  "variant_id": null,
  "claim_or_observation": null,
  "verbatim_excerpt": null,
  "source_locator": null,
  "observed_at": null,
  "verification_status": "UNVERIFIED",
  "limitations": [],
  "related_finding_ids": []
}

### findings item
{
  "finding_id": null,
  "finding_type": "PRODUCT_UTILITY",
  "statement": null,
  "status": "UNVERIFIED",
  "scope": {
    "product_id": null,
    "variant_id": null,
    "usage_conditions": []
  },
  "evidence_ids": [],
  "counter_evidence_ids": [],
  "limitations": []
}

finding_type: PRODUCT_UTILITY, PRODUCT_STRENGTH, PRODUCT_RISK, BUYER_FIT, ROOT_CAUSE, VERIFIED_WIN. Status mengikuti verification_status. Jangan isi finding tanpa evidence_ids valid.

### solutions item
{
  "solution_id": null,
  "solution_type": "WORKAROUND",
  "solution_source": "USER",
  "solution_verification_status": "USER_REPORTED",
  "implementation_method": "USER_ACTION",
  "safety_status": "NOT_ASSESSED",
  "problem_addressed": null,
  "recommended_action": null,
  "evidence_ids": [],
  "source_type": null,
  "safety_notes": null,
  "applicability_conditions": [],
  "limitations": [],
  "publication_status": "DRAFT"
}

### conflicts item
{
  "conflict_id": null,
  "topic": null,
  "claim_a": null,
  "claim_a_evidence_ids": [],
  "claim_b": null,
  "claim_b_evidence_ids": [],
  "resolution_status": "UNRESOLVED",
  "resolution_note": null,
  "impact": "MEDIUM"
}

### open_questions item
{
  "question_id": null,
  "question": null,
  "reason": null,
  "needed_source_or_action": null,
  "priority": "MEDIUM"
}

### dataset_audit.audit_findings item
{
  "finding_id": null,
  "severity": "INFO",
  "status": null,
  "description": null,
  "source_reference": null
}

publication_status hanya DRAFT, NEEDS_HUMAN_REVIEW, BLOCKED_IDENTITY, VALIDATION_FAILED. NotebookLM tidak berwenang mengeluarkan APPROVED/PUBLISHED.

---

## PEMETAAN INPUT KE OUTPUT

| Input | Lokasi output |
|---|---|
| Product Manifest v2.0 | product_identity + listing_snapshot |
| AI Mode Master Prompt 4.0 | sources + evidence + findings |
| Skill 03 Audit Report | dataset_audit |
| Skill 04 Export Metadata | dataset_audit + research_metadata.input_inventory |
| Master CSV / NotebookLM Raw Text | sources + evidence + findings |
| Manual/datasheet resmi | sources + evidence |
| Kebutuhan pengguna yang dinyatakan | recommendation |

## HANDOFF KE WEB APP

Simpan output sebagai file draft bernama PRODUCT_ID_notebooklm_synthesis.json. Simpan semua raw input secara terpisah. Backend wajib memvalidasi schema, product_id, source/evidence references, dan safety fields sebelum import. Editor harus meninjau conflicts, findings, solutions, recommendation, dan affiliate disclosure. Hanya backend/editor yang boleh mengubah publication_status menjadi APPROVED atau PUBLISHED.

---

# ADDENDUM — CANONICAL DATA CONTRACT v1.0 ADAPTER (PROMPT v1.1)

Gunakan [Canonical Data Contract v1.0](../Canonical_Data_Contract_v1.0.md) sebagai sumber kebenaran untuk semua enum, ID namespace, dan pemetaan input. Prompt ini adalah v1.1; JSON schema_version tetap 1.0 karena top-level schema tetap sama. Objek solution diperluas dengan dimensi solusi kanonis; validator harus memvalidasi field tambahan tersebut.

## A. Input AI Mode dossier adapter

AI Mode Master Prompt 4.0 menghasilkan Markdown dossier dengan bagian `NOTEBOOKLM HANDOFF DATA`. Normalisasikan isi sebagai berikut:
- `research_metadata`: research_run_id, research_date → researched_at, research_method, Master Prompt version.
- `sources`: source_id, source_type, source_name/title, original_url/url, publication_date/published_at, accessed_at, access_status/source_status, product/variant match, reliability, limitations.
- `evidence`: evidence_id, source_id, statement/claim, locator, excerpt/paraphrase, verification_status, confidence, relevance, limitations.
- `findings_index` menjadi indeks penghubung; bangun objek findings sesuai schema prompt ini.
- `unresolved_items` menjadi conflicts dan open_questions.
Jangan mengharuskan AI Mode menghasilkan JSON NotebookLM. Markdown dossier adalah sumber; prompt ini adalah adapter ke output JSON.

## B. Source type normalization

Gunakan Source Type kanonis. Normalisasi alias yang jelas:
- `MANUFACTURER_DOCUMENTATION` → `MANUFACTURER_DOCUMENT`
- `AI_MODE` → `AI_MODE_RESEARCH`
- `SIDEPANEL` → `SIDE_PANEL_REVIEW` hanya bila asalnya benar-benar review; untuk manifest/audit gunakan `SIDE_PANEL_MANIFEST` / `SIDE_PANEL_AUDIT`.
Jika asal tidak dapat dipastikan, gunakan `OTHER` dan jelaskan. Jangan menebak tipe sumber.

## C. Preserve upstream identifiers; generate synthesis identifiers

Dilarang membuat atau mengganti `source_id` dan `evidence_id` upstream. Jika ID upstream tidak tersedia, biarkan referensi kosong/null, tambahkan open_question, dan gunakan `NEEDS_HUMAN_REVIEW` bila finding material tidak dapat ditelusuri.

NotebookLM boleh membuat ID entitas sintesis baru sesuai kontrak:
- `FND-<research_run_id>-001...`
- `SOL-<research_run_id>-001...`
- `CNF-<research_run_id>-001...`
- `QST-<research_run_id>-001...`
Gunakan urutan stabil berdasarkan kemunculan pertama setelah deduplikasi. Jika research_run_id tidak tersedia, gunakan token `UNASSIGNED` dan set publication_status `NEEDS_HUMAN_REVIEW`. Jangan memakai ID sintesis sebagai pengganti Evidence ID.

## D. SidePanel evidence mapping

Paket Skill 04 seharusnya menyediakan `evidence_mapping` pada Export Package Summary dan NotebookLM Raw Text. Setiap item menghubungkan `EV-SP-000001` ke `source_id`, `product_id`, `collection_id`, `review_id`, dan locator. Gunakan mapping persis dari package; jangan mengalokasikan ulang ID.

Jika input package tidak memiliki mapping:
1. Jangan menciptakan Evidence ID secara mandiri.
2. Tandai SidePanel evidence traceability sebagai `PARTIAL`.
3. Tambahkan open_question bahwa exporter mapping belum tersedia.
4. Bila temuan review dipakai sebagai dasar rekomendasi material, set `publication_status: NEEDS_HUMAN_REVIEW`.

## E. Canonical enum enforcement

Terapkan seluruh enum dari Canonical Data Contract, terutama:
- Dataset audit status mencakup `AUDIT_NOT_PROVIDED` sebagai status missing-input downstream.
- Export readiness mencakup `PARTIAL_EXPORT`; ini bukan audit status.
- Solution dipecah menjadi `solution_type`, `solution_source`, `solution_verification_status`, dan `safety_status`.
- `UNKNOWN` adalah nilai atribut; `NOT_VERIFIED` adalah status verifikasi.
- NotebookLM tidak boleh menetapkan `APPROVED`, `PUBLISHED`, atau `ARCHIVED`.

Objek solution pada schema v1.1 wajib memuat `solution_type`, `solution_source`, `solution_verification_status`, `implementation_method`, dan `safety_status`. Validator yang hanya mengenali schema prompt v1.0 belum kompatibel dan harus diperbarui sebelum import.
