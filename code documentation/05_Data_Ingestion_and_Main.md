# 5. Data Ingestion and Pipeline Orchestration

## Data Ingestion (`src/ingestion/`)

### Fetching External Ontologies (`fetcher.py`)
**What it does:** Automates the downloading of the ClinVec ontology files (`ClinGraph_nodes.csv` and `ClinGraph_edges.csv`) directly from the Harvard Dataverse.
**How it works:** It uses the `requests` library to stream the files over HTTP, employing `tqdm` to display a progress bar.
**Why it works this way:** Hardcoding ontologies in the repo limits scalability. Dynamically fetching them ensures the system can always bootstrap itself with the latest academic ontologies on fresh installs (like in Google Colab).

### Loading Local Data (`loader.py`)
**What it does:** Reads local CSV files containing clinical notes and the ontology graphs into pandas DataFrames.
**How it works:** 
- `load_clinical_notes`: Uses a robust path-resolution algorithm (checking `data/`, `datasets/`, `/content/LLM_KG`, etc.) to find the CSV. It then searches for standard text columns (`text`, `note`, `description`, `NEV_TEXT`) and extracts them as a list of strings.
- `load_ontology`: Searches for the fetched ClinGraph CSVs and loads them.
**Why:** This defensive programming approach prevents the script from crashing when executed across different environments (local VS Code vs. Jupyter Notebook vs. Colab) where relative path roots might differ.

## Pipeline Orchestration (`src/main.py`)

### The CLI Interface
`main.py` is the primary entry point for local execution. It uses `argparse` to provide a robust Command Line Interface.
Arguments include:
- `--notes`: Path to the raw notes CSV.
- `--fetch-ontology`: Flag to trigger `fetcher.py`.
- `--limit`: Restricts the number of documents processed (crucial for local testing).
- `--neo4j` and `--clear-neo4j`: Flags to dictate whether to sync the results to the cloud graph database.
- `--query`: Allows the user to bypass ingestion and directly query an existing Neo4j graph using natural language.

### Step-by-step Orchestration
1. **Pre-flight:** Fetches ontologies if requested and loads the `.env` variables (vital for Neo4j and Ollama configurations).
2. **Data Loading:** Passes the CSV paths to `loader.py`.
3. **Pipeline Initialization:** Compiles the LangGraph workflow via `create_agentic_workflow()` and creates a new `GraphBuilder`.
4. **Execution Loop:**
   - Iterates through the list of notes.
   - For each note, it populates the initial `AgentState` dictionary (`input_text`, `domain`, `is_valid=False`, etc.).
   - Calls `workflow.invoke(initial_state)`. This hands control over to the LangGraph state machine (Planner -> Extractor -> Validator -> Deduplicator).
   - Extracts the resulting triples and appends them to the master list.
5. **Graph Operations:**
   - Saves the extracted triples to `knowledge_graph.json`.
   - Generates the Pyvis interactive HTML map.
6. **Neo4j Sync:** If `--neo4j` was passed, it initializes `Neo4jManager`. If requested, it clears the DB, then pushes all accumulated triples using optimized `MERGE` queries, and generates a fresh visual representation straight from the DB.
7. **Querying & Evaluation:** If a `--query` was provided, it compiles a simplified `create_query_workflow` and uses the LangChain `GraphCypherQAChain` to answer the user's question. Finally, it attempts to run metrics from `evaluation/metrics.py` to grade the graph's quality against the reference ontology.
