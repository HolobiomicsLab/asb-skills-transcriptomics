---
name: gene-level-read-count-quantification
description: Use when starting from raw RNA-seq FASTQ files of a patient cohort and
  per-gene read counts plus DE statistics (P value, regulation direction) are needed
  as downstream input for node-level pathway aggregation, e.g. FunHoP/KEGG network
  analysis.
when_to_use_negative:
- Input is already a gene-level count matrix (e.g. TCGA read counts with gene identifiers
  downloaded directly) — only the voom DE step and gene-length adjustment are needed,
  not alignment/assignment.
- The goal is isoform-level or transcript-level quantification — KEGG provides no
  curated isoform enzyme-activity data, so this skill is deliberately limited to the
  gene level.
- Analysis targets KEGG glycan-related pathways (category 1.7, e.g. 'Glycan biosynthesis
  and metabolism') — these use 'links' instead of nodes and are excluded from FunHoP.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: TopHat2
  role: splice-aware alignment of raw RNA-seq reads to the hg19 transcriptome
- name: featureCounts
  role: assigns aligned reads to each gene, taking gene length as the sum of exons
- name: Voom
  role: differential expression analysis comparing cancer vs normal tissue samples
- name: FunHoP
  role: 'downstream consumer: collapses length-adjusted counts of all genes in a multi-gene
    KEGG node into one expression value'
  repo: https://github.com/kjerstirise/FunHoP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:d6eca0b11dd3766192bcce2020bcb69fdd51c1d8682c317f49b11472b7f12893
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - FunHoP
  - TopHat2
  - Voom
  - featureCounts
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-level-read-count-quantification

## Summary

Quantifies raw RNA-seq FASTQ reads into per-gene read counts and per-gene differential expression statistics, following the FunHoP pipeline: TopHat2 splice-aware alignment to hg19, featureCounts assignment (gene length = sum of exons), and voom differential expression between cancer and normal tissue.

## When to use

Use when starting from raw RNA-seq FASTQ files of a patient cohort and per-gene read counts plus DE statistics (P value, regulation direction) are needed as downstream input for node-level pathway aggregation, e.g. FunHoP/KEGG network analysis.

## When NOT to use

- Input is already a gene-level count matrix (e.g. TCGA read counts with gene identifiers downloaded directly) — only the voom DE step and gene-length adjustment are needed, not alignment/assignment.
- The goal is isoform-level or transcript-level quantification — KEGG provides no curated isoform enzyme-activity data, so this skill is deliberately limited to the gene level.
- Analysis targets KEGG glycan-related pathways (category 1.7, e.g. 'Glycan biosynthesis and metabolism') — these use 'links' instead of nodes and are excluded from FunHoP.

## Inputs

- raw RNA-seq fastq files from the patient cohort (e.g. dbGap phs000443.v1.p1 PCa raw reads)
- hg19 reference transcriptome for TopHat2 splice-aware alignment
- sample metadata distinguishing cancer vs normal/control tissue

## Outputs

- per-gene read-count matrix with counts adjusted by gene length (sum of exons)
- differential expression table (e.g. prensner_gene_counts_de.tsv) with P values, regulation direction, and transformed Value = log2(Pvalue) × (-10) × regulation for DEGs with P < 0.05

## How to apply

Download the cohort's raw fastq files (for the Prensner PCa cohort, from dbGap phs000443.v1.p1, project No. 5870, with data-access approval) and map reads to the hg19 transcriptome using TopHat2 splice-aware alignment. Assign aligned reads to each gene with featureCounts, taking gene length as the sum of exons. Run voom differential expression analysis comparing cancer vs normal tissue samples, extract DEGs with P value below 0.05, and transform each as Value = log2(Pvalue) × (-10) × regulation, with regulation = 1 for up-regulated and -1 for down-regulated genes. Additionally adjust all read counts for gene length by multiplying with the factor (gene length as sum of exons) / (average gene length over all genes), and write counts plus transformed DE statistics to a table such as prensner_gene_counts_de.tsv. The gene length correction matters because FunHoP collapses multi-gene KEGG nodes by weighting reads on gene length and ratio, so the correction flag ('correct' set to True in load_countdata) must match whether counts are already length-corrected.

## Related tools

- **TopHat2** (splice-aware alignment of raw RNA-seq reads to the hg19 transcriptome)
- **featureCounts** (assigns aligned reads to each gene, taking gene length as the sum of exons)
- **Voom** (differential expression analysis comparing cancer vs normal tissue samples)
- **FunHoP** (downstream consumer: collapses length-adjusted counts of all genes in a multi-gene KEGG node into one expression value) — https://github.com/kjerstirise/FunHoP

## Examples

```
./funhop.py hsa_list.tsv expression_table.tsv metadata.tsv expression_counts.tsv updated_gene_symbols.tsv
```

## Evaluation signals

- Output table (e.g. prensner_gene_counts_de.tsv) exists and contains read counts plus DE fields (P value, regulation, transformed Value) for all genes.
- DE statistics follow the paper's transform: Value = log2(Pvalue) × (-10) × regulation, with regulation = 1/-1 for up/down-regulated genes and only DEGs with P < 0.05 retained.
- Read counts are adjusted by the factor (gene length as sum of exons) / (average gene length over all genes) before downstream use.
- Expression levels are broadly consistent with the TCGA cohort as reported in the paper (highly correlated read counts between Prensner and TCGA datasets, Figure 2C).
- Count matrix gene set matches the gene-name list from the expression table (same number of genes), as required by the FunHoP input convention.

## Limitations

- Genes with particularly short or long dominant isoforms compared to the canonical isoform model may lead to aberrant expression level estimation.
- Enzyme kinetics/efficiency data are usually not available, so the equal enzyme rate assumption behind read-count weighting is a simplification.
- Raw reads from dbGap require data-access approval; the correction flag in load_countdata (and the gene-name/metadata column positions in Calculate_counts.py) is hard-coded and must be checked manually.

## Evidence

- [methods] Raw RNA-seq reads were mapped to the hg19 transcriptome using TopHat2: "Raw RNA-seq reads were mapped to the hg19 transcriptome using TopHat2"
- [methods] featureCounts assign reads: "featureCounts [25] was used to assign the reads to each gene"
- [methods] gene length adjustment: "All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes"
- [readme] funhop invocation: "./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>"
- [readme] load_countdata correction flag: "If the data is not corrected on gene length, "correct" should be set to True when calling the load_countdata function"
