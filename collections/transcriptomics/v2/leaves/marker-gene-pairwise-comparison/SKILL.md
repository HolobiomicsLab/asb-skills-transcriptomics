---
name: marker-gene-pairwise-comparison
description: Use when you have a cell-type marker gene list (e.g., scRNA-seq-derived
  T cell subtype markers) plus bulk RNA-seq/microarray data and need expression-level
  pairwise relations as input to survival modeling.
when_to_use_negative:
- Input is single-cell expression data where direct cell-cell communication tools
  (e.g., scSeqComm) can be applied instead of bulk-inferred gene pairs
- No matched survival/clinical outcome information is available, since prognostic
  gene-pair selection via TimiCOX cannot be powered
- The gene set of interest is continuous dosage data rather than marker dichotomies
  and cont=TRUE behavior has not been validated for the cohort
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: TimiGenePair
  role: Generates the Marker Pair Score matrix via logical pairwise comparison of
    markers (cont=FALSE)
  repo: https://github.com/CSkylarL/TimiGP
- name: TimiPrePropress
  role: 'Upstream preprocessing: extracts marker subset, log-transforms, and performs
    gene-wise median normalization of the transcriptome'
  repo: https://github.com/CSkylarL/TimiGP
- name: TimiCheckEvent
  role: Filters low-quality clinical event/time-to-event information before pairwise
    analysis
  repo: https://github.com/CSkylarL/TimiGP
- name: TimiCOX
  role: Consumes the MPS matrix for univariate Cox regression of each marker pair
    with BH adjustment
  repo: https://github.com/CSkylarL/TimiGP
- name: R
  role: Implementation language; TimiGP installed via devtools::install_github
  repo: https://github.com/r-lib/devtools
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:1332e4aa52ac6ddeee283f4197dd7a92d05d2d494714b92533ab740b78dea099
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiCOX
  - TimiCheckEvent
  - TimiFS
  - TimiGP
  - TimiGenePair
  - TimiPermFDR
  - TimiPrePropress
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# marker-gene-pairwise-comparison

## Summary

Generates a Marker Pair Score (MPS) matrix from a cell-type marker gene list and a bulk transcriptomic profile by logical pairwise comparison of marker expression within each sample, the first analytical step of the TimiGP framework that enables downstream Cox-based prognostic ranking of gene pairs.

## When to use

Use when you have a cell-type marker gene list (e.g., scRNA-seq-derived T cell subtype markers) plus bulk RNA-seq/microarray data and need expression-level pairwise relations as input to survival modeling.

## When NOT to use

- Input is single-cell expression data where direct cell-cell communication tools (e.g., scSeqComm) can be applied instead of bulk-inferred gene pairs
- No matched survival/clinical outcome information is available, since prognostic gene-pair selection via TimiCOX cannot be powered
- The gene set of interest is continuous dosage data rather than marker dichotomies and cont=TRUE behavior has not been validated for the cohort

## Inputs

- Cell-type marker gene list (e.g., scRNA-seq-derived markers from Tirosh2016/Zheng2021, or bundled sets like Immune_Marker_n1293, CellType_Newman2015_LM22)
- Preprocessed bulk transcriptomic profile (log-transformed, gene-wise median normalized; e.g., SKCM06rna from TCGA SKCM06)
- Filtered clinical survival information (via TimiCheckEvent; e.g., SKCM06info)

## Outputs

- Marker Pair Score (MPS) matrix (gene pairs x samples) of logical pairwise comparison results
- Downstream: Cox regression statistics per gene pair (via TimiCOX) and enrichment background gene pairs (via TimiBG)

## How to apply

First preprocess the transcriptome with TimiPrePropress, which extracts the profile of the selected markers and cohorts, performs log transformation and gene-wise median normalization. Then run TimiGenePair with logical (non-continuous) pairwise comparison (cont=FALSE) to capture the logical relation of any two marker pairs, producing a Marker Pair Score (MPS) matrix where each entry records which gene of a pair is more highly expressed in a sample. The MPS matrix is subsequently fed to TimiCOX for univariate Cox regression on each marker pair (with BH adjustment) and to TimiBG for building the enrichment background. Rationale: starting from pairwise expression relations sidesteps the immune co-infiltration bias that confounds single-marker or deconvolution-based estimates of prognostic association.

## Related tools

- **TimiGenePair** (Generates the Marker Pair Score matrix via logical pairwise comparison of markers (cont=FALSE)) — https://github.com/CSkylarL/TimiGP
- **TimiPrePropress** (Upstream preprocessing: extracts marker subset, log-transforms, and performs gene-wise median normalization of the transcriptome) — https://github.com/CSkylarL/TimiGP
- **TimiCheckEvent** (Filters low-quality clinical event/time-to-event information before pairwise analysis) — https://github.com/CSkylarL/TimiGP
- **TimiCOX** (Consumes the MPS matrix for univariate Cox regression of each marker pair with BH adjustment) — https://github.com/CSkylarL/TimiGP
- **R** (Implementation language; TimiGP installed via devtools::install_github) — https://github.com/r-lib/devtools

## Examples

```
SKCM06rna_m <- TimiPrePropress(marker = skcm06_markers, rna = SKCM06rna); mps <- TimiGenePair(rna = SKCM06rna_m, marker = skcm06_markers, cont = FALSE)
```

## Evaluation signals

- MPS matrix dimensions equal the number of marker gene pairs (n*(n-1)/2 for logical comparison) by number of filtered samples
- MPS entries are logical/binary scores reflecting which gene of each pair dominates per sample, not raw expression values
- Downstream TimiCOX pairs with HR>1 are reversed so all selected pairs orient toward favorable prognosis before enrichment
- On TCGA_SKCM06 with T cell-subtype markers, the pipeline reproduces differential prognostic association of CD8+ GZMK+ Tex vs CD8+ TCF7+ Tex and yields unfavorable scores for stromal/tumor cells

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs
- Sensitivity to identify a specific cell-cell interaction is affected by the number of corresponding marker genes; cell types with fewer marker genes are less likely to be identified
- No wet-lab experiments were performed to validate the important findings, raising uncertainty about the reliability of inferred cell-cell interactions

## Evidence
