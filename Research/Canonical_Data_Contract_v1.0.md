# CANONICAL DATA CONTRACT v1.0
## Shopee Consumer Intelligence Research Pipeline

**Contract ID:** `CDC-SHOPEE-001`  
**Contract Version:** `1.0`  
**Status:** Normative — source of truth for cross-document data exchange  
**Applies to:** SidePanel Skills 01–04, Google AI Mode Master Prompt 4.0, NotebookLM Synthesis, Web App backend and database.

---

## 1. Tujuan dan prioritas

Dokumen ini menetapkan istilah, identitas, provenance, Evidence ID, struktur pertukaran, enum, dan aturan kompatibilitas lintas pipeline.

Urutan otoritas:
1. Dokumen ini mengatur kontrak lintas sistem.
2. Prompt/skill mengatur cara kerja masing-masing tahap dan tidak boleh mendefinisikan kontrak yang bertentangan.
3. Product Manifest dan raw data adalah data sumber, bukan spesifikasi kontrak.
4. Blueprint Web App menjelaskan implementasi dan harus mengikuti kontrak ini.

Jika ditemukan konflik, jangan menebak atau melakukan normalisasi diam-diam. Pertahankan nilai sumber, catat conflict, dan tandai kebutuhan human review.

## 2. Prinsip identitas

### 2.1 Identifier yang berbeda fungsi

| Field | Makna | Aturan |
|---|---|---|
| `product_id` | ID satu listing Shopee | Primary listing key; satu ID hanya untuk satu listing |
| `research_key` | Penghubung proses riset | Bukan identitas produk |
| `canonical_product_id` | ID produk kanonis lintas listing | Null sampai identitas produk/model benar-benar diverifikasi |
| `product_url` | URL listing | Pertahankan URL sumber dan jangan samakan listing berbeda |
| `variant_id` | ID varian jika sumber menyediakannya | Jangan dibuat dari tebakan nama varian |
| `research_run_id` | Satu pelaksanaan riset | Stabil dalam satu research run |

`UNKNOWN` berarti nilai atribut tidak diketahui. `NOT_VERIFIED` berarti status verifikasi belum ditetapkan atau belum berhasil dibuktikan. Keduanya tidak dapat saling menggantikan.

### 2.2 Identity matching enum

Gunakan:
- `EXACT_LISTING_MATCH`
- `EXACT_PRODUCT_MATCH`
- `PROBABLE_PRODUCT_MATCH`
- `VARIANT_MATCH`
- `PRODUCT_FAMILY_MATCH`
- `SIMILAR_PRODUCT`
- `MISMATCH`
- `UNKNOWN`

Sumber yang hanya cocok pada keluarga/kemiripan produk tidak boleh diperlakukan sebagai bukti produk target.

### 2.3 Canonical identity status

- `VERIFIED`
- `PARTIALLY_VERIFIED`
- `NOT_VERIFIED`
- `CONFLICTING`
- `NOT_APPLICABLE`

Hanya status `VERIFIED` yang mengizinkan pengisian `canonical_product_id`. Status lain wajib menggunakan null untuk ID kanonis.

### 2.4 Listing-to-Manifest identity status

Untuk membandingkan input listing/review dengan Product Manifest gunakan:
- `MATCH`
- `PARTIAL_MATCH`
- `CONFLICT`
- `NOT_VERIFIED`
- `NOT_APPLICABLE`

Status ini bukan status canonical identity. Dalam JSON NotebookLM gunakan nama field `listing_identity_status`; jangan memakai field generik `identity_status` untuk dua makna berbeda.

### 2.5 Provenance status

- `COMPLETE`
- `PARTIAL`
- `CONFLICT`
- `NOT_PROVIDED`
- `NOT_ASSESSABLE`

### 2.6 Evidence mapping status

- `AVAILABLE`
- `PARTIAL`
- `UNAVAILABLE`

