# Financial Document OCR Benchmark — Dashboard

Interactive benchmark dashboard comparing **8 OCR models** (5 local + 3 LLM-based) across **50 financial documents**, scored by 5 independent AI judges. 2,000 evaluations total.

## Results

| Rank | Model              | Avg Score | Type  | Notes                                        |
|------|--------------------|-----------|-------|----------------------------------------------|
| 🥇   | Claude Sonnet 4.6  | **88.9**  | LLM   | Best on degraded scans (86.1) and completeness (90.6) |
| 🥈   | GPT-5.4            | **86.6**  | LLM   | Best on SEC filings (97.1) and general docs (88.9) |
| 🥉   | GLM-OCR            | **84.5**  | Local | Best on receipts (91.0) and financial mixed (82.9) |
| 4th  | Gemini 3.1 Pro     | **79.8**  | LLM   | Strong text accuracy (85.9); low completeness (71.8) |
| 5th  | docTR              | **71.9**  | Local | Solid on receipts (88.2); weak on table structure |
| 6th  | Docling            | **63.4**  | Local | IBM pipeline; best local on general docs (74.8) |
| 7th  | Tesseract          | **60.9**  | Local | Reliable on clean text; collapses on SEC filings (53.2) |
| 8th  | EasyOCR            | **52.0**  | Local | Lowest on financial mixed (19.5) and degraded scans (28.6) |

## Key Findings

- **LLM-based models sweep the top 3** — Claude, GPT-5.4, and GLM-OCR lead all purely local engines by a large margin, confirming vision-language models understand document context rather than just pixel patterns
- **No single model dominates every category** — Claude leads on degraded scans (86.1), GPT-5.4 on SEC filings (97.1), GLM-OCR on receipts (91.0) and financial mixed (82.9)
- **Gemini 3.1 Pro underperforms vs Claude and GPT on completeness (71.8)** — strong text accuracy (85.9) but consistently misses content on longer documents
- **Numeric precision is the highest-scoring criterion for every model** — Claude reaches 95.4 and even EasyOCR scores 63.8, well above its 52.0 overall average
- **Degraded scans expose a 57-point spread** — EasyOCR drops to 28.6 while Claude holds at 86.1; the single largest performance differentiator
- **Financial mixed is the hardest category** — every model scores below its overall average; EasyOCR collapses to 19.5

## Category Breakdown

| Category         | Claude 4.6 | GPT-5.4 | GLM-OCR | Gemini 3.1 | docTR | Docling | Tesseract | EasyOCR |
|-----------------|-----------|---------|---------|-----------|-------|---------|-----------|---------|
| invoice_receipt | 89.0      | 87.0    | **91.0**| 79.9      | 88.2  | 74.7    | 80.6      | 70.3    |
| sec_filing      | 97.0      | **97.1**| 80.3    | 83.5      | 70.5  | 63.8    | 53.2      | 58.6    |
| financial_mixed | 81.0      | 79.2    | **82.9**| 67.1      | 67.4  | 42.7    | 51.7      | 19.5    |
| general_doc     | 88.1      | **88.9**| 84.1    | 86.7      | 67.3  | 74.8    | 64.9      | 61.7    |
| degraded_scan   | **86.1**  | 76.2    | 83.7    | 72.2      | 66.4  | 44.9    | 47.6      | 28.6    |

---

## File Structure

```
ui/                             ← serve this directory
├── index.html                  # Single-file dashboard (Bootstrap 5, dark/light theme)
├── benchmark_final.json        # Aggregated scores, rankings, category breakdown
├── extraction_summary.json     # Per-model per-doc timing and metadata
├── document_manifest.json      # 50 test document paths and categories
├── README.md                   # This file
│
├── outputs/                    # OCR extracted text — included in repo (~3 MB)
│   ├── glm_ocr/doc_001.txt … doc_050.txt
│   ├── doctr/
│   ├── docling/
│   ├── tesseract/
│   ├── easyocr/
│   ├── gemini_ocr/             # Gemini 3.1 Pro via OpenRouter
│   ├── gpt_ocr/                # GPT-5.4 via OpenRouter
│   └── claude_ocr/             # Claude Sonnet 4.6 via OpenRouter
│
├── evaluations/                # Per-judge JSON scores — included in repo (~9.5 MB)
│   ├── Gemini_Flash/           # {model}_{doc_id}.json  e.g. claude_ocr_doc_001.json
│   ├── Claude_Sonnet/
│   ├── GPT_4.1_Mini/
│   ├── Qwen_2.5_VL_72B/
│   └── DeepSeek_V3/
│
├── benchmark_docs/             # All 50 benchmark document images — included (~7 MB)
│   ├── sroie_000.jpg … sroie_009.jpg          # Receipt images (docs 1–10)
│   ├── multifinben_000.png … multifinben_009.png  # SEC filings (docs 11–20)
│   ├── sujet_000.png … sujet_004.png          # Financial mixed (docs 21–25)
│   ├── omni_0.png … omni_113.png              # General docs (docs 26–40)
│   └── degraded_000.jpg … degraded_009.jpg    # Degraded scans (docs 41–50, auto-generated)
│
└── datasets/                   # ⚠ NOT included — only needed to re-run extraction
    ├── sroie/data/img/
    ├── multifinben-images/
    ├── sujet-images/
    └── omni-ocr-benchmark/test/images/
```

