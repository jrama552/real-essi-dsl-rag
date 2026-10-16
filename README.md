# Real-ESSI DSL Generation with Dynamic Few-Shot RAG

Sanitized version of a retrieval-augmented generation (RAG) pipeline that converts natural-language descriptions into FEI code, the domain-specific language (DSL) of the [Real-ESSI Simulator](http://real-essi.us/) (a nonlinear finite element platform for soil-structure interaction). Built during the UC Davis Young Scholars Program (summer 2025) under Prof. Boris Jeremić.

> **Sanitized release.** Proprietary simulator inputs, outputs and credentials were removed with permission-conscious redaction. The scripts show the pipeline design and methodology but **cannot be run end to end** from this repository. See [What was removed](#what-was-removed).

## Problem

Real-ESSI has no GUI. Users must know both geomechanics and its FEI syntax, which limits who can use it. General LLMs tend to hallucinate FEI syntax, so we tested whether retrieving similar examples at generation time improves the generated code.

## Approach

1. **Build a dataset:** collect FEI projects and generate a natural-language prompt for each with GPT-4.1.
2. **Embed and retrieve:** embed prompts with a sentence-transformer and retrieve the most similar training examples by cosine similarity for each test prompt.
3. **Dynamic few-shot generation:** put the retrieved prompt/code pairs in the context and generate FEI code (GPT-4.1). A baseline uses the same prompt with no retrieved examples.
4. **Evaluate** generated code against the original with:
   - **CodeBERT** embedding cosine similarity (semantic similarity)
   - **Normalized Longest Common Subsequence Similarity (LCSS)** (ordered token overlap, a syntax-oriented measure)

## Results (24 test pairs, from the accompanying paper)

| Metric | Baseline | Dynamic few-shot |
| --- | --- | --- |
| Avg. normalized LCSS, original prompts | 0.0780 | 0.3093 (about 4x, +296%) |
| Avg. CodeBERT similarity, original prompts | 0.9683 | 0.9790 (+1.1 points) |
| Avg. CodeBERT similarity, detailed prompts | 0.9722 | 0.9801 (+0.8 points) |

With more detailed prompts the baseline improves and the LCSS gain shrinks to about +98%. Generated code was not executed in Real-ESSI, and the test set is small. See the paper for limitations.

## Repository contents

| File | Purpose |
| --- | --- |
| `(0) Mounting.py` | Loads and prepares the FEI project data |
| `(2) prompt-code pairs.py` | Generates synthetic natural-language prompts from FEI code |
| `(3) prompt-code pairs output.txt` | Sample output of step 2 |
| `(4) DSL_RAG.py` | Embedding, similarity retrieval and few-shot generation |
| `(6) Evaluation.py` | CodeBERT and LCSS evaluation |
| `Optimization RAG for Real-ESSI DSL.pdf` | Full paper: methods, results, limitations |

Steps (1) and (5) (input FEI codes and generated output files) are **omitted** because they contain the simulator's source material.

## What was removed

- Step (1): the FEI input code dataset
- Step (5): generated and reference output files
- API keys (replace with an environment variable, e.g. `OPENAI_API_KEY`)

## Setup (for reference)

```bash
pip install openai sentence-transformers transformers torch scikit-learn scipy numpy matplotlib
export OPENAI_API_KEY=...
```

## Authors

Jiro Ramacciotti, Grant Kopania, Albert Wu. Advised by Prof. Boris Jeremić, UC Davis.

## License

Code released under the MIT License. The Real-ESSI Simulator and the sanitized-out data belong to their respective owners.