`PARTIAL` berarti bukti dapat ditelusuri ke record tetapi satu atau lebih locator/content fields tidak lengkap.

## 3. Provenance dan ID namespace

### 3.1 Source ID

ID sumber mempertahankan namespace asal:
- `SRC-SP-###` — sumber/capture SidePanel.
- `SRC-AI-###` — sumber eksternal yang ditemukan melalui AI Mode.
- `SRC-MAN-###` — dokumen produsen/manual/datasheet jika dialokasikan oleh workflow.
- `SRC-MI-###` — input manual pengguna jika perlu diregistrasikan.

Jangan mengganti ID yang sudah ada. Jangan mengklaim bahwa namespace menentukan reliabilitas sumber.

### 3.2 Evidence ID

ID bukti juga mempertahankan namespace asal:
- `EV-SP-###` — bukti turunan dari record review SidePanel atau metadata listing.
- `EV-AI-###` — bukti spesifik dari sumber riset AI Mode.
- `EV-MAN-###` — bukti spesifik dari dokumen produsen.
- `EV-MI-###` — bukti dari input manual pengguna.

Satu Source dapat memiliki banyak Evidence. Satu Evidence harus menunjuk tepat ke satu Source. Evidence bukan ringkasan bebas; ia harus memiliki locator yang dapat ditelusuri.

### 3.3 Deterministic SidePanel evidence mapping

Skill 04 wajib membuat `evidence_mapping` di Export Package Summary dan NotebookLM Raw Text tanpa menambah kolom CSV.

Untuk setiap review row yang memenuhi syarat, urutkan deterministik berdasarkan:
1. `collection_id` secara leksikografis;
2. `source_page` numerik jika tersedia (nilai tidak diketahui setelah nilai numerik);
3. `source_position` numerik jika tersedia (nilai tidak diketahui setelah nilai numerik);
4. `review_id` secara leksikografis sebagai tie-breaker.

Alokasikan `EV-SP-000001`, `EV-SP-000002`, dan seterusnya dalam satu export package. Simpan mapping persis; jangan mengalokasikan ulang ID saat package yang sama diekspor ulang. Bila package baru menggabungkan data lama, pertahankan ID lama dan lanjutkan nomor dari nomor tertinggi yang sudah tercatat. Jika ID lama tidak tersedia, buat mapping baru dan catat `mapping_version` serta bahwa ID tersebut baru dialokasikan.

Contoh:
```yaml
evidence_mapping_version: "1.0"
evidence_mapping:
  - evidence_id: "EV-SP-000001"
    source_id: "SRC-SP-001"
    evidence_origin: "SIDEPANEL_REVIEW"
    product_id: "P001"
    collection_id: "P001-R1"
    review_id: "P001-R1-P02-REV04"
    locator:
      source_page: 2
      source_position: 4
    evidence_status: "AVAILABLE"
```

Mapping wajib menyertakan semua record yang diekspor, termasuk record parsial, tetapi gunakan `evidence_status: "PARTIAL"` bila teks/locator tidak lengkap. Record yang tidak memiliki review_id stabil tidak boleh diberi Evidence ID; catat pada mapping exceptions. Evidence ID menunjuk ke record mentah, bukan menyatakan review tersebut benar atau autentik.

### 3.4 AI Mode evidence mapping

Master Prompt membuat source/evidence records dengan ID deterministik dalam satu research run:
- `SRC-AI-001...` per sumber unik.
- `EV-AI-001...` per klaim/bukti atomik yang memiliki locator.
ID yang sudah ditetapkan tidak boleh berubah pada revisi dossier yang sama. Satu bukti harus mencatat source_id, locator, exact/short excerpt atau paraphrase terlabel, product/variant match, verification status, dan limitations.

### 3.5 Synthesis-generated IDs

