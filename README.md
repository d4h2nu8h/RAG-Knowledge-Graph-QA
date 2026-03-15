# Hybrid RAG with Knowledge Graph for Document Question Answering

> A document-grounded question answering system that combines vector similarity retrieval with a Neo4j knowledge graph to deliver semantically rich, contextually accurate responses — powered by a locally hosted Ollama LLM and deployed via Streamlit.

---

## Overview

Standard RAG pipelines retrieve document chunks by vector similarity alone — an approach that works well for surface-level factual queries but struggles with questions that require understanding relationships between entities, multi-hop reasoning, or structured domain knowledge. This project addresses that gap by introducing a hybrid architecture that augments vector retrieval with a structured knowledge graph.

The system processes PDF documents through a three-stage pipeline: text extraction, entity-relationship graph construction, and interactive querying. At inference time, the Streamlit interface routes user queries through both a vector embedding store and a Neo4j knowledge graph, combining the outputs before passing them to a locally running Ollama language model for generation. The result is a system that can answer both semantically similar queries and relationship-aware questions from the same document corpus — without sending any data to external APIs.

---

## Dataset

**Source:** User-supplied PDF documents

The system is document-agnostic — any PDF can be processed through the pipeline. The provided research paper (`PraxiPaaS_A_Decomposable_Machine_Lear...`) was used as the primary test document. Text is extracted to `extracted_text.txt` and serves as the input to both the vector embedding store and the knowledge graph construction stage.

| Property | Detail |
|---|---|
| Input format | PDF documents |
| Intermediate output | `extracted_text.txt` |
| Extraction library | LlamaIndex |
| Knowledge graph store | Neo4j |
| LLM | Ollama (running locally) |

---

## Methodology

### System Architecture

```
PDF Document
      ↓
extraction.py  →  extracted_text.txt
      ↓                    ↓
graphcreation.py      Vector Embeddings
      ↓                    ↓
Neo4j Knowledge Graph ←→  QA Engine (Ollama)
                               ↓
                        Streamlit Interface
                               ↓
                             User
```

### Stage 1 — Text Extraction (`extraction.py`)

PDF documents are parsed using the **LlamaIndex** (`llama-index`) library, which converts PDF content into machine-readable plain text. The extracted text is cleaned, structured, and written to `extracted_text.txt` for use in downstream stages.

### Stage 2 — Knowledge Graph Construction (`graphcreation.py`)

The extracted text is processed to identify key entities and the relationships between them. These entity-relationship pairs are used to construct and populate a **Neo4j** knowledge graph, which represents the semantic and structural context of the document. The knowledge graph enables relationship-aware retrieval that pure vector similarity search cannot provide — for example, querying which entities are connected, or tracing multi-hop relationships across the document.

### Stage 3 — Interactive Querying (`app.py`)

A **Streamlit** web application provides the user-facing query interface. On receiving a query:

1. Relevant information is retrieved from both the vector embedding store and the Neo4j knowledge graph
2. Retrieved context from both sources is combined and passed to the locally running **Ollama** language model
3. The model generates a coherent, contextually grounded response which is returned to the user

Running Ollama locally means all inference happens on-device — no data is sent to external APIs, making the system suitable for sensitive or proprietary document corpora.

---

## Results

The hybrid retrieval approach improves response quality over vector-only RAG on queries that involve named entities, relationships between concepts, and structured domain knowledge within the document. The knowledge graph provides a complementary signal to vector similarity — where embeddings capture semantic closeness, the graph captures explicit structural connections — and the combination is passed to Ollama for coherent generation.

> Qualitative evaluation was performed on the PraxiPaaS research paper. The system successfully answered entity-specific, relationship-aware, and summary-level questions from the document.

---

## Limitations & Future Work

**Current Limitations:**

- The system requires Ollama to be installed and running locally; there is no fallback to a hosted API model
- Knowledge graph construction is dependent on the quality of entity and relationship extraction, which may be incomplete for highly technical or domain-specific documents
- The current pipeline processes one document at a time; multi-document indexing and cross-document relationship queries are not yet supported
- There is no persistent user session or query history in the Streamlit interface

**Future Directions:**

- Extend to multi-document corpora with cross-document entity linking in the Neo4j graph
- Add graph visualisation in the Streamlit interface to let users explore the knowledge graph interactively
- Benchmark hybrid retrieval (vector + graph) against vector-only RAG on a standardised QA dataset to quantify the accuracy improvement
- Integrate support for additional file formats (Word documents, web pages, structured CSVs) beyond PDF

---

## How to Run This Project

### Prerequisites

```bash
Python 3.7+
Neo4j (running locally or via cloud)
Ollama (running locally)
```

### 1. Clone the Repository

```bash
git clone https://github.com/d4h2nu8h/hybrid-rag-knowledge-graph.git
cd hybrid-rag-knowledge-graph
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Run the Scripts in Order

**Step 1 — Extract text from your PDF:**

```bash
python extraction.py
```

**Step 2 — Build the knowledge graph:**

```bash
python graphcreation.py
```

**Step 3 — Launch the Streamlit app:**

```bash
streamlit run app.py
```

> Ollama must be running locally on your machine before launching the app.

---

## Tech Stack

| Category | Tools |
|---|---|
| Language | Python 3.7+ |
| Text Extraction | LlamaIndex (`llama-index`) |
| Knowledge Graph | Neo4j |
| Language Model | Ollama (local inference) |
| Vector Embeddings | LlamaIndex |
| Web Interface | Streamlit |
| Input Format | PDF |

---

## Author

**Dhanush Sambasivam**

[![GitHub](https://img.shields.io/badge/GitHub-d4h2nu8h-181717?style=flat&logo=github)](https://github.com/d4h2nu8h)

---

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.
