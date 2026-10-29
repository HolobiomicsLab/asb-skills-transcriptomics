---
name: marker-gene-pair-score-matrix-construction
description: Use when you have a bulk transcriptomic profile (RNA-seq or microarray)
  plus a cell-type marker annotation and need expression-level pairwise relations
  as input to prognostic gene-pair analysis.
when_to_use_negative:
- Input expression data has not been preprocessed (no log transformation or gene-wise
  median normalization) — run TimiPrePropress first, since unnormalized values distort
  the pairwise comparisons
- The goal is estimating absolute cell-type abundance proportions (e.g., deconvolution
  with CIBERSORTx or xCell) rather than pairwise expression relations for prognostic
  network analysis
- No cell-type marker annotation is available to define the gene pairs
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: TimiGP
  role: R package providing TimiGenePair (builds the MPS matrix) and TimiPrePropress
    (marker extraction with log transformation and gene-wise median normalization)
  repo: https://github.com/CSkylarL/TimiGP
- name: R
  role: Runtime environment; TimiGP is installed via devtools::install_github
- name: devtools
  role: Installs the TimiGP package from GitHub
  repo: https://github.com/r-lib/devtools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:e1dd8931c8858c1d97ef47e8cba4fcf7e050f0ec548eb3e0d1663295b88134cc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiGP
  - devtools
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# Marker gene pair score matrix construction

## Summary

TimiGP's Marker Pair Score (MPS) construction step: convert a bulk transcriptomic profile and a cell-type marker set into a binary pairwise-comparison matrix where each marker pair (gene A vs gene B) scores 1 when gene A expression exceeds gene B in a sample. This MPS is the substrate for downstream univariate Cox selection and cell-cell interaction inference.

## When to use

Use when you have a bulk transcriptomic profile (RNA-seq or microarray) plus a cell-type marker annotation and need expression-level pairwise relations as input to prognostic gene-pair analysis.

## When NOT to use

- Input expression data has not been preprocessed (no log transformation or gene-wise median normalization) — run TimiPrePropress first, since unnormalized values distort the pairwise comparisons
- The goal is estimating absolute cell-type abundance proportions (e.g., deconvolution with CIBERSORTx or xCell) rather than pairwise expression relations for prognostic network analysis
- No cell-type marker annotation is available to define the gene pairs

## Inputs

- Bulk transcriptomic profile (RNA-seq or microarray), e.g., SKCM06rna from TCGA SKCM06 metastatic melanoma
- Cell-type marker annotations, e.g., CellType_Bindea2013_cancer or Immune_Marker_n1293
- Clinical info (event and time-to-event) used by TimiCheckEvent for sample filtering

## Outputs

- Binary Marker Pair Score (MPS) matrix: one row per sample, one column per marker pair, values 1/0 from the expression comparison

## How to apply

First preprocess with TimiCheckEvent to filter low-quality clinical info (event and time-to-event) and TimiPrePropress with log=TRUE and GMNorm=TRUE to extract the selected markers' expression and apply log transformation and gene-wise median normalization. Then run TimiGenePair (Pairwise Comparison step, section 2.1 of the workflow), which captures the logical relation of any two markers and generates the Marker Pair Score (MPS) matrix: binary values (1 when gene A expression exceeds gene B, 0 otherwise) per sample. The MPS columns enumerate pairwise marker combinations across the marker set (e.g., the Bindea2013 cancer cell-type markers in the TCGA SKCM06 melanoma demonstration). The rationale is that the sign of a marker-pair comparison within a sample encodes the relative abundance ordering of the markers, which later supports directed gene-gene network construction and Cox-based prognostic selection.

## Related tools

- **TimiGP** (R package providing TimiGenePair (builds the MPS matrix) and TimiPrePropress (marker extraction with log transformation and gene-wise median normalization)) — https://github.com/CSkylarL/TimiGP
- **R** (Runtime environment; TimiGP is installed via devtools::install_github)
- **devtools** (Installs the TimiGP package from GitHub) — https://github.com/r-lib/devtools

## Examples

```
devtools::install_github("CSkylarL/TimiGP")
```

## Evaluation signals

- MPS matrix is strictly binary: every entry is 1 or 0, derived from whether gene A expression exceeds gene B in that sample
- Matrix columns equal the number of pairwise marker combinations over the selected marker set (e.g., the Bindea2013 cancer markers), rows equal the samples surviving TimiCheckEvent filtering
- Markers in the MPS correspond exactly to the expression rows extracted by TimiPrePropress after log transformation and gene-wise median normalization
- Downstream sanity check: TimiCOX on the MPS should produce a QV (BH-adjusted p-value) per pair, and the workflow validation keeps only pairs with QV<0.05
- Reproducing on the packaged TCGA SKCM06 data via the MSofTimiGP scripts should yield the same MPS-derived gene-gene network (gene_network.sif, gene_edge.txt)

## Limitations

- To effectively identify prognostic gene pairs downstream, TimiGP requires a large gene expression dataset with matched survival information
- Sensitivity to identify a specific cell-cell interaction is affected by the number of corresponding marker genes; cell types with fewer marker genes are less likely to be identified as having significant interactions
- The power of the survival analysis is determined by the sample size, the number of events, and the quality of the survival information

## Evidence