NotebookLM boleh membuat ID hanya untuk entitas sintesis baru, bukan untuk mengganti Source ID/Evidence ID input:
- `FND-<research_run_id>-001`
- `SOL-<research_run_id>-001`
- `CNF-<research_run_id>-001`
- `QST-<research_run_id>-001`

Jika `research_run_id` tidak tersedia, gunakan `UNASSIGNED` dalam ID dan set publication status ke `NEEDS_HUMAN_REVIEW`. ID sintesis harus stabil pada revisi output untuk input dan urutan yang sama. Jangan mengarang Source ID/Evidence ID untuk menutup referensi yang hilang.

## 4. Kontrak CSV review

Header CSV Skill 02–04 tetap persis seperti berikut dan tidak boleh diubah tanpa versi kontrak baru dan migrasi:
```csv
review_id,collection_id,product_id,product_name,product_url,rating_filter,sort_order,source_page,source_position,rating,variant,review_date,usage_duration,original_text,sentiment,tags,review_type,has_image,has_video,purchase_badge,seller_response,extraction_status,duplicate_status
```

Evidence mapping disimpan sebagai metadata terpisah. `original_text` harus dipertahankan verbatim. CSV adalah data review mentah/terstruktur, bukan hasil sintesis atau penilaian kualitas produk.

## 5. Canonical enum registry

### 5.1 General value and access status
- Unknown scalar: `UNKNOWN` atau null sesuai tipe/schema; jangan gunakan placeholder bebas.
- Access status: `FULL_ACCESS`, `PARTIAL_ACCESS`, `SEARCH_SNIPPET_ONLY`, `NOT_ACCESSIBLE`, `NOT_SEARCHED`, `NO_RELEVANT_FINDINGS`.

### 5.2 Source type
- `SIDE_PANEL_MANIFEST`
- `SIDE_PANEL_REVIEW`
- `SIDE_PANEL_AUDIT`
- `AI_MODE_RESEARCH`
- `MANUFACTURER_DOCUMENT`
- `SELLER_LISTING`
- `INDEPENDENT_REVIEW`
- `COMMUNITY_REPORT`
- `REGULATORY_SOURCE`
- `MANUAL_INPUT`
- `OTHER`

### 5.3 Verification status
- `MANUFACTURER_CONFIRMED`
- `INDEPENDENTLY_TESTED`
- `MULTI_SOURCE_CORROBORATED`
- `USER_REPORTED`
- `SELLER_CLAIM`
- `MANUFACTURER_CLAIM`
- `HYPOTHESIS`
- `CONFLICTING`
- `UNVERIFIED`
- `NOT_AVAILABLE`

### 5.4 Confidence and evidence quality
Confidence: `HIGH`, `MEDIUM`, `LOW`, `UNVERIFIED`. Confidence is qualitative, not a probability.
Source reliability: `HIGH`, `MEDIUM`, `LOW`, `UNKNOWN`.
Evidence relevance: `HIGH`, `MEDIUM`, `LOW`, `NOT_RELEVANT`.

### 5.5 Dataset audit status
- `PASS`
- `PASS_WITH_WARNINGS`
- `NEEDS_REVIEW`
- `FAIL`
- `AUDIT_NOT_PROVIDED`

`AUDIT_NOT_PROVIDED` is a downstream missing-input state, not a successful audit result.

### 5.6 Audit finding severity
- `CRITICAL`
- `HIGH`
- `MEDIUM`
- `LOW`
- `INFO`

### 5.7 Export readiness
- `READY_FOR_EXPORT`
- `READY_WITH_WARNINGS`
- `NOT_READY`
- `PARTIAL_EXPORT`

Skill 03 handoff remains limited to `READY_FOR_EXPORT`, `READY_WITH_WARNINGS`, `NOT_READY`. Skill 04 may assign `PARTIAL_EXPORT` when only a clearly bounded subset can be exported. Do not relabel a Skill 03 handoff value as export readiness without this explicit mapping.

