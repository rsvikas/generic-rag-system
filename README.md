# Karnataka Legal RAG System - RunPod Build + PC Query

Current code is for Karnataka legal only. Not generic yet.

I am experimenting with Karnataka legal as a test subject. Legal is not the main product. The RAG system is the main product.
But in current code, only the legal test is implemented. Generic RAG for any subject is the future goal, not yet in code.

---

## What this repo actually does - from code

This is a 2-stage system. Do not skip Stage 1.

### Stage 1: Build Index on RunPod GPU

Run on RunPod GPU. Paths in code are `/workspace/...`.

1. `scripts/ocr_ai.py` - Reads PDFs, OCR + native text, fixes OCR with local LLM
   Output: `*_final.txt` in `/workspace/output_final`

2. `scripts/chunk_text.py` - Reads `*_final.txt`
   - Splits by: CHAPTER / Chapter / SECTION / Section / numbered clauses like `1.`
   - If that gives <=1 chunk, fallback: split by blank line paragraphs
   - Packs to MAX 1500 chars, MIN 500 chars
   Output: `*_chunks.txt` in `/workspace/chunks`, joined by `---CHUNK---`
   Note: This file does NOT save source or chunk_id. That is added in next file.

3. `scripts/embed_chunks.py` - Reads `*_chunks.txt`
   - Calls Ollama `http://localhost:11434/api/embeddings`, model `nomic-embed-text`
   - Retries 3 times, sleeps 1.5s after every chunk
   Output: `embeddings.jsonl` in `/workspace`
   Each line: `{text, embedding, source = file name, chunk_id = "i.j"}`

4. `scripts/build_faiss.py` - Reads `embeddings.jsonl`
   - Makes `faiss.IndexFlatL2`, adds vectors
   Output: `legal_index.faiss` + `legal_meta.json` in `/workspace`
   Note: FAISS only. No BM25. No hybrid.

Download those 2 files from RunPod to your PC.

### Stage 2: Ask Questions on PC

Run on PC. No GPU needed.

Put the 2 files in:
```
C:/legal_rag/
```

Then run:
```bash
python scripts/legal_rag.py
```

What `legal_rag.py` does (from earlier check):
- Loads `legal_index.faiss` + `legal_meta.json`
- Embeds your question, finds TOP_K 3 chunks
- Asks local LLM `mistral` with prompt: Legal Assistant, quote from Act, Sections, Provisos
- If not in PDFs, says: Not found in provided documents

---

## Test Data: Karnataka Legal

Test data is Karnataka property laws: KAOA, KAOR, KOFA, RERA, KCSA, TOPA, RDPR, IGR.

This test does NOT give legal advice. It only finds text from your PDFs.

---

## Hard-coded in code - what you must know

- Paths: `/workspace/...` for build, `C:/legal_rag` for query - hard-coded in each file
- Models: `phi4-mini` for correction, `nomic-embed-text` for embeddings, `mistral` for query - hard-coded
- Chunk size: MAX 1500, MIN 500 - hard-coded
- No `config.yaml`, no `requirements.txt`
- File names have `legal_` prefix: `legal_index.faiss`, `legal_meta.json`, `legal_rag.py`

To use a different subject today, you must edit 5 files. That is why it is not generic yet.

---

## Limitations - from code

- Chunking only understands legal headings. For college notes (Unit / Lesson), it will miss headings.
- Prompt always says Legal / Act, even if data is not legal.
- Build needs GPU for OCR correction + embeddings. Slow on CPU.
- Tables may lose visual structure.
- Always verify against original PDFs.

---

## Disclaimer

This tool only retrieves text from documents you give it. It does not give legal advice.

---

## Repo Map

- `generic-rag-system` (this repo) = Build on RunPod GPU, Use on PC. Legal code only for now.
- `generic-rag-colab` = Fully on Google Colab. Uses ready-made index. Legal code only for now.
