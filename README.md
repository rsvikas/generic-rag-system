# Generic Offline RAG System

Build your own Knowledge DB from your own PDFs. Offline. No cloud. No API key.

This is **not** a legal tool.

Karnataka legal laws are used here only as **one test example** to check if the tool works.
Tomorrow you can use the same tool with medical books, college notes, company manuals, or any other PDFs.

---

## How to use this repo - 2 Stages

This repo has **2 stages**. Do not skip Stage 1.

### Stage 1: Build Index on RunPod GPU

Run these on **RunPod GPU** (RTX 4090 recommended). Building needs GPU.

Why GPU? OCR correction + embeddings are slow on CPU.

```bash
python scripts/ocr_ai.py
python scripts/chunk_text.py
python scripts/embed_chunks.py
python scripts/build_faiss.py
```

Paths in Stage 1 are `/workspace/...` - that is RunPod.

Outputs from Stage 1:
- `legal_index.faiss`
- `legal_meta.json`

Download these 2 files from RunPod to your PC.

### Stage 2: Ask Questions Locally on PC

Run this on your **PC**, no GPU needed.

Put the 2 files from Stage 1 into:
```
C:/legal_rag/
```

Then run:
```bash
python scripts/legal_rag.py
```

Ask your question. It answers only from your PDFs.

---

## What does it do?

You give it PDFs. It reads them. It answers your questions **only from those PDFs**.

Example:
- Put Karnataka property laws PDFs in -> Ask about Karnataka property law
- Put college notes PDFs in -> Ask about your college notes
- Put company manual PDFs in -> Ask about your company manual

Same tool. Different data.

---

## Test Example Included: Karnataka Legal

We test this tool with Karnataka property laws like KAOA, KAOR, KOFA, RERA, KCSA, TOPA, RDPR, IGR.

Why this test?
- Legal PDFs are hard: scanned pages, tables, mixed portrait + landscape, OCR spelling errors.
- If the tool works here, it will work for simpler PDFs too.

This test does **not** give legal advice. It only finds text from your PDFs.

---

## Pipeline

```
Your PDFs (RunPod)
  ↓
1. Extract text (OCR + native text) - scripts/ocr_ai.py - RunPod
  ↓
2. Fix OCR errors with local LLM - scripts/ocr_ai.py - RunPod
  ↓
3. Split into small chunks - scripts/chunk_text.py - RunPod
  ↓
4. Make embeddings (local) - scripts/embed_chunks.py - RunPod
  ↓
5. Save in FAISS index - scripts/build_faiss.py - RunPod
  ↓
Download index.faiss + meta.json to PC
  ↓
6. Ask question -> Find best chunks -> Local LLM answers only from those chunks - scripts/legal_rag.py - PC
```

If answer is not in your PDFs, it says:

> Not found in provided documents

It does not guess. It does not use outside knowledge.

---

## Technologies Used

- Python 3.10+
- Tesseract OCR, OpenCV, pdfplumber
- FAISS
- Ollama (local LLM runner)
- nomic-embed-text
- Open-source local LLMs only (phi4-mini for correction, mistral for query)

---

## Why Offline & Open-Source?

- No API rate limits
- No data leakage - your PDFs never leave your computer / RunPod
- Full control over embeddings and answers
- Reproducible
- Cheap for large document sets

---

## Current Status

- Text extraction: ✅ Complete
- OCR correction: ✅ Complete
- Chunking: ✅ Complete
- Embeddings: ✅ Complete
- FAISS index: ✅ Complete
- Query / RAG: 🧪 Testing / tuning

Note: Current code is tuned for the Karnataka legal test (prompts mention legal, chunking looks for CHAPTER / SECTION). The concept is generic, code will be made fully generic in next update.

---

## Limitations

- Answer quality depends on scan quality
- Tables may lose some visual structure
- Tool finds text, it does not interpret or advise
- Always verify against your original PDFs
- This repo builds only FAISS index. For FAISS + BM25 hybrid index used in Colab repo, you need an extra hybrid build step.

---

## Disclaimer

This tool only retrieves text from documents you give it.

It does not give legal advice, medical advice, or any professional advice.

For the Karnataka legal test: verify all answers against official Acts.

---

## Repo Map

- `legal-rag-system` (this repo) = Build on RunPod GPU, Use on PC. Main pipeline.
- `karnataka-legal-rag-assistant` = Fully on Google Colab. Uses ready-made index. No build step.
