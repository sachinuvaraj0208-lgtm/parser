<!-- Before pushing: replace every ⟨placeholder⟩ with a measured value, then delete this comment. -->

<div align="center">

# 🔍 ParseAnything

**Every page parsed. Every number traceable. Nothing hallucinated.**

A universal document parser that turns PDFs, scans, Office files, spreadsheets and emails into structured Markdown + JSON — with page, bounding box and confidence on every block.



<img src="docs/assets/hero.gif" alt="Click a number, see its source" width="800"/>

</div>

---

## Problem

In finance and legal work, one missed line item can cost a deal. Existing parsers return empty output on scans, flatten tables, drop charts and equations, scramble columns, and give **no way to verify where an answer came from.**

## Solution

One entry point. Four stages:

```
DETECT & ROUTE  →  EXTRACT & ASSEMBLE  →  VERIFY  →  FLAG & FAIL-SAFE
(per page/region)  (typed blocks, order,   (sum-checks,  (confidence per block,
                    cross-page tables)      chart re-plot) never invents text)
```

## Key features

- **Provenance Viewer** — click any output block to see its exact bounding box on the source page
- **Self-verifying tables** — totals are recomputed; mismatches are flagged and re-read automatically
- **Chart → data** — series extracted and re-plotted over the original as proof
- **Equations → LaTeX**, multi-column reading order, cross-page table stitching
- **Refuses to guess** — low-confidence regions are withheld and flagged, not hallucinated
- **Cost-aware ladder** — native → CPU OCR → specialist models → vision-LLM, escalating only when needed
- **Benchmark Arena** — compare against baselines with the same metrics
- **Self-hostable** — one `docker compose up`; permissive-license core; optional offline VLM

**Formats:** PDF (digital/scanned) · images · DOCX/PPTX · XLSX/CSV · legacy DOC/XLS/PPT · EML/MSG (with attachments)

## Quick start

```bash
git clone ⟨repo-url⟩ && cd parseanything
cp .env.example .env        # set GEMINI_API_KEY, or VLM_PROVIDER=ollama | none
docker compose up --build
```

- App: http://localhost:3000 → drop files or click **Try the Torture Test**
- API docs: http://localhost:8000/docs

```bash
curl -F "file=@samples/torture_test/debt_schedule_scan.pdf" \
     "http://localhost:8000/v1/parse?sync=true"
```

## Output

Every block carries provenance and confidence:

```json
{ "id": "b_0008", "type": "table", "page": 3, "spans_pages": [3, 4],
  "bbox": [0.08, 0.16, 0.92, 0.88], "confidence": 0.91,
  "table": { "checks": [{ "type": "column_sum", "expected": 48210, "found": 48210, "ok": true }] } }
```

Confidence `< 0.50` → flagged `needs_review`; `< 0.30` → text withheld, source crop attached.

## Fail-safe

Corrupt, encrypted, empty or unsupported files return a structured error within the 60-second budget — never a crash or silent empty output:

```json
{ "status": "error", "error": { "code": "E_CORRUPT_FILE", "stage": "preflight", "elapsed_ms": 412 } }
```

## Benchmarks

Measured on ⟨corpus, N docs⟩, ⟨hardware⟩. Reproduce: `python -m eval.runner --corpus gold --system all`

| Metric | pypdf | pdfplumber | **ParseAnything** |
|---|---|---|---|
| Text fidelity ↑ | ⟨…⟩ | ⟨…⟩ | **⟨…⟩** |
| Reading order ↑ | ⟨…⟩ | ⟨…⟩ | **⟨…⟩** |
| Table structure (TEDS) ↑ | — | ⟨…⟩ | **⟨…⟩** |
| Bounding-box IoU ↑ | none | none | **⟨…⟩** |
| Chart value error ↓ | n/a | n/a | **⟨…⟩** |
| Pages / sec ↑ | ⟨…⟩ | ⟨…⟩ | ⟨…⟩ |
| **Cost per 1k pages** | — | — | **⟨$X.XX⟩** |

## Architecture

```
Next.js viewer ⇄ FastAPI + SSE ⇄ Orchestrator (page-parallel, 55 s deadline)
                                   ├─ Format adapters (pdf, image, office, sheets, email)
                                   ├─ Extractors (text, OCR, tables, charts, math)
                                   ├─ Verifier (sum-checks, re-plot, self-heal)
                                   └─ VLM gateway (Gemini | Ollama, cached, budget-guarded)
```

More in [`docs/`](docs/).

## Limitations

- Handwriting and non-Latin scripts are outside our accuracy claims (low-confidence output is flagged).
- 3D or heavily stylized charts are flagged rather than digitized.
- Vision-LLM features degrade gracefully to non-VLM paths if the provider is unavailable.

## Team & license

⟨Name⟩ · ⟨Name⟩ · ⟨Name⟩ · ⟨Name⟩ — released under the ⟨MIT / Apache-2.0⟩ License.

Built for DataQuest 3.0 on the Cenizas Labs problem statement *ParseAnything: High-Fidelity Universal Document Ingestion Engine*.
