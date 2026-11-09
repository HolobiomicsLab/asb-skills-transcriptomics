---
name: annotation-accuracy-ari-evaluation
description: Use when comparing cell annotation methods on scRNA-seq data with known
  reference labels — e.g. difficult PBMC splits with p = 10/20/50/80% training proportions
  or ASD vs control cross-condition directions — to rank methods by accuracy and ARI.
when_to_use_negative:
- No true labels are available for the testing data — accuracy and ARI cannot be computed
  and only qualitative checks (e.g. marker concordance) are possible.
- The sample may contain unknown or novel cell types absent from the training data
  — NeuCA's exhaustive assignment is restricted to settings where all cell types in
  the new sample already exist in the reference data.
- Goal is cluster discovery rather than label transfer evaluation — use unsupervised
  clustering validation metrics, not accuracy against a fixed reference.
tools:
- name: NeuCA
  role: Supervised neural-network annotation method being benchmarked; its predicted.label
    vector is evaluated against true labels
  repo: https://bioconductor.org/packages/NeuCA/
- name: scmap
  role: Baseline supervised annotation method (scmap-cluster, scmap-cell) evaluated
    in the same accuracy/ARI framework
- name: CHETAH
  role: Baseline hierarchical supervised method whose unassigned/intermediate label
    fraction is tracked alongside accuracy
- name: Seurat
  role: Baseline unsupervised clustering method scored by accuracy and ARI in cross-condition
    benchmarks
- name: SC3
  role: Baseline unsupervised clustering method scored by accuracy and ARI in cross-condition
    benchmarks
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:79d018f5d66e6222f0e4600a1226353d7e10f1c1c76b1da549c786c52194a9e9
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# annotation-accuracy-ari-evaluation

## Summary

Evaluate single-cell label assignment by computing annotation accuracy and adjusted Rand index (ARI) of predicted cell labels against published true labels, used in the NeuCA paper to benchmark NeuCA against scmap-cluster, scmap-cell, CHETAH, CellAssign, SCINA, Seurat, and SC3 across intra-study, inter-study, and cross-condition settings.

## When to use

Use when comparing cell annotation methods on scRNA-seq data with known reference labels — e.g. difficult PBMC splits with p = 10/20/50/80% training proportions or ASD vs control cross-condition directions — to rank methods by accuracy and ARI.

## When NOT to use

- No true labels are available for the testing data — accuracy and ARI cannot be computed and only qualitative checks (e.g. marker concordance) are possible.
- The sample may contain unknown or novel cell types absent from the training data — NeuCA's exhaustive assignment is restricted to settings where all cell types in the new sample already exist in the reference data.
- Goal is cluster discovery rather than label transfer evaluation — use unsupervised clustering validation metrics, not accuracy against a fixed reference.

## Inputs

- Predicted cell label vectors from each annotation method (NeuCA, scmap-cluster, scmap-cell, CHETAH, CellAssign, SCINA, Seurat, SC3)
- Published true cell type labels for the testing dataset
- Preprocessed SingleCellExperiment objects with normcounts and overlapped genes
- Split specifications (training proportions p and test sizes n)

## Outputs

- Per-method annotation accuracy and ARI for every split and direction
- Fraction of 'unassigned' labels per method per cell type
- Benchmark summary tables (benchmark_results.csv, asd_crosscondition_benchmark.csv)
- Confusion matrix and Sankey diagram of predicted vs true labels

## How to apply

For each train–test split (e.g. training proportions p = 10%, 20%, 50%, 80% with test sizes n = 800, 1600, 4000 on the 10X PBMC data, or the ASD-vs-control cross-condition directions), run every method on the same preprocessed data (normalized matrices subset to overlapped genes). Collect each method's predicted label vector and compare it against the published true labels. Compute per-method annotation accuracy (fraction of exactly matched labels) and the adjusted Rand index (ARI) for every split and direction. Also record the fraction of 'unassigned' labels per method per cell type, since CHETAH in particular produces unassigned/intermediate labels when researchers expect few or no unknown types. Compile the metrics into a benchmark table (e.g. benchmark_results.csv or asd_crosscondition_benchmark.csv) and verify expected orderings such as NeuCA highest on the five correlated T-cell types and scmap-cell second-best among supervised methods.

## Related tools

- **NeuCA** (Supervised neural-network annotation method being benchmarked; its predicted.label vector is evaluated against true labels) — https://bioconductor.org/packages/NeuCA/
- **scmap** (Baseline supervised annotation method (scmap-cluster, scmap-cell) evaluated in the same accuracy/ARI framework)
- **CHETAH** (Baseline hierarchical supervised method whose unassigned/intermediate label fraction is tracked alongside accuracy)
- **Seurat** (Baseline unsupervised clustering method scored by accuracy and ARI in cross-condition benchmarks)
- **SC3** (Baseline unsupervised clustering method scored by accuracy and ARI in cross-condition benchmarks)

## Examples

```
table(predicted.label, Seg_true_cell_label)
```

## Evaluation signals

- NeuCA achieves the highest accuracy and ARI in both ASD→control and control→ASD cross-condition directions.
- On the difficult five-T-cell-type PBMC setting, NeuCA leads all methods in accuracy and ARI; scmap-cell is second-best supervised with accurately assigned rate around 0.7.
- Existing methods assign >50% 'unassigned' labels to naive T, CD4 T helper, and regulatory T cells, while CD56 NK and CD14 monocytes are annotated correctly by all methods.
- Confusion matrix of predicted.label vs Seg_true_cell_label is near-diagonal (e.g. alpha 328/331 correct) when labels are accurate.
- Benchmark CSV contains one accuracy and one ARI value per method per direction/split with no missing entries.

## Limitations

- NeuCA is restricted to settings where all cell types in the new sample already exist in the reference/training data; novel cell types require a separate identification step.
- Cross-condition prediction success (ASD data) may not hold in scenarios other than the ASD data application.
- Deciding whether unknown/novel cell types exist in the data is not addressed; consulting biological/clinical experts is recommended.
- No version numbers are given for keras, TensorFlow, or R in the software description.

## Evidence

- [abstract] Stable cross-condition performance claim: "the applications on eight real datasets show that NeuCA has stable performance for intra- and inter-study annotation, as well as cross-condition annotation"
- [readme] README evaluation via confusion matrix: "If you have the true cell type labels for the testing dataset, you may evaluate the predictive performance by a confusion matrix"
- [intro] Supervised vs unsupervised benchmark rationale: "when training data are available, the improved accuracy, speed, and reproducibility of supervised methods easily outperform unsupervised ones"
- [intro] CHETAH unassigned-label issue motivating per-method unassigned rates: "CHETAH still produces an unignorable amount of unassigned/intermediate cell labels"
