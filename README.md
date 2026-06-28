# NavaVeda - Intelligent Document Question Answering System

RAG-based system using FAISS vector DB and Sentence Transformers for semantic PDF search; Llama3 via Ollama for context-aware answers.

---

## Project Overview

Reading long PDF documents to find specific answers is time-consuming. NavaVeda lets you upload any PDF and ask questions in plain language. The system retrieves the most relevant sections from the document and passes them to a local LLM to generate a precise, context-aware answer — all running offline.

```
PDF Upload -> Text Extraction -> Chunking -> Embeddings -> FAISS Index -> Semantic Search -> Llama3 -> Answer
```

---

## Objective

- Extract and preprocess text from uploaded PDF documents
- Split large text into overlapping chunks for better retrieval
- Convert chunks into vector embeddings using Sentence Transformers
- Store and search embeddings using FAISS vector database
- Retrieve the top-3 most relevant chunks for any user question
- Generate context-aware answers using Llama3 running locally via Ollama
- Display the full RAG pipeline in an interactive Streamlit UI

---

## Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Frontend UI | Streamlit | PDF upload, question input, answer display |
| PDF Parsing | PyPDF2 | Extract text from PDF pages |
| Embeddings | Sentence Transformers (all-MiniLM-L6-v2) | Convert text to vector representations |
| Vector Database | FAISS | Store embeddings and perform similarity search |
| LLM | Llama3 via Ollama | Generate final answer from retrieved context |
| Language | Python 3.x | Core development |

---

## Project Structure

```
navaveda-rag-qa/
|
|- app.py               # Main Streamlit application
|- requirements.txt     # Python dependencies
|- README.md
```

---

## System Workflow

### Step 1 - PDF Upload
User uploads a PDF file through the Streamlit UI. PyPDF2 extracts raw text page by page.

### Step 2 - Text Chunking
Large extracted text is split into overlapping chunks of 500 characters with 100-character overlap to preserve context across boundaries.

```
chunk_size = 500 characters
overlap    = 100 characters
```

### Step 3 - Embedding Generation
Each chunk is converted into a 384-dimensional vector using the `all-MiniLM-L6-v2` Sentence Transformer model.

### Step 4 - FAISS Vector Database
All chunk embeddings are stored in a FAISS `IndexFlatL2` index for fast nearest-neighbour search.

### Step 5 - Question Processing
User types a question. It is converted into an embedding using the same Sentence Transformer model.

### Step 6 - Semantic Search
FAISS searches for the top-3 most similar chunks to the question embedding using L2 distance.

### Step 7 - Context + Prompt Construction
The 3 retrieved chunks are combined into a context block and sent to Ollama with a structured prompt:

```
Instructions:
- Answer only from the given context
- Do not use outside knowledge
- If answer not found, say so clearly
```

### Step 8 - LLM Answer Generation
Llama3 running locally through Ollama generates the final answer based on the retrieved context only.

---

## RAG Architecture

```
PDF Upload
    |
Extract Text (PyPDF2)
    |
Split into Chunks (500 chars, 100 overlap)
    |
Generate Embeddings (Sentence Transformers)
    |
Store in FAISS Vector Database
    |
User Asks Question
    |
Convert Question to Embedding
    |
Search Top-3 Similar Chunks (FAISS)
    |
Build Context from Retrieved Chunks
    |
Send Context + Question to Llama3 (Ollama)
    |
Generate Final Answer
```

---

## Sample Output

```
Question : What is the refund policy mentioned in the document?

Retrieved Context : [top 3 matching chunks from PDF]

Answer :
According to the document, refunds are processed within 7 working
days of the cancellation request. The refund is credited to the
original payment method only. No refund is applicable after 30 days
of purchase.
```

---

## How to Run

### 1. Clone the Repository
```bash
git clone https://github.com/yourusername/navaveda-rag-qa.git
cd navaveda-rag-qa
```

### 2. Install Dependencies
```bash
pip install -r requirements.txt
```

### 3. Install and Start Ollama
```bash
# Install Ollama from https://ollama.com
ollama pull llama3
ollama serve
```

### 4. Run the App
```bash
streamlit run app.py
```

UI runs at: http://localhost:8501

---

## Requirements

```
streamlit
pypdf2
faiss-cpu
numpy
sentence-transformers
requests
```

Install all:
```bash
pip install streamlit pypdf2 faiss-cpu numpy sentence-transformers requests
```

---

## Key Design Decisions

**Overlapping Chunks**
A 100-character overlap between consecutive chunks ensures that sentences split across chunk boundaries are still retrievable. Without overlap, a key sentence at the edge of a chunk could be missed entirely.

**all-MiniLM-L6-v2 Model**
This lightweight Sentence Transformer produces high-quality 384-dimensional embeddings at fast inference speed, making it ideal for local offline use without GPU.

**FAISS IndexFlatL2**
Flat L2 index performs exact nearest-neighbour search. For small-to-medium PDFs this is fast enough and avoids the approximation errors of IVF or HNSW indexes.

**Context-Only Prompting**
The LLM prompt explicitly instructs Llama3 to answer only from the provided context and not use external knowledge. This prevents hallucination and keeps answers grounded in the uploaded document.

**@st.cache_resource for Embedding Model**
The Sentence Transformer model is loaded once and cached using Streamlit's resource cache, avoiding repeated slow model loading on every user interaction.

---

## Future Enhancements

- [ ] Support multi-PDF upload and cross-document search
- [ ] Add chat history for multi-turn question answering
- [ ] Switch between Ollama models (Llama3, Mistral, Gemma) from UI
- [ ] Highlight the exact paragraph in the PDF where the answer was found
- [ ] Export Q&A session as PDF report
- [ ] Add support for DOCX and TXT files alongside PDF
- [ ] Deploy on cloud with persistent vector store (Pinecone / Weaviate)

---

## Author

**Rushikesh Chavhan**
MCA Student | Python Developer | AI & Backend Enthusiast

---

## License

This project is for educational purposes only.
