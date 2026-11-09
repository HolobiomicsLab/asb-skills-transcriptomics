---
name: sra-fastq-data-retrieval
description: Use when raw RNA-seq data for a study must be fetched from NCBI SRA via
  SRR/ERR run accessions or a study accession (SRP) before quantification with tools
  like Seq2Fun 2.0.
when_to_use_negative:
- Count tables (e.g., salamander_ortholog_counts.tsv) or processed FASTQ files are
  already available locally
- The study's data are not deposited in NCBI SRA or accessions are invalid/unreleased
- The analysis starts from an existing gene expression matrix rather than raw reads
  (the ExpressAnalystR gene-matrix workflow)
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NCBI SRA Toolkit (prefetch/fasterq-dump)
  role: Downloads the raw paired-end FASTQ files for each SRR accession from NCBI
    SRA prior to Seq2Fun quantification
- name: ExpressAnalyst
  role: Downstream web platform that consumes the retrieved FASTQ files for Seq2Fun
    read quantification and functional analysis
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:0d5f7a83c77a048dc1463cbec683902121c729415ac5881afa86bb5284bcd4b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpressAnalyst
  - NCBI SRA SRR7499348–SRR7499365
  - NCBI SRA Toolkit (prefetch/fasterq-dump)
  - Seq2Fun 1.0
  - Seq2Fun 2.0
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sra-fastq-data-retrieval

## Summary

Fetch raw paired-end FASTQ files from NCBI SRA by accession (e.g., SRR7499348–SRR7499365 salamander samples or SRP299836 zebrafish samples) as the entry point for downstream Seq2Fun/ExpressAnalyst RNA-seq quantification in non-model species.

## When to use

Use when raw RNA-seq data for a study must be fetched from NCBI SRA via SRR/ERR run accessions or a study accession (SRP) before quantification with tools like Seq2Fun 2.0.

## When NOT to use

- Count tables (e.g., salamander_ortholog_counts.tsv) or processed FASTQ files are already available locally
- The study's data are not deposited in NCBI SRA or accessions are invalid/unreleased
- The analysis starts from an existing gene expression matrix rather than raw reads (the ExpressAnalystR gene-matrix workflow)

## Inputs

- List of NCBI SRA run accessions (e.g., SRR7499348–SRR7499365)
- NCBI SRA study accession (e.g., SRP299836)
- QA/QC-derived exclusion list of outlier accessions (e.g., SRR7499350)

## Outputs

- Per-sample paired-end FASTQ files (R1/R2) named by accession
- Retrieval manifest mapping each accession to downloaded FASTQ files and exclusion status

## How to apply

Collect the run accessions cited by the paper (e.g., salamander samples SRR7499348–SRR7499365, or the twelve zebrafish samples SRR13332314–SRR13332325 from study SRP299836) and retrieve them from NCBI SRA, e.g., using the SRA Toolkit's prefetch followed by fasterq-dump to produce paired-end FASTQ files. Exclude flagged samples during retrieval or afterwards — the salamander case skips SRR7499350, which was identified as an outlier by QA/QC in the original publication. Keep each accession's R1/R2 pair together so per-sample FASTQ pairs can be passed directly to Seq2Fun 2.0 with the calibrated parameters (max mismatches = 2, minimum matching length = 19, minimum BLOSUM62 score = 80).

## Related tools

- **NCBI SRA Toolkit (prefetch/fasterq-dump)** (Downloads the raw paired-end FASTQ files for each SRR accession from NCBI SRA prior to Seq2Fun quantification)
- **ExpressAnalyst** (Downstream web platform that consumes the retrieved FASTQ files for Seq2Fun read quantification and functional analysis)

## Examples

```
prefetch SRR7499348 && fasterq-dump SRR7499348 --split-files --gzip
```

## Evaluation signals

- All cited accessions resolve and download successfully (18 salamander accessions SRR7499348–SRR7499365; 12 zebrafish accessions SRR13332314–SRR13332325)
- Each accession yields exactly one R1/R2 FASTQ pair; file sizes are non-trivial and sequence headers are consistent
- Excluded sample SRR7499350 is either not retrieved or marked as outlier, so the downstream count matrix has 17 salamander samples
- Retrieved samples match the source study metadata (e.g., study SRP299836) via the SRA trace portal

## Limitations

- Sample-level details may be missing from the original publication (e.g., 'Details on sex were not included in the original manuscript'), complicating metadata completeness checks
- Retrieval is only the entry point: reads from a sample must be mapped to the correctly selected taxonomic sub-group database, otherwise key functional groups may be missed

## Evidence

- [discussion] Salamander RNA-seq data were obtained from NCBI's SRA at sample accessions SRR7499348 - SRR7499365: "Salamander RNA-seq data were obtained from NCBI’s SRA at sample accessions SRR7499348 - SRR7499365"
- [discussion] excluding sample SRR7499350 identified as an outlier by QA/QC: "excluding sample SRR7499350 which was identified as an outlier by QA/QC performed by the original publication"
- [discussion] zebrafish sample accessions in study SRP299836: "sample accessions SRR13332314 – SRR13332325 (https://trace.ncbi.nlm.nih.gov/Traces/?view=study&acc=SRP299836)"
