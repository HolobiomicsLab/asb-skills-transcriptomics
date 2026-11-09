---
name: variant-coverage-fraction-calculation
description: Use when you have per-gene MRSD scores from MRSDexplorer (.results.mrsd.txt)
  plus a list of VUS-harboring genes and need the fraction of variants assayable at
  a given read-depth threshold.
when_to_use_negative:
- When the goal is differential expression or allele-specific expression suitability
  — the MRSD model cannot directly be extended to predict suitability for detecting
  allele-specific expression biases and differential gene expression
- When the input RNA-seq is long-read, single-cell, or spatially resolved, or non-poly-A
  enrichment — MRSD was validated only for bulk short-read poly-A enrichment RNA-seq
  processed with a specific bioinformatics pipeline
- When the target genes sit in patients with disorders involving widespread transcriptomic
  changes (e.g. interferonopathies) — MRSD predictions may not accurately reflect
  coverage there
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_0622
tools:
- name: MRSDexplorer
  role: Generates the per-gene .results.mrsd.txt MRSD score tables that the variant-fraction
    calculation joins to the VUS gene list
  repo: https://github.com/mcgm-mrsd/mrsd-explorer
- name: Python
  role: Used with pandas to join variant genes to MRSD scores across the four biosamples
    and compute the low-MRSD fraction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ajhg.2021.12.014
    title: 'MRSD: A quantitative approach for assessing suitability of RNA-seq in
      the investigation of mis-splicing in Mendelian disease'
  source_content_hash: sha256:666df1bc4a5c72a7f4084af58762e477dcce555f7a8dbd6dc0fc1748eda040b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgm-mrsd/mrsd-explorer
  tools:
  - MRSDexplorer
  - Python
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# variant-coverage-fraction-calculation

## Summary

Computes the fraction of SpliceAI-prioritized ClinVar VUSs that fall in genes with low MRSD (≤100 million reads) in at least one biosample (whole blood, LCL, skeletal muscle, cultured fibroblast), quantifying how many splice-impacting VUSs could be assayed by RNA-seq.

## When to use

Use when you have per-gene MRSD scores from MRSDexplorer (.results.mrsd.txt) plus a list of VUS-harboring genes and need the fraction of variants assayable at a given read-depth threshold.

## When NOT to use

- When the goal is differential expression or allele-specific expression suitability — the MRSD model cannot directly be extended to predict suitability for detecting allele-specific expression biases and differential gene expression
- When the input RNA-seq is long-read, single-cell, or spatially resolved, or non-poly-A enrichment — MRSD was validated only for bulk short-read poly-A enrichment RNA-seq processed with a specific bioinformatics pipeline
- When the target genes sit in patients with disorders involving widespread transcriptomic changes (e.g. interferonopathies) — MRSD predictions may not accurately reflect coverage there

## Inputs

- MRSD score tables (.results.mrsd.txt) for whole blood, LCL, skeletal muscle, and cultured fibroblast GTEx-derived datasets
- One-per-line gene list of 7,507 SpliceAI-filtered ClinVar VUS-harboring genes (GENCODE v19 gene symbol and/or Ensembl gene ID, two-column format)

## Outputs

- Per-variant classification table (each variant annotated with low-MRSD status per biosample)
- Final fraction of SpliceAI-prioritized ClinVar VUSs in low-MRSD genes (e.g. written to mrsd_variant_fraction.csv)

## How to apply

Load the .results.mrsd.txt files produced by MRSDexplorer.py for the four GTEx-derived biosamples, computed with --number_reads 8, --sj_prop 0.75, --mrsd_param 0.95, --read_type unique. Load the one-per-line gene list of the 7,507 SpliceAI-filtered ClinVar VUS-harboring genes (GENCODE v19 symbols and/or Ensembl IDs) and, in Python (pandas), join each variant's gene to its MRSD score in each biosample. Treat '-' entries as genes whose median junction coverage was 0 reads (not assayable). Flag each gene as low-MRSD if its MRSD is ≤100 million reads in at least one biosample, then compute the fraction of variants in low-MRSD genes, stratifying across model stringency settings to yield the reported range (25.8%–67.8%).

## Related tools

- **MRSDexplorer** (Generates the per-gene .results.mrsd.txt MRSD score tables that the variant-fraction calculation joins to the VUS gene list) — https://github.com/mcgm-mrsd/mrsd-explorer
- **Python** (Used with pandas to join variant genes to MRSD scores across the four biosamples and compute the low-MRSD fraction)

## Examples

```
python MRSDexplorer.py --tissues blood,lcl,muscle,fibroblast --number_reads 8 --sj_prop 0.75 --mrsd_param 0.95 --read_type unique vus_genes.txt
```

## Evaluation signals

- The computed fraction must reproduce the reported range of 25.8%–67.8%
- The upper bound (67.8%) must correspond to the least stringent model setting in at least one biosample
- Genes with '-' in .results.mrsd.txt must be treated as not assayable (median junction coverage of 0 reads), matching the README definition
- Variant-to-gene join must resolve both GENCODE v19 symbols and Ensembl gene IDs; unmatched IDs should be surfaced rather than silently dropped
- Low-MRSD flag must be TRUE when MRSD ≤100 million reads in ANY of the four biosamples, not requiring all biosamples to pass

## Limitations

- The 25.8%–67.8% range depends on model stringency; the upper bound applies only at the least stringent setting
- MRSD does not explicitly account for eQTLs or sQTLs, so gene-level coverage predictions may be off for affected individuals
- Pre-computed GTEx MRSD scores are most accurate for Illumina 75 bp paired-end poly-A enrichment workflows; other methodologies require regenerating scores
- MRSD does not account for genomic background influencing gene expression (e.g. spliceosome disruption, chromatin structure disorders)

## Evidence

- [abstract] up to 67.8% of the variants of uncertain significance in ClinVar that are predicted to impact splicing could be assayed by RNA sequencing in at least one of the biosamples: "up to 67.8% of the variants of uncertain significance in ClinVar that are predicted to impact splicing could be assayed by RNA sequencing in at least one of the biosamples"
- [readme] Where '-' is given as a required read depth, median coverage of splice junctions for the given gene was 0 reads.: "Where '-' is given as a required read depth, median coverage of splice junctions for the given gene was 0 reads."
- [readme] This file can have up to two columns, containing the GENCODE v19 gene symbol and/or the Ensembl gene ID.: "This file can have up to two columns, containing the GENCODE v19 gene symbol and/or the Ensembl gene ID."
- [intro] MRSD scores (see web resources for access) can be utilized to select the most appropriate biosample to detect specific splicing aberrations and to guide required depth of sequencing.: "MRSD scores (see web resources for access) can be utilized to select the most appropriate biosample to detect specific splicing aberrations and to guide required depth of sequencing."
