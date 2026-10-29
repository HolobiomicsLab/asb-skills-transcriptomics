---
name: command-line-rna-seq-classification-execution
description: Use when you have a human RNA-seq gene-expression matrix (TSV, CSV, HDF5,
  or 10x directory) that needs automated cell type annotation without user-supplied
  marker genes or labeled training data.
when_to_use_negative:
- The input data are from a species other than human — CellO's training set is human
  primary cell bulk RNA-seq from the SRA, and no comprehensive mouse training set
  exists yet
- The analysis requires species-specific or lineage/developmental relationships between
  cell types that the Cell Ontology does not encode
- The user must supply custom labeled training data or marker gene sets for a bespoke
  taxonomy rather than classify against the Cell Ontology graph
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3518
tools:
- name: CellO
  role: Python package providing the cello_predict command-line tool that performs
    hierarchical cell type classification against the Cell Ontology using the IR or
    CLR algorithm
  repo: https://github.com/deweylab/CellO
- name: CellO Viewer
  role: Web application for exploring CellO's gene-wise linear model coefficients
    across the Cell Ontology to interpret cell type expression signatures
  repo: https://github.com/uwgraphics/CellOViewer
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:315d6734efc4acea663e82e979c38f48ea41680fe8e5f12a3737b834c3c9d5df
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# command-line-rna-seq-classification-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Running CellO's cello_predict command-line tool to hierarchically classify human scRNA-seq or bulk RNA-seq expression matrices against the Cell Ontology, using isotonic regression (IR) as the default algorithm and producing probability, binary-decision, and most-specific cell type tables.

## When to use

Use when you have a human RNA-seq gene-expression matrix (TSV, CSV, HDF5, or 10x directory) that needs automated cell type annotation without user-supplied marker genes or labeled training data.

## When NOT to use

- The input data are from a species other than human — CellO's training set is human primary cell bulk RNA-seq from the SRA, and no comprehensive mouse training set exists yet
- The analysis requires species-specific or lineage/developmental relationships between cell types that the Cell Ontology does not encode
- The user must supply custom labeled training data or marker gene sets for a bespoke taxonomy rather than classify against the Cell Ontology graph

## Inputs

- Gene-expression matrix in TSV, CSV, HDF5, or 10x directory format (matrix.mtx, genes.tsv, barcodes.tsv)
- Expression units flag: COUNTS, CPM, LOG1_CPM, TPM, or LOG1_TPM
- Assay type flag: 3_PRIME or FULL_LENGTH
- Optional pre-trained model file (*.model.dill) via -m, or pre-computed cluster assignments via -p

## Outputs

- <output_prefix>.probability.tsv: NxM classification probability table (N cells x M cell types)
- <output_prefix>.binary.tsv: NxM binary-decision matrix of 0/1 Cell Ontology term predictions
- <output_prefix>.most_specific.tsv: mapping of each cell to its most-specific predicted cell type
- <output_prefix>.model.dill: retrained model when the -t flag is used
- <output_prefix>.log/clustering.tsv: cell-to-cluster assignments (unless -p pre-clustering is provided)

## How to apply

Install CellO with `pip install cello-classify`, then run `cello_predict` specifying the data type with `-d` (TSV/CSV/10x/HDF5), expression units with `-u` (COUNTS, CPM, LOG1_CPM, TPM, LOG1_TPM), and assay type with `-s` (3_PRIME or FULL_LENGTH) matching the input dataset. The default algorithm is isotonic regression (`-a IR`); the alternative is cascaded logistic regression ('CLR'). The input matrix's genes must match the genes expected by the pre-trained classifier (trained on nearly all human primary cell bulk RNA-seq data from the SRA); if they do not, retrain with the `-t` flag so a model is fit on only the input genes, saved as <output_prefix>.model.dill. Collect the three output tables — <output_prefix>.probability.tsv (NxM confidence probabilities), <output_prefix>.binary.tsv (NxM binary-decision matrix), and <output_prefix>.most_specific.tsv (each cell mapped to its most-specific predicted cell type) — where the hierarchy lets cells be placed at internal ontology nodes rather than only at leaves.

## Related tools

- **CellO** (Python package providing the cello_predict command-line tool that performs hierarchical cell type classification against the Cell Ontology using the IR or CLR algorithm) — https://github.com/deweylab/CellO
- **CellO Viewer** (Web application for exploring CellO's gene-wise linear model coefficients across the Cell Ontology to interpret cell type expression signatures) — https://github.com/uwgraphics/CellOViewer

## Examples

```
cello_predict -d 10x -u COUNTS -s 3_PRIME example_input/Zheng_PBMC_10x -o test
```

## Evaluation signals

- All three output tables (<output_prefix>.probability.tsv, .binary.tsv, .most_specific.tsv) are produced for the given output prefix
- The most_specific.tsv labels agree with known assignments, e.g. acinar cells annotated as glandular epithelial cells and microglial cells as phagocytes on challenging datasets (Segerstolpe islets, La Manno fetal neural)
- No genes are silently missing: when -t retraining fires, genes_absent_from_training_set.tsv records which training-set genes were absent from the input
- Specified -u units and -s assay flags match the actual sequencing assay and expression units of the input matrix
- The .binary.tsv matrix is hierarchy-consistent: cells placed at internal ontology nodes carry positive calls for all ancestor cell types

## Limitations

- The input expression data's genes must match the genes expected by the trained classifier; otherwise a new model must be trained with -t on only the input genes
- CellO is trained on human, healthy, untreated primary samples and is pre-trained only on human SRA bulk data — mouse or other species are unsupported pending MetaSRA metadata
- Classifier calibration gap: performance with F1-score lags average precision, affecting binary yes-no decisions, and calibration is expected to improve only as more training data per cell type become available
- Optimal clustering resolution in scRNA-seq is an open problem that affects CellO's cluster-based annotation accuracy

## Evidence
