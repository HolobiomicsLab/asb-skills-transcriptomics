---
name: pseudobulk-aggregation-per-perturbation
description: Use when Perturb-seq single-cell data have been quantified to 3'UTR isoform
  level with scUTRquant and per-perturbation (per-guide) bulk-level profiles are needed
  before computing differential UTR-length statistics.
when_to_use_negative:
- Data have not yet been processed through scUTRquant and still exist as raw gene-count
  matrices without 3'UTR isoform resolution
- Goal is per-cell, not per-perturbation, 3'UTR-isoform characterization (e.g. cell-type
  UTR usage across the 474 cell types)
- Input is already a pseudobulk count table — re-aggregation would double-count cells
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scUTRquant
  role: Produces the 3'UTR isoform SingleCellExperiment objects (via kallisto bus
    pseudoalignment and bustools correct) that are the input to pseudobulk pooling
  repo: https://github.com/Mayrlab/scUTRquant
- name: kallisto-bustools
  role: Underlying pseudoalignment and barcode-correction workflow used by scUTRquant
    to generate the single-cell UTR isoform counts being aggregated
- name: scUTRboot
  role: Consumes the pseudobulk perturbation profiles to compute zdWUI z-scores of
    significant 3'UTR-length responses relative to non-targeting controls
  repo: https://github.com/mfansler/scutrboot
- name: gwps-sq
  role: Analysis repository implementing the filtering, guide-assignment metadata,
    and summarization-to-pseudobulk stages for the K562/RPE1 Perturb-seq data
  repo: https://github.com/Mayrlab/gwps-sq
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:d8c11e14988e02af2fa3dee0286cef022379b36573e6ba9f2f058aafc416852c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - Conda
  - gwps-sq
  - kallisto-bustools
  - scUTRboot
  - scUTRquant
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Pseudobulk aggregation of UTR isoform counts per perturbation

## Summary

Collapses single-cell 3'UTR isoform counts from scUTRquant SingleCellExperiment objects into one pseudobulk profile per perturbation by pooling all cells sharing a guide, converting Perturb-seq data into a perturbation-by-UTR-isoform count matrix suitable for downstream statistical testing of 3'UTR-length responses.

## When to use

Use when Perturb-seq single-cell data have been quantified to 3'UTR isoform level with scUTRquant and per-perturbation (per-guide) bulk-level profiles are needed before computing differential UTR-length statistics.

## When NOT to use

- Data have not yet been processed through scUTRquant and still exist as raw gene-count matrices without 3'UTR isoform resolution
- Goal is per-cell, not per-perturbation, 3'UTR-isoform characterization (e.g. cell-type UTR usage across the 474 cell types)
- Input is already a pseudobulk count table — re-aggregation would double-count cells

## Inputs

- 3'UTR isoform SingleCellExperiment object from scUTRquant (gene-level and isoform-level matrices)
- Cell-level metadata mapping cell barcodes to guide identities / perturbations
- List of non-targeting control guides used as the reference group (97 controls in the K562 analysis)

## Outputs

- Perturbation-by-3'UTR-isoform pseudobulk count matrix (one column/profile per guide)
- Perturbation metadata table aligned to pseudobulk profiles

## How to apply

After running scUTRquant on the Perturb-seq BAMs (e.g. the 48 K562 Perturb-seq inputs with the 'utrome_hg38_v1' target, with min_umis cell filtering) to obtain 3'UTR isoform SingleCellExperiment objects, group cells by their guide/perturbation identity from the guide-assignment metadata and sum UMI counts across all cells of each guide, yielding one pseudobulk profile per perturbation. Rationale: pseudobulking pools counts so that each perturbation has sufficient depth for isoform-level testing, and in this work the pseudobulk matrix feeds scUTRboot, which computes zdWUI z-scores for each perturbation relative to the 97 non-targeting control guides. The filtering, guide assignment, and summarization steps are implemented in the Mayrlab/gwps-sq analysis repository, which reprocesses the K562 and RPE1 Perturb-seq data from the GWPS resource. Validate the output by checking one row/profile per unique guide, preserved isoform columns, and totals that equal the sum of the pooled single-cell counts.

## Related tools

- **scUTRquant** (Produces the 3'UTR isoform SingleCellExperiment objects (via kallisto bus pseudoalignment and bustools correct) that are the input to pseudobulk pooling) — https://github.com/Mayrlab/scUTRquant
- **kallisto-bustools** (Underlying pseudoalignment and barcode-correction workflow used by scUTRquant to generate the single-cell UTR isoform counts being aggregated)
- **scUTRboot** (Consumes the pseudobulk perturbation profiles to compute zdWUI z-scores of significant 3'UTR-length responses relative to non-targeting controls) — https://github.com/mfansler/scutrboot
- **gwps-sq** (Analysis repository implementing the filtering, guide-assignment metadata, and summarization-to-pseudobulk stages for the K562/RPE1 Perturb-seq data) — https://github.com/Mayrlab/gwps-sq

## Evaluation signals

- One pseudobulk profile exists per unique guide (e.g. 836 perturbations retained downstream in the K562 matrix of 836 x 883 response genes)
- Pseudobulk isoform counts equal the column-wise sums of the pooled single-cell UMI counts for each guide
- Guide-assignment metadata covers all cells passing the min_umis filter, with no unassigned cells silently dropped
- Downstream scUTRboot call yields zdWUI z-scores for every perturbation when compared against the 97 non-targeting control guides
- Filtered response-gene matrix reproduces the expected 836 perturbations x 883 response genes dimensions feeding the walktrap clustering

## Limitations

- Pseudobulking discards within-perturbation single-cell heterogeneity; it is intended for perturbation-level 3'UTR response testing, not cell-level analysis
- Requires reliable guide-to-cell assignment; cells with ambiguous or failed guide calls cannot be correctly pooled
- Most 10x reads do not span 3' end cleavage sites, so isoform quantification depends on the truncated-transcript UTRome annotation rather than exact 3' end mapping

## Evidence

- [readme] It includes both some preprocessing code used to created the metadata tables used in the scUTRquant pipeline, and the initial stages of filtering, summarization to pseudobulk,: [evidence span withheld — anchor status: external_doc]
- [discussion] Coupled with scUTRboot, significant differences in 3′UTRs across samples are identified, which facilitates the integration of 3′UTR quantification into standard scRNA-seq data analysis: "Coupled with scUTRboot, significant differences in 3′UTRs across samples are identified, which facilitates the integration of 3′UTR quantification into standard scRNA-seq data analysis"
- [abstract] simultaneous quantification of gene and 3'UTR isoform expression from single-cell RNA sequencing (scRNA-seq) data: "simultaneous quantification of gene and 3'UTR isoform expression from single-cell RNA sequencing (scRNA-seq) data"
- [methods] `min_umis`: minimum number of UMIs per cell; cells below this threshold are excluded: "`min_umis`: minimum number of UMIs per cell; cells below this threshold are excluded"
