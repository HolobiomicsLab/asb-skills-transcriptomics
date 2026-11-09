---
name: differential-transcript-usage-analysis
description: Use when you have condition-grouped RNA-seq samples (e.g., 3 control
  vs 3 LPS-stimulated monocytes from GSE118165) with sample-level IPA usage estimates
  and need to identify differentially used IPA isoforms between groups.
when_to_use_negative:
- You only need to quantify APA usage in annotated 3'UTRs — tools like DaPars, QAPA,
  or LABRAT suffice, as InPACT/DRIMSeq here targets novel intronic IPA isoforms
- Your reads are not aligned to a sorted BAM with gene-level grouping feasible, or
  transcript counts are too low to fit the Dirichlet-multinomial (e.g., genes filtered
  out by TPM < 1)
- You lack replication or condition grouping — differential usage testing requires
  multiple samples per group
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: InPACT
  role: Identifies novel skipped and composite IPA sites, assembles novel IPA isoforms
    (InPACT_transcript), and computes per-sample IPA usage (InPACT_quantify) to feed
    differential testing
  repo: https://github.com/YY-TMU/InPACT
- name: Salmon
  role: Indexes and quantifies transcript-level abundance (salmon index -k 31, salmon
    quant --validateMappings) of annotated plus novel IPA transcripts extracted with
    gffread
  repo: https://github.com/COMBINE-lab/salmon
- name: DRIMSeq
  role: Performs differential transcript usage analysis between control and LPS-stimulated
    groups with a Dirichlet-multinomial framework
- name: ClusterProfiler
  role: Version 3.18.1; performs Gene Ontology biological-process enrichment on genes
    carrying differentially used IPA sites at FDR < 0.05
- name: gffread
  role: Extracts transcript sequences from merged.gtf to FASTA for building the Salmon
    index
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:b96c0c3b249fd9e7a95ca88ea13af29c3c53ef4b085da07cb7012d99f90c1236
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - ClusterProfiler
  - DRIMSeq
  - InPACT
  - Salmon
  - gffread
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# differential-transcript-usage-analysis

## Summary

Detect IPA isoforms whose relative usage changes between biological conditions by combining InPACT-derived IPA usage estimates with DRIMSeq's Dirichlet-multinomial differential transcript usage framework, as demonstrated on untreated vs LPS-stimulated human monocyte RNA-seq.

## When to use

Use when you have condition-grouped RNA-seq samples (e.g., 3 control vs 3 LPS-stimulated monocytes from GSE118165) with sample-level IPA usage estimates and need to identify differentially used IPA isoforms between groups.

## When NOT to use

- You only need to quantify APA usage in annotated 3'UTRs — tools like DaPars, QAPA, or LABRAT suffice, as InPACT/DRIMSeq here targets novel intronic IPA isoforms
- Your reads are not aligned to a sorted BAM with gene-level grouping feasible, or transcript counts are too low to fit the Dirichlet-multinomial (e.g., genes filtered out by TPM < 1)
- You lack replication or condition grouping — differential usage testing requires multiple samples per group

## Inputs

- Coordinate-sorted BAM files of RNA-seq samples from two condition groups (e.g., GSE118165 control and 100 ng/ml LPS 6 h monocytes, 3 vs 3)
- RefSeq GRCh38 GTF annotation (hg38.109.20211119.ncbiRefSeq.gtf.gz)
- Pre-scored polyA site annotation for GRCh38 (InPACT_polyAsites.hg38.saf)
- Per-sample IPA usage / transcript-level TPM table from InPACT_quantify and Salmon quant.sf output

## Outputs

- Table of novel IPA sites classified as skipped or composite IPA events
- DRIMSeq differential transcript usage table with log2 fold change and FDR per IPA isoform (e.g., 204 significant events at |log2FC| > 1, FDR < 0.05)
- ClusterProfiler Gene Ontology biological-process enrichment table for genes with differentially used IPA sites (FDR < 0.05)

## How to apply

