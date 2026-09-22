# pharma

Pharmacology and medical-directory data tooling: OCR of a 617-page Persian drug monograph,
transcription of handwritten pharmacology notes, a generation pipeline for a 182-drug
reference handbook, scrapers for Iranian doctor and appointment directories, and the
substance-reference dataset behind those. Everything here is Python-first, RTL/Persian-aware,
and built against real clinical source material.

This is a collection repo. Each subdirectory is an independent tool that came from a different
working folder on the author's machine; they are grouped because they serve the same domain, not
because they share code.

**Stack:** Python 3 (openpyxl, PyMuPDF, Pillow, requests, python-docx), Node + `docx`, some Go
## Layout

| Path | What it is |
|---|---|
| `handbook-pipeline/` | 56 scripts that build the handbook: page population (`populate_all_handbook_pages.py`, 1 474 lines), cleanup (`clean_populator.py`), master-data generation (`generate_master_data.py`, `handbook_data.py`), category chunking (`chunk_antibiotics_*.py`), and successive assembly passes (`build_the_ultimate_182_pages_handbook.py`, `assemble_complete_mega_handbook.py`). Three parallel data variants exist for the editorial modes: pure, verbatim, and with-notes. |
| `monograph-ocr/` | Extraction of a 617-page Persian drug monograph to Markdown - `build_md.py` and numbered successors (`build_md10.py` … `build_md14.py`) refining RTL line/column reflow under printed box captions. PyMuPDF + Tesseract. |
| `drug-transcripts/` | Transcription of 182 pages of handwritten Persian pharmacology notes, plus `docx-build/` (Node generator that renders the handbook DOCX) and `parts/`. |
| `directory-scrapers/` | The visital.ir data acquisition pipeline. `doctoreto_scraper.py` walks doctoreto.com city/specialty listings; `import_nobat.py` handles nobat.ir appointment data; `scraper_comments.py` pulls reviews; `build_category_mapping.py` maps source categories onto the site taxonomy; `import_*_live.py` / `import_shiraz_*.py` / `import_qom_*.py` push per-city results into the WordPress API via `ahura_api.py`. `mock_ahura_api.py` is an offline stand-in for that API. Docs: `AHURA_PIPELINE.md`, `SCRAPER_GUIDE.md`. |
| `substance-reference/` | Extracted structured substance data (`all_200_extracted.json`, `clean_200_drugs.json`, `ocr_extracted_200.json`, `complete_page_map_200_verified.json`) with the scripts that produced them - the reference dataset keyed by mechanism of action. |

## What is deliberately not here

- **Generated outputs.** No scanned page images, rendered handbook previews, PDFs, or DOCX
  results. The handbook pipeline writes ~1 800 PNG intermediates and 46 DOCX files; those are
  build artifacts, and the handbook itself derives from a copyrighted source text, so the
  outputs carry that copyright even though the scripts do not.
- **Scraped personal records.** The per-city clinician datasets produced by
  `directory-scrapers/` (~130 MB of merged JSON and Excel: Tehran, Mashhad, Isfahan, Karaj,
  Shiraz, Qom, Rasht, Ahvaz, Zanjan) are real practitioners' data and stay out of the repo.
- **Browser profiles and session state.** The scrapers were driven by persistent Chrome/Opera
  profiles; those are excluded entirely.
- **Credentials.** API keys present in the working copies are replaced with
  `<REDACTED_*>` placeholders in `ahura_api.py`, `import_to_ahura.py` and `mock_ahura_api.py`.
  Set `AHURA_API_KEY` in the environment instead. The keys in the original working copies
  should be considered exposed and rotated.

## Related, published separately

- `piru` - an iOS/Android substance dose journal and pharmacopeia (1 689 entries, fully offline).
  Same domain, different medium, so it is its own repository.
- The WordPress plugin exposing the `ahura/v1` REST API that `directory-scrapers/` writes into.

## Running

Each tool has its own entry point and is not wired together by a shared runner. Typical order
for the handbook: OCR/transcription input → `monograph-ocr/build_md*.py` or `drug-transcripts/`
→ `handbook-pipeline/generate_master_data.py` → a `build_*_handbook.py` assembler. For the
directory pipeline, read `directory-scrapers/AHURA_PIPELINE.md` first; `mock_ahura_api.py` lets
you exercise imports without touching the live site.
