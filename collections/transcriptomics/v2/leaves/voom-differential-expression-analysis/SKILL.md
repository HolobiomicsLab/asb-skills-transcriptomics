---
name: voom-differential-expression-analysis
description: Use when gene-level RNA-seq read counts (featureCounts output for the
  Prensner PCa cohort) need cancer-vs-normal DE testing with voom and conversion to
  log2(Pvalue)×(-10)×regulation values for FunHoP KEGG node scoring.
when_to_use_negative:
- Counts are already collapsed to node-level KEGG measurements — voom DE must run
  on per-gene featureCounts counts before FunHoP aggregates genes within a node
- Only the probe-level expression table is available (FunHoP reads it solely to extract
  gene names from column 0) with no RNA-seq read-count matrix — voom requires raw
  counts
- Input is raw fastq reads that have not been aligned and counted — run TopHat2 (hg19)
  and featureCounts first
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: voom (limma)
  role: 'Core method: converts raw gene-level read counts to log2-CPM with precision
    weights and fits the cancer-vs-normal linear model to produce the DE statistics'
- name: featureCounts
  role: Produces the per-gene read-count matrix voom consumes, assigning aligned reads
    to genes with gene length defined as the sum of exons
- name: TopHat2
  role: Upstream splice-aware aligner mapping the Prensner-cohort raw fastq reads
    to the hg19 transcriptome before counting
- name: FunHoP
  role: 'Downstream consumer: its count-calculation step takes the voom DE values
    and length-adjusted counts to collapse multi-gene KEGG nodes for Cytoscape'
  repo: https://github.com/kjerstirise/FunHoP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:ed50dd78178a22ba85586ed2cadcc81abf10262c071c3baa371490366577a996
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - FunHoP
  - TopHat2
  - Voom
  - featureCounts
  - voom (limma)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# voom-differential-expression-analysis

## Summary

Runs limma voom differential expression on gene-level RNA-seq read counts (featureCounts on TopHat2/hg19 alignments) to compare cancer vs normal prostate tissue. The resulting P < 0.05 DEG statistics and gene-length-adjusted counts feed FunHoP's collapse of multi-gene KEGG nodes.

## When to use

Use when gene-level RNA-seq read counts (featureCounts output for the Prensner PCa cohort) need cancer-vs-normal DE testing with voom and conversion to log2(Pvalue)×(-10)×regulation values for FunHoP KEGG node scoring.

## When NOT to use

- Counts are already collapsed to node-level KEGG measurements — voom DE must run on per-gene featureCounts counts before FunHoP aggregates genes within a node
- Only the probe-level expression table is available (FunHoP reads it solely to extract gene names from column 0) with no RNA-seq read-count matrix — voom requires raw counts
- Input is raw fastq reads that have not been aligned and counted — run TopHat2 (hg19) and featureCounts first

## Inputs

- Gene-level RNA-seq read-count matrix (featureCounts output, one column per sample)
- Sample metadata with tissue status (cancer vs normal/control); FunHoP's define_groups reads it from column 7
- Expression/gene-name table with gene names in the first row (load_genenames) used to align identifiers
- Gene length table (sum of exons per gene) for the length-adjustment factor

## Outputs

- prenser_gene_counts_de.tsv: gene-length-adjusted read counts plus transformed DE statistics for all genes
- DEG set at P < 0.05 with Value = log2(Pvalue) × (-10) × regulation (regulation = +1 up, -1 down)
- Summary of expression counts per gene used to build Cytoscape colour schemes

## How to apply

Build the input count matrix by mapping Prensner-cohort fastq reads (dbGap phs000443.v1.p1, project No. 5870) to the hg19 transcriptome with TopHat2 and assigning reads to genes with featureCounts, taking gene length as the sum of exons. Run voom on the raw counts with a design encoding the two tissue groups (cancer vs normal) that FunHoP's define_groups function locates in the metadata (column 7). Extract DEGs with P value below 0.05 and transform each P value to Value = log2(Pvalue) × (-10) × regulation, where regulation = 1 for up-regulated and -1 for down-regulated genes, so positive/negative Values track fold-change direction. Adjust all read counts for gene length by multiplying with a factor of each gene's length (sum of exons) divided by the average gene length over all genes; in FunHoP's load_countdata this correction is toggled with the 'correct' flag, set True when the input counts are not yet corrected. Write the length-adjusted counts plus transformed DE statistics for all genes to the prensner gene-counts/DE TSV (prenser_gene_counts_de.tsv), keeping gene names in the first row as load_genenames expects. Validate that the table contains counts and DE fields for all genes and that expression levels are broadly consistent with the TCGA cohort's read counts (highly correlated, Figure 2C).

## Related tools

- **voom (limma)** (Core method: converts raw gene-level read counts to log2-CPM with precision weights and fits the cancer-vs-normal linear model to produce the DE statistics)
- **featureCounts** (Produces the per-gene read-count matrix voom consumes, assigning aligned reads to genes with gene length defined as the sum of exons)
- **TopHat2** (Upstream splice-aware aligner mapping the Prensner-cohort raw fastq reads to the hg19 transcriptome before counting)
- **FunHoP** (Downstream consumer: its count-calculation step takes the voom DE values and length-adjusted counts to collapse multi-gene KEGG nodes for Cytoscape) — https://github.com/kjerstirise/FunHoP

## Examples

```
./funhop.py <hsa list> <expression table> <metadata> <expression counts> <updated gene symbols>
```

## Evaluation signals

- prenser_gene_counts_de.tsv exists and contains read counts plus DE fields for all genes
- Every reported DEG has P < 0.05 and Value = log2(Pvalue) × (-10) × regulation, with sign matching fold-change direction (positive for up-, negative for down-regulated genes)
- Count matrix passes README consistency checks: same number of genes as gene names in the expression table, same number of samples as tissue statuses
- Gene-length adjustment verified: each gene's counts scaled by (sum-of-exons gene length) / (average gene length over all genes)
- Expression levels broadly consistent with the TCGA cohort — highly correlated read counts between the two datasets (Figure 2C)

## Limitations

- FunHoP's Calculate_counts.py hardcodes three assumptions — gene names in the first row (load_genenames), tissue types in column 7 (define_groups), and the gene-length-correction switch ('correct' in load_countdata) — inputs must match or the code must be edited
- Genes with particularly short or long dominant isoforms compared to the canonical isoform model may lead to aberrant expression level estimation
- Regenerating the Prensner counts requires approved dbGap access (phs000443.v1.p1, project No. 5870); KEGG pathway XMLs and the hsa list also need periodic re-download since KEGG is constantly updated

## Evidence

- [methods] Voom used for differential expression analysis: "Voom [26] was further used for differential expression analysis"
- [methods] DEG extraction cutoff: "DEGs with a P value below 0.05 were extracted"
- [methods] P-value transformation formula for DE values: "P values were log2 transformed by:(1)Value=log2Pvalue×(-10)×regulation"
- [methods] Gene length adjustment factor definition: "All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes"
- [methods] featureCounts assigns reads to genes: "featureCounts [25] was used to assign the reads to each gene"
- [readme] Gene-length correction switch in load_countdata: "If the data is not corrected on gene length, "correct" should be set to True when calling the load_countdata function"
- [readme] define_groups locates the two tissue types: "In the function define_groups, where it finds the two different types of tissue used for differential expression later on"
- [readme] Counts-matrix consistency requirement: "A matrix containing all the read counts for all the samples. Note that this should have the same number of genes as the gene names found in the expression table."
