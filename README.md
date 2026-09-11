# Data Type XXL

![Data Type XXL](datatypexxl_preview.png)

**Build training datasets, then train models on them — from collection to a model trained from scratch, in one tool.**

Data Type XXL is a Windows desktop application (and an equivalent notebook/Python API) for building clean knowledge and source-code datasets and turning them into models. It collects the web, ingests your local files and whole sites, repairs and explains the data, generates synthetic and multimodal data, and trains — LoRA adapters *or* a model from scratch — all in the standard dataset format so every stage round-trips.

- **Version:** 2.0.0  ·  **Python:** 3.14 (cp314)  ·  **Platform:** Windows 11 (desktop exe) + notebook API
- **API surface:** 181 functions in `datatypexxl.py`  ·  **Source tests:** 81 (run on every build)
- **License:** Apache-2.0

> Origin: the project exists to build datasets and models for a real need — designing plastic injection **moulds / CAD** (dimensioned 2D DWG + 3D) from a natural request, instead of the weeks it takes by hand.

---

## Contents
- [Two ways to use it](#two-ways-to-use-it)
- [What it can do](#what-it-can-do)
- [Command-line (exe)](#command-line-exe)
- [Tested](#tested)
- [Benchmarks](#benchmarks)
- [API / exe / zip parity](#api--exe--zip-parity)
- [Build from source](#build-from-source)
- [Honesty & scope](#honesty--scope)

---

## Two ways to use it

**Desktop app (exe).** `DataTypeXXL.exe` — a tabbed GUI: Sources, Internet Search, Full Coder, Training & Output, Data Prompt, Model Adaptation, Run Model, Merge Datasets, Model Viewer, **Scratch Training**, Performance. Code-signed, self-contained (`DataTypeXXL_runtime/`), ~44 MB.

**Notebook / Python API.** `import datatypexxl as dx` — the same features as functions. On-demand dependencies live in `.datatypexxl_deps`; the sources ship in `zippeddata.zip`.

```python
import datatypexxl as dx
dx.doctor()                                   # environment health check
run = dx.crawl_site("netplas.com", max_pages=50, name="netplas")
dx.enrich_run(run["run_directory"])           # add the "what is this" explanation layer
dx.train_from_scratch(run["run_directory"], preset="small", tokenizer="cl100k_base")
```

---

## What it can do

### Data collection
- Multi-website runs; Common Crawl archive ingestion at web-corpus scale; related-web discovery.
- Wikipedia/Wikibooks/Stack Overflow/GitHub Code Search presets + ready-to-select docs (MDN, Python, Rust, Go, PyTorch, …).
- **Full Coder** mode: 371-entry language/DSL catalog, 140-source verified Code Memory, two code-quality passes, 97-language multilingual research, GitHub/GitLab/Codeberg repo ingestion with test/impl pairing and SPDX licensing.
- Hardware-aware workers (auto or up to 64), local/remote browser pool, Dask backends, resumable frontier, safe stop.
- Output: JSONL/JSON/TXT/CSV; tokenizers `cl100k_base`, `o200k_base`, UTF-8 bytes, or text-only.

### Web scraping & whole-site harvesting  *(`web_scrape`)*
- `web_scrape(url)` — pull a page's title/headings/paragraphs/table-cells/links (stdlib only), explained by the small AI.
- `crawl_site(url)` — follow internal links across a whole site into one self-explaining dataset.
- `collect_site_files(url, preset=…/omega=True)` — download every **safe, useful** file on a site and tokenise it through local ingest. **Strict filtering:** executables/scripts blocked (by extension *and* by magic bytes — a `.pdf` that is really an `.exe` is rejected), ad/tracker URLs skipped, empty/oversized files dropped, de-duplicated. Presets: `omega`, `drawings`, `documents`, `code`, `quick`.

### Local ingest & tokenization  *(`ingest_folder`, `local_ingest`)*
- Folder or `.zip` → dataset; text, code, CAD (`.dxf/.dwg`), meshes (`.obj/.stl/.step`), PDFs, Office docs, notebooks.
- Optional `<MESH>` tokenisation, per-document explanation headers, and URL ingest in the same run.

### The "small AI" explanation layer  *(`describe`)*
- `explain_content` / `explain_header` — for any content: **what it is / what it describes / what a user might ask**, plus key points. Heuristic and offline by default; richer, structured output when a local Ollama model is present.
- `enrich_run` — apply that layer to any finished run as a non-destructive `_explanations.jsonl` sidecar.

### Token repair & cleaning  *(`token_repair`)*
- Fixes mojibake (`Ã§elik` → `çelik`), strips replacement/control/zero-width characters, NFC-normalises; scores brokenness and **drops irredeemably corrupt documents**. Runs automatically in ingest/scrape; structure-preserving for code/CAD.
- `clean_run` — repair an already-collected run into a new, cleaned, re-tokenised copy (originals untouched).

### Dataset tooling  *(`dataset_tools`)*
- `enrich_run`, `expand_instructions` (turn a run into question→content SFT pairs), `clean_run`.
- `near_duplicates` — MinHash + LSH to find *near*-identical documents (not just exact), numpy-only.
- `combine_datasets` / `list_runs` — merge several runs (de-duplicated) for multi-dataset training.
- split, PII scan, duplicate report, token histogram, dataset health/coverage/overlap.

### Global / public datasets  *(`global_datasets`)*
- Curated registry (`tiny_shakespeare`, `wikitext-2/103`, `code_search_net`, `the_stack_smol`, `openwebtext`, `oscar_en`). URL datasets load through the harvester+ingest; Hugging Face datasets stream via the optional `datasets` add-on. Everything lands in the standard format and mixes with your own runs.

### Synthetic data (self-instruct)  *(`synthetic_data`)*
- `generate_synthetic_dataset(topic, kind='qa'|'instruction'|'cad')` — a local Ollama model writes realistic instruction/response pairs into the standard format. Bootstraps domains the web is thin on (e.g. mould/CAD prompts).

### Generation harnesses (multimodal)
- **Image** (`image_gen`), **Video** (`video_gen`), **Audio** (`audio_gen`) — text→media with quality tiers, prompt enhancement, and dataset synthesis (media+caption manifests for training). Gated behind the relevant add-on + GPU; config/planning/gating run on CPU.

### CAD / 3D / mesh
- `<PART>` command language ↔ dimensioned DXF (with tolerances); `part_to_step` (2D→3D), `part_to_image` (PNG drawing), engineering sheets, DFM/tolerance/symmetry precision layer.
- numpy-only mesh repair/quality; MeshGPT-style `<MESH>` tokens; Hunyuan3D harness.

### Train from scratch  *(`scratch_training`)* — headline
- `train_from_scratch(runs, preset=…, tokenizer=…)` — a **fresh, randomly-initialised GPT** (presets nano→base, ~1.8M–355M params) trained on **your** dataset(s) and tokenizer. Not a fine-tune.
- **Multi-dataset:** one run or many (yours + global), with optional **mixture weights**.
- **Fast training:** mixed-precision autocast, fused/foreach AdamW, cosine LR + warmup, gradient clipping, TF32, optional `torch.compile`.
- **Resumable:** `checkpoint.pt` (weights + optimizer + step); `resume=True` continues.
- `generate_text` (temperature/top-k sampling), `evaluate_model` (loss/perplexity), `export_model` (safetensors/.pt + manifest), `train_tokenizer` (SentencePiece on your data), `scratch_plan` (VRAM-aware sizing).

### Fine-tuning / adapters
- `train_adapter` (LoRA/QLoRA on one or many runs, merged), `train_dtxxl_lora`, `train_full` / `continued_pretraining`, VRAM planner, DeepSpeed ZeRO offload, 4/8-bit quantisation. Base weights read-only; held-out validation gate.

### Diagnostics, drivers & feedback
- `doctor()` / `diagnostics()` — PASS/WARN/FAIL health across every subsystem; `import_health()` is the API↔exe consistency test.
- `gpu_drivers()` — detects GPU/driver/CUDA/VRAM via `nvidia-smi` (no torch needed) and torch.
- `progress_reporter()` — rate + ETA feedback; `checkpoint_report()` — progress/ETA/resumable for long runs.

---

## Command-line (exe)

The packaged exe runs headless utilities without the notebook:

```
DataTypeXXL.exe --doctor                    # full environment/health report (exit!=0 on FAIL)
DataTypeXXL.exe --gpu-info                   # detected GPUs, driver, CUDA, VRAM
DataTypeXXL.exe --repair-run <run_dir>       # clean a dataset into a new copy
DataTypeXXL.exe --enrich-run <run_dir>       # write the explanation sidecar
DataTypeXXL.exe --train-scratch <run> [preset]   # plan + train from scratch (needs torch)
DataTypeXXL.exe --generate <model_dir> [prompt]  # sample text from a scratch model
DataTypeXXL.exe --evaluate <model_dir> <run_dir> # loss / perplexity
DataTypeXXL.exe --export-model <model_dir> [out] # export to safetensors/.pt
```

---

## Tested

81 source tests run automatically on **every build** (the build fails if any fail). Coverage includes:

- **Exact-dimension CAD** (box/cavity/plates to the millimetre), CAD-command ↔ DXF round-trip, `part_to_image` PNG.
- **Mesh** repair/quality/watertightness (numpy-only), `<MESH>` tokens, multi-view.
- **Precision** (ISO fit tolerances, DFM, symmetry, hole pitch).
- **Web scrape/crawl** parsing, link normalisation, same-site rules; **site-file filtering** (executables, disguised-exe magic bytes, ad/tracker URLs, empty files all correctly rejected).
- **Token repair** (mojibake fix, control/replacement stripping, code-indentation preserved, brokenness scoring, drop-irredeemable) — verified end-to-end in ingest.
- **Explain layer** (3-way output, AI-JSON parsing + heuristic fallback).
- **Dataset tools** (instruction expansion, clean_run repairs+drops into a new run, near-duplicate clustering).
- **Scratch training** — parameter accounting, VRAM planning, multi-run streaming, mixture-weight oversampling; and, with torch present, a **real nano model trained on CPU** (weights saved), **resume** from checkpoint, **generate**, **evaluate**, **export** (safetensors) all verified.
- **Diagnostics/doctor/drivers**, **progress/ETA**, **checkpoint report**, **global-dataset registry** gating, **synthetic** prompt/parse + Ollama gating.

Honesty is enforced in the tests: GPU/torch/Ollama-only paths are gated and skip cleanly when the backend is absent, rather than pretending to run.

---

## Benchmarks

Measured on this machine — **not estimates**. Environment: Windows 11, NVIDIA GeForce RTX 4060 Laptop (8 GB, driver 581.57) but **CPU-only PyTorch** (`torch 2.13.0+cpu`, no CUDA), Python 3.14.6.

| Operation | Result | Notes |
|---|---:|---|
| API import (`import datatypexxl`) | **0.11 s** | loads 181-function surface |
| Token repair | **15.8 MB/s** | mojibake + control/replacement cleaning |
| Web page parse | **~899 pages/s** | stdlib HTML parser |
| Near-duplicate detection | **~495 docs/s** | MinHash + LSH, numpy-only |
| Train-from-scratch (nano, CPU) | **~77 tok/s** | single machine, CPU torch, `fast=True` |

The from-scratch figure is **CPU-only** (this box has no CUDA build); on a CUDA GPU the fast path (mixed precision, fused optimiser, TF32, `torch.compile`) applies and throughput is far higher. That number is not measured here and is not claimed.

---

## API / exe / zip parity

Verified for this release:

- **API:** 181 functions in `datatypexxl.__all__`, all resolve and are callable.
- **zip ↔ source:** all **123 `.py` files** in `zippeddata.zip` are **byte-identical** (SHA-256) to the source tree — no drift, nothing missing or extra.
- **exe:** every feature module (`scratch_training`, `global_datasets`, `synthetic_data`, `dataset_tools`, `token_repair`, `web_scrape`, `image_gen`, `video_gen`, `audio_gen`, `describe`, `diagnostics`, `datatypexxl`, …) is present in the packaged PYZ.
- **Version:** `APP_VERSION`, the exe's ProductVersion, and the engine.py inside the zip all read **2.0.0**.
- **torch:** installed into `.datatypexxl_deps` (works notebook/Colab, deps-only verified) and deliberately **not** bundled into the exe, which stays ~44 MB.

---

## Build from source

Windows, **Python 3.14** (the deps and PyInstaller build target cp314; the default 3.12 breaks cp314 numpy):

```powershell
# prepend Python 3.14 to PATH, then:
.\build.ps1 -ReusePreparedTools
```

Produces `DataTypeXXL.exe` + `DataTypeXXL_runtime/` in the parent folder, runs the 81 source tests, prepares tokenizer/parser caches, and code-signs the exe. (Linux: `build.sh`.)

---

## Honesty & scope

- Everything that runs on the CPU here is **tested and measured**; GPU-only performance is flagged as such and never fabricated.
- Model training/generation needs the `train` add-on (**torch**) — installed in the deps for the API/notebook, gated in the exe. Frontier-quality results depend on your data and GPU fine-tune, not the tool alone: the tool produces clean, exact, standards-checked **drafts** and the training infrastructure; an engineer verifies before machining.
- Image/video/audio generation and Hugging Face dataset loading need their respective add-ons + (for real output) a GPU; they gate cleanly when absent.
- No telemetry. Third-party code/data is fetched at collection time, not redistributed in the app.
