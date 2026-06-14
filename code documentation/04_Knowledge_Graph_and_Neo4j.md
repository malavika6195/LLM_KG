# 4. Knowledge Graph and Neo4j

## Overview
The files within `src/graph/` are responsible for taking the structured arrays of `Triple` objects produced by the Agentic Workflow and translating them into tangible, queryable, and visualizable graph structures.

## In-Memory Construction (`builder.py`)
**What it is:** The `GraphBuilder` class wraps a `networkx.MultiDiGraph`.
**How it works:** It iterates over extracted triples, adding subjects and objects as nodes, and the predicates as directed edges with a `confidence` attribute. It provides methods to save and load the entire graph structure to a JSON file format (`data/processed/knowledge_graph.json`).
**Why:** NetworkX is excellent for fast, in-memory graph operations, topological analysis, and acting as a bridge to visualization libraries before committing to a heavy database transaction.

## Intermediate Storage (`milestone.py`)
**What it is:** The `MilestoneManager` class manages a local JSON file (`milestone.json`).
**How it works:** During processing, especially across hundreds of documents, an interruption could cause massive data loss. The `MilestoneManager` continually appends newly extracted triples to this JSON file.
**Why:** It acts as a fault-tolerant cache. If the pipeline crashes on document 50 of 100, the previous 49 are safely persisted and can be reloaded or pushed directly to Neo4j.

## Visualization (`visualizer.py`)
**What it is:** This script uses `pyvis.network.Network` to render interactive HTML maps of the graph.
**How it works:**
- It provides `visualize_graph` which takes a `NetworkX` graph. It colors nodes dynamically based on the domain configuration (e.g., medical entities might be colored differently than legal entities based on substring matching).
- It injects custom JavaScript physics options (Barnes-Hut) to ensure the graph spreads out cleanly and is highly readable even with hundreds of nodes.
- It also provides `visualize_from_neo4j`, which connects directly to the Aura instance, pulls the current graph topology using Cypher, and pipes it into Pyvis.

## Graph Database Persistence (`neo4j_manager.py`)
**What it is:** The `Neo4jManager` is a wrapper around the official `neo4j` Python driver, specifically optimized for connecting to a Neo4j Aura cloud database.
**How it works:**
- **Initialization:** Connects via URI, user, and password specified in environment variables.
- **Upload Logic (`upload_triples`):** 
  - It sanitizes predicates to conform to Cypher syntax (e.g., replacing spaces with underscores, ensuring it doesn't start with a number).
  - It uses the `MERGE` clause for both nodes and edges.
**Why `MERGE`?** `MERGE` operates like an "UPSERT" (Update or Insert). If a node for "Aspirin" already exists, it matches it. If it doesn't, it creates it. The same logic applies to the relationship. This prevents the database from creating duplicate nodes every time a new document mentions the same entity, automatically building a densely connected graph network.
