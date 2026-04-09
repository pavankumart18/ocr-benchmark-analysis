# Financial Document OCR Benchmark — Dashboard

Interactive benchmark dashboard comparing 5 OCR models across 25 financial documents, scored by 5 independent AI judges.

## Results

| Rank | Model      | Avg Score | Notes                                    |
|------|------------|-----------|------------------------------------------|
| 🥇   | GLM-OCR    | **85.1**  | Vision-language model; best on degraded scans |
| 🥈   | docTR      | **77.4**  | Strong on receipts and SEC filings       |
| 🥉   | Docling    | **63.9**  | IBM pipeline; tied with Tesseract        |
| 4th  | Tesseract  | **63.9**  | Unbeatable on clean typeset (SEC: 98.1)  |
| 5th  | EasyOCR    | **55.5**  | Collapses on degraded scans (16.1)       |

## Key Findings

- **GLM-OCR dominates** — leads by 7.7 points over docTR; vision-language context gives it an edge
- **Numeric precision is the bright spot** — even weaker models score higher on digits than characters
- **SEC filings are easy** — clean typeset pushes Tesseract to 98.1; all models score >74
- **Degraded scans separate the field** — EasyOCR: 16.1; GLM-OCR: 84.8; ~70 point spread
- **Financial mixed is the hardest category** — Tesseract drops to 31.0; even GLM-OCR only 69.4
- **Docling and Tesseract tie overall** but have complementary strengths by category

## Category Breakdown

| Category         | GLM-OCR  | docTR | Docling | Tesseract | EasyOCR |
|-----------------|----------|-------|---------|-----------|---------|
| invoice_receipt | 87.1     | 82.7  | 74.2    | 69.0      | 58.9    |
| sec_filing      | 96.7     | 95.8  | 74.5    | **98.1**  | 86.7    |
| financial_mixed | 69.4     | 68.8  | 52.9    | 31.0      | 45.7    |
| general_doc     | 87.0     | 65.9  | 72.8    | 67.6      | 61.5    |
| degraded_scan   | **84.8** | 71.9  | 38.0    | 49.7      | 16.1    |

---

## File Structure

```
ui/                             ← serve this directory
├── index.html                  # Single-file dashboard (Bootstrap 5, dark/light theme)
├── benchmark_final.json        # Aggregated scores, rankings, category breakdown
├── extraction_summary.json     # Per-model per-doc timing and metadata
├── document_manifest.json      # 25 test document paths and categories
├── README.md                   # This file
│
├── outputs/                    # OCR extracted text — included in repo (~1.1 MB)
│   ├── glm_ocr/doc_001.txt … doc_025.txt
│   ├── doctr/
│   ├── docling/
│   ├── tesseract/
│   └── easyocr/
│
├── evaluations/                # Per-judge JSON scores — included in repo (~3 MB)
│   ├── Gemini_Flash/           # {model}_{doc_id}.json  e.g. glm_ocr_doc_001.json
│   ├── Claude_Sonnet/
│   ├── GPT_4.1_Mini/
│   ├── Qwen_2.5_VL_72B/
│   └── DeepSeek_V3/
│
├── benchmark_docs/             # Degraded scan images — included in repo (~204 KB)
│   ├── degraded_000.jpg … degraded_003.jpg
│
└── datasets/                   # ⚠ NOT included — download manually (5.3 GB total)
    ├── sroie/data/img/         # Receipt images (.jpg)
    ├── multifinben-images/     # SEC filing images (.png)
    ├── sujet-images/           # Financial doc images (.png)
    └── omni-ocr-benchmark/test/images/  # General doc images (.png)
```

> **Note:** The dashboard works fully without the `datasets/` folder. Document images for
> receipts, SEC filings, financial mixed, and general docs will show a placeholder.
> Degraded scan images are included in `benchmark_docs/` and always display.

---

## Getting the Source Datasets

The `datasets/` folder is not included due to size (~5.3 GB). Place downloaded data at
`ui/datasets/` (or `~/ocr-benchmark/datasets/`) so the dashboard can display source images.

### 1. SROIE — Receipt images (invoice_receipt, docs 1–6)

```bash
# Kaggle dataset: dainis-boumber/sroie2019
pip install kaggle
kaggle datasets download -d dainis-boumber/sroie2019
unzip sroie2019.zip -d datasets/sroie
# Images expected at: datasets/sroie/data/img/*.jpg
```

Or download manually from:  
https://www.kaggle.com/datasets/dainis-boumber/sroie2019

The benchmark uses images: `000.jpg`, `001.jpg`, `002.jpg`, `003.jpg`, `004.jpg`, `005.jpg`

---

### 2. MultiFin / multifinben — SEC filing images (sec_filing, docs 7–11)

