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
