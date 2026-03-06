# 📊 StrategyAI — RAG Chatbot for Business Strategy

**Team 12 · Advanced NLP & AI Applications · March 2025**

---

## 👥 Team & Contributions

| # | Name | Contribution |
|---|------|---|
| 1 | **Akansha** | Core RAG pipeline design & implementation · Multi-query retrieval with RRF · ChromaDB integration & chunk deduplication · Flask backend API · Browser frontend (`strategyai.html`) · Follow-up query resolver · Notebook architecture · System integration & testing |
| 2 | **Manasvi** | Document corpus curation & preprocessing · Chunking strategy evaluation & comparison · Technical writeup drafting |
| 3 | **Yijia** | Evaluation methodology & benchmark question set · Ablation testing (single-query vs. multi-query) · Results analysis & metrics reporting |

---

A production-ready Retrieval-Augmented Generation chatbot over 50+ business strategy documents, featuring multi-query retrieval with Reciprocal Rank Fusion, persistent conversation memory, and a browser-native frontend served from a Jupyter notebook.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────┐
│             Jupyter Notebook                │
│                                             │
│  ┌──────────────┐    ┌────────────────────┐ │
│  │  Flask API   │◄───│   ask() / RAG      │ │
│  │  (bg thread) │    │   pipeline         │ │
│  └──────┬───────┘    └────────┬───────────┘ │
│         │                     │             │
│         │            ┌────────▼───────────┐ │
│         │            │  ChromaDB          │ │
│         │            │  (HNSW, cosine)    │ │
│         │            └────────────────────┘ │
└─────────┼───────────────────────────────────┘
          │ HTTP (localhost:5050)
          ▼
┌─────────────────────┐
│  strategyai.html    │  ← Opens in real browser tab
│  (HTML/JS frontend) │
└─────────────────────┘
```

## ⚙️ Tech Stack

| Layer | Technology |
|---|---|
| LLM | Google Gemini 2.5 Flash |
| Embeddings | `text-embedding-004` (768-dim) |
| Vector DB | ChromaDB (HNSW index, cosine similarity, persistent storage) |
| Backend | Flask (background thread in notebook) |
| Frontend | Standalone HTML/JS (browser-native, no framework) |
| Retrieval | Multi-Query + Reciprocal Rank Fusion (RRF) |

---

## 🚀 Quickstart

### 1. Install dependencies

```bash
pip install flask flask-cors google-generativeai chromadb
```

### 2. Set your API key

```python
import os
os.environ["GOOGLE_API_KEY"] = "your-key-here"
```

### 3. Run the notebook

Open `StrategyAI.ipynb` and run all cells in order:

| Cell | What it does |
|---|---|
| 1 — Ingest | Chunks and embeds all documents into ChromaDB |
| 2 — RAG pipeline | Defines `ask()` with multi-query RRF retrieval |
| 3 — Flask server | Starts API on `localhost:5050` in a background thread |
| 4 — Launch UI | Opens `strategyai.html` in your browser |

### 4. Place the frontend

Make sure `strategyai.html` is in the **same folder** as the notebook.

---

## 🔍 How Retrieval Works

```
User query
    │
    ▼
┌─────────────────────────────────────┐
│  Query Expansion (Gemini)           │
│  Generate 3 semantic variations     │
└──────┬──────────────────────────────┘
       │
   [q1] [q2] [q3]
    │    │    │
    ▼    ▼    ▼
  top-10 chunks each (ChromaDB)
       │
       ▼
┌─────────────────────────────────────┐
│  Reciprocal Rank Fusion             │
│  RRF(d) = Σ 1 / (60 + rank(d))     │
└──────┬──────────────────────────────┘
       │
   Top 5 unique chunks
       │
       ▼
  Gemini 2.5 Flash → Final answer + cited sources
```

**Why RRF?** Parameter-free, robust to score-scale differences between queries, and naturally rewards chunks that appear across multiple query variations. Improved answer accuracy from 71% → 84% in ablation testing.

---

## 📂 Project Structure

```
├── StrategyAI.ipynb          # Main notebook
├── strategyai.html           # Browser frontend
├── documents/                # Source PDFs, DOCXs, HTMLs
├── chroma_db/                # Persisted vector store (auto-created)
└── README.md
```

---

## 📊 Evaluation Results

| Metric | Single-Query | Multi-Query (Ours) | Δ |
|---|---|---|---|
| Answer Accuracy (50 Q) | 71% | **84%** | +13 pts |
| Retrieval Recall@5 | 0.68 | **0.81** | +19% |
| Source Citation Precision | 0.79 | **0.87** | +10% |
| Avg Query Latency | 1.8s | 2.6s | +0.8s overhead |
| Multi-turn Coherence | 0.74 | **0.89** | +20% |

---

## 💡 Key Design Decisions

**Paragraph-based chunking** — splits on double-newlines, merges chunks under 30 words, splits overlong chunks (>300 words) at sentence boundaries. Chosen over fixed-size chunking to preserve semantic units.

**MD5 chunk IDs** — identical text always maps to the same ID, enabling safe upsert and incremental ingestion without full re-indexing.

**Follow-up resolver** — short queries (<6 words) are detected and enriched with the last user message before embedding, resolving ~90% of follow-up retrieval failures.

**Flask in background thread** — runs the API server directly inside the notebook kernel, requiring no separate terminal or Docker setup for local development.

---

## 🗂️ Document Corpus

50+ curated strategy documents covering: Porter's Five Forces · BCG Matrix · Blue Ocean Strategy · Disruptive Innovation · Value Chain Analysis · M&A Synergies · Resource-Based View · Balanced Scorecard · SWOT/PESTLE · Game Theory in Strategy
