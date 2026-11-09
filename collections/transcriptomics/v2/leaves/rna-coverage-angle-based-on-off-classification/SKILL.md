---
name: rna-coverage-angle-based-on-off-classification
description: Use when preparing base-level RNA-seq pileup matrices (e.g., TCGA cohort)
  for SCISSOR shape-change detection and genes with many non-expressed samples must
  be identified and excluded.
when_to_use_negative:
- Genes have already been filtered by expression-level thresholds (e.g., percentile
  cutoffs on median and MAD of normalized RSEM values), so on/off shape classification
  would be redundant
- The cohort is too small or too heterogeneous in library preparation for a gene mean
  vector to be a meaningful shape reference (SCISSOR has been used successfully down
  to ~30 samples)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCISSOR
  role: R package providing build_pileup/get_Ranges for the pileup matrix, angle.hy
    for the sample-vs-mean angle statistic, and get_exon_offstat/get_offstat for per-gene
    off-sample fractions
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:8013081aeabeeeded182394cc8be3679ae3d3edb93a80daf17714f4c086ce5b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# rna-coverage-angle-based-on-off-classification

## Summary

A SCISSOR-based on/off gene classification step that measures, per gene, the high-dimensional angle between each sample's base-level coverage vector and the gene mean vector, then flags genes with many 'off' samples (>20%) for exclusion before genome-wide shape-change analysis.

## When to use

Use when preparing base-level RNA-seq pileup matrices (e.g., TCGA cohort) for SCISSOR shape-change detection and genes with many non-expressed samples must be identified and excluded.

## When NOT to use

- Genes have already been filtered by expression-level thresholds (e.g., percentile cutoffs on median and MAD of normalized RSEM values), so on/off shape classification would be redundant
- The cohort is too small or too heterogeneous in library preparation for a gene mean vector to be a meaningful shape reference (SCISSOR has been used successfully down to ~30 samples)

## Inputs

- Base-level RNA-seq coverage pileup matrix per gene (from SCISSOR build_pileup)
- Genomic ranges for gene models (from SCISSOR get_Ranges, e.g., built from Homo_sapiens.GRCh37.87.gtf)
- Sample set already screened for RNA degradation (e.g., 522 TCGA HNSC tumors minus 70 severely degraded cases)

## Outputs

- Per-gene table of off-sample fractions (e.g., onoff_gene_fractions.csv)
- List of excluded on/off genes (genes with >20% off samples; 5802 genes, ~28%, in TCGA HNSC)
- Filtered pileup matrix restricted to retained genes for downstream GSC/LSC analysis

## How to apply

Load the genome-wide base-level pileup matrix produced by SCISSOR's build_pileup (for TCGA HNSC, the 522-sample cohort after excluding 70 severely degraded cases) together with genomic ranges from get_Ranges. For each gene, compute the mean coverage vector across all samples as the gene-shape reference. Use SCISSOR's angle computation (angle.hy) to measure the angle between each individual sample vector and the gene mean vector as the shape-similarity statistic, then apply the on/off classification to each sample-gene pair. Aggregate the per-gene fraction of off samples with get_exon_offstat / get_offstat and flag genes with more than 20% off samples as excluded. Rationale: SCISSOR develops a per-gene definition of 'not expressed' versus 'expressed' specifically so genes with many non-expressed samples do not confound downstream shape-change detection; on TCGA HNSC this excluded 5802 genes (~28%).

## Related tools

- **SCISSOR** (R package providing build_pileup/get_Ranges for the pileup matrix, angle.hy for the sample-vs-mean angle statistic, and get_exon_offstat/get_offstat for per-gene off-sample fractions) — https://github.com/hyochoi/SCISSOR

## Examples

```
library(devtools); install_github("hyochoi/SCISSOR"); library(SCISSOR); pileup <- build_pileup(...); ranges <- get_Ranges(...)
```

## Evaluation signals

- The number of genes flagged with >20% off samples should reproduce the reported 5802 genes (~28% of genes) on the TCGA HNSC 522-sample cohort (after removing 70 degraded cases)
- Every gene in the excluded list has an off-sample fraction strictly greater than 0.20 in the per-gene table
- The per-gene off-fraction table contains both off fractions and the excluded-gene list (e.g., onoff_gene_fractions.csv written without missing gene rows)
- Downstream shape-change (GSC/LSC) detection runs only on retained genes, i.e., the input matrix dimensions match the retained-gene list

## Limitations

- SCISSOR tuning parameters such as log normalization and the normality cutoff value likely require optimization when a significantly different wet lab RNA-seq protocol is used, which shifts on/off classifications
- The angle-based on/off definition depends on the pileup being base-level and uncompressed; exon-collapsed representations (DEXSeq, SpliceTrap) cannot support it
- Genes excluded as on/off (~28% in HNSC) are removed from all downstream analysis, so shape changes in those genes are not discoverable

## Evidence

- [results] Per gene definition of 'not expressed' versus 'expressed' developed to exclude genes with many non-expressed samples: "A per gene definition of “not expressed” versus “expressed” was developed for the purposes of excluding genes with many non-expressed samples"
- [results] 5802 genes (~28%) with many non-expressed samples excluded from downstream analysis: "we identified 5802 genes (~28%) and they were excluded from the downstream analysis"
- [methods] Pileup matrix and genomic ranges as inputs to SCISSOR functions: "We first obtain pileup data matrix using `build_pileup` and genomic ranges using `get_Ranges`"
- [methods] SCISSOR availability as an R package: "`SCISSOR` is free software and available on [Github](https://github.com/hyochoi/SCISSOR)."
- [readme] SCISSOR's base-level, uncompressed view enabling unbiased structural-alteration screening: "SCISSOR considers a novel shape property of aligned short read data through a base-level pileup file."
