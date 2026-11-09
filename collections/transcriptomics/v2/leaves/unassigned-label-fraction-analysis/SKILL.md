---
name: unassigned-label-fraction-analysis
description: Use when benchmarking scRNA-seq cell annotation methods on data where
  all true cell types are known, to check whether each method assigns labels exhaustively
  or dumps many cells as 'unassigned'/'intermediate'.
when_to_use_negative:
- When the data genuinely contain unknown or rare cell types — the paper recommends
  identifying those cells first with dedicated tools, and NeuCA is restricted to settings
  where all test cell types exist in the training data
- When the goal is annotation accuracy alone (accuracy/ARI) rather than exhaustiveness
  of label assignment
- When clustering rather than supervised label assignment is being evaluated — unassigned
  fractions are an artifact of label-assignment methods like CHETAH
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: NeuCA
  role: Exhaustive neural-network-based annotator used as the reference method whose
    outputs contain essentially no unassigned labels
  repo: https://bioconductor.org/packages/NeuCA
- name: CHETAH
  role: Comparison method whose unassigned/intermediate labels are quantified to demonstrate
    the unassigned-fraction failure mode
- name: scmap
  role: Comparison annotation method included in the unassigned-fraction benchmark
- name: CellAssign
  role: Comparison marker-based annotation method included in the benchmark
- name: SCINA
  role: Comparison marker-based annotation method included in the benchmark
- name: R
  role: Environment in which the methods are run and the unassigned-fraction metrics
    are computed and written to benchmark_results.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:c27c8e2d097ccdef341c2372a83f274fd53e35d24398b9a1734e700ac9722260
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHETAH
  - CellAssign
  - NeuCA
  - R
  - SC3
  - SCINA
  - Seurat
  - TSCAN
  - scmap
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# unassigned-label-fraction-analysis

## Summary

Quantify, per annotation method and per cell type, the fraction of cells left 'unassigned' after label assignment, to expose methods that exhaustively refuse to label cells even when no unknown cell types are expected. Used in the PBMC benchmark to show existing methods (e.g. CHETAH) leave >50% of naive T, CD4 T helper, and regulatory T cells unassigned under high-correlation conditions while NeuCA annotates exhaustively.

## When to use

Use when benchmarking scRNA-seq cell annotation methods on data where all true cell types are known, to check whether each method assigns labels exhaustively or dumps many cells as 'unassigned'/'intermediate'.

## When NOT to use

- When the data genuinely contain unknown or rare cell types — the paper recommends identifying those cells first with dedicated tools, and NeuCA is restricted to settings where all test cell types exist in the training data
- When the goal is annotation accuracy alone (accuracy/ARI) rather than exhaustiveness of label assignment
- When clustering rather than supervised label assignment is being evaluated — unassigned fractions are an artifact of label-assignment methods like CHETAH

## Inputs

- Predicted label vectors per method per split (e.g. NeuCA's predicted.label output)
- True cell type labels for the held-out PBMC test cells
- Benchmark splits of normalized 10X Genomics PBMC expression matrices subset to overlapped genes

## Outputs

- Per-method, per-cell-type unassigned-label fraction table
- benchmark_results.csv with accuracy, ARI, and unassigned fractions per method per split

## How to apply

For each simulated split (training proportions p = 10%, 20%, 50%, 80% and test sizes n = 800, 1600, 4000 from 10X Genomics PBMC data), run each annotation method — NeuCA, scmap, CHETAH, CellAssign, SCINA — and record predicted labels. For every method and cell type, compute the fraction of test cells receiving an 'unassigned' label, alongside accuracy and ARI. Contrast the difficult setting (five highly correlated T-cell types: CD4 T helper, memory T, naive cytotoxic T, naive T, regulatory T) with the easy setting (five T-cell types merged into one 'T cells' type among major cell types). Judge correctness by verifying that existing methods assign >50% 'unassigned' labels to naive T, CD4 T helper, and regulatory T cells in the difficult setting, that CD56 NK and CD14 monocytes are annotated correctly by all methods, and that NeuCA stays exhaustive while leading in accuracy and ARI; write all metrics to benchmark_results.csv.

## Related tools

- **NeuCA** (Exhaustive neural-network-based annotator used as the reference method whose outputs contain essentially no unassigned labels) — https://bioconductor.org/packages/NeuCA
- **CHETAH** (Comparison method whose unassigned/intermediate labels are quantified to demonstrate the unassigned-fraction failure mode)
- **scmap** (Comparison annotation method included in the unassigned-fraction benchmark)
- **CellAssign** (Comparison marker-based annotation method included in the benchmark)
- **SCINA** (Comparison marker-based annotation method included in the benchmark)
- **R** (Environment in which the methods are run and the unassigned-fraction metrics are computed and written to benchmark_results.csv)

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE); frac_unassigned <- mean(predicted.label == "unassigned")
```

## Evaluation signals

- In the difficult five-T-cell-type setting, existing methods assign >50% 'unassigned' labels to naive T, CD4 T helper, and regulatory T cells
- CD56 NK and CD14 monocytes are annotated correctly by all methods (their unassigned fractions stay low)
- NeuCA's predicted.label vector covers every test cell with a concrete type label (no unassigned category) in the easy setting
- Per-split accuracy and ARI confirm NeuCA leads all methods in the difficult setting while unassigned fractions are recorded
- All metrics for every method/split combination are present in benchmark_results.csv

## Limitations

- High unassigned fractions are method-specific: CHETAH produces unignorable amounts of unassigned/intermediate labels even when researchers expect no unknown cell types
- NeuCA is designed for settings where all cell types in the new sample already exist in the reference/training data; exhaustive assignment is inappropriate if novel types are truly present
- Deciding whether unknown/novel cell types exist in the data is not addressed; expert consultation is recommended for tissues from novel conditions

## Evidence

- [abstract] existing methods label a considerable number of cells as unassigned even when unknown cell types are not expected: "when unknown cell types are not expected, existing methods still label a considerable number of cells as unassigned, which is not desirable"
- [intro] CHETAH unassigned labels: "CHETAH still produces an unignorable amount of unassigned/intermediate cell labels"
- [methods] NeuCA correlation threshold: [evidence span withheld — anchor status: not_found]
- [readme] NeuCA predicted label output: "`predicted.label` is a vector of the same length with the number of cells in the testing dataset, containing all cell’s predicted cell type"
- [abstract] NeuCA hierarchical rationale: "NeuCA can utilize the hierarchical structure information of the cell types to improve the annotation accuracy, which is especially helpful when data contain closely correlated cell types"
