---
name: cross-cohort-expression-consistency-check
description: Use when a derived gene read-count or DE table from one RNA-seq cohort
  must be validated against a second public cohort (e.g. dbGap vs TCGA) before downstream
  pathway interpretation.
when_to_use_negative:
- Only one cohort is available, so no independent replication dataset exists to correlate
  against
- The cohorts used different aligners or gene definitions that were not harmonized,
  making direct read-count correlation invalid
- The goal is pathway visualization itself rather than validation of expression measurements
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: TopHat2
  role: Splice-aware alignment of raw RNA-seq fastq reads to hg19 for each cohort
    so counts are comparable
- name: featureCounts
  role: Assigns aligned reads to each gene (gene length = sum of exons), producing
    the per-cohort count matrix used for cross-cohort correlation
- name: Voom
  role: Differential expression analysis (cancer vs normal) generating the P values
    and regulation signs validated for cohort consistency
- name: FunHoP
  role: Downstream method whose input counts and DE statistics are the objects of
    this consistency check
  repo: https://github.com/kjerstirise/FunHoP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:e7350b05d5eef665447018a6792693bc4af08cfc3c1df34a392b658d42c3b6d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - FunHoP
  - TopHat2
  - Voom
  - featureCounts
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cross-cohort-expression-consistency-check

## Summary

A validation step for the FunHoP prostate cancer workflow that checks whether independently processed RNA-seq cohorts (Prensner dbGap phs000443.v1.p1 and TCGA) yield broadly consistent gene expression measurements before cohort-specific results are trusted for pathway analysis.

## When to use

Use when a derived gene read-count or DE table from one RNA-seq cohort must be validated against a second public cohort (e.g. dbGap vs TCGA) before downstream pathway interpretation.

## When NOT to use

- Only one cohort is available, so no independent replication dataset exists to correlate against
- The cohorts used different aligners or gene definitions that were not harmonized, making direct read-count correlation invalid
- The goal is pathway visualization itself rather than validation of expression measurements

## Inputs

- Gene-level read-count table from cohort 1 (e.g. Prensner cohort, derived from TopHat2 + featureCounts)
- Read counts and gene identifiers downloaded from TCGA
- prensner_gene_counts_de.tsv with length-adjusted counts and DE statistics (P value, regulation direction)

## Outputs

- Cross-cohort read-count correlation assessment (Figure 2C-style comparison of Prensner vs TCGA)
- Validation verdict confirming the output table contains counts and DE fields for all genes and expression levels are broadly consistent with the TCGA cohort

## How to apply

After independently processing each cohort — aligning raw fastq reads to hg19 with TopHat2, assigning reads to genes with featureCounts (gene length = sum of exons), and running voom differential expression comparing cancer vs normal tissue — compute the read counts for each cohort and correlate them across genes. In the FunHoP prostate cancer study this meant checking the Prensner cohort (dbGap phs000443.v1.p1, project No. 5870) against TCGA read counts and gene identifiers, with the expected outcome of highly correlated read counts between the two datasets (paper Figure 2C). Broad consistency of expression levels across cohorts is the acceptance criterion that justifies using the newly generated prensner_gene_counts_de.tsv (counts plus log2-transformed DE statistics, Value = log2(Pvalue) x (-10) x regulation, DEGs with P < 0.05) as a faithful replication of previously published metabolic signals.

## Related tools

- **TopHat2** (Splice-aware alignment of raw RNA-seq fastq reads to hg19 for each cohort so counts are comparable)
- **featureCounts** (Assigns aligned reads to each gene (gene length = sum of exons), producing the per-cohort count matrix used for cross-cohort correlation)
- **Voom** (Differential expression analysis (cancer vs normal) generating the P values and regulation signs validated for cohort consistency)
- **FunHoP** (Downstream method whose input counts and DE statistics are the objects of this consistency check) — https://github.com/kjerstirise/FunHoP

## Examples

```
./funhop.py hsa_list.txt expression_table.tsv metadata.tsv counts.tsv updated_gene_symbols.tsv  # then correlate resulting counts against TCGA read counts
```

## Evaluation signals

- Read counts between the Prensner cohort and the TCGA cohort are highly correlated (paper Figure 2C expectation)
- prensner_gene_counts_de.tsv exists and contains read counts plus DE fields (P value, log2-transformed Value, regulation sign) for all genes
- Length adjustment factor (gene length / average gene length over all genes) was applied to all read counts before comparison
- DEG extraction threshold of P value below 0.05 was respected, with Value = log2(Pvalue) x (-10) x regulation producing positive values for up- and negative for down-regulated genes
- Expression levels are broadly consistent with the TCGA cohort as reported in the paper

## Limitations

- Cross-cohort agreement requires matching gene identifiers and comparable preprocessing; different mapping or counting choices can mask true consistency
- KEGG's lack of curated isoform-level enzyme activity data and the canonical isoform assumption may cause aberrant expression estimates for genes with atypical dominant isoforms, affecting cross-cohort agreement at gene level
- Glycan-related KEGG pathways (category 1.7) using 'links' instead of nodes are excluded, so consistency checks only cover supported human metabolic pathways

## Evidence

- [methods] RNA-seq data for PCa (read counts and gene identifiers) were downloaded from The Cancer Genome Atlas (TCGA): "RNA-seq data for PCa (read counts and gene identifiers) were downloaded from The Cancer Genome Atlas (TCGA) [22]"
- [methods] Raw RNA-seq reads were mapped to the hg19 transcriptome using TopHat2: "Raw RNA-seq reads were mapped to the hg19 transcriptome using TopHat2"
- [methods] All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes: "All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons)"
- [other] verify the output table exists and contains counts and DE fields for all genes, and confirm the expression levels are broadly consistent with the TCGA cohort as reported in the paper: [evidence span withheld — anchor status: not_found]
- [methods] DEGs with a P value below 0.05 were extracted: "DEGs with a P value below 0.05 were extracted"
