# SITL_Neurosymbolic_KG

# Symbolic-in-the-Loop (SITL) Neuro-Symbolic Verification

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Neo4j](https://img.shields.io/badge/Neo4j-AuraDB-blue.svg)](https://neo4j.com/)
[![Ollama](https://img.shields.io/badge/Ollama-Llama3-black.svg)](https://ollama.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> **Achieving high-assurance autonomous execution by eliminating LLM hallucination through deterministic Knowledge Graph (KG) verification.**

This repository contains the core implementation and adversarial benchmark dataset for the **Symbolic-in-the-Loop (SITL)** architecture. The pipeline integrates a probabilistic Large Language Model (Llama 3) with a deterministic Neo4j Knowledge Graph to enforce a **Strict Closed World Assumption (CWA)**, reducing semantic drift by forcing the LLM to output only provably true statements.

---

## 🧠 Architecture Overview

*(Note: Upload `architecture_diagram.png` to your repository root for this image to display)*
![Architecture Diagram](architecture_diagram.png)

The SITL pipeline operates in three major phases:
1. **Targeted Extraction:** The LLM's draft response is programmatically parsed into `(Subject, Predicate, Object)` triples using Context-Aware Prompt Injection to ignore conversational fluff.
2. **Logic Gate Verification:** The extracted triples are evaluated against the Neo4j Knowledge Graph using deterministic Cypher queries. 
3. **Iterative Refinement ($k=3$):** If a fact is unverified or hallucinated, the system generates a structured error signal and forces the LLM to regenerate. To prevent infinite loops against an incomplete graph, the system utilizes a strict $k=3$ iteration limit, followed by a deterministic "Escape Hatch" (graceful abort).

---

## 📂 Repository Structure

| File | Description |
| :--- | :--- |
| `SITL_Neurosymbolic.ipynb` | The primary Google Colab Notebook containing the full pipeline (Ollama server boot, Prompts, Neo4j connection, and Evaluation loop). |
| `benchmark.json` | A 50-question adversarial micro-benchmark divided into four tiers: Direct Facts, Negative/Trick, Fluff Triggers, and Out-of-Domain. |
| `semantic_drift_progress.json` | *(Generated at runtime)* Checkpoint file to save evaluation progress and auto-heal if the local LLM server crashes. |

---

## ⚙️ Setup and Installation

This pipeline is optimized to run locally or in **Google Colab**. 

### 1. Database Prerequisites
You must have an active **Neo4j AuraDB** instance (or local Neo4j desktop) populated with a Blockchain domain ontology.
* Create a free instance at [Neo4j Aura](https://neo4j.com/cloud/platform/aura-graph-database/).

### 2. Environment Variables
If using Google Colab, click on the **Secrets (🔑)** tab on the left sidebar and add your Neo4j credentials:
* `NEO4J_URI`: Your database URI (e.g., `neo4j+s://<your-db-id>.databases.neo4j.io`)
* `NEO4J_USERNAME`: Your database username (usually `neo4j`)
* `NEO4J_PASSWORD`: Your database password

### 3. Dependencies
The notebook automatically handles the installation of required packages, including setting up a local Ollama server in the background of the Colab instance.
```bash
pip install langchain langchain-community neo4j
apt-get install zstd
curl -fsSL [https://ollama.com/install.sh](https://ollama.com/install.sh) | sh
```

## 🚀 How to Run the Evaluation

1. Open `SITL_Neurosymbolic.ipynb` in Google Colab or your local Jupyter environment.
2. Ensure `benchmark.json` is uploaded to the root directory (e.g., `/content/` in Colab).
3. Select **Runtime -> Run All**.

The script will automatically:
* Boot the Ollama server and pull the `llama3` model.
* Connect to your Neo4j database.
* Iterate through all 50 questions in `benchmark.json`.
* Output a real-time terminal trace of the extraction, database validation, and iterative feedback loops.

---

## 📊 Evaluation Metrics & Results

The pipeline evaluates **Semantic Drift** mathematically. A "hallucination" is strictly defined as any extracted triple whose Subject, Object, or Relational Predicate does not exist within the Neo4j graph boundary.

Upon completing the 50-question benchmark, the script outputs the final comparative metrics. In our primary test run, the pipeline achieved a **100% reduction in semantic drift**:

```text
============================================================
📊 FINAL IEEE EVALUATION METRICS: SEMANTIC DRIFT 📊
============================================================
Total Baseline Triples Extracted: 35
Baseline Hallucinations (Raw LLM): 27
-> Baseline Hallucination Rate (H_base): 77.14%

Total Final Triples Extracted: 8
Final Hallucinations (Pipeline): 0
-> Final Hallucination Rate (H_final): 0.00%

🚀 SEMANTIC DRIFT REDUCTION: 100.00%
============================================================
