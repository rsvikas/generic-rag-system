# Generic Offline RAG System

Build your own Knowledge DB from your own PDFs. Offline. No cloud. No API key.

This is **not** a legal tool.

Karnataka legal laws are used here only as **one test example** to check if the tool works.
Tomorrow you can use the same tool with medical books, college notes, company manuals, or any other PDFs.

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

## How it works

```
Your PDFs
  ↓
1. Extract text (OCR + native text)
  ↓
2. Fix OCR errors with local LLM
  ↓
3. Split into small chunks
  ↓
4. Make embeddings (local)
  ↓
5. Save in FAISS index
  ↓
6. Ask question -> Find best chunks -> Local LLM answers only from those chunks
```

If answer is not in your PDFs, it says:

> Not found in provided documents

It does not guess. It does not use outside knowledge.

---

## Use it with your own subject - 3 steps

### Step 1: Put your PDFs

```
data/<your-subject>/your-files.pdf
```

Example for test:
```
data/karnataka-legal/*.pdf
```

### Step 2: Edit config

Open `config.yaml` and change only these 3 lines:

```yaml
subject_name: "karnataka-legal"
docs_folder: "data/karnataka-legal"
answer_rule: "Quote verbatim. Use only retrieved text."
```

For your own subject, change to:

```yaml
subject_name: "my-college-notes"
docs_folder: "data/my-college-notes"
```

### Step 3: Run pipeline

```bash
python scripts/extract_text.py
python scripts/chunk_text.py
python scripts/embed_chunks.py
python scripts/build_index.py
python scripts/query.py
```

That's it. Same tool, new Knowledge DB.

---

## Pipeline Details

### 1. Extract Text
- Handles scanned PDFs, image + text pages, tables, portrait + landscape
- Tools: `pdfplumber`, `pytesseract`, OpenCV
- Script: `scripts/extract_text.py`

### 2. Fix OCR Errors (Offline LLM)
- Fixes common OCR errors like `rn -> m`, `cl -> d`, broken words
- Uses local LLM only. No OpenAI / Gemini API.
- Script: `scripts/extract_text.py`

### 3. Chunk Text
- Splits by headings, paragraphs, then token limit
- Keeps structure, so retrieval stays correct
- No hard-coded legal sections. Separators come from `config.yaml`
- Script: `scripts/chunk_text.py`

### 4. Make Embeddings (Local)
- Uses `nomic-embed-text` via Ollama, fully offline
- One embedding per chunk
- Script: `scripts/embed_chunks.py`

### 5. Build FAISS Index
- Stores embeddings in FAISS locally
- Metadata (chunk text + source file) stored separately
- Script: `scripts/build_index.py`
- Outputs (not pushed to Git):
  - `legal_index.faiss` -> renamed to `index.faiss`
  - `legal_meta.json` -> renamed to `meta.json`

### 6. Ask Questions (RAG)
- Your question -> embedding -> FAISS search -> best chunks -> local LLM
- LLM rule: Use only retrieved text. Quote verbatim. No guessing.
- Script: `scripts/query.py`

---

## Technologies Used

- Python 3.10+
- Tesseract OCR, OpenCV, pdfplumber
- FAISS
- Ollama (local LLM runner)
- nomic-embed-text
- Open-source local LLMs only

---

## Why Offline & Open-Source?

- No API rate limits
- No data leakage - your PDFs never leave your computer
- Full control over embeddings and answers
- Reproducible
- Cheap for large document sets

---

## Project Status

- Text extraction: ✅ Complete
- OCR correction: ✅ Complete
- Chunking: ✅ Complete
- Embeddings: ✅ Complete
- FAISS index: ✅ Complete
- Query / RAG: 🧪 Testing / tuning

---

## Limitations

- Answer quality depends on scan quality
- Tables may lose some visual structure
- Tool finds text, it does not interpret or advise
- Always verify against your original PDFs

---

## Disclaimer

This tool only retrieves text from documents you give it.

It does not give legal advice, medical advice, or any professional advice.

For the Karnataka legal test: verify all answers against official Acts.

---

## Repo Map

- `legal-rag-system` (this repo) = Main engine. Generic. Works with any subject.
- `karnataka-legal-rag-assistant` = Colab demo of this engine, using Karnataka legal as test data.
