---
name: scrna-seq-fusion-gene-detection
description: Use when running targeted fusion-gene detection on paired-end scRNA-seq
  FASTQs from cancer datasets (e.g., K562 Smart-seq2 or multiple myeloma t(4;14) cohorts)
  and comparing sensitivity against STAR-Fusion, FusionCatcher, Arriba, FusionInspector,
  or scFusion.
when_to_use_negative:
- The sequencing data are single-end RNA-seq, whole-genome, or whole-exome sequencing
  — Anchored-fusion only supports paired-end RNA-seq
- The fusions of interest do not include the user-supplied target gene — only fusions
  containing the anchored gene are reported
- The user cannot provide a reference transcript sequence for the target gene — this
  input is required
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: Anchored-fusion
  role: Primary targeted fusion detector that anchors the gene of interest and recovers
    non-unique short-read matches in scRNA-seq
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: BWA-MEM
  role: Alignment tool mapping paired-end reads to the anchored gene reference and
    unmapped mates to the whole genome
- name: BLAT
  role: Local alignment of candidate chimeric reads to predict the precise fusion
    location
- name: scFusion
  role: scRNA-seq-specific competing fusion detection method
  repo: https://github.com/XiDsLab/scFusion
- name: STAR-Fusion v1.12.0
  role: Bulk-RNA-seq comparator fusion caller run on the same per-cell FASTQs
  repo: https://github.com/STAR-Fusion
- name: FusionInspector v2.8.0
  role: Bulk-RNA-seq comparator fusion caller
  repo: https://github.com/FusionInspector/FusionInspector
- name: FusionCatcher v1.30
  role: Bulk-RNA-seq comparator fusion caller
  repo: https://github.com/ndaniel/fusioncatcher
- name: Arriba v2.4.0
  role: Bulk-RNA-seq comparator fusion caller
  repo: https://github.com/suhrig/arriba
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:b62c9dfe456cd78ef78ef971e167765ca57aec6fdead1dde13a8a06fd1d7715a
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
  - Arriba v2.4.0
  - BLAT
  - BWA-MEM
  - FusionCatcher
  - FusionCatcher v1.30
  - FusionInspector
  - FusionInspector v2.8.0
  - STAR-Fusion
  - STAR-Fusion v1.12.0
  - scFusion
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# scrna-seq-fusion-gene-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect fusion genes in paired-end single-cell RNA-seq data using Anchored-fusion, which anchors a user-specified gene of interest (e.g., BCR, NSD2) and recovers non-unique short-read matches that de novo tools discard, enabling detection even at shallow scRNA-seq depths (~1.3 M reads/cell).

## When to use

Use when running targeted fusion-gene detection on paired-end scRNA-seq FASTQs from cancer datasets (e.g., K562 Smart-seq2 or multiple myeloma t(4;14) cohorts) and comparing sensitivity against STAR-Fusion, FusionCatcher, Arriba, FusionInspector, or scFusion.

## When NOT to use

- The sequencing data are single-end RNA-seq, whole-genome, or whole-exome sequencing — Anchored-fusion only supports paired-end RNA-seq
- The fusions of interest do not include the user-supplied target gene — only fusions containing the anchored gene are reported
- The user cannot provide a reference transcript sequence for the target gene — this input is required

## Inputs

- Paired-end scRNA-seq FASTQ files (per-cell, e.g., Smart-seq2 from SRA:SRP291312 or SRP158590)
- Anchored target-gene transcript CDS FASTA (e.g., BCR or NSD2 from NCBI Gene)
- Genome FASTA (hg38 or hg19) and compatible GTF annotation from GENCODE/Ensembl
- HVLD positive training samples (PCAWG fusions) and homologous gene file from the program's 'data' folder
- Pre-trained HVLD model file ('data' folder) for the --not_train_filter_model option

## Outputs

- Per-cell fusion calls (fusion-positive/negative) for the anchored gene (e.g., 35/350 BCR-ABL1-positive K562 cells; 99 NSD2-IGH-positive MM cells)
- Precise fusion breakpoints from BLAT local alignment (e.g., chr4:1,902,353 and chr4:1,905,943 for NSD2-IGH)
- Per-patient/per-sample detection-rate tables comparing Anchored-fusion against STAR-Fusion, FusionInspector, FusionCatcher, Arriba, and scFusion
- Filtered fusion candidate list with spanning and split read support and HVLD probabilities