### 5.8 Coverage status
- `TARGET_REACHED`
- `IN_PROGRESS`
- `NOT_SPECIFIED`
- `PRIOR_DATA_NOT_AVAILABLE`
- `POTENTIAL_COVERAGE_GAP`
- `NOT_ASSESSABLE`

Coverage of a sample is not proof of a complete review census.

### 5.9 Extraction and duplicate status
Extraction: `EXTRACTED`, `PARTIAL`, `UNREADABLE`.
Duplicate: `UNASSESSED`, `UNIQUE`, `POSSIBLE_DUPLICATE`, `EXACT_DUPLICATE`.

### 5.10 Solution dimensions

Do not combine solution authority, action type, verification, and safety into one enum.

**solution_type**
- `MAINTENANCE`
- `REPAIR`
- `TROUBLESHOOTING`
- `WORKAROUND`
- `PREVENTION`

**implementation_method**
- `DIY`
- `PROFESSIONAL`
- `USER_ACTION`
- `NOT_APPLICABLE`

`DIY_FIX` pada prompt lama bukan lagi nilai solution_type. Petakan menjadi solution_type sesuai tindakan aktual, lalu set implementation_method=`DIY`. `BASIC_TROUBLESHOOTING` dipetakan ke solution_type=`TROUBLESHOOTING`; `TEMPORARY_WORKAROUND` ke `WORKAROUND`; `MAINTENANCE_PROCEDURE` ke `MAINTENANCE`; `REPAIR_PROCEDURE` ke `REPAIR`.

**solution_source**
- `MANUFACTURER`
- `USER`
- `COMMUNITY`
- `PROFESSIONAL`
- `INDEPENDENT_TEST`
- `UNKNOWN`

**solution_verification_status** uses the Verification Status enum in §5.3.

**safety_status**
- `REVIEWED_SAFE`
- `NEEDS_REVIEW`
- `UNSAFE`
- `NOT_ASSESSED`

A user workaround is never labeled as official manufacturer guidance unless the manufacturer independently confirms it.

### 5.11 Recommendation
- `SUITABLE`
- `SUITABLE_WITH_CONDITIONS`
- `NOT_RECOMMENDED_FOR_STATED_NEED`
- `INSUFFICIENT_EVIDENCE`
- `NEEDS_HUMAN_REVIEW`

### 5.12 Publication status
- `DRAFT`
- `VALIDATION_FAILED`
- `NEEDS_HUMAN_REVIEW`
- `APPROVED`
- `PUBLISHED`
- `ARCHIVED`
- `BLOCKED_IDENTITY`

NotebookLM may emit only `DRAFT`, `VALIDATION_FAILED`, `NEEDS_HUMAN_REVIEW`, or `BLOCKED_IDENTITY`. Only the authorized Web App editorial workflow may set `APPROVED`, `PUBLISHED`, or `ARCHIVED`.

### 5.13 Conflict status
- `OPEN`
- `RESOLVED`
- `NEEDS_HUMAN_REVIEW`

## 6. AI Mode → NotebookLM mapping

Master Prompt 4.0 may produce a Markdown research dossier. NotebookLM must treat it as a research source and normalize it into the canonical synthesis JSON. The Markdown dossier is not itself the canonical JSON schema.

| Master Prompt 4.0 concept/section | Canonical NotebookLM destination | Mapping rule |
|---|---|---|
| research_id / research_date / method | research_metadata.research_run_id / researched_at / research_method | Preserve exact values; missing values null |
| Product identity and listing reconciliation | product_identity + listing_snapshot | Keep listing identity separate from canonical identity |
| Manufacturer and seller claims | evidence + findings | Preserve claim attribution and verification status |
| Technical profile | findings.product_utility / strengths / risks_and_limitations | Split atomic statements; cite evidence |
| Reality checks / independent tests | findings + evidence | Preserve test conditions and limitations |
| User experiences | findings + evidence | Mark USER_REPORTED; do not infer population prevalence |
| Failure modes | findings.risks_and_limitations + root_cause_assessments | Root cause remains hypothesis unless proven |
| Problem-solution intelligence | root_cause_assessments + solutions | Use multidimensional solution fields |
| Alternatives / competitors | findings.buyer_fit and open_questions, plus product references if schema supports | Never merge alternative identity into target product |
| Evidence ledger | sources + evidence | One source record per source; one evidence record per atomic claim/locator |
| Recommendation | recommendation | Use canonical recommendation enum; retain rationale/evidence |
| Research limitations / unanswered questions | open_questions + conflicts | Do not discard unresolved items |

