---
name: f1-score-benchmarking
description: Use when comparing cell annotation algorithms (e.g., CellO IR/CLR, 1NN,
  scMatch, SingleR) on a labeled single-cell dataset, or when testing whether classification
  performance is stable across parameter settings such as Leiden clustering resolution.
when_to_use_negative:
- Input predictions are probabilities or binary decisions (<output_prefix>.probability.tsv
  or .binary.tsv) rather than most-specific cell type calls — the F1 evaluation here
  targets the most_specific table
- The dataset lacks trustworthy reference cell labels, since per-cell-type F1 computation
  requires ground truth
- Non-human RNA-seq data, since CellO is trained on human primary cell bulk RNA-seq
  from the SRA
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellO
  role: Classifier under evaluation; run via cello_predict with '-a IR' (isotonic
    regression) and '-a CLR' (cascaded logistic regression), plus the '-p' option
    to supply pre-computed Leiden clusterings
  repo: https://github.com/deweylab/CellO
- name: Scanpy
  role: Used to cluster cells with the Leiden algorithm at multiple resolution settings
    and export pre-clustering TSVs
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:310c865f5df18addceb55caf0e998958b5513add5e22fb56c6394bbb6fa70a46
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
  - Scanpy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# f1-score-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark hierarchical cell type classifiers such as CellO's IR and CLR by computing per-cell-type F1-scores against reference labels and aggregating them (e.g., median across cell types) for method comparison and robustness assessment.

## When to use

Use when comparing cell annotation algorithms (e.g., CellO IR/CLR, 1NN, scMatch, SingleR) on a labeled single-cell dataset, or when testing whether classification performance is stable across parameter settings such as Leiden clustering resolution.

## When NOT to use

- Input predictions are probabilities or binary decisions (<output_prefix>.probability.tsv or .binary.tsv) rather than most-specific cell type calls — the F1 evaluation here targets the most_specific table
- The dataset lacks trustworthy reference cell labels, since per-cell-type F1 computation requires ground truth
- Non-human RNA-seq data, since CellO is trained on human primary cell bulk RNA-seq from the SRA

## Inputs

- Gene expression matrix in 10x format (matrix.mtx, genes.tsv, barcodes.tsv), e.g., the Zheng et al. (2017) FAC-sorted PBMC dataset
- Reference cell type labels per cell (e.g., from Zheng et al. 2017)
- Classifier outputs: <output_prefix>.most_specific.tsv prediction tables from cello_predict runs
- Optional pre-clustering TSV files mapping cell IDs to cluster integers (compatible with the -p option)

## Outputs

- Per-cell-type F1-scores for each method/parameter combination
- Aggregated comparison table, e.g., f1_scores_by_resolution.csv with one row per algorithm x resolution
- Median F1-score per method for benchmark comparison (e.g., Figure 6D reproduction)

## How to apply

Run each classifier (e.g., cello_predict with '-a IR' and '-a CLR' on 10x-formatted Zheng PBMC data using '-d 10x -u COUNTS -s 3_PRIME') and read the '<output_prefix>.most_specific.tsv' prediction table. Match predicted cell types per cell to reference labels from Zheng et al. (2017) and compute a per-cell-type F1-score for every method. Aggregate into a table (one row per algorithm x parameter combination, e.g., f1_scores_by_resolution.csv) and take the median across cell types to compare methods, as done to show IR (0.97), CLR (0.96), and 1NN (0.95) ordering. For robustness testing, vary Leiden resolution (default 1.0 plus four alternatives, supplying each via the '-p' pre-clustering option) and confirm F1-scores show small variance across resolutions, reproducing the stability reported in Figure S15.

## Related tools

- **CellO** (Classifier under evaluation; run via cello_predict with '-a IR' (isotonic regression) and '-a CLR' (cascaded logistic regression), plus the '-p' option to supply pre-computed Leiden clusterings) — https://github.com/deweylab/CellO
- **Scanpy** (Used to cluster cells with the Leiden algorithm at multiple resolution settings and export pre-clustering TSVs)

## Examples

```
cello_predict -d 10x -u COUNTS -s 3_PRIME -a IR example_input/Zheng_PBMC_10x -o test
```

## Evaluation signals

- Median F1-scores across cell types should reproduce the reported ordering and magnitudes: IR 0.97, CLR 0.96, 1NN 0.95 on the Zheng PBMC dataset (Figure 6D)
- F1-scores across the five Leiden resolution settings should have small variance for both IR and CLR, indicating robustness (Figure S15)
- Each cello_predict run must produce the expected output files: <output_prefix>.probability.tsv, .binary.tsv, and .most_specific.tsv
- The aggregated table must contain one row per algorithm x resolution combination (10 rows for 2 algorithms x 5 resolutions)
- Pre-clustering TSVs must map cell IDs (column names of the input matrix) to integer cluster labels without column names, per the -p option spec

## Limitations

- Determining the optimal clustering in scRNA-seq data is a challenging, open problem; CellO's accuracy depends on the clustering resolution
- F1-score performance lags average precision performance; calibration for binary yes-no decisions needs improvement
- Confounding by sorting/preparation procedures such as FACS may affect gene expression and thus the labeled reference used for scoring

## Evidence

- [results] Robustness to Leiden resolution: [evidence span withheld — anchor status: not_found]
- [readme] README: description of the most_specific output table used for F1 scoring: [evidence span withheld — anchor status: not_found]
