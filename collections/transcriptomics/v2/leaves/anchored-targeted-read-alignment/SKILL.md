---
name: anchored-targeted-read-alignment
description: Use when searching for fusions involving one known driver gene (e.g.,
  BCR-ABL1, NSD2-IGH) in paired-end bulk or single-cell RNA-seq (e.g., Smart-seq2)
  where de novo tools miss multi-mapping/homologous or shallow-depth reads.
when_to_use_negative:
- Input is single-end RNA-seq, whole-genome, or whole-exome sequencing data — Anchored-fusion
  requires paired-end RNA-seq
- The fusion of interest does not include the user-provided target gene — only fusions
  containing the anchored gene are reported
- A comprehensive de novo fusion discovery is needed rather than a targeted search
  for a known driver gene
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Anchored-fusion
  role: Main tool implementing the anchored read-alignment and fusion search workflow
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: BWA-MEM
  role: Aligns paired-end reads to the anchored gene transcript reference and unmapped
    mates to the whole genome
- name: BLAT
  role: Performs local alignment on candidate chimeric reads to predict the precise
    fusion location
- name: STAR-Fusion
  role: Bulk-RNA-seq competing method used for benchmarking
  repo: https://github.com/STAR-Fusion
- name: FusionInspector
  role: Bulk-RNA-seq competing method used for benchmarking
  repo: https://github.com/FusionInspector/FusionInspector
- name: FusionCatcher
  role: Bulk-RNA-seq competing method used for benchmarking
  repo: https://github.com/ndaniel/fusioncatcher
- name: Arriba
  role: Bulk-RNA-seq competing method used for benchmarking
  repo: https://github.com/suhrig/arriba
- name: scFusion
  role: Single-cell RNA-seq competing method used for benchmarking
  repo: https://github.com/XiDsLab/scFusion
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:e3e2ee8dd993901a9faaadf6e73dadf811d7c36b72cb284efd2c71d76e79593a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  repo_url: https://github.com/ShenLab-Genomics/Anchored-Fusion
  tools:
  - Anchored-fusion
  - Arriba
  - BLAT
  - BWA-MEM
  - FusionCatcher
  - FusionInspector
  - STAR-Fusion
  - scFusion
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# anchored-targeted-read-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A targeted fusion-detection strategy from Anchored-fusion that anchors a user-specified gene of interest, maps all paired-end reads to that gene's transcript reference with BWA-MEM, realigns unmapped mates to the whole genome, and uses BLAT local alignment on chimeric reads to pinpoint the fusion breakpoint.

## When to use

Use when searching for fusions involving one known driver gene (e.g., BCR-ABL1, NSD2-IGH) in paired-end bulk or single-cell RNA-seq (e.g., Smart-seq2) where de novo tools miss multi-mapping/homologous or shallow-depth reads.

## When NOT to use

- Input is single-end RNA-seq, whole-genome, or whole-exome sequencing data — Anchored-fusion requires paired-end RNA-seq
- The fusion of interest does not include the user-provided target gene — only fusions containing the anchored gene are reported
- A comprehensive de novo fusion discovery is needed rather than a targeted search for a known driver gene

## Inputs

- Paired-end RNA-seq FASTQ files (bulk or single-cell, e.g., Smart-seq2 from SRA:SRP291312 or SRP158590)
- Target gene transcript FASTA (from NCBI Gene)
- Genome FASTA (hg38 or hg19, from GENCODE/Ensembl)
- Gene annotation GTF compatible with the genome FASTA version
- Optional: positive training samples and homologous gene file from the program's 'data' folder for HVLD retraining

## Outputs

- Per-cell or per-sample fusion calls including the anchored target gene (e.g., BCR-ABL1, NSD2-IGH positive/negative calls)
- Precise fusion breakpoint coordinates from BLAT local alignment (e.g., chr4:1,902,353 and chr4:1,905,943)
- Spanning-read and split-read support counts per candidate fusion
- HVLD-filtered candidate list (candidates with probability < 0.1 removed)

## How to apply

