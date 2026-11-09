---
name: leiden-resolution-sensitivity-analysis
description: Use when validating that a cluster-based cell type annotation pipeline
  (e.g., CellO with pre-clustering via -p) is robust to the choice of Leiden clustering
  resolution, or when reporting sensitivity of F1-scores across clustering granularity
  settings.
when_to_use_negative:
- When the goal is to find the single optimal clustering resolution rather than to
  assess robustness — the paper notes determining optimal clustering in scRNA-seq
  is a challenging, open problem outside this skill's scope
- When the input expression matrix genes do not match CellO's pretrained training
  set and no compatible model is available (CellO requires matching genes unless a
  new model is trained with -t)
- When annotating non-human species — CellO is trained only on human primary cell
  bulk RNA-seq from the SRA
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellO
  role: Hierarchical cell type classifier run once per resolution with -a IR and -a
    CLR, using pre-computed clusters supplied via -p
  repo: https://github.com/deweylab/CellO
- name: Scanpy
  role: Performs Leiden clustering at multiple resolution settings and exports pre-clustering
    TSVs for CellO
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:eff8629353445c19dfa50fd79121f1c4f35ba63357e3b82219751dba37b4b161
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# leiden-resolution-sensitivity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A robustness assessment skill that tests whether CellO's hierarchical cell type classification (IR and CLR algorithms) is sensitive to the granularity of Leiden clustering by sweeping multiple resolution values and comparing per-cell-type F1-scores.

## When to use

Use when validating that a cluster-based cell type annotation pipeline (e.g., CellO with pre-clustering via -p) is robust to the choice of Leiden clustering resolution, or when reporting sensitivity of F1-scores across clustering granularity settings.

## When NOT to use

- When the goal is to find the single optimal clustering resolution rather than to assess robustness — the paper notes determining optimal clustering in scRNA-seq is a challenging, open problem outside this skill's scope
- When the input expression matrix genes do not match CellO's pretrained training set and no compatible model is available (CellO requires matching genes unless a new model is trained with -t)
- When annotating non-human species — CellO is trained only on human primary cell bulk RNA-seq from the SRA

## Inputs

- Zheng_PBMC_10x dataset (10x format: matrix.mtx, genes.tsv, barcodes.tsv; COUNTS units; 3_PRIME assay)
- Five Leiden pre-clustering TSV files (cell IDs mapped to cluster integers, no header) at resolutions including 1.0
- CellO pretrained model file (e.g., .model.dill)
- Reference cell labels from Zheng et al. (2017)

## Outputs

- Ten sets of CellO predictions (<output_prefix>.most_specific.tsv, .binary.tsv, .probability.tsv) — one per algorithm x resolution
- f1_scores_by_resolution.csv with per-cell-type F1-scores for each algorithm x resolution combination
- Robustness conclusion (stable vs. unstable F1 variance across resolutions)

## How to apply

Load a 10x-format scRNA-seq dataset (e.g., the Zheng_PBMC_10x example: 1,000 cells subsampled from Zheng et al. 2017, COUNTS units, 3_PRIME assay) and cluster cells with the Leiden algorithm at five resolution settings (default 1.0 plus four alternatives) using Scanpy, exporting each clustering as a pre-clustering TSV mapping cell IDs to cluster integers (no column headers) compatible with CellO's -p option. For each resolution, run cello_predict twice — once with isotonic regression (-a IR) and once with cascaded logistic regression (-a CLR) — supplying the corresponding pre-clustering file and the pretrained model. From each run's <output_prefix>.most_specific.tsv predictions, compute per-cell-type F1-scores against the reference cell labels from Zheng et al. (2017). Aggregate all F1-scores into a single table (one row per algorithm x resolution combination, saved as f1_scores_by_resolution.csv) and check that scores are stable (small variance) across resolutions, reproducing the robustness shown in Figure S15.

## Related tools

- **CellO** (Hierarchical cell type classifier run once per resolution with -a IR and -a CLR, using pre-computed clusters supplied via -p) — https://github.com/deweylab/CellO
- **Scanpy** (Performs Leiden clustering at multiple resolution settings and exports pre-clustering TSVs for CellO)

## Examples

```
cello_predict -d 10x -u COUNTS -s 3_PRIME -a IR -p clusters_res1.0.tsv -m cello.model.dill -o out_res1.0 example_input/Zheng_PBMC_10x
```

## Evaluation signals

- Each pre-clustering TSV has exactly two columns (cell IDs and integer clusters), no header, and covers all cells in the input expression matrix
- The F1 table contains one row per algorithm x resolution combination (2 algorithms x 5 resolutions = 10 rows)
- F1-scores across the five resolutions show small variance for both IR and CLR, indicating robustness to clustering granularity
- Each cello_predict run produced <output_prefix>.most_specific.tsv and no <output_prefix>.log/clustering.tsv (confirming pre-computed clusters were used)
- The variance pattern is consistent with the robustness reported in Figure S15

## Limitations

- Determining the optimal clustering in scRNA-seq data remains a challenging, open problem; this skill only measures sensitivity, it does not identify a best resolution
- Robustness conclusions are specific to the tested dataset (Zheng et al. PBMC) and may not transfer to other tissues or assays
- Results depend on the pretrained CellO model matching the input gene set; mismatched genes require retraining with -t

## Evidence

- [results] CellO was tested with five Leiden resolution parameter values on the Zheng et al. PBMC data set, and both IR and CLR performed similarly across these settings: [evidence span withheld — anchor status: not_found]
