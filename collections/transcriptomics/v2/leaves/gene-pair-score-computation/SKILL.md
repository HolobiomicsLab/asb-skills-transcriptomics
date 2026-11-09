---
name: gene-pair-score-computation
description: Use when a bulk transcriptomic cohort has been preprocessed with marker-gene
  selection and you need a pairwise logical-relational matrix of marker expression
  comparisons to feed Cox regression or enrichment of cell-cell interactions.
when_to_use_negative:
- Input is single-cell expression data with per-cell resolution where direct cell-type
  abundance deconvolution methods (xCell, CIBERSORTx) are appropriate
- The cohort lacks matched survival or response information, since downstream directed
  IMGP selection relies on Cox regression of the MPS matrix
- Only a small number of marker genes are available, since enrichment-based interaction
  sensitivity is affected by marker gene count
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: TimiGP
  role: R package providing TimiGenePair, TimiPrePropress, and TimiCheckEvent used
    to build the MPS matrix
  repo: https://github.com/CSkylarL/TimiGP
- name: devtools
  role: Used to install the TimiGP package from GitHub
  repo: https://github.com/r-lib/devtools
- name: R
  role: Runtime environment for the TimiGP pipeline
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:a302c88f28738a0ce6f78c9eb2650767b4452e2bf086d4242d8a9b22c273cf19
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiGP
  - devtools
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# gene-pair-score-computation

## Summary

Compute the binary Marker Pair Score (MPS) matrix used by the TimiGP framework, capturing pairwise logical relations between marker genes (e.g., 1 = gene A expression > gene B) from log-transformed, gene-wise median-normalized bulk expression data, as the basis for inferring prognostic cell-cell interactions.

## When to use

Use when a bulk transcriptomic cohort has been preprocessed with marker-gene selection and you need a pairwise logical-relational matrix of marker expression comparisons to feed Cox regression or enrichment of cell-cell interactions.

## When NOT to use

- Input is single-cell expression data with per-cell resolution where direct cell-type abundance deconvolution methods (xCell, CIBERSORTx) are appropriate
- The cohort lacks matched survival or response information, since downstream directed IMGP selection relies on Cox regression of the MPS matrix
- Only a small number of marker genes are available, since enrichment-based interaction sensitivity is affected by marker gene count

## Inputs

- Preprocessed marker expression matrix (RNA-seq or microarray) from TimiPrePropress, e.g. SKCM06rna restricted to CellType_Bindea2013_cancer genes
- Filtered clinical information (event and time-to-event) from TimiCheckEvent, e.g. SKCM06info
- Cell-type marker list defining the genes to pair (e.g. CellType_Bindea2013_cancer, Immune_Marker_n1293)

## Outputs

- Binary Marker Pair Score (MPS) matrix with one row per sample and one column per marker pair
- List of tested marker pairs used later as enrichment background (via TimiBG)

## How to apply

First preprocess the transcriptome profile with TimiPrePropress (log=TRUE, GMNorm=TRUE) to extract the selected marker genes and apply log transformation and gene-wise median normalization; clinical records should be filtered with TimiCheckEvent. Then run TimiGenePair on the normalized marker expression matrix: by default it captures the logical relation of any two markers and generates the Marker Pair Score (MPS) matrix where a pair scores 1 when gene A expression > gene B expression. The resulting binary MPS matrix is the substrate for downstream steps such as TimiCOX univariate Cox regression (favorable-prognosis IMGP selection) and TimiGenePair-based background generation (TimiBG). Decision point: the logical relation is directional, so pair orientation must be preserved when reversing pairs with HR>1 during directed IMGP selection.

## Related tools

- **TimiGP** (R package providing TimiGenePair, TimiPrePropress, and TimiCheckEvent used to build the MPS matrix) — https://github.com/CSkylarL/TimiGP
- **devtools** (Used to install the TimiGP package from GitHub) — https://github.com/r-lib/devtools
- **R** (Runtime environment for the TimiGP pipeline)

## Examples

```
library(TimiGP); mps <- TimiGenePair(rna_norm, marker_genes)  # after rna_norm <- TimiPrePropress(rna, genes=CellType_Bindea2013_cancer, log=TRUE, GMNorm=TRUE)
```

## Evaluation signals

- MPS matrix must be strictly binary (0/1), with 1 encoded as gene A expression > gene B expression under the default logical relation
- MPS columns must cover only genes present in the supplied cell-type marker list after TimiPrePropress filtering
- Applying TimiCOX to the MPS on TCGA_SKCM06 with Bindea2013 markers must support the downstream validation of 54 significant cell pairs out of 462 (FDR<0.05)
- Sample count of the MPS matrix must equal the cohort retained after TimiCheckEvent filtering of low-quality clinical records

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs
- The power of the survival analysis is determined by the sample size, the number of events, and the quality of the survival information
- Since cell-cell interactions are identified based on enrichment analysis, the sensitivity to identify a specific interaction is affected by the number of corresponding marker genes; cell types with fewer marker genes are less likely to be identified

## Evidence
