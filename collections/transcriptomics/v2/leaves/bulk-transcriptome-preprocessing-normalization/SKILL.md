---
name: bulk-transcriptome-preprocessing-normalization
description: Use when bulk RNA-seq or microarray expression data with matched clinical
  event/time-to-event survival information must be filtered, log-transformed, gene-wise
  median normalized, and converted into marker pair scores before Cox-based immune
  prognostic analysis.
when_to_use_negative:
- Input data has no matched event and time-to-event survival information, since TimiCheckEvent
  filters on event and time-to-event quality and the downstream module is prognosis/response-based
- Expression data are already log-transformed and gene-wise median normalized, or
  only a precomputed feature/deconvolution abundance table is available rather than
  a raw bulk transcriptomic profile
- The analysis target is single-cell expression quantification itself rather than
  preparing marker pair scores for bulk-data cell-cell interaction inference
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: TimiCheckEvent
  role: Filters low-quality clinical info (event and time-to-event) from the cohort
    before expression preprocessing
  repo: https://github.com/CSkylarL/TimiGP
- name: TimiPrePropress
  role: Extracts the transcription profile of selected markers and performs log transformation
    and gene-wise median normalization (log=TRUE, GMNorm=TRUE)
  repo: https://github.com/CSkylarL/TimiGP
- name: TimiGenePair
  role: Captures the logical relation of any two marker pairs and generates the binary
    Marker Pair Score (MPS) matrix consumed by TimiCOX
  repo: https://github.com/CSkylarL/TimiGP
- name: R
  role: Runtime environment in which the TimiGP package functions are executed
  repo: https://github.com/CSkylarL/TimiGP
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_003
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:da8d4f69c135a5402f280b03f1c491df72b8dd456252a28cfb2eb45cf1882d2b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiCheckEvent
  - TimiGP
  - TimiGenePair
  - TimiPrePropress
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# bulk-transcriptome-preprocessing-normalization

## Summary

Quality-filter clinical survival records and preprocess bulk transcriptomic profiles (RNA-seq or microarray) for TimiGP marker-pair analysis, using TimiCheckEvent, TimiPrePropress (log transformation and gene-wise median normalization), and TimiGenePair to build the binary Marker Pair Score (MPS) matrix.

## When to use

Use when bulk RNA-seq or microarray expression data with matched clinical event/time-to-event survival information must be filtered, log-transformed, gene-wise median normalized, and converted into marker pair scores before Cox-based immune prognostic analysis.

## When NOT to use

- Input data has no matched event and time-to-event survival information, since TimiCheckEvent filters on event and time-to-event quality and the downstream module is prognosis/response-based
- Expression data are already log-transformed and gene-wise median normalized, or only a precomputed feature/deconvolution abundance table is available rather than a raw bulk transcriptomic profile
- The analysis target is single-cell expression quantification itself rather than preparing marker pair scores for bulk-data cell-cell interaction inference

## Inputs

- Clinical survival info table with event and time-to-event fields (e.g., SKCM06info, TNBCaPD1info)
- Bulk transcriptome profile (RNA-seq or microarray expression matrix, e.g., SKCM06rna, TNBCaPD1rna)
- Cell-type marker gene list (e.g., CellType_Bindea2013_cancer, CellType_Newman2015_LM22, CellType_Charoentong2017_Bindea2013_Xu2018_Immune)

## Outputs

- Filtered clinical info (low-quality event/time records removed)
- Marker expression matrix extracted from the cohort, log-transformed and gene-wise median normalized
- Binary Marker Pair Score (MPS) matrix over all marker pairs (1 if A>B, 0 if A<B)

## How to apply

First run TimiCheckEvent on the clinical info (e.g., SKCM06info, TNBCaPD1info) to filter out low-quality event and time-to-event records. Then run TimiPrePropress on the transcriptome profile (e.g., SKCM06rna, TNBCaPD1rna) with log=TRUE and GMNorm=TRUE, using the marker genes from the chosen annotation (e.g., CellType_Bindea2013_cancer) to extract, log-transform, and gene-wise median normalize expression of selected markers. Finally, apply TimiGenePair to capture the logical relation of any two marker pairs and generate the Marker Pair Score (MPS) matrix, where by default A_B=1 when gene A expression > gene B and 0 when A < B. Rationale: TimiGP starts from pairwise expression relations, so correct event filtering and marker-specific normalization directly determine which pairs enter downstream TimiCOX univariate regression with BH adjustment.

## Related tools

- **TimiCheckEvent** (Filters low-quality clinical info (event and time-to-event) from the cohort before expression preprocessing) — https://github.com/CSkylarL/TimiGP
- **TimiPrePropress** (Extracts the transcription profile of selected markers and performs log transformation and gene-wise median normalization (log=TRUE, GMNorm=TRUE)) — https://github.com/CSkylarL/TimiGP
- **TimiGenePair** (Captures the logical relation of any two marker pairs and generates the binary Marker Pair Score (MPS) matrix consumed by TimiCOX) — https://github.com/CSkylarL/TimiGP
- **R** (Runtime environment in which the TimiGP package functions are executed) — https://github.com/CSkylarL/TimiGP

## Examples

```
devtools::install_github("CSkylarL/TimiGP"); mps <- TimiGenePair(TimiPrePropress(SKCM06rna, gene = geneset, log = TRUE, GMNorm = TRUE))
```

## Evaluation signals

- TimiCheckEvent output contains no records with low-quality event or time-to-event fields relative to the input clinical table
- TimiPrePropress output matrix contains only the selected marker genes and each gene's expression is log-transformed and gene-wise median normalized across the cohort
- TimiGenePair MPS matrix is binary per pair, with A_B=1 exactly where gene A expression > gene B and 0 where A < B
- Number of MPS columns/rows matches the expected number of marker pairs from the chosen marker annotation (e.g., 4352 top pairs retained per cancer type in the pan-cancer run)
- Downstream TimiCOX on the preprocessed MPS reproduces favorable-prognosis-associated pairs after reversing pairs with HR>1 under BH adjustment

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs
- Power of the survival analysis is determined by sample size, number of events, and quality of the survival information
- Because downstream identification is enrichment-analysis based, sensitivity to a specific cell-cell interaction is affected by the number of corresponding marker genes; cell types with fewer marker genes are less likely to be identified

## Evidence

- [methods] TimiCheckEvent filters low-quality clinical info: "`TimiCheckEvent` will filter out the low-quality clinical info(event and time-to-event)."
- [methods] TimiPrePropress log transform and gene-wise median normalization: "`TimiPrePropress` will extract the transcription profile of selected markers and cohorts, then perform log transformation and gene-wise median normalization."
- [methods] TimiGenePair generates Marker Pair Score matrix: "In default, `TimiGenePair` will capture the logical relation of any two marker pairs, and generate a matrix of Marker Pair Score(MPS):"
- [methods] TimiCOX fits each marker pair as a variable: "`TimiCOX` will perform univariate Cox regression that fits each marker pair as a variable."
- [other] TimiGP is an R package for tumor microenvironment inference from bulk transcriptomics: [evidence span withheld — anchor status: external_doc]
