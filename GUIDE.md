# Data Type XXL — Complete User Guide

**Build clean training datasets, then train models on them — from collection to a model trained from scratch, in one tool.**

This is the full manual: how to open it, what every part does, how to use both the **desktop app (exe)** and the **Python API**, every feature, the command line, the tokenizers, and the version history.

- **Current version:** 2.2.0
- **Python:** 3.14 (cp314)
- **Platform:** Windows 11 (desktop exe) + notebook/Python API
- **License:** Apache-2.0

---

## Table of contents
1. [What it is & who it's for](#1-what-it-is--who-its-for)
2. [Install & open](#2-install--open)
3. [Quick start](#3-quick-start)
4. [The desktop app (exe) — tab by tab](#4-the-desktop-app-exe--tab-by-tab)
5. [The Python API — by area](#5-the-python-api--by-area)
6. [Command-line (exe flags)](#6-command-line-exe-flags)
7. [Tokenizers (including our own dttx)](#7-tokenizers-including-our-own-dttx)
8. [Training a model from scratch](#8-training-a-model-from-scratch)
9. [Full feature reference](#9-full-feature-reference)
10. [Version history (2.0 → 2.1 → 2.2)](#10-version-history)
11. [Requirements & honest scope](#11-requirements--honest-scope)
12. [File locations](#12-file-locations)

---

## 1. What it is & who it's for

Data Type XXL is a **dataset factory + training toolkit**. You point it at the web, your own files, or whole sites; it collects, cleans, explains, de-duplicates and tokenises the data into one standard format; then you can train — a LoRA adapter on an existing model, *or* a model **from scratch** on your own data and tokenizer.

It exists for a real goal: building the data and infrastructure to design plastic injection **moulds / CAD** (dimensioned 2D DWG + 3D) from a natural request, instead of the weeks it takes by hand. But it is general — any knowledge or source-code dataset works.

**Two equal ways to use it:**
- **Desktop app** (`DataTypeXXL.exe`) — a tabbed window, no Python needed.
- **Python API** (`import datatypexxl as dx`) — the same features as functions, for notebooks/Colab.

Both are the **same version** and the **same features**.

---

## 2. Install & open

### Desktop app (exe)
1. Download `DataTypeXXL_v2.2.0_windows.zip`.
2. Unzip it anywhere.
3. Double-click **`DataTypeXXL.exe`**. (It needs the `DataTypeXXL_runtime` folder next to it — keep them together.)

That's it — no Python install. The window is **scrollable** (vertical + horizontal scrollbars), so it works on small screens too. Mouse wheel scrolls vertically; **Shift + wheel** scrolls horizontally.

### Python API
1. Get the `api` folder (the `apidatatypexxl` source).
2. In a notebook or script:
```python
import sys
sys.path.insert(0, r"path/to/apidatatypexxl")
import datatypexxl as dx
print(dx.version())        # 2.2.0
dx.doctor()                # environment health check
```
- **Light functions** (explain, token repair, web scrape, dataset tooling, the dttx tokenizer) need nothing extra.
- **Heavy functions** (numpy / torch / ezdxf / tiktoken — training, CAD, near-dup) need the dependency set: put a `.datatypexxl_deps` folder next to `apidatatypexxl`, **or** `pip install numpy torch ezdxf tiktoken`, **or** let the API download them on first heavy call (needs internet).

---

## 3. Quick start

**Collect a site and train a small model (API):**
```python
import datatypexxl as dx
run = dx.crawl_site("example.com", max_pages=50, name="mysite")   # collect
dx.enrich_run(run["run_directory"])                               # add explanations
dx.clean_run(run["run_directory"], name="mysite_clean")           # repair corrupt text
dx.train_from_scratch(run["run_directory"], preset="small",       # train from scratch
                      tokenizer="dttx")
```

**Build a dataset from your own files (exe):**
1. Open the app → **Sources** or **Training & Output**.
2. Point it at a folder (or use built-in sources / Full Coder / FLC).
3. Pick a tokenizer (try **dttx** — our efficient one).
4. Press the collect/run button and watch the live log.

---

## 4. The desktop app (exe) — tab by tab

The window is split: **left** = tabs (setup), **right** = output folder + live runtime status. Everything scrolls.

- **Sources** — Pick built-in sources (Wikipedia, Stack Overflow, docs…) and add your own named websites (name + URL). Multiple can be selected in one run.
- **Internet Search** — Unlimited named internet sources, each with its own search text and optional seed website.
- **Full Coder** — Code-focused mode: a 371-entry language/DSL catalog, 140-source verified Code Memory, two code-quality passes, 97-language research, GitHub/GitLab/Codeberg repo ingestion with test/impl pairing and SPDX licensing.
- **Training & Output** — The main run settings: **Content mode** (Auto / Knowledge / Code / Full Coder / Full Knowledge / Full Everything / Full Omega Code / Wanted 3D / **FLC**), code languages, item/byte limits, workers, **Tokenizer** (cl100k_base / o200k_base / **dttx** / UTF-8 bytes / None), chunk size/overlap, force pages, quality score, output format (JSONL/JSON/TXT/CSV), dataset folder name.
- **Data Prompt** — An explicit system-training document: model definition, training rules, target code-percentage, curriculum order, vocabulary size, final sequence length.
- **Model Adaptation** — Fine-tune a local Hugging Face causal model + one completed run into a **LoRA/QLoRA** adapter, in an external CUDA Python runtime. Base weights read-only; a held-out validation gate quarantines a degrading adapter.
- **Run Model** — Point at an extracted model folder (or a `.gguf`), inspect it, and run prompts. Also runs any model your local Ollama server has pulled.
- **Merge Datasets** — Select two or more runs and fuse them into one (byte-for-byte shard copy, duplicates kept once).
- **Model Viewer** — Inspect a 3D model / mesh.
- **Scratch Training** — Train a model **from scratch** on one or many runs: pick the runs (`;`-separated / "Add run…"), a global public dataset, a preset (nano→base), a tokenizer, VRAM, and a **Fast** toggle. Buttons: Plan, Prepare tokenizer, Check GPU/drivers, Train from scratch (with live output).
- **Performance** — Compute backend (threads / multi-process CPU / local & remote Dask), workers, memory limits.

---

## 5. The Python API — by area

`import datatypexxl as dx` — 192 functions. The main ones by area:

### Collection
- `flc_collector()`, `full_omega_code_collector()`, `everything_collector()`, `knowledge_collector()`, `wanted_3d_collector()`, `video_collector()`, `file_editing_collector()`
- `collect_multi(["flc", "full_coder"])` — **select several built-in collectors at once** (auto sources)
- `sources()`, `target_plan()`, `site_plan()`, `capabilities()`

### Web scraping & whole-site harvesting
- `web_scrape(url)` — one page → title/headings/paragraphs/tables/links + explanation
- `crawl_site(url, max_pages=…)` — a whole site → one self-explaining dataset
- `collect_site_files(url, preset=…/omega=True)` — download every **safe** file and tokenise it (blocks executables by extension *and* magic bytes, skips ads/trackers, empties, oversized; presets: omega/drawings/documents/code/quick)
- `harvest_presets()` — the ready-made parameter bundles

### Local ingest
- `ingest_folder(folder_or_zip, …)` — folder/zip → dataset (text, code, CAD, meshes, PDFs, Office, notebooks); optional `<MESH>` tokens, URL ingest, explanation headers

### The "small AI" explanation layer
- `explain_content(text)` / `explain_header(text)` — what it is / what it describes / what a user would ask (+ key points; richer with a local Ollama model)
- `enrich_run(run_dir)` — add explanations to any finished run (sidecar)

### Token repair & cleaning
- `repair_text(text)` — fix mojibake, strip control/replacement/zero-width chars
- `text_brokenness(text)`, `is_broken_text(text)`, `clean_tokens(text)`
- `clean_run(run_dir)` — repair a run into a new, cleaned, re-tokenised copy

### Dataset tooling
- `dataset_card(run_dir)` — a Hugging-Face-style Markdown card
- `redact_run(run_dir)` — a PII-redacted copy (emails/phones/keys masked)
- `curriculum_order(run_dir)` — order documents easy→hard for training
- `language_report(run_dir)`, `near_duplicates(run_dir)`, `cross_run_duplicates([...])`
- `expand_instructions(run_dir)` — turn a run into question→content SFT pairs
- `combine_datasets([...])`, `list_runs("datasets")`
- `split_dataset`, `scan_pii`, `duplicate_report`, `token_histogram`, `dataset_health`, `dataset_coverage`, `dataset_overlap`, `dataset_report`

### Tokenizers
- `train_dttx_tokenizer(run_dir, vocab_size=16000)` — train our own tokenizer on your data
- `dttx_efficiency(text, tokenizer=…)`, `tokenizer_benchmark(text)` — compare tokenizers

### Global / public datasets
- `global_datasets()`, `describe_global_dataset(name)`, `load_global_dataset(name)` (tiny_shakespeare, wikitext, code_search_net, the_stack_smol, openwebtext, oscar_en)

### Synthetic data (self-instruct)
- `generate_synthetic_dataset(topic, kind='qa'|'instruction'|'cad')` — a local Ollama model writes instruction/response pairs

### Generation harnesses (multimodal)
- Image: `generate_image`, `plan_image_dataset`, `generate_image_dataset`
- Video: `video_create`, `image_to_video`, `generate_video_dataset`, `tokenize_video`
- Audio: `generate_audio`, `generate_audio_dataset`
- `generation_capabilities()` — readiness across image/video/audio/3D/CAD

### CAD / 3D / mesh
- `cad_program`, `part_to_dxf`, `part_to_step`, `part_to_image`, `cad_box/cylinder/…`, `engineering_sheet`, `finalize_drawing`
- `mesh_program`, `mesh_repair`, `mesh_analyze`, `mesh_to_step`, `mesh_to_part_drawing`
- `iso_fit`, `dfm_check`, `apply_tolerances`, `enforce_symmetry`

### Train from scratch
- `scratch_capabilities()`, `scratch_architectures()`, `scratch_plan(run_dir, preset, vram_gb)`
- `train_from_scratch(runs, preset, tokenizer, weights=…, resume=…)`
- `generate_text(model_dir, prompt)`, `evaluate_model(model_dir, run_dir)`, `export_model(model_dir)`
- `train_tokenizer(run_dir)` (SentencePiece)

### Fine-tuning / adapters
- `train_adapter([...], base_model, out)`, `train_dtxxl_lora`, `train_full`, `continued_pretraining`, `train_lora`, `train_dpo`, `vram_plan`, `deepspeed_config`, `quantize_model`

### Diagnostics, drivers & feedback
- `doctor()`, `diagnostics()`, `import_health()`, `module_versions()`, `debug_mode(on)`
- `gpu_report()`, `gpu_drivers()`
- `progress_reporter(total, label)`, `checkpoint_report(dir)`, `checkpoint_status(dir)`, `resumable_workspace(dir)`

---

## 6. Command-line (exe flags)

Run these on the packaged exe without the notebook:
```
DataTypeXXL.exe --doctor                        # full environment/health report
DataTypeXXL.exe --gpu-info                       # GPUs, driver, CUDA, VRAM
DataTypeXXL.exe --repair-run  <run_dir>          # clean a dataset into a new copy
DataTypeXXL.exe --enrich-run  <run_dir>          # write the explanation sidecar
DataTypeXXL.exe --train-scratch <run> [preset]   # plan + train from scratch (needs torch)
DataTypeXXL.exe --generate     <model_dir> [prompt]   # sample text from a scratch model
DataTypeXXL.exe --evaluate     <model_dir> <run_dir>  # loss / perplexity
DataTypeXXL.exe --export-model <model_dir> [out] # export to safetensors/.pt
```

---

## 7. Tokenizers (including our own dttx)

You can tokenise with any of these (GUI dropdown or API `tokenizer=` / `Tokenizer(name)`):

| Tokenizer | Needs | Notes |
|---|---|---|
| `cl100k_base` | tiktoken | GPT-4 class BPE |
| `o200k_base` | tiktoken | newer OpenAI BPE |
| **`dttx`** | **nothing (stdlib)** | **our own** byte-level BPE — lossless + efficient |
| `UTF-8 bytes` | nothing | 1 token per byte, lossless, simple |
| `Words` / `Characters` | nothing | simple splits |

**dttx** is Data Type XXL's own tokenizer: a byte-level BPE with a **byte foundation** (lossless on any input — no `<unk>`) and **learned merges** (efficient — fewer tokens). Measured **~4.1× fewer tokens than raw UTF-8** on real code. It ships with a trained default model (16k vocab) bundled in both the exe and the API, and needs no dependency (unlike cl100k/o200k which need tiktoken). Train your own on your corpus with `dx.train_dttx_tokenizer(run_dir)`.

---

## 8. Training a model from scratch

`train_from_scratch` builds a **fresh, randomly-initialised GPT** (not a fine-tune) and trains it on your data with your tokenizer.

- **Presets:** nano (~1.8M) · micro · tiny · small (~110M) · base (~355M)
- **Multi-dataset:** one run or many (`run_directories=[...]`), with optional **mixture weights**
- **Fast training:** mixed-precision (GPU), fused AdamW, cosine LR + warmup, gradient clipping, TF32, optional `torch.compile`
- **Resumable:** saves `checkpoint.pt` (weights + optimizer + step); `resume=True` continues
- **Then:** `generate_text` (temperature/top-k), `evaluate_model` (loss/perplexity), `export_model` (safetensors)

Example:
```python
dx.train_from_scratch(["datasets/a/run_…", "datasets/b/run_…"],
                      preset="tiny", tokenizer="dttx",
                      weights=[1, 3], epochs=1)
txt = dx.generate_text("scratch_model", "def netplas(")
print(dx.evaluate_model("scratch_model", "datasets/held_out/run_…"))
```

> Real training needs the **`train` add-on (torch)** and a CUDA GPU for anything past a small run. On CPU it works but is slow. The from-scratch loop is verified (a real nano model trained, loss dropped from 66 → 3.25).

---

## 9. Full feature reference

Everything, grouped:
- **Collection:** multi-site, Common Crawl, related-web discovery, Full Coder, FLC (604 sources), repo ingestion, `collect_multi`
- **Web:** scrape, crawl, whole-site file harvest with strict safety filtering
- **Ingest:** folder/zip, code/CAD/mesh/PDF/Office/notebooks, URL ingest
- **Explain:** 3-way explanation layer, run enrichment
- **Clean:** token repair (mojibake/corruption), clean_run, redact_run (PII)
- **Dedup:** exact, near-duplicate (MinHash+LSH), cross-run
- **Tokenizers:** cl100k, o200k, **dttx** (own), UTF-8 bytes; train dttx/SentencePiece; benchmark
- **Datasets:** dataset_card, language_report, curriculum_order, split, PII scan, histograms, health/coverage/overlap
- **Global datasets:** curated public-dataset loader
- **Synthetic:** self-instruct generation via local model
- **Generation:** image / video / audio harnesses + video tokenisation
- **CAD/3D:** `<PART>` ↔ dimensioned DXF, 2D→3D STEP, PNG drawings, mesh repair/quality, precision (ISO/DFM/symmetry), Hunyuan3D harness
- **Training:** from-scratch GPT (multi-dataset, weights, resume, fast), LoRA/QLoRA, full/continued pretraining, VRAM planner, quantisation, DeepSpeed
- **Model use:** run a downloaded model, Ollama models, model viewer
- **Diagnostics:** doctor, import health, module versions, GPU drivers, progress/ETA, checkpoint reports
- **UI:** scrollable window (vertical + horizontal)

---

## 10. Version history

### 2.0.0
- First public release. Data collection (web/local/site), token repair, the explanation layer, multimodal + synthetic data harnesses, **train-from-scratch** and LoRA, CAD/3D/mesh tools, diagnostics, GPU driver detection, CLI (`--doctor`, `--train-scratch`, …). API + exe parity, 81 tests.

### 2.1.0
- **Fixed from-scratch training** — it now really learns (loss 66 → 3.25). Fixed LR warmup on short runs, CPU mixed-precision, and the UTF-8-bytes vocab.
- **Auto-dependency activation** — `import datatypexxl` alone puts `.datatypexxl_deps` on the path.
- **Benchmark & Proof Pack** — real, executed evidence.

### 2.2.0
- **dttx tokenizer** — our own byte-level BPE, a first-class tokenizer option (GUI + engine + pipeline), lossless + **~4.1× more efficient than raw bytes**, with a trained default model bundled, no dependency needed. Faster (word-frequency) BPE training.
- **New dataset tools** — `dataset_card`, `redact_run`, `curriculum_order`, `language_report`, `cross_run_duplicates`, `tokenizer_benchmark`.
- **`collect_multi`** — select several built-in collectors at once (FLC + Full Coder together).
- **`tokenize_video`** — turn video into a compact token sequence.
- **Scrollable window** — vertical + horizontal scrollbars for small screens.

---

## 11. Requirements & honest scope

- **Desktop app:** Windows 10/11. Self-contained; no Python needed.
- **API:** Python 3.14. Light features need only the standard library; heavy features need their deps (numpy/torch/ezdxf/tiktoken/…).
- **Training / generation:** needs the `train` add-on (**torch**) and, for real speed/scale, a CUDA GPU. Image/video/audio generation and Hugging Face dataset loading need their own add-ons. These all **gate cleanly** when a backend is absent — the tool says what's missing instead of pretending.
- **Honesty:** benchmarks in this project were measured on CPU (no CUDA on the test machine); GPU numbers are higher and are **not claimed**. The tool builds frontier-scale **data and training infrastructure** — the quality of a final model depends on your data, your GPU, and your fine-tuning, not the tool alone.
- **Privacy:** no telemetry. Third-party code/data is fetched at collection time, not redistributed.

---

## 12. File locations

Default layout on this machine:
```
Desktop\
 ├─ derle\                              ← latest release build (kept current)
 │   ├─ exe\
 │   │   ├─ DataTypeXXL.exe             ← the desktop app
 │   │   └─ DataTypeXXL_runtime\        ← its runtime (keep next to the exe)
 │   └─ api\
 │       ├─ README.md
 │       └─ apidatatypexxl\             ← the Python API source (import from here)
 │           ├─ datatypexxl.py          ← the API module
 │           ├─ dttx_tokenizer.py + dttx_tokenizer.json
 │           ├─ engine.py, app.py, … (all source)
 │           └─ tests\, build.ps1, build.sh
 ├─ DataTypeXXL_v2.2.0_windows.zip      ← release package (exe + runtime)
 ├─ zippeddata.zip                      ← API source (same as derle\api)
 ├─ benchmark\                          ← proof pack (FLC, training, parity, dttx…)
 └─ data_type_xxl\
     ├─ data_type_xxl_source_code\      ← the working source tree
     ├─ .datatypexxl_deps\              ← on-demand dependencies (numpy, torch, …)
     ├─ DataTypeXXL.exe + DataTypeXXL_runtime\
     └─ zippeddata.zip
```

- **To run the app:** `derle\exe\DataTypeXXL.exe`
- **To use the API:** import from `derle\api\apidatatypexxl`
- **To build from source:** `data_type_xxl\data_type_xxl_source_code\build.ps1` (Python 3.14)
