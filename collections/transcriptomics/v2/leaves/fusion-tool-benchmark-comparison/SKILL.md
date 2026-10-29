---
name: fusion-tool-benchmark-comparison
description: Use when benchmarking fusion-gene detection sensitivity across tools
  on paired-end bulk or single-cell RNA-seq (e.g., Smart-seq2 FASTQs), especially
  when shallow depth or sequence homology suppresses de novo caller recall.
when_to_use_negative:
- Input is single-end RNA-seq, whole-genome, or whole-exome sequencing data — Anchored-fusion
  requires paired-end RNA-seq
- The fusions of interest do not include the anchored target gene — Anchored-fusion
  only reports fusions containing the user-specified gene
- The dataset is too small to retrain the HVLD filter — default model parameters from
  GitHub must be used instead
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: Anchored-fusion
  role: 'Primary sensitivity benchmark: anchors the target gene (BCR), maps paired-end
    reads with BWA-MEM, refines breakpoints with BLAT, and filters false positives
    via the HVLD deep-learning module'
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: STAR-Fusion v1.12.0
  role: Competing de novo fusion caller run on the same per-cell FASTQs for benchmark
    comparison
  repo: https://github.com/STAR-Fusion
- name: FusionInspector v2.8.0
  role: Competing fusion verification/inspection tool used as bRNA-seq benchmark method
  repo: https://github.com/FusionInspector/FusionInspector
- name: FusionCatcher v1.30
  role: Competing fusion detection tool used as bRNA-seq benchmark method
  repo: https://github.com/ndaniel/fusioncatcher
- name: Arriba v2.4.0
  role: Competing STAR-based fusion caller used as bRNA-seq benchmark method
  repo: https://github.com/suhrig/arriba
- name: scFusion
  role: Competing single-cell-specific fusion detection method for the scRNA-seq benchmark
  repo: https://github.com/XiDsLab/scFusion
- name: BWA-MEM
  role: Alignment tool used internally by Anchored-fusion to map reads to the anchored
    gene and genome
- name: BLAT
  role: Local alignment of candidate chimeric reads to precisely predict fusion breakpoints
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:24811519402e43e2bd40398442f8606cbbaf7eef47de93b995c2111d3e6cbb49
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
  edam_operation: http://edamontology.org/operation_2928
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

# fusion-tool-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking workflow that runs Anchored-fusion alongside competing fusion callers (STAR-Fusion v1.12.0, FusionInspector v2.8.0, FusionCatcher v1.30, Arriba v2.4.0, scFusion) on the same paired-end RNA-seq FASTQs and tabulates per-cell fusion calls to quantify relative sensitivity. Anchored-fusion anchors a user-specified target gene (e.g., BCR) and recovers non-unique short-read matches that de novo tools discard, enabling comparison on shallow scRNA-seq data.

## When to use

Use when benchmarking fusion-gene detection sensitivity across tools on paired-end bulk or single-cell RNA-seq (e.g., Smart-seq2 FASTQs), especially when shallow depth or sequence homology suppresses de novo caller recall.

## When NOT to use

- Input is single-end RNA-seq, whole-genome, or whole-exome sequencing data — Anchored-fusion requires paired-end RNA-seq
- The fusions of interest do not include the anchored target gene — Anchored-fusion only reports fusions containing the user-specified gene
- The dataset is too small to retrain the HVLD filter — default model parameters from GitHub must be used instead

## Inputs

- Paired-end RNA-seq FASTQ files (e.g., per-cell FASTQ pairs from Smart-seq2 scRNA-seq)
- Target gene transcript reference FASTA (e.g., BCR transcript from NCBI Gene)
- Genome FASTA (hg38 or hg19 from GENCODE/Ensembl)
- Gene annotation GTF compatible with the genome FASTA version
- Anchored-fusion 'data' folder with positive training samples and homologous gene file for HVLD retraining

## Outputs

