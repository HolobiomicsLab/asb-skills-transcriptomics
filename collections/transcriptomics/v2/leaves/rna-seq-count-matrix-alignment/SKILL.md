---
name: rna-seq-count-matrix-alignment
description: Use when loading expression count data plus a sample metadata/design
  table (e.g. BRCA_normCounts and BRCA_metadata from DEGGs) prior to generate_subnetworks
  or any per-sample subgroup analysis.
when_to_use_negative:
- Data already passed through a pipeline that key-joins samples by ID (alignment is
  then enforced downstream)
- Count matrix and metadata come from different cohorts with non-overlapping sample
  identifiers — re-matching is a data-integration task, not a trivial order check
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: DEGGs
  role: R package whose BRCA_normCounts/BRCA_metadata datasets and generate_subnetworks
    workflow this alignment check guards; run in R before network generation
  repo: https://github.com/elisabettasciacca/DEGGs
- name: R
  role: Language in which the identical(colnames, rownames) alignment assertion is
    executed
provenance:
  source_task_ids:
  - task_007
  - task_006
  - task_005
  - task_008
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad192
    title: 'DEGGs: an R package with shiny app for the identification of differentially
      expressed gene–gene interactions in high-throughput sequencing data'
  source_content_hash: sha256:e85d10b5e2e488203f299c21e35c0b61603a88a4b480b12b9632be1b019e7cea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elisabettasciacca/DEGGs
  tools:
  - DEGGs
  - MASS
  - R
  - devtools
  - org.Hs.eg.db
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# rna-seq-count-matrix-alignment

## Summary

Verify that a normalized RNA-seq count matrix and its clinical metadata table are sample-aligned before differential network analysis, using identical() between matrix column names and metadata row names in the DEGGs workflow.

## When to use

Use when loading expression count data plus a sample metadata/design table (e.g. BRCA_normCounts and BRCA_metadata from DEGGs) prior to generate_subnetworks or any per-sample subgroup analysis.

## When NOT to use

- Data already passed through a pipeline that key-joins samples by ID (alignment is then enforced downstream)
- Count matrix and metadata come from different cohorts with non-overlapping sample identifiers — re-matching is a data-integration task, not a trivial order check

## Inputs

- Normalized count matrix (e.g. BRCA_normCounts) with samples as columns
- Clinical/metadata table (e.g. BRCA_metadata) with samples as rows and subgroup labels such as SUBTYPE

## Outputs

- Boolean alignment verdict from identical(colnames(counts), rownames(metadata)); analysis proceeds only if TRUE
- Optionally, an ordering-corrected count matrix/metadata pair ready for generate_subnetworks

## How to apply

After loading the packaged data with data("BRCA_normCounts") and data("BRCA_metadata"), run identical(colnames(BRCA_normCounts), rownames(BRCA_metadata)) and require it to return TRUE before proceeding. Column order in the count matrix must exactly match row order in the metadata, because DEGGs pairs each expression column to its clinical subgroup label (e.g. SUBTYPE = BRCA_Her2 vs BRCA_LumA) positionally rather than by re-keying. If the check fails, reorder columns/rows (e.g. match metadata rownames against colnames) rather than assuming internal consistency. This guard is step 1 of every DEGGs task card and protects the robust linear regression interaction-term fits from mislabeled samples.

## Related tools

- **DEGGs** (R package whose BRCA_normCounts/BRCA_metadata datasets and generate_subnetworks workflow this alignment check guards; run in R before network generation) — https://github.com/elisabettasciacca/DEGGs
- **R** (Language in which the identical(colnames, rownames) alignment assertion is executed)

## Examples

```
library(DEGGs); data("BRCA_normCounts"); data("BRCA_metadata"); stopifnot(identical(colnames(BRCA_normCounts), rownames(BRCA_metadata)))
```

## Evaluation signals

- identical(colnames(BRCA_normCounts), rownames(BRCA_metadata)) returns TRUE
- Number of metadata rows equals number of count-matrix columns
- Each subgroup (e.g. BRCA_Her2, BRCA_LumA) has matching sample counts on both sides of the alignment
- Downstream generate_subnetworks runs without sample-mismatch errors and the returned object is of class DEGGs

## Limitations

- DEGGs assumes positional sample correspondence; identical() detects order/label mismatch but cannot detect silently swapped sample identities when labels were already permuted consistently
- The original DEGGs repository is no longer maintained; the package moved to multiDEGGs, where the equivalent entry point is get_diffNetworks with assayData/metadata arguments

## Evidence

- [methods] verify column/row alignment with identical(): [evidence span withheld — anchor status: not_found]
- [readme] data loading from DEGGs package: "Load package and sample data   `library(DEGGs)  
data("BRCA_metadata")  
data("BRCA_normCounts")`"
- [methods] TCGA-BRCA dataset provenance: "This vignette uses breast cancer RNA-seq data from The Cancer Genome Atlas program (TCGA)"