```bash
# HuggingFace dataset: multilingual financial document images
pip install huggingface_hub
python3 - <<'EOF'
from huggingface_hub import snapshot_download
snapshot_download(
    repo_id="nlpaueb/multifinben",
    repo_type="dataset",
    local_dir="datasets/multifinben-images"
)
EOF
# Images expected at: datasets/multifinben-images/*.png  (000.png … 004.png)
```

Or browse at: https://huggingface.co/datasets/nlpaueb/multifinben

---

### 3. Sujet Finance — Financial mixed images (financial_mixed, docs 12–16)

```bash
# HuggingFace dataset: sujet-ai/Sujet-Finance-Instruct-177k or sujet finance 10-K images
python3 - <<'EOF'
from huggingface_hub import snapshot_download
snapshot_download(
    repo_id="sujet-ai/sujet-finance-10k",
    repo_type="dataset",
    local_dir="datasets/sujet-images"
)
EOF
# Images expected at: datasets/sujet-images/*.png  (000.png … 004.png)
```

Or browse at: https://huggingface.co/datasets/sujet-ai/sujet-finance-10k

---

### 4. Omni OCR Benchmark — General document images (general_doc, docs 17–21)

```bash
# HuggingFace dataset: omni-ocr benchmark test split
python3 - <<'EOF'
from huggingface_hub import snapshot_download
snapshot_download(
    repo_id="ucaslcl/GOT-OCR2_0",   # or the omni-ocr-benchmark source
    repo_type="dataset",
    local_dir="datasets/omni-ocr-benchmark"
)
EOF
# Images expected at: datasets/omni-ocr-benchmark/test/images/0.png, 1.png, 100.png, 101.png, 103.png
```

Alternatively, the exact images used are referenced in `document_manifest.json` — match filenames
to the test split of whichever OCR benchmark dataset you source.

---

### Directory layout after downloading

```
ui/datasets/
├── sroie/
│   └── data/img/
│       ├── 000.jpg
│       ├── 001.jpg
│       ├── 002.jpg
│       ├── 003.jpg
│       ├── 004.jpg
│       └── 005.jpg
├── multifinben-images/
│   ├── 000.png
│   ├── 001.png
│   ├── 002.png
│   ├── 003.png
│   └── 004.png
├── sujet-images/
│   ├── 000.png … 004.png
└── omni-ocr-benchmark/
    └── test/images/
        ├── 0.png
        ├── 1.png
        ├── 100.png
        ├── 101.png
        └── 103.png
```

Once in place, refresh the dashboard — thumbnails and full-size images will load automatically.
The degraded scan images (`benchmark_docs/`) are already included and require no extra setup.

---

## Running the Dashboard

Requires a local HTTP server (browsers block `fetch()` on `file://`):

```bash
# Python — serve from the ui/ directory
cd ~/ocr-benchmark/ui
python3 -m http.server 8080
# Open http://localhost:8080

# Node.js
npx serve ~/ocr-benchmark/ui

# VS Code Live Server
# Right-click index.html → "Open with Live Server"
```

---

## Benchmark Methodology

### Test Documents (25 total)

| Category         | Count | Source Dataset      | Doc IDs       |
|-----------------|-------|---------------------|---------------|
| invoice_receipt | 6     | SROIE               | doc_001–006   |
| sec_filing      | 5     | MultiFin            | doc_007–011   |
| financial_mixed | 5     | Sujet Finance       | doc_012–016   |
| general_doc     | 5     | Omni OCR Benchmark  | doc_017–021   |
| degraded_scan   | 4     | SROIE + degradation | doc_022–025   |

### OCR Models

| Model      | Type                                   |
|-----------|----------------------------------------|
| GLM-OCR   | Vision-language model (GLM-4V) via API |
| docTR     | Deep learning OCR (Mindee)             |
| Docling   | Document understanding pipeline (IBM)  |
| Tesseract | LSTM-based OCR engine (Google)         |
| EasyOCR   | PyTorch multi-language OCR             |

### AI Judges

Each judge receives the source image + OCR text and scores 5 criteria (0–100):

| Judge             | Provider  |
|------------------|-----------|
| Gemini 2.5 Flash | Google    |
| Claude Sonnet    | Anthropic |
| GPT-4.1 Mini     | OpenAI    |
| Qwen 2.5 VL 72B  | Alibaba   |
| DeepSeek V3      | DeepSeek  |

### Scoring Rubric

| Criterion            | Weight | What is measured                      |
|---------------------|--------|---------------------------------------|
| text_accuracy       | 30%    | Character-level fidelity to source    |
| table_structure     | 25%    | Tables, columns, rows preserved       |
| numeric_precision   | 20%    | Numbers, dates, amounts correct       |
| layout_preservation | 15%    | Reading order and formatting          |
| completeness        | 10%    | All text captured, no omissions       |

Final score = weighted sum of 5 criteria, averaged across all 5 judges.

### Scale

- 125 extractions (5 models × 25 documents)
- 625 evaluations (125 × 5 judges)
- Evaluation date: April 8, 2026
