# 1. Introduction and Architecture

## System Overview
The **Personalized Medicine Knowledge Graph** is an advanced Agentic GraphRAG system designed to dynamically construct Knowledge Graphs (KGs) from unstructured clinical and legal text. By leveraging local Large Language Models (LLMs) via Ollama, LangGraph for agentic state management, and Neo4j for graph persistence, the system bridges the gap between raw unstructured data and highly structured relational knowledge.

## Core Concepts: Agentic GraphRAG
Unlike standard Retrieval-Augmented Generation (RAG) which relies primarily on semantic vector similarity, **GraphRAG** grounds LLM generation in deterministic, structured relationships. 
This project takes it a step further by implementing **Agentic GraphRAG**: instead of using a single LLM call to extract data, it employs a multi-agent workflow (Planner -> Extractor -> Validator -> Deduplicator). This multi-step deliberation ensures that extracted triples (Subject -> Predicate -> Object) are highly accurate, domain-compliant, and grounded in the source text.

## LLM Models
The system emphasizes local, privacy-preserving AI using **Ollama**. During its benchmarking and execution phases, it utilizes several state-of-the-art open-weight models:
1. **Llama 3 (`llama3`)**: Used as the primary driver for extraction and validation due to its strong reasoning capabilities.
2. **Mistral (`mistral`)**: Benchmarked for speed and instructional adherence.
3. **Gemma 2 (`gemma2`)**: Benchmarked for its compact size and high efficiency.

## Architectural Flow
The pipeline operates in the following sequential phases:
1. **Data Ingestion**: Raw clinical notes (MIMIC-IV schema) and domain ontologies (like Harvard's ClinVec) are loaded into memory.
2. **Agentic Pipeline (LangGraph)**:
   - **Planner**: Reads the text and domain configuration to formulate an extraction strategy.
   - **Extractor**: Uses the strategy to pull Triples from the text.
   - **Validator**: Checks if the triples are faithful to the text and the ontology. If not, it loops back to the Extractor with feedback.
   - **Deduplicator**: Cleans and normalizes entity names.
3. **In-Memory Graph & Milestones**: Extracted triples are stored incrementally (MilestoneManager) and added to an in-memory `NetworkX` graph.
4. **Visualization**: `Pyvis` renders the NetworkX graph into an interactive HTML visualization.
5. **Persistence**: The validated triples are upserted into a **Neo4j** database using optimized Cypher queries.
6. **Querying (Graph QA)**: A downstream LangChain `GraphCypherQAChain` translates natural language questions into Cypher queries against Neo4j to provide answers.

This architecture ensures a robust, fault-tolerant, and domain-adaptable approach to Knowledge Graph generation.
