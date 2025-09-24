# BioBART + LoRA for Medical QA (RAG on CPU) — WIP

**Short description:** Merge **MedMCQA** + **LiveQA** into one cleaned JSONL, fine-tune **`GanjinZero/biobart-base`** with **LoRA** on ~5k samples (compute-limited), and answer questions with **RAG** (BM25 → MiniLM → CrossEncoder) on **CPU**.

> **Status:** Work in progress — some cells still error (generation kwargs, prompt echo, slow indexing). I’m actively fixing and improving.

---

## Overview

- **Preprocess** datasets in `notebooks/Cleaner.ipynb` → unified JSONL.  
- **Train** BioBART with **LoRA** on ~5k samples → save **adapter only**.  
- **Serve** answers via **RAG** over the unified corpus on CPU.  
- *(Optional)* entailment (NLI) gate to accept only evidence-supported answers.

> **Medical disclaimer:** Research/education only. Not medical advice.

---

## Data: Merge & Preprocess (Cleaner)

Notebook: `notebooks/Cleaner.ipynb`

1. Load **MedMCQA** and **LiveQA**.  
2. Normalize to a single schema:  
   `id`, `question`, `answer`, `options` (optional), `explanation` (optional), `source` (optional).  
3. Clean text (strip HTML/refs, collapse whitespace).  
4. Write unified file: `data/merged_corpus_cleaned_Final.jsonl`.

---

## Training (LoRA)

Notebook: `notebooks/Bio_bart__Training.ipynb`

- **Base:** `GanjinZero/biobart-base`  
- **Sample:** ~5,000 rows (random, compute-limited)  
- **X (input):** `question` (+ `options` when MCQ)  
- **Y (target):** `answer` (and/or brief explanation, per notebook)  
- **Procedure:** tokenize → seq2seq fine-tune with **PEFT/LoRA** → validate → **save adapter only**:
  - `adapter_config.json`
  - `adapter_model.safetensors` (or `.bin`)

At inference, load the same base model and attach this adapter.

---

## Inference (RAG)

Notebook: `notebooks/bio_bartipynb.ipynb`

1. **Load model:** base BioBART + attach LoRA adapter (PEFT), `eval()` on **CPU**.  
2. **Build sentence index** from the unified JSONL:  
   - **BM25** → keyword shortlist  
   - **MiniLM** → semantic rerank  
   - **CrossEncoder** → precise final rerank  
   - *(Optional)* **NLI/entailment** check  
3. **Answer flow:** detect question type (Yes/No, Compare, General) → retrieve top sentences → build a short prompt with evidence → generate a concise, clinically safe answer.

**RAG in one line:** retrieve evidence → place it in the prompt → generate the answer.

---

