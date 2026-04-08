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

| Category         | GLM-OCR | docTR | Docling | Tesseract | EasyOCR |
|-----------------|---------|-------|---------|-----------|---------|
| invoice_receipt | 87.1    | 82.7  | 74.2    | 69.0      | 58.9    |
| sec_filing      | 96.7    | 95.8  | 74.5    | **98.1**  | 86.7    |
| financial_mixed | 69.4    | 68.8  | 52.9    | 31.0      | 45.7    |
| general_doc     | 87.0    | 65.9  | 72.8    | 67.6      | 61.5    |
| degraded_scan   | **84.8**| 71.9  | 38.0    | 49.7      | 16.1    |

## File Structure

```
ui/
├── index.html              # Single-file dashboard (Bootstrap 5, dark mode)
├── benchmark_final.json    # Aggregated scores, rankings, category breakdown
├── extraction_summary.json # Per-model per-doc timing and metadata
├── document_manifest.json  # 25 test documents with paths and categories
└── README.md               # This file

../outputs/                 # OCR text outputs (loaded via ../outputs/{model}/{doc_id}.txt)
├── glm_ocr/
├── doctr/
├── docling/
├── tesseract/
└── easyocr/

../evaluations/             # Per-judge JSON scores (loaded via ../evaluations/{judge}/{model}_{doc_id}.json)
├── Gemini_Flash/
├── Claude_Sonnet/
├── GPT_4.1_Mini/
├── Qwen_2.5_VL_72B/
└── DeepSeek_V3/

../datasets/                # Source images (loaded relative to ui/)
├── sroie/data/img/         # Receipt images (.jpg)
├── multifinben-images/     # SEC filing images (.png)
├── sujet-images/           # Financial doc images (.png)
├── omni-ocr-benchmark/test/images/  # General doc images (.png)
└── benchmark_docs/         # Degraded scan images (.jpg)
```

## Running the Dashboard

The dashboard loads data via `fetch()` and requires a local HTTP server (browsers block `fetch` on `file://`):

```bash
# Python (simplest)
cd ~/ocr-benchmark/ui
python3 -m http.server 8080
# Open http://localhost:8080

# Node.js
npx serve ~/ocr-benchmark/ui

# VS Code
# Install "Live Server" extension, right-click index.html → "Open with Live Server"
```

## Benchmark Methodology

### Test Documents (25 total)

| Category         | Count | Source Dataset         |
|-----------------|-------|------------------------|
| invoice_receipt | 6     | SROIE                  |
| sec_filing      | 5     | MultiFin               |
| financial_mixed | 5     | Sujet Finance          |
| general_doc     | 5     | Omni OCR Benchmark     |
| degraded_scan   | 4     | SROIE + added degradation |

### OCR Models

| Model      | Type                         |
|-----------|------------------------------|
| GLM-OCR   | Vision-language model (GLM-4V) via API |
| docTR     | Deep learning OCR (Mindee)   |
| Docling   | Document understanding pipeline (IBM) |
| Tesseract | LSTM-based OCR engine (Google) |
| EasyOCR   | PyTorch multi-language OCR   |

### AI Judges

Each judge receives the source image + OCR text and scores 5 criteria (0–100):

| Judge              | Provider   |
|-------------------|------------|
| Gemini 2.5 Flash  | Google     |
| Claude Sonnet     | Anthropic  |
| GPT-4.1 Mini      | OpenAI     |
| Qwen 2.5 VL 72B   | Alibaba    |
| DeepSeek V3       | DeepSeek   |

### Scoring Rubric

| Criterion           | Weight | Description                           |
|--------------------|--------|---------------------------------------|
| text_accuracy      | 30%    | Character-level fidelity              |
| table_structure    | 25%    | Tables, columns, rows preserved       |
| numeric_precision  | 20%    | Numbers, dates, amounts correct       |
| layout_preservation| 15%    | Reading order, formatting             |
| completeness       | 10%    | All text captured (no omissions)      |

Final score = weighted sum of 5 criteria, averaged across all 5 judges.

### Scale

- 125 extractions (5 models × 25 documents)
- 625 evaluations (125 × 5 judges)
- Evaluation date: April 8, 2026
