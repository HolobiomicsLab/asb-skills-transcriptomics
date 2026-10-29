---
name: classifier-baseline-comparison
description: Use when evaluating a new cell type annotation method on a labeled scRNA-seq
  dataset and you need a simple baseline (1NN) plus tools (scMatch, SingleR) for fair
  comparison via median F1.
when_to_use_negative:
- The input data has no ground-truth cell type labels, so F1-scores cannot be computed
  and only qualitative annotation is possible
- The dataset is non-human or non-primary-cell tissue, since CellO's pretrained models
  cover only human primary cell bulk RNA-seq from the SRA
- You need a calibrated yes/no binary decision per cell type rather than a ranking
  comparison — the paper notes CellO's F1-score performance lags its average precision
  due to calibration gaps
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: CellO
  role: Hierarchical classifier under evaluation, run via cello_predict with IR (isotonic
    regression) and CLR (cascaded logistic regression) algorithms
  repo: https://github.com/deweylab/CellO
- name: SingleR
  role: Reference-based baseline classifier run with Blueprint+ENCODE, HPCA, and Monaco
    immune reference sets
- name: scMatch
  role: Reference-based baseline classifier run with Blueprint+ENCODE, HPCA, and Monaco
    immune reference sets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:69419fd1dc2c3c63e6cd8a128456c111600d06e0424966bd563cdc6011349037
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deweylab/CellO
  repo_url: https://github.com/deweylab/CellO
  tools:
  - CellO
  - SingleR
  - scMatch
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# classifier-baseline-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark CellO's hierarchical classifiers (isotonic regression, IR; cascaded logistic regression, CLR) against a simple 1NN baseline and reference-based methods (scMatch, SingleR) by computing per-cell-type F1-scores and taking the median across cell types.

## When to use

Use when evaluating a new cell type annotation method on a labeled scRNA-seq dataset and you need a simple baseline (1NN) plus state-of-the-art tools (scMatch, SingleR) for fair comparison via median F1.

## When NOT to use

- The input data has no ground-truth cell type labels, so F1-scores cannot be computed and only qualitative annotation is possible
- The dataset is non-human or non-primary-cell tissue, since CellO's pretrained models cover only human primary cell bulk RNA-seq from the SRA
- You need a calibrated yes/no binary decision per cell type rather than a ranking comparison — the paper notes CellO's F1-score performance lags its average precision due to calibration gaps

## Inputs

- 10x-formatted gene expression matrix directory (matrix.mtx, genes.tsv, barcodes.tsv) for the Zheng et al. (2017) PBMC dataset, subsampled to 20,000 FAC-sorted cells
- Ground-truth per-cell cell type labels for F1-score computation
- Immune reference sets for baseline classifiers: Blueprint+ENCODE, HPCA, and Monaco

## Outputs

- CellO output tables per algorithm: <prefix>.probability.tsv (N cells x M cell types), <prefix>.binary.tsv, and <prefix>.most_specific.tsv
- Per-cell-type F1-scores for each method (IR, CLR, 1NN, scMatch, SingleR)
- Median F1-score across all cell types per method, for ranking (Figure 6D-style comparison)

## How to apply

Run CellO's cello_predict on the target expression matrix once per hierarchical algorithm, using '-a IR' (default) and '-a CLR' with the appropriate data type, units, and assay flags (e.g. '-d 10x -u COUNTS -s 3_PRIME'). Run a 1NN nearest-neighbor baseline classifier on the same input features, then run scMatch and SingleR with immune reference sets (Blueprint+ENCODE, HPCA, Monaco). For every method, compute per-cell-type F1-scores against ground-truth labels and take the median F1 across all cell types. Compare methods by the relative ordering of these medians (in the paper: IR 0.97 > CLR 0.96 > 1NN 0.95 on the FAC-sorted 20,000-cell Zheng et al. 2017 PBMC subset), checking that the learned hierarchical methods beat the simple baseline and are competitive with or superior to scMatch and SingleR (Figure 6D).

## Related tools

- **CellO** (Hierarchical classifier under evaluation, run via cello_predict with IR (isotonic regression) and CLR (cascaded logistic regression) algorithms) — https://github.com/deweylab/CellO
- **SingleR** (Reference-based baseline classifier run with Blueprint+ENCODE, HPCA, and Monaco immune reference sets)
- **scMatch** (Reference-based baseline classifier run with Blueprint+ENCODE, HPCA, and Monaco immune reference sets)

## Examples

```
cello_predict -d 10x -u COUNTS -s 3_PRIME -a CLR Zheng_PBMC_10x -o test_clr
```

## Evaluation signals

- Median F1-score ordering reproduces IR > CLR > 1NN (0.97 vs 0.96 vs 0.95 on the Zheng PBMC benchmark)
- IR and CLR median F1 are competitive with or superior to scMatch and SingleR baselines (Figure 6D)
- cello_predict produces all three expected output files: .probability.tsv, .binary.tsv, and .most_specific.tsv
- Per-cell-type F1-scores are computed for every method on the same cell set and same ground-truth labels before taking the median
- 1NN baseline uses the same input features/units (e.g. COUNTS on 3_PRIME assay data) as the CellO runs for a fair comparison

## Limitations

- CellO's classifier calibration is imperfect: F1-score performance lags average precision, so binary yes-no decisions may underperform ranking quality
- The extent to which FACS/sorting procedures confound gene expression versus cell type remains unclear, which may affect benchmark fairness on sorted datasets
- Determining the optimal clustering resolution in scRNA-seq data is an open problem that affects CellO's cluster-based accuracy

## Evidence

- [results] benchmark finding: [evidence span withheld — anchor status: not_found]
