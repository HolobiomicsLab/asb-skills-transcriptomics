---
name: tumor-microenvironment-cell-identification
description: Use when annotating human scRNA-seq cells whose exact types may be missing
  from the classifier's training set (e.g. endothelial, fibroblast, plasmacytoid dendritic
  cells in LX675/LX679/LX682 lung tumor samples), and hierarchical Cell Ontology-consistent
  labels are wanted.
when_to_use_negative:
- 'Non-human data: CellO is trained on human primary cell bulk RNA-seq from the SRA,
  and no comprehensive curated mouse training set exists yet.'
- Single-cell datasets whose genes are expected to match an already-trained model
  but don't, unless you are willing to retrain with -t on the reduced gene set.
- Flat, non-hierarchical labeling tasks where you need leaf-node-only labels; CellO
  may deliberately place cells at internal (ancestral) ontology nodes when uncertain.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CellO
  role: Hierarchical cell type classifier run via cello_predict against the Cell Ontology;
    used with IR (isotonic regression) algorithm, optional retraining (-t) on datasets
    with incompatible gene sets
  repo: https://github.com/deweylab/CellO
- name: CellO Viewer
  role: Web application for exploring CellO's model coefficients (cell type expression
    signatures) across the ontology when interpreting tumor-microenvironment calls
  repo: https://uwgraphics.github.io/CellOViewer/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:1a269cf148a2f50774820927c9d136739a212aa1e14fb7ec23cff2f37dd29fab
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# tumor-microenvironment-cell-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate cells in tumor-microenvironment and other challenging scRNA-seq datasets (e.g. Laughney lung adenocarcinoma, Segerstolpe pancreatic islets, La Manno fetal neural) with CellO, which makes hierarchical predictions against the Cell Ontology using classifiers pre-trained on human bulk RNA-seq from the SRA, and can place cells at ancestral ontology nodes when the specific type is absent from training.

## When to use

Use when annotating human scRNA-seq cells whose exact types may be missing from the classifier's training set (e.g. endothelial, fibroblast, plasmacytoid dendritic cells in LX675/LX679/LX682 lung tumor samples), and hierarchical Cell Ontology-consistent labels are wanted.

## When NOT to use

- Non-human data: CellO is trained on human primary cell bulk RNA-seq from the SRA, and no comprehensive curated mouse training set exists yet.
- Single-cell datasets whose genes are expected to match an already-trained model but don't, unless you are willing to retrain with -t on the reduced gene set.
- Flat, non-hierarchical labeling tasks where you need leaf-node-only labels; CellO may deliberately place cells at internal (ancestral) ontology nodes when uncertain.

## Inputs

- scRNA-seq gene-expression matrix for the dataset (10x directory with matrix.mtx/genes.tsv/barcodes.tsv, TSV, CSV, or HDF5)
- Expression units and assay type metadata (e.g. COUNTS with 3_PRIME, LOG1_TPM with FULL_LENGTH)
- Optional pre-computed cluster assignments TSV (for the -p option)
- Optional pretrained CellO model file (.model.dill, via -m)

## Outputs

- <output_prefix>.binary.tsv: NxM binary-decision matrix (1 if cell i is predicted to be of cell type j)
- <output_prefix>.probability.tsv: NxM classification probability table
- <output_prefix>.most_specific.tsv: mapping of each cell to its most-specific predicted cell type
- <output_prefix>.model.dill: retrained model when -t is used
- <output_prefix>.log/ files including genes_absent_from_training_set.tsv and clustering.tsv

## How to apply

Install CellO with `pip install cello-classify` and run `cello_predict` per dataset, specifying the data type ('-d' one of TSV/CSV/HDF5/10x), expression units ('-u' from COUNTS/CPM/LOG1_CPM/TPM/LOG1_TPM), and assay type ('-s' 3_PRIME or FULL_LENGTH) matching each dataset; the default algorithm is isotonic regression ('-a IR'). The input's genes must match the pre-trained classifier's genes, so if the gene set is incompatible (as with tumor TSVs like LX653), retrain with '-t' to fit a model on only the present genes, saved as <output_prefix>.model.dill. Collect the three output tables: <output_prefix>.binary.tsv (binary decisions), <output_prefix>.probability.tsv (confidence values), and <output_prefix>.most_specific.tsv (most-specific predicted type per cell). Validate by checking reproduced calls against the paper's reported assignments: acinar cells annotated as glandular epithelial cells (an ancestral Cell Ontology type), microglia as phagocytes, and correct endothelial/fibroblast/plasmacytoid dendritic cell calls in the LX675, LX679, and LX682 tumor samples.

## Related tools

- **CellO** (Hierarchical cell type classifier run via cello_predict against the Cell Ontology; used with IR (isotonic regression) algorithm, optional retraining (-t) on datasets with incompatible gene sets) — https://github.com/deweylab/CellO
- **CellO Viewer** (Web application for exploring CellO's model coefficients (cell type expression signatures) across the ontology when interpreting tumor-microenvironment calls) — https://uwgraphics.github.io/CellOViewer/

## Examples

```
cello_predict -u COUNTS -s 3_PRIME -t -o test example_input/LX653_tumor.tsv
```

## Evaluation signals

- The most_specific.tsv labels reproduce the paper's reported assignments: acinar-to-glandular-epithelial in Segerstolpe, microglia-to-phagocyte in La Manno fetal neural, and correct endothelial/fibroblast/plasmacytoid dendritic cell calls in LX675, LX679, and LX682 tumor samples.
- The three output tables exist with consistent dimensions: N cells x M cell types in binary.tsv and probability.tsv, and one row per cell in most_specific.tsv.
- If -t was used, <output_prefix>.log/genes_absent_from_training_set.tsv records which training-set genes were missing from the input, and <output_prefix>.model.dill was saved.
- Predicted labels are logically consistent with the Cell Ontology hierarchy (cells labeled with a specific type are also labeled as its ancestors in binary.tsv).
- Units and assay flags (-u, -s) match the dataset (e.g. LOG1_TPM/FULL_LENGTH for Kallisto-quantified output).

## Limitations

- Specific cell types absent from the bulk RNA-seq training set can only be labeled as ancestral Cell Ontology types (e.g. acinar cells annotated as glandular epithelial cells).
- The input expression data's genes must match the genes expected by the trained classifier; otherwise the classifier must be retrained on the input genes with -t.
- Determining optimal clustering in scRNA-seq data is an unsolved problem affecting CellO's cluster-based accuracy.
- Calibration gap: performance with F1-score lags average precision, affecting binary yes-no decisions.
- Cells without pre-computed clusters are clustered internally (clustering.tsv); providing -p skips this step.

## Evidence

- [other] CellO correctly annotated acinar cells, whose specific type is absent from the training set, as glandular epithelial cells, an ancestral cell type in the Cell Ontology.: [evidence span withheld — anchor status: not_found]
