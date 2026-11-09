---
name: single-cell-rna-seq-cell-type-annotation
description: Use when annotating human single-cell RNA-seq cells with standardized,
  ontology-grounded cell type labels without supplying marker genes or a user-labeled
  training set, especially when uncertain cells should be placed at ancestral ontology
  nodes.
when_to_use_negative:
- The data are non-human (e.g. mouse); CellO's training set covers only human primary
  samples and mouse training would require curated SRA training data not yet available
- The user already has a curated marker gene set or a labeled reference specific to
  their tissue, where a flat classifier against that reference may suffice
- Cell type labels for lineages absent from CellO's bulk RNA-seq training set are
  required at leaf-level specificity; CellO can only place such cells at ancestral
  ontology nodes (e.g. acinar cells annotated as glandular epithelial cells)
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: CellO
  role: Primary hierarchical classifier annotating cells against the Cell Ontology
    using IR or CLR algorithms with pre-trained SRA bulk RNA-seq models
  repo: https://github.com/deweylab/CellO
- name: CellO Viewer
  role: Web application for exploring the gene-wise linear model coefficients (cell
    type expression signatures) learned by CellO across the ontology
  repo: https://github.com/uwgraphics/CellOViewer
- name: Scanpy
  role: Python library through which CellO's API integrates into a single-cell analysis
    pipeline, e.g. for Leiden clustering passed via the -p option
  repo: https://github.com/deweylab/CellO
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_004
  - task_008
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:917feda1a5662bae271983b3d4460d6655b95c1fec214afa35604378d4a3cf64
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
  - CellO Viewer
  - Python
  - Scanpy
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# single-cell-rna-seq-cell-type-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Hierarchical cell type classification of human scRNA-seq data against the Cell Ontology using CellO, a supervised tool pre-trained on human primary cell bulk RNA-seq data from the SRA, benchmarked against scMatch, SingleR, and 1NN baselines.

## When to use

Use when annotating human single-cell RNA-seq cells with standardized, ontology-grounded cell type labels without supplying marker genes or a user-labeled training set, especially when uncertain cells should be placed at ancestral ontology nodes.

## When NOT to use

- The data are non-human (e.g. mouse); CellO's training set covers only human primary samples and mouse training would require curated SRA training data not yet available
- The user already has a curated marker gene set or a labeled reference specific to their tissue, where a flat classifier against that reference may suffice
- Cell type labels for lineages absent from CellO's bulk RNA-seq training set are required at leaf-level specificity; CellO can only place such cells at ancestral ontology nodes (e.g. acinar cells annotated as glandular epithelial cells)

## Inputs

- Human single-cell RNA-seq expression matrix in 10x format (matrix.mtx, genes.tsv, barcodes.tsv), TSV, CSV, or HDF5
- Optional TSV of pre-clustered cells (cell IDs in column 1, cluster integers in column 2, no header) for the -p option
- Optional comma-separated Uberon ontology term IDs for anatomical filtering (-l)
- Pre-trained CellO model file (.model.dill) or SRA bulk RNA-seq training data from Zenodo deposit 10.5281/zenodo.4289064 for retraining with -t

## Outputs

- <output_prefix>.probability.tsv — N cells x M cell types classification probability table
- <output_prefix>.binary.tsv — NxM binary-decision matrix of predicted cell types
- <output_prefix>.most_specific.tsv — per-cell mapping to the most-specific predicted cell type
- <output_prefix>.model.dill — retrained model when the -t flag is used
- Per-cell-type F1-score, precision, and average-precision metric tables (e.g. TSV/CSV) for benchmarking

## How to apply

Install CellO with `pip install cello-classify` and run `cello_predict` on the expression matrix, specifying the input format (-d 10x/TSV/CSV/HDF5), expression units (-u COUNTS/CPM/LOG1_CPM/TPM/LOG1_TPM), and assay (-s 3_PRIME or FULL_LENGTH) matching the dataset. Use the default isotonic regression algorithm (-a IR) or cascaded logistic regression (-a CLR); IR achieved median F1 of 0.97 and CLR 0.96 on the Zheng et al. 10x PBMC data, outperforming a 1NN baseline (0.95) and the flat ready-to-run tools scMatch and SingleR after projecting their labels onto the Cell Ontology. The input gene set must match the pre-trained classifier's genes; if not, retrain with the -t flag so a model is fit on only the genes present, saved as <output_prefix>.model.dill. Optionally supply pre-computed Leiden clusters via -p (performance is robust across five resolution settings) or filter tissue-resident cell types with -l using Uberon term IDs. Collect the three outputs — <output_prefix>.probability.tsv (per-cell probabilities), <output_prefix>.binary.tsv (binary decisions), and <output_prefix>.most_specific.tsv (most-specific label per cell) — and validate F1/precision metrics against the paper's reported medians.

## Related tools

- **CellO** (Primary hierarchical classifier annotating cells against the Cell Ontology using IR or CLR algorithms with pre-trained SRA bulk RNA-seq models) — https://github.com/deweylab/CellO
- **CellO Viewer** (Web application for exploring the gene-wise linear model coefficients (cell type expression signatures) learned by CellO across the ontology) — https://github.com/uwgraphics/CellOViewer
- **Scanpy** (Python library through which CellO's API integrates into a single-cell analysis pipeline, e.g. for Leiden clustering passed via the -p option) — https://github.com/deweylab/CellO

## Examples

```
cello_predict -d 10x -u COUNTS -s 3_PRIME example_input/Zheng_PBMC_10x -o test
```

## Evaluation signals

- Median per-cell-type F1-scores on the 4,936-cell non-droplet test set should approximate 0.81 (IR), 0.85 (CLR), and 0.63 (1NN) within small tolerance
- On the Zheng et al. 10x PBMC data, IR, CLR, and 1NN median F1-scores should be approximately 0.97, 0.96, and 0.95, with IR/CLR competitive with or superior to scMatch and SingleR (Figure 5B/6D)
- All three output tables (probability.tsv, binary.tsv, most_specific.tsv) must be populated for every input cell, with binary decisions consistent with probabilities
- F1-scores across at least five Leiden resolution settings should show small variance for both IR and CLR, indicating robustness to clustering granularity
- On challenging datasets, spot-check specific calls: Segerstolpe acinar cells as glandular epithelial cells, La Manno microglia as phagocytes, and correct endothelial/fibroblast/plasmacytoid dendritic cell labels in LX675/LX679/LX682 tumors

## Limitations

- F1-score performance lags average precision; calibration of the binary yes-no decisions is imperfect and expected to improve only as more per-cell-type training data become available
- Specific cell types absent from the bulk RNA-seq training set can only be annotated at ancestral Cell Ontology terms, not at leaf specificity
- Performance depends on Leiden clustering, and the optimal clustering resolution for scRNA-seq annotation remains an open problem
- It is unclear to what extent sorting/preparation procedures such as FACS affect gene expression and confound cell type
- The input gene set must match the pre-trained classifier's genes, otherwise a new model must be trained with -t; independent classifiers can be inconsistent for cell types sharing an 'is a' parent but with divergent evolutionary origins

## Evidence

- [results] IR and CLR outperformed the 1NN baseline, producing median F1-scores of 0.81, 0.85, and 0.63 on the non-droplet test set: [evidence span withheld — anchor status: not_found]
- [results] acinar cells, whose specific type is absent from the training set, annotated as glandular epithelial cells: [evidence span withheld — anchor status: not_found]
