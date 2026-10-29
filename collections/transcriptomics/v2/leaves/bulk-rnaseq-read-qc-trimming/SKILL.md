---
name: bulk-rnaseq-read-qc-trimming
description: Use when raw bulk RNA-seq FASTQ files (e.g. anti-PD-1 vs control CD8
  TIL samples from GEO/SRA, such as GSE62771) need QC and trimming before Salmon quasi-mapping
  quantification.
when_to_use_negative:
- Input is single-cell RNA-seq data (Smart-seq2 plate-based or 10X droplet-based),
  which follows a different QC/alignment path than bulk FASTQ trimming
- Data is already quantified transcript-level abundance (e.g. Salmon/Kallisto output)
  and only needs tximport aggregation and DESeq2 modeling
- Input is already a gene-level count matrix — trimming has no effect at this stage
edam_operation: http://edamontology.org/operation_3220
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: Trimmomatic
  role: Trims adapters and low-quality bases from bulk RNA-seq reads before quantification
- name: Salmon
  role: 'Downstream quantifier: quasi-maps trimmed reads against the mouse transcriptome
    to yield transcript abundances'
  repo: https://github.com/COMBINE-lab/salmon/releases
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:ca8fb869bf75a8a6c586f0fe387c22ea2b0a0a46b0afbb341fdead065b6b23ea
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/carmonalab/TILPRED
  tools:
  - BiocManager
  - DESeq2
  - R
  - Salmon
  - Trimmomatic
  - clusterProfiler
  - tximport
  edam_operation: http://edamontology.org/operation_3220
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# bulk-rnaseq-read-qc-trimming

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quality-control and adapter/low-quality trimming of raw bulk RNA-seq FASTQ reads (via Trimmomatic) as the first stage of a pipeline that quantifies CD8 TIL bulk transcriptomes with Salmon and tests EM-like signature enrichment upon anti-PD-1 therapy.

## When to use

Use when raw bulk RNA-seq FASTQ files (e.g. anti-PD-1 vs control CD8 TIL samples from GEO/SRA, such as GSE62771) need QC and trimming before Salmon quasi-mapping quantification.

## When NOT to use

- Input is single-cell RNA-seq data (Smart-seq2 plate-based or 10X droplet-based), which follows a different QC/alignment path than bulk FASTQ trimming
- Data is already quantified transcript-level abundance (e.g. Salmon/Kallisto output) and only needs tximport aggregation and DESeq2 modeling
- Input is already a gene-level count matrix — trimming has no effect at this stage

## Inputs

- Raw bulk RNA-seq FASTQ files (paired-end) downloaded from GEO/SRA, e.g. GSE62771 anti-PD-1 and control CD8 TIL samples
- Sample metadata (anti-PD-1 vs control condition labels)

## Outputs

- Adapter- and quality-trimmed FASTQ files ready for Salmon quasi-mapping
- QC summary of removed adapters/low-quality bases per sample

## How to apply

Download the raw FASTQ files together with sample metadata (anti-PD-1 vs control CD8 TIL samples) from GEO/SRA — for the Gubin et al. murine sarcoma dataset this is accession GSE62771. Trim adapters and low-quality bases from the reads with Trimmomatic, removing contaminating adapter sequences and low-quality tails so they cannot corrupt quasi-mapping. Feed the trimmed FASTQ files directly into Salmon using quasi-mapping against the mouse transcriptome to produce per-sample transcript-level abundance estimates. Rationale: trimming precedes quantification because untrimmed adapters and degraded bases degrade Salmon's mapping accuracy, which in turn propagates into the tximport gene-level counts, the DESeq2 anti-PD-1 vs control differential expression model, and the final GSEA test of the EM-like CD8 TIL signature (target: reproduce the nominal p = 0.035).

## Related tools

- **Trimmomatic** (Trims adapters and low-quality bases from bulk RNA-seq reads before quantification)
- **Salmon** (Downstream quantifier: quasi-maps trimmed reads against the mouse transcriptome to yield transcript abundances) — https://github.com/COMBINE-lab/salmon/releases

## Evaluation signals

- Trimming report shows adapter and low-quality base removal without discarding the majority of reads from GSE62771 samples
- Salmon quasi-mapping succeeds on trimmed reads and produces per-sample transcript abundance estimates consistent with the mouse transcriptome reference
- tximport aggregation to gene-level counts feeds a DESeq2 model that converges on anti-PD-1 vs control contrast
- Downstream GSEA with the EM-like CD8 TIL signature reproduces the reported nominal p = 0.035

## Limitations

- The enrichment result is dataset-specific (Gubin et al. murine sarcoma, GSE62771); the reference CD8 TIL map was built in B16 melanoma and a single mouse strain, and the authors note 'Future studies of additional mouse strains and cancer types will help us further refine the CD8 TIL landscape'
- Reanalyzed external bulk/scRNA-seq datasets in the article are cited without accession numbers or repository identifiers, complicating provenance

## Evidence

- [other] FASTQ download and metadata: [evidence span withheld — anchor status: not_found]
- [other] Trimmomatic trimming step: [evidence span withheld — anchor status: not_found]
- [other] Salmon quantification on trimmed reads: [evidence span withheld — anchor status: not_found]
- [discussion] GSEA target result: "Our meta-analysis of bulk and single-cell transcriptomic data showed a selective enrichment of the EM-like signature"
- [readme] Salmon as alternative quantifier: [evidence span withheld — anchor status: external_doc]
