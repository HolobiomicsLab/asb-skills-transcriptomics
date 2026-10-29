---
name: cell-type-held-out-benchmarking
description: Use when benchmarking scRNA-seq integration methods on two datasets with
  known cell-type compositions and you want to verify that a cell type present in
  only one dataset is left unmatched instead of incorrectly aligned.
when_to_use_negative:
- 'When the current implementation constraint matters: BATMAN only supports integration
  of two datasets, so multi-dataset benchmarks need other approaches'
- When the goal is to integrate (not benchmark) datasets containing genuinely shared
  cell types — removing a cell type is a stringency test, not a standard integration
  step
edam_operation: http://edamontology.org/operation_0566
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BATMAN
  role: 'Primary integration method: minimum-weight matching batch correction via
    anchors, Euclidean-distance-weighted bipartite graph, and local shifting vectors'
  repo: https://github.com/mandricigor/batman
- name: Scanorama
  role: State-of-the-art comparison method run on the same preprocessed dataset pair
- name: Seurat
  role: Provides V3.0 pre-processing to obtain log-counts and serves as a comparison
    integration method
- name: MNNcorrect
  role: MNN-based comparison integration method run on the same dataset pair
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:57a69a3fcc7edb839d6de877694a73d0327b145443e8a5b767b8a3d4438bb0c5
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
  edam_operation: http://edamontology.org/operation_0566
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cell-type-held-out-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A stringent scRNA-seq integration benchmark design in which the most abundant cell type is removed from one dataset before integration, to test whether methods can correctly leave unmatched cell types (e.g., alpha cells) rather than forcibly mixing them.

## When to use

Use when benchmarking scRNA-seq integration methods on two datasets with known cell-type compositions and you want to verify that a cell type present in only one dataset is left unmatched instead of incorrectly aligned.

## When NOT to use

- When the current implementation constraint matters: BATMAN only supports integration of two datasets, so multi-dataset benchmarks need other approaches
- When the goal is to integrate (not benchmark) datasets containing genuinely shared cell types — removing a cell type is a stringency test, not a standard integration step

## Inputs

- Muraro et al. CEL-Seq2 pancreatic scRNA-seq dataset (2,285 cells)
- Segerstolpe et al. Smart-Seq2 pancreatic scRNA-seq dataset (2,394 cells)
- Cell-type annotations used to identify and exclude alpha cells (37% of CEL-Seq2 cells)

## Outputs

- Integrated (batch-corrected) expression matrices from BATMAN, Scanorama, Seurat V3.0, and MNNcorrect
- Mean iLISI and cLISI scores per method
- Wall-clock runtimes per method
- Comparison table (e.g., pancreas_1heldout_results.csv) with iLISI, cLISI, and runtime columns

## How to apply

Load the Muraro et al. CEL-Seq2 (2,285 cells) and Segerstolpe et al. Smart-Seq2 (2,394 cells) pancreas datasets and apply Seurat V3.0 pre-processing to obtain log-counts. Exclude the most abundant cell type — alpha cells, 37% of cells in the CEL-Seq2 dataset — to set up the 1-held-out condition. Run BATMAN (bipartite graph with edges weighted by Euclidean distance, minimum-weight matching via the exact blossom-based algorithm or greedy approximation with ratio 2, local shifting vectors, batch correction) alongside Scanorama, Seurat V3.0, and MNNcorrect on the same preprocessed pair. Compute mean iLISI and cLISI on the integrated data, verify that alpha cells from the Smart-Seq2 dataset remain unmatched by all methods, and record wall-clock runtimes. In the reference run, BATMAN achieves the best iLISI (1.39) while MNNcorrect and Scanorama produce the worst, and all methods leave the alpha cells unmatched.

## Related tools

- **BATMAN** (Primary integration method: minimum-weight matching batch correction via anchors, Euclidean-distance-weighted bipartite graph, and local shifting vectors) — https://github.com/mandricigor/batman
- **Scanorama** (State-of-the-art comparison method run on the same preprocessed dataset pair)
- **Seurat** (Provides V3.0 pre-processing to obtain log-counts and serves as a comparison integration method)
- **MNNcorrect** (MNN-based comparison integration method run on the same dataset pair)

## Examples

```
python3.7 batman.py -data celseq2_alpha_removed.csv,smartseq2.csv -output integrated.csv -anchors 100
```

## Evaluation signals

- BATMAN should achieve the best mean iLISI in the 1-held-out experiment (reference value: 1.39)
- MNNcorrect and Scanorama should produce the worst iLISI scores in this condition
- Alpha cells from the Smart-Seq2 dataset should remain unmatched by all tested methods — this is the key correctness invariant of the held-out design
- All methods must receive identical Seurat V3.0 log-count pre-processed inputs so iLISI/cLISI comparisons are fair
- Wall-clock runtimes should be recorded for every method alongside iLISI and cLISI in the results table

## Limitations

- The current BATMAN implementation works only with two datasets; multi-dataset integration is not implemented
- Alpha cells constitute 37% of CEL-Seq2 and 42% of Smart-Seq2 cells, so the held-out removal substantially changes dataset composition
- The greedy matching option is a heuristic with approximation ratio 2, recommended only for extra large datasets

## Evidence
