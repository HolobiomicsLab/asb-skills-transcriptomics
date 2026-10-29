---
name: minimum-weight-bipartite-matching
description: Use when integrating two scRNA-seq datasets (e.g., log-count matrices
  pre-processed with Seurat V3.0) where batch correction must preserve local cell-to-cell
  structure, and a one-to-one matching of representative anchor cells across datasets
  is needed.
when_to_use_negative:
- You need to integrate more than two datasets — the current BATMAN implementation
  works only with two datasets (multi-dataset integration is possible only 'in principle').
- You have extra-large datasets where exact blossom matching would be too slow — prefer
  the -greedy heuristic, or a different method if even greedy matching is infeasible.
- Your goal requires matching cell types absent from one dataset (e.g., alpha cells
  held out of CEL-Seq2) — the matching leaves such cells unmatched by design.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BATMAN
  role: Implements anchor identification, bipartite graph construction, minimum-weight
    matching (exact or greedy), local shifting vectors, and batch correction via batman.py
  repo: https://github.com/mandricigor/batman
- name: Scanorama
  role: Comparison baseline integration method run on the same preprocessed dataset
    pair (e.g., pancreas 1-held-out benchmark)
- name: Seurat
  role: V3.0 pre-processing to obtain log-counts for each dataset before matching;
    also used as an integration baseline
- name: MNNcorrect
  role: Mutual-nearest-neighbor baseline for benchmarking the matching-based correction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:a62a6e328320feb86535998a9ef62aae72e82072ed585eed0062d0387007395f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mandricigor/batman
  repo_url: https://github.com/mandricigor/batman
  tools:
  - BATMAN
  - MNNcorrect
  - Scanorama
  - Seurat
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# minimum-weight-bipartite-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Core algorithmic step of BATMAN for scRNA-seq batch integration: anchors identified in each of two datasets are connected in a weighted bipartite graph (edges weighted by Euclidean distance), and a parsimonious one-to-one minimum-weight matching is found either exactly (blossom-based) or via a greedy approximation with ratio 2.

## When to use

Use when integrating two scRNA-seq datasets (e.g., log-count matrices pre-processed with Seurat V3.0) where batch correction must preserve local cell-to-cell structure, and a one-to-one matching of representative anchor cells across datasets is needed.

## When NOT to use

- You need to integrate more than two datasets — the current BATMAN implementation works only with two datasets (multi-dataset integration is possible only 'in principle').
- You have extra-large datasets where exact blossom matching would be too slow — prefer the -greedy heuristic, or a different method if even greedy matching is infeasible.
- Your goal requires matching cell types absent from one dataset (e.g., alpha cells held out of CEL-Seq2) — the matching leaves such cells unmatched by design.

## Inputs

- Two pre-processed scRNA-seq log-count datasets (e.g., Muraro CEL-Seq2, 2,285 cells; Segerstolpe Smart-Seq2, 2,394 cells), Seurat V3.0 log-counts
- Anchor count (-anchors) and optional Pearson correlation filtering threshold (-filter-cor)

## Outputs

- A minimum-weight one-to-one matching between anchors of the two datasets (exact blossom-based or greedy ratio-2 solution)
- Local shifting vectors and batch-corrected integrated expression matrix

## How to apply

First identify a set of anchor points in each dataset (number set with the -anchors parameter). Build a weighted bipartite graph whose partitions correspond to the two datasets, with each edge weighted by the Euclidean distance between the cells in N-dimensional gene space. Find the minimum-weight matching in this graph using either the exact blossom-based algorithm or the greedy approximation (approximation ratio 2, recommended for extra large datasets), then use the matched anchor pairs to determine local shifting vectors and perform batch correction. Optionally filter anchors with a Pearson correlation threshold (-filter-cor) before matching. This parsimonious matching is what lets BATMAN apply the minimal total correction necessary, unlike mutual-nearest-neighbor approaches that can distort local structure.

## Related tools

- **BATMAN** (Implements anchor identification, bipartite graph construction, minimum-weight matching (exact or greedy), local shifting vectors, and batch correction via batman.py) — https://github.com/mandricigor/batman
- **Scanorama** (Comparison baseline integration method run on the same preprocessed dataset pair (e.g., pancreas 1-held-out benchmark))
- **Seurat** (V3.0 pre-processing to obtain log-counts for each dataset before matching; also used as an integration baseline)
- **MNNcorrect** (Mutual-nearest-neighbor baseline for benchmarking the matching-based correction)

## Examples

```
python3.7 batman.py -data muraro_celseq2.csv,segerstolpe_smartseq2.csv -output integrated.csv -anchors 500 -greedy
```

## Evaluation signals

- Integrated data achieves improved mixing without destroying local structure: e.g., in the LB scenario BATMAN reached mean iLISI 1.84 (vs 1.82 original) while mean 50-RNN stayed at 0.92 (vs 1.00 original), far above baselines like Scanorama (0.14).
- Greedy approximation should stay within its guaranteed approximation ratio 2 of the exact blossom-based matching weight.
- Cells of a type absent from one dataset (e.g., alpha cells excluded from CEL-Seq2 in the 1-held-out pancreas setup) remain unmatched by all methods — an expected negative control.
- Corrected datasets should show cell-type-consistent alignment, measurable via cLISI alongside iLISI.
- CLI runs without errors for both matching modes (default exact and -greedy) and produces an output file at the -output path.

## Limitations

- The current implementation of BATMAN works only with two datasets; multi-dataset integration is not implemented.
- The greedy option is an approximation (ratio 2), trading optimality of the matching weight for speed on extra large datasets.
- Anchor count (-anchors) and the -filter-cor Pearson correlation threshold must be chosen; poor choices can degrade the matching.
- In the 1-held-out pancreas experiment, alpha cells from the Smart-Seq2 dataset were left unmatched by all methods, including BATMAN.

## Evidence

- [readme] -greedy               use greedy heuristic for the minimum weight matching
                        (recommended for extra large datasets): "-greedy               use greedy heuristic for the minimum weight matching
                        (recommended for extra large datasets)"
