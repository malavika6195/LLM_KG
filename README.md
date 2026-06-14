# Personalized Medicine Knowledge Graph (LLM-KG)

Build a high-fidelity, ontologically-aligned Knowledge Graph (KG) from unstructured clinical notes using **Agentic GraphRAG**.

## 🚀 Overview

This project implements an advanced pipeline for **Personalized Medicine** by transforming raw clinical text (like MIMIC-IV) into a structured knowledge graph. It leverages **LangChain** and **LangGraph** to orchestrate a multi-agent workflow that ensures every extracted relationship is medically sound and aligned with established ontologies (**ClinVec/ClinGraph**).

### Key Features
- **Agentic GraphRAG**: Uses specialized agents (Planner, Extractor, Validator) to minimize hallucinations and maximize clinical accuracy.
- **Multi-Model Support**: Benchmarked against **Llama 3**, **Mistral**, and **Gemma 2** via Ollama for domain-specific recall.
- **Neo4j Persistence**: Seamlessly syncs validated triples to **Neo4j Aura** for scalable graph querying and storage.
- **Ontological Grounding**: Automatically fetches and aligns data with the Harvard Zitnik Lab's **ClinVec** ontology (17,000+ diseases, 5M+ relationships).
- **Interactive Visualization**: Generates dynamic, browser-based network maps of patient data using **Pyvis**.

---

## 🏗 Architecture

The system follows a stateful multi-agent architecture:

1.  **Ingestion**: Fetches the ClinVec ontology and loads clinical notes (MIMIC-IV) or legal corpora (BillSum).
2.  **Planner Agent**: Analyzes notes to determine the extraction strategy.
3.  **Extractor Agent**: Performs Named Entity Recognition (NER) and Relation Extraction (RE) using local LLMs (via Ollama) or Gemini.
4.  **Validator Agent**: A "medical/legal critic" that cross-references triples against the ontology, providing feedback for re-extraction.
5.  **Graph Layer**: Consolidates validated triples into **Neo4j** or **NetworkX**.
6.  **Visualization**: Exports the KG as an interactive HTML map.

---

## 🛠 Tech Stack

- **LLMs**: Llama 3, Mistral, Gemma 2 (Local via Ollama), Google Gemini 1.5
- **Framework**: LangChain & LangGraph
- **Database**: Neo4j (Graph Persistence)
- **Graph Engine**: NetworkX
- **Visualization**: Pyvis
- **Data Analysis**: Pandas
- **Ontology**: ClinVec (Harvard Dataverse)

---

## 📊 Benchmarking LLMs for KG Construction

Can LLMs help build Knowledge Graphs? **Yes.** Our multi-domain benchmark shows high efficiency in extracting structured triples from raw text.

| Model | Domain | Triples Extracted | Avg Time/Doc | Efficiency |
| :--- | :--- | :--- | :--- | :--- |
| **Llama 3** | Legal | 57 | 64.5s | 0.09 T/s |
| **Llama 3** | Medical | 20 | 32.3s | 0.06 T/s |
| **Mistral** | Legal | 52 | 68.5s | 0.08 T/s |
| **Gemma 2** | Legal | 68 | 83.2s | 0.08 T/s |

*Llama 3 demonstrated the best balance between extraction speed and recall accuracy.*

---

## 📥 Setup & Installation

### 1. Clone the Repository
```bash
git clone https://github.com/malavika6195/LLM_KG.git
cd LLM_KG
```

### 2. Install Dependencies
```bash
pip install -r requirements.txt
```

### 3. Configure Environment
Create a `.env` file or export your API key:
```bash
GEMINI_API_KEY=your_api_key_here
```

---

## 🏃 Usage

### 1. Download Ontology Data
Fetch the latest ClinGraph nodes and edges (approx. 450MB):
```bash
python3 src/ingestion/fetcher.py
```

### 2. Run the Pipeline
Process clinical notes and generate the graph:
```bash
# To run with real Gemini API
python3 src/main.py --notes data/raw/notes_sample.csv --limit 3

# To run in MOCK MODE (No API Key needed)
export MOCK_MODE=true
python3 src/main.py --notes data/raw/notes_sample.csv --limit 3
```

### 3. View Results
- **Graph Data**: `data/processed/knowledge_graph.json`
- **Visualization**: Open `data/processed/kg_visualization.html` in your browser.

---

## 📂 Project Structure
```text
LLM_KG/
├── src/
│   ├── agents/      # LangGraph node and state definitions
│   ├── graph/       # KG construction and Pyvis visualization
│   └── ingestion/   # Data fetching and MIMIC-IV loading
├── docs/            # Detailed design and usage manuals
├── data/
│   ├── raw/         # Sample clinical notes
│   └── ontology/    # ClinVec CSV files (git-ignored)
└── main.py          # Unified CLI entry point
```

## 📄 License
This project is for educational and research purposes. Data from ClinVec and MIMIC-IV are subject to their respective licenses.