First obtain the target gene transcript FASTA (e.g., BCR or NSD2 from NCBI Gene) plus a genome FASTA (hg38 or hg19) and a compatible GTF from GENCODE/Ensembl. Run Anchored_Fusion.py so BWA-MEM maps all paired-end reads to the anchored gene's CDS reference; collect reads aligned to gene X, then select pairs where only one read maps and align the unmapped mate to the entire genome. Candidate chimeric reads are locally aligned with BLAT to predict the precise fusion location, using spanning reads and split reads across the fusion site as supporting evidence; genes with homologous regions to the anchored gene are removed from the potential partner set, and candidates with uncertain positions or imbalanced spanning/split read counts are filtered. Finally, the HVLD deep-learning module (retrained per dataset with PCAWG positives, or the default model for small inputs) discards candidates with probability below 0.1. Validate by benchmarking recovery of known positives, e.g., 35 BCR-ABL1-positive K562 cells and 99 NSD2-IGH-positive cells (92.5% of positive cells) in the MM cohort.

## Related tools

- **Anchored-fusion** (Main tool implementing the anchored read-alignment and fusion search workflow) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **BWA-MEM** (Aligns paired-end reads to the anchored gene transcript reference and unmapped mates to the whole genome)
- **BLAT** (Performs local alignment on candidate chimeric reads to predict the precise fusion location)
- **STAR-Fusion** (Bulk-RNA-seq competing method used for benchmarking) — https://github.com/STAR-Fusion
- **FusionInspector** (Bulk-RNA-seq competing method used for benchmarking) — https://github.com/FusionInspector/FusionInspector
- **FusionCatcher** (Bulk-RNA-seq competing method used for benchmarking) — https://github.com/ndaniel/fusioncatcher
- **Arriba** (Bulk-RNA-seq competing method used for benchmarking) — https://github.com/suhrig/arriba
- **scFusion** (Single-cell RNA-seq competing method used for benchmarking) — https://github.com/XiDsLab/scFusion

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=BCR.fasta --fastq1=cell_1.fastq.gz --fastq2=cell_1.fastq.gz --out_folder=output_dir --file_ref_seq=hg38.fa --file_ref_ann=gencode.gtf
```

## Evaluation signals

- Known-positive recovery: 35 BCR-ABL1-positive cells detected in the 350-cell K562 Smart-seq2 dataset (SRP291312)
- NSD2-IGH validation: 99 positive cells in the 597-cell MM cohort, 92.5% of positive cells, per-patient rates of 32.6%/59.5%/77.6%
- Breakpoint correctness: NSD2-IGH breakpoints on chr4 at positions 1,902,353 and 1,905,943 resolved by BLAT
- HVLD filter behavior: candidates with HVLD probability < 0.1 removed while true fusions are retained
- All reported fusions contain the anchored target gene; other fusion genes are absent from the output

## Limitations

- Fusion genes that do not include the target gene will not be detected
- Anchored-fusion can only detect fusion genes from paired-end RNA-seq; single-end RNA-seq, WGS, and WES data are unsupported
- Users must supply the reference sequence of their target gene
- For small input datasets, retrained HVLD parameters may be unreliable; use the default model parameters provided on GitHub
- Pre-trained filter models not trained on the user's input data may give deviating results

## Evidence

- [discussion] Anchored-fusion first maps all paired-end reads to the reference sequence of the anchored gene X. All reads aligned to gene X are collected: "Anchored-fusion first maps all paired-end reads to the reference sequence of the anchored gene X. All reads aligned to gene X are collected"
- [discussion] BLAT local alignment for breakpoint prediction: "Anchored-fusion utilizes the BLAT tool36 to perform local alignment on candidate chimeric reads to accurately predict the precise location of gene fusion."
- [discussion] HVLD probability threshold: "genes with a low probability (less than 0.1) will be filtered out."
- [readme] README run command: "python Anchored_Fusion.py --file_anchored_cds=test/target_gene.fasta --fastq1=test/test_sample_1.fastq.gz  --fastq2=test/test_sample_1.fastq.gz --out_folder=output_dir"
- [discussion] Spanning and split read support: "The process of searching for fusion genes considers paired-end reads spanning across fusion sites (referred to as spanning reads) and single reads containing fusion sites (referred to as split reads)"