- Per-cell fusion-positive/negative call table for each tool (e.g., BCR-ABL1 status for all 350 cells)
- Sensitivity/recovery metrics per tool against evidence-stratified cell categories (e.g., 83.3% recovery of 42 evidence-bearing cells, 10/15 very-low-evidence cells)
- Anchored-fusion candidate fusions with spanning/split read support and precise fusion breakpoint predicted by BLAT

## How to apply

Download paired-end FASTQs (e.g., 350-cell K562 Smart-seq2 from SRA SRP291312, ~1.3 M reads/cell) and run Anchored_Fusion.py per cell with the target-gene transcript FASTA (BCR from NCBI Gene) via --file_anchored_cds, the hg38/hg19 genome FASTA via --file_ref_seq, and a version-compatible GTF via --file_ref_ann, with the HVLD deep-learning filter retrained using positive samples and the homologous gene file in the 'data' folder. Run the competing tools (STAR-Fusion v1.12.0, FusionInspector v2.8.0, FusionCatcher v1.30, Arriba v2.4.0, and scFusion for scRNA-seq) on the identical per-cell FASTQs. Tabulate fusion-positive/negative calls per cell for the target fusion (BCR-ABL1) and stratify cells by supporting-evidence level. Expected benchmark outcomes in the K562 example: Anchored-fusion detected 35/350 BCR-ABL1-positive cells, recovered 83.3% of 42 evidence-bearing cells, and detected 10/15 very-low-evidence cells, exceeding all competitors.

## Related tools

- **Anchored-fusion** (Primary sensitivity benchmark: anchors the target gene (BCR), maps paired-end reads with BWA-MEM, refines breakpoints with BLAT, and filters false positives via the HVLD deep-learning module) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **STAR-Fusion v1.12.0** (Competing de novo fusion caller run on the same per-cell FASTQs for benchmark comparison) — https://github.com/STAR-Fusion
- **FusionInspector v2.8.0** (Competing fusion verification/inspection tool used as bRNA-seq benchmark method) — https://github.com/FusionInspector/FusionInspector
- **FusionCatcher v1.30** (Competing fusion detection tool used as bRNA-seq benchmark method) — https://github.com/ndaniel/fusioncatcher
- **Arriba v2.4.0** (Competing STAR-based fusion caller used as bRNA-seq benchmark method) — https://github.com/suhrig/arriba
- **scFusion** (Competing single-cell-specific fusion detection method for the scRNA-seq benchmark) — https://github.com/XiDsLab/scFusion
- **BWA-MEM** (Alignment tool used internally by Anchored-fusion to map reads to the anchored gene and genome)
- **BLAT** (Local alignment of candidate chimeric reads to precisely predict fusion breakpoints)

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=BCR_transcript.fasta --fastq1=cell_001_1.fastq.gz --fastq2=cell_001_2.fastq.gz --out_folder=output_dir --file_ref_seq=GRCh38.genome.fa --file_ref_ann=gencode.annotation.gtf
```

## Evaluation signals

- Reproduce the published benchmark: Anchored-fusion detects 35 BCR-ABL1-positive cells among 350 K562 Smart-seq2 cells while competitor tools report lower counts
- Confirm 83.3% recovery of the 42 evidence-bearing cells (35/42) by Anchored-fusion
- Confirm detection in two-thirds of very-low-evidence cells (10/15)
- Verify all tools were run on identical per-cell FASTQ inputs and calls were tabulated per cell, avoiding input-inconsistency artifacts
- Check that Anchored-fusion output contains only fusions including the anchored target gene (BCR), as other fusions are not represented

## Limitations

- Anchored-fusion cannot detect fusions that do not include the user-provided target gene
- Anchored-fusion requires paired-end RNA-seq and cannot be used on single-end RNA-seq, whole-genome, or whole-exome data
- Users must supply the reference sequence of their target gene
- For very small input datasets, the retrained HVLD filter is unreliable and default GitHub model parameters should be used
- The loss-balancing hyperparameter alpha in the HVLD loss function is not specified in the article
- Sequence homology (e.g., BCR homologous regions) suppresses multi-mapping read recovery in most de novo tools, which is the bias this benchmark is designed to expose

## Evidence