> **Note:** All 50 document images are included in `benchmark_docs/` — the dashboard works
> fully out of the box with no additional downloads required.
> The `datasets/` folder is only needed if you want to re-run `run_extraction.py`.

---

## Running the Dashboard

Requires a local HTTP server (browsers block `fetch()` on `file://`):

```bash
cd ~/ocr-benchmark/ui
python3 -m http.server 8080
# Open http://localhost:8080
```

```bash
# Node.js alternative
npx serve ~/ocr-benchmark/ui
```

---

## Re-running the Benchmark

### Prerequisites

```bash
pip install pillow pytesseract easyocr doctr docling transformers torch requests
export OPENROUTER_API_KEY=sk-or-v1-xxxxxxxxxxxx   # required for LLM OCR + judges
```

### Step 1 — Extract text (8 models × 50 docs = 400 extractions)

```bash
cd ~/ocr-benchmark
python run_extraction.py
```

Local models (GLM-OCR, docTR, Docling, Tesseract, EasyOCR) run without an API key.
LLM models (Gemini 3.1 Pro, GPT-5.4, Claude Sonnet 4.6) require `OPENROUTER_API_KEY`.

### Step 2 — Evaluate outputs (5 judges × 8 models × 50 docs = 2,000 evaluations)

```bash
python run_evaluation.py
```

### Step 3 — Copy results to UI

```bash
cp benchmark_final.json extraction_summary.json document_manifest.json ui/
cp -r evaluations ui/evaluations
cp -r outputs ui/outputs
```

---

## Getting the Source Datasets (for re-extraction only)

The `datasets/` folder is not included. Place downloaded data at `~/ocr-benchmark/datasets/`.

### SROIE — Receipt images (docs 1–10)

```bash
pip install kaggle
kaggle datasets download -d dainis-boumber/sroie2019
unzip sroie2019.zip -d datasets/sroie
# Images at: datasets/sroie/data/img/000.jpg … 009.jpg
```

### MultiFin — SEC filing images (docs 11–20)

```bash
python3 -c "
from huggingface_hub import snapshot_download
snapshot_download(repo_id='nlpaueb/multifinben', repo_type='dataset',
                  local_dir='datasets/multifinben-images')
"
# Images at: datasets/multifinben-images/000.png … 009.png
```

### Sujet Finance — Financial mixed images (docs 21–25)

```bash
python3 -c "
from huggingface_hub import snapshot_download
snapshot_download(repo_id='sujet-ai/sujet-finance-10k', repo_type='dataset',
                  local_dir='datasets/sujet-images')
"
# Images at: datasets/sujet-images/000.png … 004.png
```

### Omni OCR Benchmark — General docs (docs 26–40)

```bash
python3 -c "
from huggingface_hub import snapshot_download
snapshot_download(repo_id='ucaslcl/GOT-OCR2_0', repo_type='dataset',
                  local_dir='datasets/omni-ocr-benchmark')
"
# Images at: datasets/omni-ocr-benchmark/test/images/0.png, 1.png, 100–113.png
```

---

## Benchmark Methodology

### Test Documents (50 total)

| Category         | Count | Source Dataset      | Doc IDs       |
|-----------------|-------|---------------------|---------------|
| invoice_receipt | 10    | SROIE               | doc_001–010   |
| sec_filing      | 10    | MultiFin            | doc_011–020   |
| financial_mixed | 5     | Sujet Finance       | doc_021–025   |
| general_doc     | 15    | Omni OCR Benchmark  | doc_026–040   |
| degraded_scan   | 10    | SROIE + degradation | doc_041–050   |

### OCR Models

| Model             | Type  | Implementation                          |
|------------------|-------|-----------------------------------------|
| Claude Sonnet 4.6 | LLM   | `anthropic/claude-sonnet-4.6` via OpenRouter |
| GPT-5.4           | LLM   | `openai/gpt-5.4` via OpenRouter         |
| GLM-OCR           | Local | `zai-org/GLM-OCR` open-source VLM, self-hosted |
| Gemini 3.1 Pro    | LLM   | `google/gemini-3.1-pro-preview` via OpenRouter |
| docTR             | Local | Deep learning OCR (Mindee)              |
| Docling           | Local | Document understanding pipeline (IBM)   |
| Tesseract         | Local | LSTM-based OCR engine (Google)          |
| EasyOCR           | Local | PyTorch multi-language OCR              |

### AI Judges

Each judge receives the source image + OCR text and scores 5 criteria (0–100):

| Judge             | Provider  | OpenRouter ID                        |
|------------------|-----------|--------------------------------------|
| Gemini 2.5 Flash | Google    | `google/gemini-2.5-flash`            |
| Claude Sonnet    | Anthropic | `anthropic/claude-sonnet-4`          |
| GPT-4.1 Mini     | OpenAI    | `openai/gpt-4.1-mini`                |
| Qwen 2.5 VL 72B  | Alibaba   | `qwen/qwen2.5-vl-72b-instruct`       |
| DeepSeek V3      | DeepSeek  | `deepseek/deepseek-chat-v3-0324`     |

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

- 400 extractions (8 models × 50 documents)
- 2,000 evaluations (400 × 5 judges)
- Evaluation date: April 13, 2026
