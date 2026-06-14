# 2. Notebook Workflow

## Overview of `LLM_KG_Neo4j.ipynb`
The Jupyter notebook `LLM_KG_Neo4j.ipynb` serves as the primary execution environment for the project. It is specifically designed to run efficiently in environments like Google Colab, handling environment setup, executing model benchmarks, and pushing the final results to a Neo4j Aura cloud database.

## Phase 1: Robust Setup and Initialization
**What it does:**
The first cell ensures a clean slate. It changes the working directory, deletes any existing project files, and clones the latest codebase from GitHub. It then installs the necessary Python libraries (`langchain`, `langgraph`, `neo4j`, `pyvis`, etc.).
Crucially, it installs and starts the **Ollama** server locally in the background.

**Why it works this way:**
In stateless environments like Google Colab, ensuring that all dependencies and the local LLM engine are running before executing Python code is vital. By force-reloading Python modules, the notebook guarantees that it is using the latest code pulled from GitHub.

## Phase 2: Multi-Model Benchmark & Analytics
**What it does:**
The notebook defines a testing suite spanning two domains: `legal` (using the `FiscalNote/billsum` dataset) and `medical` (using sample text). It iterates over three models: `llama3`, `mistral`, and `gemma2`.
For each model and domain, it invokes the Agentic LangGraph workflow and measures:
- Total Triples extracted.
- Average Time per Document.
- Extraction Efficiency (Triples per second).

**How it works:**
The code uses `ChatOllama` to instantiate the respective local model. It records the extraction metrics and stores the resulting triples in a local JSON via the `MilestoneManager` (but only saves the results from `llama3` to keep the final graph clean). Finally, it outputs the benchmark results using the `tabulate` library.

## Phase 3: Syncing to Neo4j
**What it does:**
Taking the best results from the benchmark phase (the `llama3` milestones), the notebook initializes the `Neo4jManager`. It aggregates all the saved triples and uploads them to the configured Neo4j Aura instance.

**Why it works this way:**
Instead of uploading midway through the pipeline (which could result in dirty data if a process crashes), the notebook first saves results locally (`MilestoneManager`), verifies them, and then pushes them in a batch to the graph database.

## Phase 4: Multi-Domain Interactive Visualization
**What it does:**
Using `NetworkX` and `Pyvis`, the notebook compiles all triples into an interactive HTML graph. This graph is embedded directly into the Jupyter/Colab output cell.

**How it works:**
It reads the aggregated triples from the Milestone JSON, adds them to a `nx.MultiDiGraph`, and passes the graph to the `visualize_graph` function from `src.graph.visualizer`. The resulting HTML is base64-encoded and rendered in an `<iframe>`, providing immediate visual feedback of the extracted knowledge.
