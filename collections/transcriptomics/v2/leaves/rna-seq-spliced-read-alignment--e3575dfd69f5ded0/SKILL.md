---
name: rna-seq-spliced-read-alignment
description: Use when raw RNA-seq FASTQ files must be converted into per-gene read
  counts (e.g., for the Prensner PCa cohort from dbGap phs000443.v1.p1) prior to differential
  expression or pathway-node aggregation.
when_to_use_negative:
- Input is already a gene-level count matrix or a corrected expression table — in
  the FunHoP workflow, set correct=False semantics apply only when data is already
  gene-length corrected (load_countdata 'correct' flag)
- Data are microarray probe intensities rather than RNA-seq FASTQ reads — the expression
  table in FunHoP only supplies gene names/IDs, not reads to align
- Target pathways use KEGG category 1.7 'glycan biosynthesis and metabolism' link-based
  entries, which cannot be expanded or colored regardless of alignment quality
edam_operation: http://edamontology.org/operation_3258
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: TopHat2
  role: Splice-aware aligner mapping raw RNA-seq reads to the hg19 transcriptome
- name: featureCounts
  role: Assigns aligned reads to each gene, taking gene length as the sum of exons
- name: Voom
  role: Differential expression analysis on the gene-level counts (cancer vs normal)
- name: FunHoP
  role: 'Downstream consumer: collapses length-adjusted counts of multi-gene KEGG
    nodes into one expression value'
  repo: https://github.com/kjerstirise/FunHoP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2021.03.003
    title: 'FunHoP: Enhanced Visualization and Analysis of Functionally Homologous
      Proteins in Complex Metabolic Networks'
  source_content_hash: sha256:0cd24ca02c4ed84f4abab27735359ea09624a76b822e22062129c12b88dafae0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kjerstirise/FunHoP
  tools:
  - FunHoP
  - TopHat2
  - Voom
  - featureCounts
  edam_operation: http://edamontology.org/operation_3258
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# rna-seq-spliced-read-alignment

## Summary

Splice-aware alignment of raw RNA-seq FASTQ reads to a reference transcriptome (hg19) using TopHat2, followed by gene-level read assignment with featureCounts, as the read-count generation step underlying FunHoP's node-level expression aggregation.

## When to use

Use when raw RNA-seq FASTQ files must be converted into per-gene read counts (e.g., for the Prensner PCa cohort from dbGap phs000443.v1.p1) prior to differential expression or pathway-node aggregation.

## When NOT to use

- Input is already a gene-level count matrix or a corrected expression table — in the FunHoP workflow, set correct=False semantics apply only when data is already gene-length corrected (load_countdata 'correct' flag)
- Data are microarray probe intensities rather than RNA-seq FASTQ reads — the expression table in FunHoP only supplies gene names/IDs, not reads to align
- Target pathways use KEGG category 1.7 'glycan biosynthesis and metabolism' link-based entries, which cannot be expanded or colored regardless of alignment quality

## Inputs

- Raw RNA-seq reads in FASTQ format (e.g., dbGap phs000443.v1.p1 Prensner PCa cohort, with dbGap approval)
- Reference transcriptome hg19
- Gene annotation for featureCounts (exon boundaries for sum-of-exons gene lengths)

## Outputs

- Gene-level read-count matrix (counts per gene per sample, genes as rows)
- Gene length-adjusted read counts (factor = gene length / average gene length)
- DE statistics per gene (P value < 0.05, Value = log2(Pvalue) x (-10) x regulation)
- Combined table prensner_gene_counts_de.tsv with counts and DE fields for all genes

## How to apply

Download raw RNA-seq FASTQ files (with data-access approval, e.g. dbGap phs000443.v1.p1, project No. 5870) and align them to the hg19 transcriptome using TopHat2 splice-aware alignment so reads spanning exon junctions are mapped correctly. Assign aligned reads to each gene with featureCounts, taking gene length as the sum of exons. Proceed to voom differential expression (cancer vs normal), extract DEGs with P < 0.05, and compute length-adjusted counts by multiplying each gene's counts by (gene length / average gene length over all genes). Correct length adjustment is essential because FunHoP weights reads on gene length and ratio when collapsing multi-gene KEGG nodes into a single measurement. Write counts plus transformed DE statistics (Value = log2(Pvalue) x (-10) x regulation, regulation = +/-1) to prensner_gene_counts_de.tsv.

## Related tools

- **TopHat2** (Splice-aware aligner mapping raw RNA-seq reads to the hg19 transcriptome)
- **featureCounts** (Assigns aligned reads to each gene, taking gene length as the sum of exons)
- **Voom** (Differential expression analysis on the gene-level counts (cancer vs normal))
- **FunHoP** (Downstream consumer: collapses length-adjusted counts of multi-gene KEGG nodes into one expression value) — https://github.com/kjerstirise/FunHoP

## Examples

```
tophat2 -o sample_tophat2_out hg19 sample_1.fastq.gz sample_2.fastq.gz && featureCounts -a genes.gtf -o gene_counts.txt sample_tophat2_out/accepted_hits.bam
```

## Evaluation signals

- prensnner_gene_counts_de.tsv exists and contains both count and DE fields for all genes
- Aligned reads assigned by featureCounts use gene length defined as the sum of exons, and length-adjusted counts use factor = gene length / average gene length over all genes
- DEG extraction retains only genes with P value below 0.05, with Value sign reflecting regulation (+1 up, -1 down)
- Expression levels broadly consistent with the TCGA cohort: highly correlated read counts between the two datasets (Figure 2C)
- Spliced reads crossing exon junctions are mapped (splice-aware TopHat2 alignment to hg19 succeeded for all samples)

## Limitations

- dbGap raw reads require data-access approval (phs000443.v1.p1, project No. 5870), so the alignment cannot be re-run without authorization
- Genes with particularly short or long dominant isoforms compared to the canonical isoform model may lead to aberrant expression level estimation
- Results are transcriptome-reference-specific (hg19); the FunHoP README notes KEGG pathway files and HSA ID lists change over time and should be refreshed

## Evidence

- [methods] Raw RNA-seq reads were mapped to the hg19 transcriptome using TopHat2: "Raw RNA-seq reads were mapped to the hg19 transcriptome using TopHat2"
- [methods] featureCounts assign reads: "featureCounts [25] was used to assign the reads to each gene"
- [methods] All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes: "All read counts were adjusted for gene lengths by a factor estimated by taking the gene length of the respective gene (sum of exons) divided by the average gene length over all genes"
- [methods] RNA-seq raw reads in fastq-format were downloaded with approval from The database of Genotypes and Phenotypes (dbGap: phs000443.v1.p1, project No. 5870): "RNA-seq raw reads in fastq-format were downloaded with approval from The database of Genotypes and Phenotypes (dbGap: phs000443.v1.p1, project No. 5870)"
- [abstract] Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads, and weighting the reads on gene length and ratio, a new expression value is calculated: "Assuming that activity for an enzymatic reaction mainly depends upon the gene with the highest number of reads, and weighting the reads on gene length and ratio, a new expression value is calculated"
