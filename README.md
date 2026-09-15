# Smart Multimedia Storage System

A Streamlit-based content management system for local journalism that
automatically enriches uploaded media (video, images, documents) with
machine-generated metadata — transcripts, OCR text, summaries, locations,
and speaker tags — so archives become searchable without manual tagging.
Built as an MSc dissertation project for the Christopher Nieper Foundation
(Spirit of Alfreton Community Project).

## Demo

[Watch the full walkthrough (demo.mp4)](demo.mp4) — upload →
automatic transcription/OCR/metadata generation → dashboard → search.

> **Note:** This demo uses sample/placeholder data (test files and a public
> BBC News clip used purely to demonstrate the transcription pipeline)
> Full source code
> is available on request, pending confidentiality clearance from the
> project sponsor.

## Problem

Local newsrooms accumulate large volumes of unstructured media (interviews,
photos, scanned documents, event footage) that are slow to search and reuse
because nothing in them is indexed beyond a filename. The goal was to build
a system that ingests any of these file types, extracts structured,
searchable information automatically, and proves — with real measurement,
not just a demo — which parts of that pipeline actually work well enough to
rely on.

## Approach

Five ML pipelines sit behind a single upload-and-search interface, backed by
a SQLite database:

- **Audio/video**: Whisper ASR for transcription, speaker diarization via
  Resemblyzer, and a text-quality pass to filter transcription
  hallucinations.
- **Images/documents**: EasyOCR-first text recognition, with a conditional
  TrOCR fallback for handwriting.
- **Entity extraction**: GLiNER zero-shot NER for locations, cross-checked
  against Nominatim geocoding.
- **Summarisation**: a hybrid extractive (Sumy/LexRank) + abstractive
  (DistilBART) pipeline.
- **Search**: fuzzy, typo-tolerant search across all extracted metadata.

The codebase is modular by responsibility (config/models/db/search engine as
shared infrastructure, one file per media pipeline, one file per Streamlit
page) rather than a single monolithic script — see the Module Map below.

## Result

Rather than just demoing the system, every component was evaluated against
hand-built ground truth using the appropriate metric for that task:

| Component | Metric | Result |
|---|---|---|
| Speech transcription (Whisper) | WER | 7.3% |
| Printed-text OCR (EasyOCR) | CER | 7.95% |
| Entity/location extraction (GLiNER) | F1 | 0.333 |
| Summarisation (hybrid pipeline) | ROUGE-L | 0.263 |

Transcription and printed-text OCR performed at a genuinely usable level.
Entity extraction and summarisation scored lower — the report treats this as
a real finding, not a flaw to hide: it explains *why* (multi-location
documents confuse the NER model; ROUGE-L is a strict lexical-overlap metric
that penalises valid paraphrasing) rather than overstating what the numbers
show.

## Tech stack

Python, Streamlit, SQLite, OpenAI Whisper, EasyOCR, Microsoft TrOCR, GLiNER,
Resemblyzer, Sumy, Hugging Face Transformers (BART), Nominatim/geopy.

## Getting started

Source code isn't public in this repo yet (see note above) — the demo GIF
and video above show the full working system. Once cleared for release,
setup will be:

```bash
pip install -r requirements.txt
streamlit run app.py
```

Run it from the project root so `news_archive.db` and the `uploads/` folder
are created alongside the code. First run downloads the ML models (Whisper,
EasyOCR, GLiNER, etc.) automatically — this can take a while and needs an
internet connection.

## Module map

| File / folder | Responsibility |
|---|---|
| `config.py` | Paths, storage routing, category taxonomy |
| `models.py` | Loads and caches every ML model |
| `db.py` | Schema, queries, filters, record updates |
| `search_engine.py` | Fuzzy search and result highlighting |
| `ingestion.py` | Runs the right pipeline per upload and stores the result |
| `pipelines/` | One file per ML pipeline (audio, vision, entity, summarisation, document parsing, text quality) |
| `page_modules/` | One file per Streamlit page (dashboard, upload, search, events, media types) |
| `ui_components.py`, `ui_theme.py` | Shared UI elements and theming |
| `app.py` | Entry point: session state, routing only |

## Limitations

- Entity extraction and summarisation are the weakest components by
  measured score — documented in the evaluation rather than glossed over.
- The diarization proxy metric (speaker-count accuracy) is not the same as
  a true Diarization Error Rate, and is labelled as such in the report.
- ML pipelines require meaningful compute/download time on first run;
  no lightweight offline mode exists.

## Files

- `demo.mp4` — walkthrough of the working system
- `README.md` — this file
- Application code (`app.py`, `config.py`, `db.py`, `models.py`,
  `search_engine.py`, `ingestion.py`, `ui_components.py`, `ui_theme.py`,
  `pipelines/`, `page_modules/`) and `requirements.txt` — available on
  request pending confidentiality clearance (see note above)