## How to apply

Download per-cell paired-end FASTQs (e.g., 350-cell K562 Smart-seq2 from SRA:SRP291312 or Jang et al. MM scRNA-seq from SRP158590), the anchored-gene transcript FASTA from NCBI Gene, and a genome FASTA plus compatible GTF (hg38/hg19) from GENCODE/Ensembl. Run Anchored_Fusion.py per cell with --file_anchored_cds set to the target transcript FASTA: BWA-MEM maps all read pairs to the anchored gene, unmapped mates of semi-aligned pairs are aligned to the whole genome, and BLAT local alignment on candidate chimeric reads resolves precise breakpoints using spanning and split reads as support. Apply the HVLD deep-learning filter — by default retrain per dataset with PCAWG positive samples and the provided homologous-gene file, removing negative training samples containing the anchored gene and reads aligned to homologous genes, and discard candidates with HVLD probability below 0.1; for small datasets use the pre-trained default model instead. Run scFusion as the scRNA-seq comparator and STAR-Fusion v1.12.0, FusionInspector v2.8.0, FusionCatcher v1.30, and Arriba v2.4.0 as bulk-RNA-seq comparators on the same FASTQs, then tabulate fusion-positive cells per tool and per patient/sample to quantify sensitivity.

## Related tools

- **Anchored-fusion** (Primary targeted fusion detector that anchors the gene of interest and recovers non-unique short-read matches in scRNA-seq) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **BWA-MEM** (Alignment tool mapping paired-end reads to the anchored gene reference and unmapped mates to the whole genome)
- **BLAT** (Local alignment of candidate chimeric reads to predict the precise fusion location)
- **scFusion** (scRNA-seq-specific competing fusion detection method) — https://github.com/XiDsLab/scFusion
- **STAR-Fusion v1.12.0** (Bulk-RNA-seq comparator fusion caller run on the same per-cell FASTQs) — https://github.com/STAR-Fusion
- **FusionInspector v2.8.0** (Bulk-RNA-seq comparator fusion caller) — https://github.com/FusionInspector/FusionInspector
- **FusionCatcher v1.30** (Bulk-RNA-seq comparator fusion caller) — https://github.com/ndaniel/fusioncatcher
- **Arriba v2.4.0** (Bulk-RNA-seq comparator fusion caller) — https://github.com/suhrig/arriba

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=BCR_transcript.fasta --fastq1=cell_1_R1.fastq.gz --fastq2=cell_1_R2.fastq.gz --out_folder=output_dir --file_ref_seq=GRCh38.fa --file_ref_ann=gencode.v38.annotation.gtf
```

## Evaluation signals

- K562 benchmark: Anchored-fusion reports 35 BCR-ABL1-positive cells out of 350 Smart-seq2 cells, with 83.3% recovery of the 42 evidence-bearing cells and 10/15 detection among very-low-evidence cells
- MM benchmark: 99 NSD2-IGH-positive cells across the three t(4;14) patients (RRMM1, RRMM2, SMM0), covering 92.5% of positive cells, with per-patient rates of 32.6%, 59.5%, and 77.6%
- NSD2-IGH breakpoints resolve to chr4:1,902,353 and chr4:1,905,943 via BLAT local alignment
- All reported fusions contain the anchored target gene; candidates with HVLD probability < 0.1, uncertain fusion positions, or imbalanced spanning/split read counts are absent from output
- Reference genome FASTA and GTF annotation versions are compatible (hg38 or hg19)

## Limitations

- Fusion genes that do not include the anchored target gene will not be detected
- Anchored-fusion only works on paired-end RNA-seq data; it cannot be used for single-end RNA-seq, whole-genome, or whole-exome sequencing data
- Users must supply the reference transcript sequence of the target gene themselves
- If the input dataset is too small, the retrained HVLD model is unreliable — use the default model parameters provided on GitHub
- Using the pre-trained filter (--not_train_filter_model) on unseen data may produce results with deviations since it was not trained on the user's input

## Evidence