First run InPACT on each coordinate-sorted BAM (with the RefSeq GRCh38 GTF and pre-scored InPACT_polyAsites.hg38.saf, minimum 5 spliced reads for skipped terminal exons and 10 spanning reads for composite terminal exons) to identify novel IPA sites and assemble IPA isoforms, then quantify transcript abundance with Salmon (gffread-extracted transcripts, salmon index -k 31, salmon quant --validateMappings) and compute per-sample IPA usage with InPACT_quantify. Feed the per-sample transcript counts/usage of annotated plus novel IPA isoforms into DRIMSeq, which models transcript proportions with a Dirichlet-multinomial to test differential transcript usage between control and LPS-stimulated groups. Filter low-abundance genes with TPM < 1 before testing, and call significantly differentially used IPA isoforms at |log2fold_change| > 1 and FDR < 0.05. Finally, run Gene Ontology biological-process enrichment on genes carrying differentially used IPA sites with ClusterProfiler version 3.18.1, defining significantly enriched terms at FDR < 0.05.

## Related tools

- **InPACT** (Identifies novel skipped and composite IPA sites, assembles novel IPA isoforms (InPACT_transcript), and computes per-sample IPA usage (InPACT_quantify) to feed differential testing) — https://github.com/YY-TMU/InPACT
- **Salmon** (Indexes and quantifies transcript-level abundance (salmon index -k 31, salmon quant --validateMappings) of annotated plus novel IPA transcripts extracted with gffread) — https://github.com/COMBINE-lab/salmon
- **DRIMSeq** (Performs differential transcript usage analysis between control and LPS-stimulated groups with a Dirichlet-multinomial framework)
- **ClusterProfiler** (Version 3.18.1; performs Gene Ontology biological-process enrichment on genes carrying differentially used IPA sites at FDR < 0.05)
- **gffread** (Extracts transcript sequences from merged.gtf to FASTA for building the Salmon index)

## Examples

```
InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5
```

## Evaluation signals

- The three output tables exist with correct structure (novel IPA sites, DRIMSeq DTU results, GO enrichment)
- Novel IPA site counts reproduce the reported totals for monocyte data: 2977 sites total, 1105 skipped and 1872 composite events
- The DRIMSeq table contains 204 significant events at the stated cutoffs (|log2fold_change| > 1 and FDR < 0.05)
- Every tested gene passed the low-abundance filter (TPM >= 1) before DRIMSeq testing
- GO enrichment results contain biological-process terms with FDR < 0.05, restricted to genes carrying differentially used IPA sites

## Limitations

- Differential usage depends on adequate transcript-level quantification of novel IPA isoforms; Salmon's index (k = 31) and the read thresholds (5 spliced reads for skipped, 10 spanning reads for composite terminal exons) can miss lowly expressed IPA events
- Genes with TPM < 1 are filtered out and cannot be tested, so differential IPA usage in very low-abundance genes is undetectable
- The cutoff |log2fold_change| > 1 with FDR < 0.05 is stringent; IPA events with moderate usage shifts will be missed
- InPACT output distinguishes only skipped and composite IPA types, and results depend on the provided pre-scored polyA site annotation for hg38/RefSeq

## Evidence

- [methods] we applied DRIMSeq to conduct differential transcript usage analysis between control and LPS stimulated group: "we applied DRIMSeq to conduct differential transcript usage analysis between control and LPS stimulated group"
- [methods] cutoff of |log2fold_change| > 1 and FDR < 0.05: "by the cutoff of | log2fold_change | > 1 and FDR < 0.05, thereby identifying the differentially used IPA sites"
- [methods] low-abundance genes with TPM < 1 were filtered out: "low-abundance genes with TPM < 1 were filtered out"
- [readme] salmon quant command with validateMappings: "salmon quant -i transcripts_index -l <LIBTYPE> -r reads.fq --validateMappings -o transcripts_quant"
- [methods] minimum spliced and span read thresholds for IPA identification: "Minimum number of spliced reads required to consider the potential skipped terminal exon. [default=5]"
