---
name: single-cell-batch-integration
description: Use when two scRNA-seq count matrices from different technologies (e.g.,
  CEL-Seq2 and Smart-Seq2) need batch correction before joint analysis, and cell-type
  mixing (iLISI), cell-type consistency (cLISI), and local-structure retention (k-RNN)
  must be quantified.
when_to_use_negative:
- 'Integration of more than two datasets: the current BATMAN implementation works
  only with two datasets (multi-dataset integration is possible only ''in principle'')'
- Input data are already integrated/corrected embeddings rather than raw or log-count
  expression matrices requiring batch correction
- Merging heterogeneous modalities or reference-free annotation tasks where batch
  alignment across two scRNA-seq batches is not the analysis goal
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BATMAN
  role: 'Primary integration method: anchor identification, bipartite graph construction,
    minimum-weight matching, local shifting vectors, and batch correction'
  repo: https://github.com/mandricigor/batman
- name: Scanorama
  role: MNN-based state-of-the-art comparison integrator run on the same log-count
    data
- name: Seurat V3.0
  role: Provides the standard pre-processing to log-counts and serves as a comparison
    integration method
- name: MNNcorrect
  role: MNN-based comparison integrator for iLISI/cLISI/runtime benchmarking
- name: Harmony
  role: State-of-the-art integration tool cited in the article's benchmark context
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:79c6a179357b8faddf898ca9bc331291a3a06b844a2133849396e7c860c08154
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
  - BERMUDA
  - Harmony
  - MNNcorrect
  - Scanorama
  - Seurat V3.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# single-cell-batch-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates two single-cell RNA-seq datasets (e.g., Muraro CEL-Seq2 vs Segerstolpe Smart-Seq2 pancreas data) by minimum-weight matching of anchors across batches (BATMAN), removing batch effects while preserving local structure, and benchmarking against MNN-based methods (Seurat V3.0, MNNcorrect, Scanorama).

## When to use

Use when two scRNA-seq count matrices from different technologies (e.g., CEL-Seq2 and Smart-Seq2) need batch correction before joint analysis, and cell-type mixing (iLISI), cell-type consistency (cLISI), and local-structure retention (k-RNN) must be quantified.

## When NOT to use

- Integration of more than two datasets: the current BATMAN implementation works only with two datasets (multi-dataset integration is possible only 'in principle')
- Input data are already integrated/corrected embeddings rather than raw or log-count expression matrices requiring batch correction
- Merging heterogeneous modalities or reference-free annotation tasks where batch alignment across two scRNA-seq batches is not the analysis goal

## Inputs

- Two single-cell RNA-seq count matrices (e.g., Muraro et al. CEL-Seq2, 2,285 cells; Segerstolpe et al. Smart-Seq2, 2,394 cells), Seurat V3.0 pre-processed to log-counts
- Number of anchor points per dataset and optional Pearson-correlation filtering threshold for anchors
- Choice of matching algorithm: exact blossom-based or greedy approximation

## Outputs

- Batch-corrected (integrated) expression matrix for the two datasets
- Integration benchmark table (mean iLISI, cLISI, runtime per method) saved as a CSV, e.g., integration_benchmark.csv
- k-RNN comparison table of mean 50-RNN per method (Original 1.00, BATMAN 0.92, Seurat V3.0 0.58, MNNcorrect 0.51, Scanorama 0.14), e.g., krnn_scores.csv

## How to apply

Apply Seurat V3.0 pre-processing to each dataset to obtain log-counts, then run BATMAN: identify the set of anchors in each dataset (set with -anchors), build a weighted bipartite graph with edges weighted by Euclidean distance between cells in N-dimensional gene space, solve the minimum-weight one-to-one matching either with the exact blossom-based algorithm or the greedy approximation (approximation ratio 2, recommended for extra-large datasets via -greedy), determine local shifting vectors from matched pairs, and apply batch correction. Optionally filter anchors with -filter-cor (threshold on Pearson correlation between anchors across datasets). Run Seurat V3.0, MNNcorrect, and Scanorama on the same log-count data as comparators, then compute mean iLISI, mean cLISI, mean 50-RNN, and wall-clock runtime per method and assemble a comparison table (e.g., integration_benchmark.csv, krnn_scores.csv). For a stringent held-out design, exclude the most abundant cell type (alpha cells, 37% of CEL-Seq2 cells) from one dataset before integration and verify unmatched cell types.

## Related tools

- **BATMAN** (Primary integration method: anchor identification, bipartite graph construction, minimum-weight matching, local shifting vectors, and batch correction) — https://github.com/mandricigor/batman
- **Scanorama** (MNN-based state-of-the-art comparison integrator run on the same log-count data)
- **Seurat V3.0** (Provides the standard pre-processing to log-counts and serves as a comparison integration method)
- **MNNcorrect** (MNN-based comparison integrator for iLISI/cLISI/runtime benchmarking)
- **Harmony** (State-of-the-art integration tool cited in the article's benchmark context)

## Examples

```
python3.7 batman.py -data muraro_celseq2.csv,segerstolpe_smartseq2.csv -output batman_corrected.csv -anchors 100 -greedy
```

## Evaluation signals

- Mean iLISI improves over no correction: e.g., all-cell-types pancreas benchmark expects BATMAN iLISI 1.55 vs no correction 1.07, with cLISI remaining near 1.05-1.08
- Local-structure retention: mean 50-RNN should stay high for BATMAN (~0.92, close to Original 1.00) and drop substantially for MNN-based comparators (Seurat V3.0 0.58, MNNcorrect 0.51, Scanorama 0.14)
- In the 1-held-out condition (alpha cells excluded from CEL-Seq2), all methods should leave alpha cells from the Smart-Seq2 dataset unmatched; BATMAN achieves best iLISI (~1.39)
- Runtime sanity check: BATMAN should complete in seconds (~9 s on the pancreas pair), far faster than MNNcorrect (~228.6 s), with Scanorama ~14 s and Seurat V3.0 ~22.3 s
- Output CSVs (e.g., pancreas_1heldout_results.csv, krnn_scores.csv) contain per-method rows for iLISI, cLISI, runtime, and mean 50-RNN with the expected value patterns

## Limitations

- The current BATMAN implementation supports integration of only two datasets; multi-dataset integration is described as possible only 'in principle'
- The greedy matching is a heuristic with approximation ratio 2 (recommended only for extra-large datasets); the exact blossom-based matching may be slower
- MNN-based comparators may fail to both correct batch effects and maintain local dataset structure, so method choice strongly affects k-RNN retention

## Evidence

- [readme] Building a weighted bipartite graph where each partition corresponds to each of the datasets and the edges are weighted by the Euclidean distance: [evidence span withheld — anchor status: external_doc]
