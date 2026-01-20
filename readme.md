# Semantic Policy Change Impact Analyzer

![Python](https://img.shields.io/badge/Python-3.9%2B-blue)
![LangChain](https://img.shields.io/badge/LangChain-Integration-green)
![LLM](https://img.shields.io/badge/AI-Gemini-orange)

**A Hybrid ML + LLM system for detecting, analyzing, and classifying semantic changes in platform policy documents.**

---

## Overview

This project is a **Semantic Policy Change Impact Analyzer** designed to solve the challenge of tracking updates in Terms of Service (ToS) and Privacy Policies.

The system employs a **cost-effective, hybrid architecture**:
1.  **Machine Learning Layer:** Uses embeddings and deterministic similarity thresholds to handle change detection and filtering.
2.  **LLM Layer:** A Large Language Model is invoked *only* when semantic ambiguity exists to assess impact severity.

This approach ensures scalability, interpretability, and rate-limit safety while providing deep insights into "breaking changes" vs. simple rephrasing.

---

## Problem Statement

Large platforms (Amazon, X, Instagram) frequently update their policies. Manual comparison is:
* **Time-consuming:** Reading thousands of lines of legal text.
* **Error-prone:** Missing subtle but critical shifts in liability.
* **Unscalable:** Impossible to track across dozens of platforms manually.

**This project automates:**
* Semantic comparison (understanding meaning, not just text diffs).
* Identification of high-impact changes.
* Confidence-aware decision support.

---

## Architecture & Workflow

The system follows a strict pipeline designed for noise reduction and efficiency:

### 1. Input & Noise Reduction
* Ingests Old and New policy versions.
* Removes identical boilerplate sentences to eliminate false positives immediately.

### 2. Semantic Chunking
* Splits policies into overlapping, context-preserving chunks rather than arbitrary line breaks.

### 3. Embedding & Retrieval
* Converts old chunks into vector embeddings (using **Sentence Transformers**).
* Retrieves the most semantically similar "Old Clause" for every "New Clause."

### 4. Semantic Change Index
* Computes **Cosine Similarity**.
* Categorizes changes based on deterministic thresholds:
    * **No Semantic Change:** (High similarity)
    * **Clarification:** (Medium similarity)
    * **Requires LLM Analysis:** (Ambiguous/Low similarity)

### 5. Selective LLM Impact Analysis
* *Only* ambiguous cases are sent to **LLM**.
* Classifies impact as: `BREAKING`, `NON_BREAKING`, or `NEW_FEATURE`.

---

## Real-World Case Studies

The system was tested on real policy updates from major platforms. Below are the aggregate results.

### 1. Amazon
* **Observation:** Balanced mix of stable clauses and moderate changes.
* **Pairs Processed:** 26
* **LLM Usage:** 34.62%
* **Avg Similarity:** 0.77 | **Avg Confidence:** 0.823
* **Original Policy Data:** [Amazon 2025](https://developer.amazon.com/support/legal/da) | [Amazon 2023](https://developer.amazon.com/support/legal/da/archive)

### 2. Twitter (X)
* **Observation:** Required the highest LLM involvement, indicating highly ambiguous or significant policy shifts.
* **Pairs Processed:** 18
* **LLM Usage:** **55.56%** (High)
* **Avg Similarity:** 0.75 | **Avg Confidence:** 0.867
* **Original Policy Data:** [Twitter Version 20](https://x.com/en/tos/previous/version_20) | [Twitter Version 19](https://x.com/en/tos/previous/version_19)

### 3. Instagram
* **Observation:** High similarity and low LLM usage suggest mostly minor updates or language clarifications.
* **Pairs Processed:** 111
* **LLM Usage:** **9.91%** (Low)
* **Avg Similarity:** 0.89 | **Avg Confidence:** 0.842
* **Original Policy Data:** [Instagram 2025](https://privacycenter.instagram.com/policy/version/10052835831416190/) | [Instagram 2024](https://privacycenter.instagram.com/policy/version/8810742435690564/)

---

## Comparative Results Across Platforms

The table below provides a side-by-side comparison of system behavior across different platforms.
It highlights how **policy complexity and ambiguity directly affect semantic similarity and LLM usage**.

### Aggregate Metrics Comparison

| Platform        | Total Clause Pairs | LLM Usage (%) | Avg Semantic Similarity | Avg Confidence |
| --------------- | -----------------: | ------------: | ----------------------: | -------------: |
| **Amazon**      |                 26 |        34.62% |                    0.77 |          0.823 |
| **Twitter (X)** |                 18 |    **55.56%** |                    0.75 |      **0.867** |
| **Instagram**   |                111 |     **9.91%** |                **0.89** |          0.842 |

### Interpretation

* **Twitter (X)** shows the *highest LLM usage*, indicating frequent **ambiguous or high-impact policy changes**.
* **Instagram** shows *high semantic stability*, with most updates being **minor clarifications**.
* **Amazon** lies in between, with a balanced mix of stable clauses and moderate semantic shifts.
* Higher **LLM usage correlates with lower semantic similarity**, validating the ML-first routing logic.

---

## Rate Limiting & Token Throttling Strategy

LLM APIs impose **rate limits and token budgets**.
Naively invoking an LLM for every clause comparison would result in:

* Frequent **HTTP 429 (Too Many Requests)** errors
* Wasted computation and partial failures
* Increased cost and unreliable execution

This project explicitly accounts for these constraints.

---

### Throttling Design

The system includes a **token-aware throttling mechanism** with the following behavior:

* Tracks estimated token usage per minute
* Safely pauses or halts execution when limits are approached
* Stops execution **gracefully** on a 429 error
* Preserves and prints **all successfully processed results**

---

## Key Design Principles

| Principle | Description |
| :--- | :--- |
| **Semantic, not Lexical** | We ignore simple phrasing changes; we look for changes in *meaning*. |
| **ML-First, LLM-Second** | We use cheap, fast embeddings for 80% of the work, and LLM reasoning for the hardest 20%. |
| **Confidence as Risk** | Confidence scores are derived from semantic signal strength, not LLM self-reporting. |
| **Deterministic** | Deterministic rules handle obvious cases efficiently. |

---

## Technologies Used

* **Language:** Python
* **Orchestration:** LangChain
* **Embeddings:** Sentence Transformers (`all-MiniLM-L6-v2` and `all-MiniLM-L12-v2`)
* **Vector Store:** FAISS
* **LLM:** Google Gemini 2.5 Flash
* **Analysis:** Pandas & Matplotlib

---

## Future Improvements

* **Checkpointing:** Resume processing after API rate-limit interruptions.
