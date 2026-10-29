---
name: pearson-correlation-benchmark-validation
description: Use when a deconvolution method's predicted cell-type abundances must
  be benchmarked against known true abundances from aggregated single-cell data to
  quantify accuracy per cell type.
when_to_use_negative:
- Input is a single-cell expression matrix that has not been aggregated into bulk/pseudo-bulk
  per-sample profiles — DCNet expects bulk-formatted input
- No ground-truth cell abundances are available for the test dataset, so a correlation-based
  benchmark cannot be computed
- The target cell types are not among the 434 cell types embedded in the DCNet network
  (33 cell types were discarded for lacking detected marker genes)
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3522
- http://edamontology.org/topic_3170
tools:
- name: DCNet
  role: Deep learning deconvolution model that predicts abundances of 434 cell types
    from bulk RNA-seq; its predictions are benchmarked via Pearson correlation
  repo: https://github.com/xindd/DCNet
- name: mxnet
  role: Python framework in which the DCNet model is coded and run to generate predicted
    abundances
- name: Python
  role: Runtime environment for running DCNet and computing Pearson correlations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:d7e590b575ec5596fb465891f784c5c8cf494181b0f8391be2e7070177fcd9d4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  - mxnet
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3522
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# pearson-correlation-benchmark-validation

## Summary

Validates bulk-expression cell-abundance deconvolution predictions by computing Pearson correlations between DCNet-predicted and true single-cell-derived abundances per cell type, as done for GSE86146 fetal germ cells (r=0.87) and somatic cells (r=0.79).

## When to use

Use when a deconvolution method's predicted cell-type abundances must be benchmarked against known true abundances from aggregated single-cell data to quantify accuracy per cell type.

## When NOT to use

- Input is a single-cell expression matrix that has not been aggregated into bulk/pseudo-bulk per-sample profiles — DCNet expects bulk-formatted input
- No ground-truth cell abundances are available for the test dataset, so a correlation-based benchmark cannot be computed
- The target cell types are not among the 434 cell types embedded in the DCNet network (33 cell types were discarded for lacking detected marker genes)

## Inputs

- Bulk-aggregated (pseudo-bulk) expression matrix with rows as human gene symbols and columns as samples, log2(x+1) transformed
- True per-cell-type abundances from aggregated single-cell RNA-seq data (e.g., GSE86146)
- DCNet index.pkl mapping input/output neurons to genes
- DCNet relation_matrix.pkl zero-one gene-cell correspondence matrix

## Outputs

- Per-cell-type Pearson correlation coefficients between predicted and true abundances (e.g., gse86146_correlations.csv)
- Statistical significance (p-values) for each correlation

## How to apply

Aggregate single-cell profiles into pseudo-bulk expression matrices per cell type (e.g., 2167 individual PGCs and gonadal niche cells from GSE86146), format rows as human gene symbols and columns as samples, and apply the recommended log2(x+1) transformation. Run DCNet (mxnet framework in Python) using the index.pkl gene-neuron mapping and the relation_matrix.pkl zero-one gene-cell correspondence matrix to predict abundances for the 434 embedded cell types. Extract predicted abundances for the leaf-node cell types of interest in the Cell Ontology hierarchy and compute Pearson correlations against true abundances. Correct application is confirmed when correlations match published values (FGC ~0.87, somatic ~0.79, both p<0.01).

## Related tools

- **DCNet** (Deep learning deconvolution model that predicts abundances of 434 cell types from bulk RNA-seq; its predictions are benchmarked via Pearson correlation) — https://github.com/xindd/DCNet
- **mxnet** (Python framework in which the DCNet model is coded and run to generate predicted abundances)
- **Python** (Runtime environment for running DCNet and computing Pearson correlations)

## Examples

```
net.load_parameters('net_M434_adam_T600_DCNetBnoise-D1.params'); pred = net(X_mxnet); pearsonr(pred[fgc_idx], true_fgc_abundance)  # expect r≈0.87, p<0.01
```

## Evaluation signals

- Pearson correlation for fetal germ cells (FGC) in GSE86146 is approximately 0.87 with p<0.01
- Pearson correlation for somatic cells in GSE86146 is approximately 0.79 with p<0.01
- Predicted abundances correspond to leaf-node cell types of the Cell Ontology hierarchy embedded in DCNet
- Expression matrix passed to DCNet is gene-symbol-by-sample oriented and log2(x+1) transformed
- Results table (e.g., gse86146_correlations.csv) contains matched predicted-vs-true abundance pairs per cell type

## Limitations

- Statistical significance thresholds and number of replicates are only reported as p<0.01 for the GSE86146 benchmark; no seed or reproducibility controls for model training are documented
- Trained model weights/checkpoints availability is not stated in the article (only source code is deposited); the README does provide net_M434_adam_T600_DCNetBnoise-D1.params
- The benchmark validates only leaf-node cell types (FGC and somatic cells) rather than all 434 embedded types, so accuracy for other lineages is not directly evidenced

## Evidence

- [results] DCNet's predicted abundance shows high correlation with true abundance: 0.87 for FGC cells and 0.79 for somatic cells (p < .01): [evidence span withheld — anchor status: not_found]
