<p align="center">
  <img src="./assert%3Aimg/font.png" alt="font" width="900">
</p>

<h1 align="center">SkillGATE: Gate-Aware Monte Carlo Tree Search for Skill Retrieval
</h1>
<div align="center">

<p align="center">
  <img src="https://img.shields.io/badge/Task-Skill%20Retrieval-blue" alt="Skill Retrieval">
  <img src="https://img.shields.io/badge/Method-Gate--Aware%20MCTS-orange" alt="Gate-Aware MCTS">
  <img src="https://img.shields.io/badge/Index-Graph%20%2B%20Hierarchy-purple" alt="Graph and Hierarchy">
  <br>
  <img src="https://img.shields.io/badge/Python-%E2%89%A53.10-3776AB?logo=python&logoColor=white" alt="Python ≥3.10">
  <img src="https://img.shields.io/badge/CUDA-PyTorch%20Compatible-76B900?logo=nvidia&logoColor=white" alt="CUDA compatible with PyTorch">
  <img src="https://img.shields.io/badge/C%2B%2B-17-00599C?logo=cplusplus&logoColor=white" alt="C++17">
  <img src="https://img.shields.io/badge/Platform-Linux-FCC624?logo=linux&logoColor=black" alt="Linux">
</p>

<p align="center">
  <img src="./assert%3Aimg/main.png" alt="framework" width="900">
</p>
</div>

coming soon...

## 📖 Paper Introduction

**SkillGATE** is a graph-guided hierarchical framework for retrieving relevant skills from large skill libraries. As these libraries grow in scale and diversity, retrieval becomes increasingly challenging: independently scoring skills can favor semantically similar distractors, graph-based search can become trapped in local neighborhoods, and hierarchical routing can exclude relevant skills after an early routing error.

🔥 What makes SkillGATE different?

Inspired by **Optimal Foraging Theory**, SkillGATE treats skill retrieval as an adaptive information-foraging process. It coordinates region-level navigation with skill-level selection, using the utility and uncertainty observed during search to decide where to explore next.

- 🕸️ **Graph-Preserving Hierarchy:** Semantic, lexical, and structured-label relations form a weighted skill graph, organized into a hierarchy while retaining skill-level connections.
- 🌳 **Gate-Aware Tree Search:** Monte Carlo Tree Search progressively explores the skill space through selection, expansion, simulation, and backpropagation.
- 🧭 **Adaptive Navigation:** The G-PUCT policy combines historical returns, return entropy, query-relevance priors, and visit statistics to guide descent, cross-region exploration, and pruning.

🚀 How well does SkillGATE perform?

The saved evaluation references cover **5,400 queries across six benchmarks** with a shared library of **26,262 skills**: TheoremQA, LogicBench, ToolQA, CHAMP, MedCalcBench, and BigCodeBench. The table below reports this repository's saved reference metrics, rounded directly to two decimal places. It does not represent a new evaluation run.
<p align="center">
  <img src="./assert%3Aimg/results.png" alt="font" width="950">
</p>

## 📂 Directory Structure

```text
configs/
  config.yaml                Model selection, model paths, and shared parameters
src/skillgate/
  cli.py                     CLI commands for indexing, retrieval, and evaluation
  pipeline.py                Search and ranking using prepared benchmark vectors
  runner.py                  Retrieval workers and prediction saving
  reranking.py               Frozen-score and live-model reranking
  evaluation.py              Prediction validation and evaluation reports
  metrics.py                 Recall@K and nDCG@K calculation
  storage.py                 Configuration loading, paths, and artifact I/O
  engine/                    G-PUCT search, routing, features, and PPR
  indexing/                  Skill graph and hierarchy construction
  ranking/                   Title processing, rank fusion, and reranker implementations
  native/                    Optional C++ PPR accelerator
  vendor/                    Adapted utility code
third_party/                 Preserved third-party license
.env.example                 API configuration template
pyproject.toml               Package metadata, dependencies, and CLI entry point
requirements.txt             Pinned core dependencies
requirements-models.txt       Optional model-inference dependencies
```

The local `artifacts/`, `models/`, and `outputs/` directories are excluded from version control. They store prepared benchmark assets, downloaded model weights, and generated indexes or run results, respectively.

## 🛠️ Installation & Environment

Python **3.10 or later** is required. Frozen retrieval and standalone evaluation run on CPU without model weights or an API key. Native PPR acceleration requires a C++17 compiler available as `g++`; use `--kernel python` to run retrieval without compilation.

Run the following from the repository root:

```bash
python3.10 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m pip install -e .
skillgate --help
```

Install the additional dependencies for index construction:

```bash
python -m pip install -e ".[indexing]"
```

For local embedding or reranker inference, also install the model dependencies:

```bash
python -m pip install -r requirements-models.txt
```

Local inference defaults to `cuda:0`. Use `--device` to select a device and ensure that your PyTorch/CUDA installation supports your NVIDIA GPU. Memory requirements depend on the model, sequence length, batch size, and number of retrieval workers.

Keep the source checkout available: `configs/config.yaml` lives outside the Python package and is not included in a standalone wheel. When running outside the repository, specify the project directory before the subcommand:

```bash
skillgate --project /path/to/skillgate retrieve --help
```

Installation does not download model weights or benchmark assets. Prepare them as described below before running indexing or retrieval. The separate `evaluate` command only requires saved retrieval outputs and matching benchmark annotations.

## ⚡ Quick Start

Complete installation and configure `embeddingmodel` and `rerankermodel` in `configs/config.yaml`. Run the following commands from the repository root.

