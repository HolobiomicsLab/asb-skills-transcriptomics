---
name: low-sequencing-depth-sensitivity-analysis
description: Use when quantifying how a fusion caller's sensitivity degrades with
  shallower bulk or single-cell RNA-seq depth, e.g. testing detection limits on CCLE
  bRNA-seq subsampled below 1 million read pairs.
when_to_use_negative:
- Input is single-end RNA-seq, whole-genome, or whole-exome sequencing data — Anchored-fusion
  only supports paired-end RNA-seq
- The goal is discovery of fusions not involving a known target gene — Anchored-fusion
  only reports fusions including the anchored gene of interest
- The question is fusion-call accuracy at full depth rather than sensitivity decay
  across depths (use standard de novo callers directly instead)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Anchored-fusion
  role: Ultra-sensitive targeted fusion caller under test; anchors the gene of interest,
    aligns with BWA-MEM, local-aligns chimeric reads with BLAT
  repo: https://github.com/ShenLab-Genomics/Anchored-Fusion
- name: STAR-Fusion v1.12.0
  role: bRNA-seq competing method benchmarked across subsampling depths
  repo: https://github.com/STAR-Fusion
- name: FusionInspector v2.8.0
  role: bRNA-seq competing method benchmarked across subsampling depths
  repo: https://github.com/FusionInspector/FusionInspector
- name: FusionCatcher v1.30
  role: bRNA-seq competing method benchmarked across subsampling depths
  repo: https://github.com/ndaniel/fusioncatcher
- name: Arriba v2.4.0
  role: bRNA-seq competing method benchmarked across subsampling depths
  repo: https://github.com/suhrig/arriba
- name: BWA-MEM
  role: Aligner used by Anchored-fusion to map reads to the anchored gene and genome
- name: BLAT
  role: Local alignment of candidate chimeric reads to pinpoint precise fusion location
- name: samtools
  role: Random halving subsampling of read pairs to generate depth series
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100733
    title: Anchored-fusion enables targeted fusion search in bulk and single-cell
      RNA sequencing data
  source_content_hash: sha256:2161d297d9821f56845a7afca467601cc71c43c1e40d02cc938df307c5c2a21a
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
  - STAR-Fusion v1.12.0
  - samtools
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# low-sequencing-depth-sensitivity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks fusion-gene detection sensitivity as sequencing depth is progressively reduced by repeated random subsampling of paired-end RNA-seq FASTQ files, verifying whether an ultra-sensitive caller (Anchored-fusion) still detects known fusions where standard tools fail below 1 M read pairs.

## When to use

Use when quantifying how a fusion caller's sensitivity degrades with shallower bulk or single-cell RNA-seq depth, e.g. testing detection limits on CCLE bRNA-seq subsampled below 1 million read pairs.

## When NOT to use

- Input is single-end RNA-seq, whole-genome, or whole-exome sequencing data — Anchored-fusion only supports paired-end RNA-seq
- The goal is discovery of fusions not involving a known target gene — Anchored-fusion only reports fusions including the anchored gene of interest
- The question is fusion-call accuracy at full depth rather than sensitivity decay across depths (use standard de novo callers directly instead)

## Inputs

- Paired-end RNA-seq FASTQ files (R1/R2) from CCLE cancer cell lines (SRA:SRP186687)
- hg38 genome fasta and compatible GTF annotation
- Target gene transcript reference fasta (downloadable via NCBI Gene)
- Known/found fusion-gene calls at full depth as positive controls

## Outputs

- 8 subsampled FASTQ depth levels per sample (7 halving rounds, final depth < 1 M read pairs)
- Tool-by-depth fusion detection matrix (which caller called which fusion at which depth)
- Detection-sensitivity comparison figure/statistics (e.g. Figure 4 pattern: Anchored-fusion positive where all four competitors fail)

## How to apply

Download paired-end FASTQ files (e.g. six CCLE cell lines K562, TE441T, NALM6, NCI-H660, NCI-H3122, KM12 from SRA:SRP186687) and create 8 depths per sample via 7 successive random halving rounds of read pairs (e.g. samtools view), ending below 1 M reads. At each depth run the competing methods STAR-Fusion v1.12.0, FusionInspector v2.8.0, FusionCatcher v1.30, and Arriba v2.4.0 against hg38 fasta and matching GTF. In parallel run Anchored-fusion with the target gene's transcript fasta, BWA-MEM alignment, and BLAT local alignment of candidate chimeric reads. Collate per-tool per-depth fusion calls into a detection matrix and verify the expected pattern: Anchored-fusion detects the fusion at depths where all competitors fail.

## Related tools

- **Anchored-fusion** (Ultra-sensitive targeted fusion caller under test; anchors the gene of interest, aligns with BWA-MEM, local-aligns chimeric reads with BLAT) — https://github.com/ShenLab-Genomics/Anchored-Fusion
- **STAR-Fusion v1.12.0** (bRNA-seq competing method benchmarked across subsampling depths) — https://github.com/STAR-Fusion
- **FusionInspector v2.8.0** (bRNA-seq competing method benchmarked across subsampling depths) — https://github.com/FusionInspector/FusionInspector
- **FusionCatcher v1.30** (bRNA-seq competing method benchmarked across subsampling depths) — https://github.com/ndaniel/fusioncatcher
- **Arriba v2.4.0** (bRNA-seq competing method benchmarked across subsampling depths) — https://github.com/suhrig/arriba
- **BWA-MEM** (Aligner used by Anchored-fusion to map reads to the anchored gene and genome)
- **BLAT** (Local alignment of candidate chimeric reads to pinpoint precise fusion location)
- **samtools** (Random halving subsampling of read pairs to generate depth series)

## Examples

```
python Anchored_Fusion.py --file_anchored_cds=target_gene.fasta --fastq1=sample_R1.fastq.gz --fastq2=sample_R2.fastq.gz --out_folder=output_dir --file_ref_seq=reference.genome.fa --file_ref_ann=genome.annotation.gtf
```

## Evaluation signals

- Each sample yields exactly 8 depth levels after 7 halving rounds, with the final depth below 1 million read pairs
- Anchored-fusion calls the known fusion at every subsampling depth (matches Figure 4 pattern)
- All four competitors (STAR-Fusion, FusionInspector, FusionCatcher, Arriba) fail to call the fusion at the lowest depths
- Full-depth runs reproduce the reference fusion calls, validating the subsampled comparison baseline
- Detection matrix rows/columns align: each tool has a call/no-call entry for every sample-depth combination

## Limitations

- Anchored-fusion cannot detect fusion genes that do not include the user-provided target gene, so the depth benchmark only covers targeted fusions
- Anchored-fusion requires paired-end RNA-seq; it cannot be applied to single-end RNA-seq or WGS/WES data
- For small input (shallow) datasets, users should rely on the default pre-trained HVLD model parameters from GitHub rather than retraining the filter
- Subsampling-based sensitivity estimates depend on the random halving procedure and a single known fusion per cell line

## Evidence