Field mapping for source records:
- Master Prompt `source_name` → canonical `title`
- `original_url` → `url`
- `publication_date` → `published_at`
- access label → `source_status` using §5.1
- product/variant match → `product_id_scope` and `variant_scope` plus identity_match
- source_type → canonical Source Type; map `MANUFACTURER_DOCUMENTATION` to `MANUFACTURER_DOCUMENT`.

Field mapping for evidence:
- `evidence_id`, `source_id` preserved exactly.
- `finding_statement` / `claim` → evidence statement or linked finding statement, not source title.
- `supporting_evidence` → finding.evidence_ids.
- `contradicting_evidence` → finding.counter_evidence_ids.
- Locator, excerpt, context, verification, confidence, limitations must be preserved.

If the AI Mode dossier contains no explicit source/evidence IDs, NotebookLM must not fabricate them. Set the affected record references empty/null, add an open question, and set `publication_status: NEEDS_HUMAN_REVIEW` when material findings cannot be traced.

## 7. NotebookLM canonical JSON contract

The current NotebookLM prompt defines the JSON object shape. Its top-level required keys are:
`schema_version`, `synthesis_prompt_version`, `research_metadata`, `product_identity`, `listing_snapshot`, `dataset_audit`, `sources`, `evidence`, `findings`, `solutions`, `recommendation`, `conflicts`, `open_questions`, `affiliate`.

Rules:
- `schema_version` is `1.1` for the NotebookLM output defined by Prompt v1.1. Canonical Data Contract document version remains `1.0`.
- Source and evidence references must resolve.
- Every finding and solution must cite at least one valid evidence item unless explicitly marked as an unresolved question or insufficient evidence.
- Synthesis-generated IDs follow §3.5.
- Null is used for unknown scalar values; empty arrays for known-empty collections; do not use empty string as a substitute for null.
- Preserve original records separately; synthesis never overwrites raw input.

## 8. Migration and compatibility

1. Keep the canonical review CSV header unchanged.
2. Add Evidence ID mapping only in export metadata.
3. Existing packages without `evidence_mapping` are `PARTIAL_EXPORT` or require mapping regeneration from stable review IDs and source metadata; do not claim full evidence traceability.
4. Preserve existing IDs and append newly allocated IDs; never renumber previously exported evidence.
5. A change to key names, required fields, or enum meaning requires a new contract/schema version and migration notes.
6. A prompt wording change that does not change JSON shape increments prompt version, not schema version.
7. Validate identity, source/evidence referential integrity, enum membership, and safety before import.
8. Dataset audit, export readiness, product quality, and publication status are separate state dimensions.

## 9. Release checklist

- [ ] All downstream documents reference this contract.
- [ ] Master Prompt emits stable AI source/evidence identifiers and uses compatible source types.
- [ ] NotebookLM maps the Master Prompt dossier using §6 and does not invent upstream IDs.
- [ ] Skill 04 emits deterministic SidePanel evidence mapping without modifying CSV.
- [ ] Skill 03 and Skill 04 distinguish handoff status from export readiness.
- [ ] Solution fields use the multidimensional taxonomy in §5.10.
- [ ] Blueprint validator checks enum membership and referential integrity.
- [ ] Existing raw data and previously allocated IDs remain unchanged.