### 1. Index

Build an index from your skill corpus. For API embeddings and skill-card extraction:

```bash
skillgate build-index --corpus /path/to/corpus.json \
  --output outputs/my-index --allow-api
```

For local embeddings with existing skill cards:

```bash
skillgate build-index --corpus /path/to/corpus.json \
  --reuse-cards /path/to/skill_cards.jsonl \
  --device cuda:0 --output outputs/my-index
```

Choose the command matching your configuration. The output directory must be new or empty. Base artifacts are saved under `outputs/my-index/base/`, and the hierarchy and retrieval indexes under `outputs/my-index/index/`.

For the existing benchmark, skip index construction and provide the matching prepared artifact bundle. The retrieval command below uses that bundle's index and saved query vectors; it does not automatically load the index built above.

### 2. Retrieve

Verify the benchmark assets, then retrieve skills for all benchmark queries:

```bash
skillgate verify

skillgate retrieve --workers 1 --kernel python \
  --output outputs/benchmark
```

The selected model pair comes from `configs/config.yaml`. Predictions are saved to `outputs/benchmark/<backend>/predictions.jsonl`, where `<backend>` is `ada`, `skillret`, or `r3`. If `rerankermodel` is enabled, reranked predictions are also saved under `<backend>_rerank/`.

Reranking uses frozen scores by default. To run the configured reranker directly, add:

- API model: `--rerank-mode live --allow-api`
- Local model: `--rerank-mode live --device cuda:0`

To retrieve one query per dataset for a quick check:

```bash
skillgate retrieve --sample-per-dataset 1 \
  --workers 1 --kernel python --output outputs/smoke
```

Retrieval does not calculate evaluation metrics. Use a new output directory when changing models or run settings.

### 3. Evaluate

Evaluate saved predictions against the benchmark annotations:

```bash
skillgate evaluate --input outputs/benchmark
```

For the sampled run, use `--input outputs/smoke` instead.

Evaluation computes Recall and nDCG without loading models, rerunning retrieval, or calling APIs. Results are saved in the same output directory:

```text
outputs/benchmark/
  <backend>/summary.json
  <backend>_rerank/summary.json   When reranking is enabled
  results.md
  paper_display_legacy.md
```

View the overall and per-dataset Recall@1 / Recall@10 tables:

```bash
cat outputs/benchmark/results.md
```

Benchmark annotations default to `artifacts/benchmark/queries.jsonl.gz`. To use the same annotation file stored elsewhere:

```bash
skillgate evaluate --input outputs/benchmark \
  --queries /path/to/queries.jsonl.gz
```

## 📥 Data Input

### Skill Corpus

Index construction accepts a JSON array of skill records. Save your skill library as `data/corpus.json`:

```json
[
  {
    "skill_id": "csv_summary",
    "name": "Summarize CSV Data",
    "description": "Compute descriptive statistics for numeric columns in a CSV file.",
    "content": "Use pandas.read_csv to load the file. Select numeric columns with select_dtypes(include='number'). Calculate the count, mean, standard deviation, minimum, and maximum for each column. Return the results as a summary table."
  },
  {
    "skill_id": "csv_deduplicate",
    "name": "Remove Duplicate CSV Rows",
    "description": "Remove duplicate rows from a CSV file and save the cleaned data.",
    "content": "Use pandas.read_csv to load the file. Apply drop_duplicates to remove duplicate rows, keeping the first occurrence. Save the cleaned DataFrame with to_csv(index=False). Report the number of rows removed."
  }
]
```

| Field | Description |
| --- | --- |
| `skill_id` | Unique, nonempty string identifying the skill. |
| `name` | Human-readable skill name. |
| `description` | Brief description of the skill's capability. |
| `content` | Full skill instructions, including procedures and relevant tools. |

With API access configured, build the index using:

```bash
skillgate build-index --corpus data/corpus.json \
  --output outputs/my-index --allow-api
```

This example illustrates the input format. Use your full skill library for index construction and benchmark retrieval.

### Benchmark Queries

Benchmark queries use JSONL: one JSON object per line. The following synthetic records illustrate the format of `queries.jsonl`:

```jsonl
{"query_id":"example_001","dataset":"bigcodebench","query":"Write Python code to calculate descriptive statistics for numeric columns in a CSV file.","gold_skill_ids":["csv_summary"]}
{"query_id":"example_002","dataset":"bigcodebench","query":"Write Python code to remove duplicate rows from a CSV file and save the cleaned result.","gold_skill_ids":["csv_deduplicate"]}
```

| Field | Description |
| --- | --- |
| `query_id` | Unique query identifier. |
| `dataset` | Dataset name: `theoremqa`, `logicbench`, `toolqa`, `champ`, `medcalcbench`, or `bigcodebench`. |
| `query` | The retrieval request. |
| `gold_skill_ids` | Nonempty list of relevant skill IDs from the corpus. Multiple relevant skills are supported. |
| `gpt_user_query` | Optional query text for GPT reranking; defaults to `query` when omitted or empty. |

The prepared benchmark stores these records in `artifacts/benchmark/queries.jsonl.gz`.

The corpus and query examples alone are not a complete runnable benchmark. The current `retrieve` command also requires matching indexes, saved vectors, and the artifact manifest; frozen reranking additionally requires cached scores. Evaluation expects 50 unique predicted skills per query, so the two-skill example above only demonstrates the data format.

## 📄 License and Attribution

SkillGATE is licensed under the [MIT License](LICENSE).
