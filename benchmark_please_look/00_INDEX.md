# Data Type XXL — Benchmark & Proof Pack (v2.0.0)

Everything that can be proven is here, produced by REAL execution (nothing fabricated).

| File | What it proves | Result |
|---|---|---|
| `01_FLC_proof.txt` | FLC collector works | **604 sources** (coder / commit / commoncrawl / wiki / stackoverflow ...) |
| `02_feature_health.json` | All feature modules are sound | doctor **7 pass / 0 fail**, **22/22 modules import** |
| `03_dataset_proof.txt` | Datasets are really produced | real ingest → standard `_index.jsonl` format, tokenized |
| `04_parity_proof.txt` | API = exe = zip | **123 .py byte-identical (SHA-256)**, versions match |
| `05_benchmark_results.json` | Performance (measured) | import 0.11 s, repair ~15.6 MB/s, parse ~985 pages/s |
| `06_training_proof.txt` | From-scratch training learns | **loss 66.4 → 3.25** (not a placeholder) |
| `DataTypeXXL_benchmark.png` | Visual summary | for preview / social preview |
| `CHECKSUMS.txt` | Download integrity | SHA-256 verification |
| `SECURITY.md` | Security policy | no telemetry, signing, honest scope |
| `BENCHMARKS.md` | Detailed benchmarks | table + loss steps |

## Overall result
- **Data collection:** frontier-scale architecture (604 sources, Common Crawl, 371 languages) — works.
- **Data processing:** ingest / cleaning / token repair / dedup — works, measured.
- **From-scratch training:** the model really learns (loss 66 → 3.25).
- **Integrity:** API / exe / zip are byte-identical, 81 tests passing.
- **Trust:** checksums + signing + open source (you can build it yourself).

## Honest environment note
Windows 11 · Python 3.14.6 · RTX 4060 (driver 581.57) but **CPU-only PyTorch** (no CUDA).
Every number here is a **CPU** number; GPU performance is higher and is **not claimed**.

