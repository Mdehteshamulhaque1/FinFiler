# FinFiler

A Q&A assistant for financial documents (loan agreements, credit card terms, insurance
policies). Upload a document, ask questions in plain language, get answers with the exact
clause and page number.

## Problem

Most people sign financial documents without reading them. The important details are buried
on page 14 in dense legal language: prepayment penalties, late fees, floating-rate clauses,
processing charges. People find out about them only when they get charged.

## Users

- Individuals taking a home, car, or education loan
- Students applying for education loans
- Freelancers and small business owners comparing credit products
- Anyone checking credit card terms before applying

## MVP features

- Upload documents (PDF; scanned PDF via OCR is optional and not in the initial build)
- Ask questions in plain language ("What is the foreclosure charge?", "What happens if I miss an EMI?")
- Cited answers: the answer, the clause text, the document name, and the page number
- "Not found" handling: if the document does not mention it, say so instead of guessing
- Multi-document comparison: upload two loan offers and ask "Which one has lower processing fees?"

## Differentiators

- Key terms summary: extract interest rate, tenure, fees and penalties into a table
- Red flag detector: highlight high penalties, variable-rate conditions, etc.
- Simple explanation mode: explain a clause in plain language while showing the original text
- Numeric answers: extract values and compute in code, not in the model

## Architecture

- Ingestion: PyMuPDF parsing, keep page numbers with every chunk
- Chunking: split by clause or section where possible, not fixed size
- Embeddings: sentence-transformers; vectors in Qdrant; metadata in PostgreSQL
- Retrieval: hybrid search (semantic + keyword/BM25) because exact terms matter
- Generation: LLM answers only from retrieved text and always cites
- Citation check: verify the cited page actually contains the supporting text
- API and UI: FastAPI backend, Streamlit frontend

## Evaluation

- Retrieval hit rate: correct page in top 3 results
- Answer correctness against hand-written answers
- Citation accuracy: cited page really supports the answer
- Refusal accuracy: says "not found" for out-of-scope questions
- Baseline comparison: semantic vs hybrid vs hybrid + rerank, with numbers for each

## Known challenges

- Tables: fee schedules parse badly; document the trade-offs
- Scanned documents: need OCR, added complexity
- Numbers and math: extract values and compute in code
- Cross-references: "subject to Section 8.2" needs more than one chunk

## Layout

```
finfiler/
├── AGENTS.md
├── .gitignore
├── backend/
├── frontend/
├── evaluation/
└── sample_docs/
```

## Rules for all code

- Backend: Python 3.11, FastAPI, Pydantic v2, SQLAlchemy 2.0 + Alembic, PostgreSQL (metadata, chunk text, and full-text keyword search using tsvector + GIN index), Qdrant (vectors), sentence-transformers, PyMuPDF (text + page numbers), pdfplumber (tables only).
- Frontend: Streamlit, calling the FastAPI backend over HTTP with the requests library. No business logic in the frontend.
- RAG orchestration: custom Python pipeline, no LangChain or LlamaIndex.
- OCR (Tesseract) is optional and NOT part of the initial build.
- Deployment: Docker + Render.
