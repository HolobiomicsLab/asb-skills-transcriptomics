---
name: multi-tool-fusion-caller-benchmarking
description: Use when a new or targeted fusion-detection method must be evaluated
  head-to-head against established callers (STAR-Fusion, FusionCatcher, Arriba, FusionInspector,
  scFusion) on shared FASTQ inputs with known positive fusion events.
when_to_use_negative:
- The RNA-seq data are single-end, whole-genome, or whole-exome — Anchored-fusion
  requires paired-end RNA-seq and the comparison design presumes transcriptomic reads
- Fusions of interest do not include the user-anchored target gene, since targeted
  anchoring will not surface them
- Only one caller is needed for discovery with no sensitivity comparison — a single
  de novo caller suffices and multi-tool benchmarking overhead is unnecessary
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: Anchored-fusion
  role: Targeted fusion caller under evaluation; anchors the gene of interest and
    recovers non-unique short-read matches
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: scFusion
  role: Single-cell RNA-seq competing fusion-detection method
  repo: https://github.com/XiDsLab/scFusion
- name: STAR-Fusion v1.12.0
  role: Bulk-RNA-seq comparator fusion caller
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
- name: BWA-MEM
  role: Aligner mapping paired reads to the anchored-gene reference with unmapped
    mates aligned to the whole genome
- name: BLAT
  role: Local alignment of candidate chimeric reads to resolve precise fusion breakpoints
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:eb6bd0fd376fa544a59956b224bfebe72022aab07a1d6a35f5bbd6a01085a6ca
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
n_claims: 16
---

# multi-tool-fusion-caller-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a targeted fusion-caller (e.g. Anchored-fusion) against competing single-cell (scFusion) and bulk-RNA-seq fusion detectors (STAR-Fusion v1.12.0, FusionInspector v2.8.0, FusionCatcher v1.30, Arriba v2.4.0) on the same RNA-seq inputs, comparing sensitivity, per-patient recovery rates, and breakpoint precision.

## When to use

Use when a new or targeted fusion-detection method must be evaluated head-to-head against established callers (STAR-Fusion, FusionCatcher, Arriba, FusionInspector, scFusion) on shared FASTQ inputs with known positive fusion events.

## When NOT to use

- The RNA-seq data are single-end, whole-genome, or whole-exome — Anchored-fusion requires paired-end RNA-seq and the comparison design presumes transcriptomic reads
- Fusions of interest do not include the user-anchored target gene, since targeted anchoring will not surface them
- Only one caller is needed for discovery with no sensitivity comparison — a single de novo caller suffices and multi-tool benchmarking overhead is unnecessary

## Inputs

- Paired-end RNA-seq FASTQ files (e.g. SRA:SRP158590 multiple myeloma scRNA-seq; SRA:SRP291312 K562 scRNA-seq; SRA:SRP186687 cancer cell line bRNA-seq)
- Anchored target-gene transcript FASTA (e.g. NSD2 from NCBI Gene)
- hg38/hg19 genome FASTA plus compatible GTF annotation from GENCODE/Ensembl
- Ground-truth fusion list (PCAWG 3540 fusion genes; Synapse:syn10003873)
- Positive training samples and homologous-gene file from the Anchored-fusion 'data' folder for HVLD model training

## Outputs

- Per-cell and per-patient fusion call tables across the cohort (e.g. 99 NSD2-IGH-positive cells across 597 cells/15 patients)
- Sensitivity/recall metrics per caller and per patient (e.g. 32.6%/59.5%/77.6% per-patient recovery)
- Precise fusion breakpoint coordinates from BLAT local alignment (e.g. chr4:1,902,353 and chr4:1,905,943)
- Comparative benchmark tables/plots ranking Anchored-fusion against scFusion, STAR-Fusion, FusionInspector, FusionCatcher, and Arriba

## How to apply

Run the method under evaluation (e.g. Anchored_Fusion.py on the anchored-gene FASTA plus paired FASTQs, with BWA-MEM mapping to the anchored reference and BLAT resolving breakpoints) and, in parallel, run the competing callers on the same data: scFusion on the scRNA-seq reads and STAR-Fusion (v1.12.0), FusionInspector (v2.8.0), FusionCatcher (v1.30), and Arriba (v2.4.0) as bulk-RNA-seq comparators. Aggregate calls across samples (e.g. 597 cells and 15 multiple-myeloma patients from SRA:SRP158590), stratify per patient/per cell, and compare against known ground truth — here the three t(4;14) patients (RRMM1, RRMM2, SMM0) expected to carry NSD2-IGH. Report sensitivity as fraction of positive cells recovered (Anchored-fusion: 92.5%; 60.3% of all cells) and per-patient rates (32.6%, 59.5%, 77.6%), and verify breakpoint concordance (chr4:1,902,353 and chr4:1,905,943). For simulated benchmarks, spike PCAWG-derived fusion transcripts (3540 fusions) into a background sample such as GM12878 and measure recovery against each caller. Treat the MM scRNA-seq task as validated only when the per-patient call counts and breakpoint positions match the published expectations.

## Related tools

- **Anchored-fusion** (Targeted fusion caller under evaluation; anchors the gene of interest and recovers non-unique short-read matches) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **scFusion** (Single-cell RNA-seq competing fusion-detection method) — https://github.com/XiDsLab/scFusion
- **STAR-Fusion v1.12.0** (Bulk-RNA-seq comparator fusion caller) — https://github.com/STAR-Fusion
- **FusionInspector v2.8.0** (Bulk-RNA-seq comparator fusion caller) — https://github.com/FusionInspector/FusionInspector
- **FusionCatcher v1.30** (Bulk-RNA-seq comparator fusion caller) — https://github.com/ndaniel/fusioncatcher
- **Arriba v2.4.0** (Bulk-RNA-seq comparator fusion caller) — https://github.com/suhrig/arriba
- **BWA-MEM** (Aligner mapping paired reads to the anchored-gene reference with unmapped mates aligned to the whole genome)
- **BLAT** (Local alignment of candidate chimeric reads to resolve precise fusion breakpoints)

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=test/target_gene.fasta --fastq1=test/test_sample_1.fastq.gz --fastq2=test/test_sample_1.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- NSD2-IGH calls in the MM cohort should be confined to the three t(4;14) patients (RRMM1, RRMM2, SMM0) — 99 positive cells out of 597
- Per-patient recovery rates should fall in published ranges: 32.6%, 59.5%, and 77.6%; overall 92.5% of NSD2-IGH-positive cells and 60.3% of all cells
- Resolved breakpoints must match chr4:1,902,353 and chr4:1,905,943
- In simulated benchmarks with PCAWG spike-ins into GM12878 background, Anchored-fusion should recover a higher fraction of true fusions than the competing callers
- HVLD-filtered candidates should retain true fusions while discarding candidates with probability below 0.1

## Limitations

- Fusion genes that do not include the user-provided target gene will not be detected by the anchored approach
- Anchored-fusion can only detect fusions from paired-end RNA-seq; it cannot be used on single-end RNA-seq, whole-genome, or whole-exome data
- For very small input datasets, retraining the HVLD filter is unreliable — use the default model parameters provided on GitHub
- Bulk-RNA-seq comparators (STAR-Fusion, FusionCatcher, Arriba, FusionInspector) are not designed for shallow scRNA-seq depth, complicating fair comparison

## Evidence
