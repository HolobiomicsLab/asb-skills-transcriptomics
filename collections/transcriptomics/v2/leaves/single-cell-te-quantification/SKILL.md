---
name: single-cell-te-quantification
description: Use when single-cell short-read data (10X scRNA, Smart-seq2 scRNA, scATAC-seq,
  Visium, multi-omics) requires TE quantification where multi-mapping reads from repetitive
  TEs must be accurately assigned to specific loci rather than collapsed to subfamily
  level.
when_to_use_negative:
- Data lacks sufficient multi-mapping TE reads — use MATES simple mode / version >=0.1.8
  fallback instead of full locus-specific inference
- Target TEs are long autonomously expressed elements like L1HS spanning several thousand
  bases — flanking-context-based multi-mapping estimation is inaccurate for TEs longer
  than 2001 bp
- Only bulk-level TE quantification is needed — bulk tools such as TEtranscripts or
  Telescope are designed for non-single-cell inputs
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0621
tools:
- name: MATES
  role: 'Core deep-learning pipeline: bam_processor, data_processor, MATES_model (AutoEncoder
    + MLP), and TE_quantifier modules for locus-specific multi-mapping read allocation
    and TE counting'
  repo: https://github.com/mcgilldinglab/MATES
- name: scTE
  role: Baseline single-cell TE quantification method for benchmarking correlation
    against long-read ground truth (R2 = 0.6499 vs MATES 0.7531 on nanopore single-cell)
- name: SoloTE
  role: Baseline single-cell TE quantification method for benchmarking (R2 = 0.6841
    vs MATES 0.7531 on nanopore single-cell)
- name: bam_processor
  role: MATES module that partitions BAMs per cell, splits unique vs multi-mapping
    reads, and constructs TE-specific coverage vectors
  repo: https://github.com/mcgilldinglab/MATES
- name: TE_quantifier
  role: MATES module using the trained MLP regressor to estimate alpha_i and compute
    locus-level per-cell TE counts
  repo: https://github.com/mcgilldinglab/MATES
- name: TE_quantifier_LongRead
  role: MATES module for TE quantification on long-read (nanopore/PacBio) single-cell
    data
  repo: https://github.com/mcgilldinglab/MATES
- name: TE_quantifier_Intronic
  role: MATES module applying intronic-mode correction by subtracting Velocyto-identified
    unspliced reads (N'=N-s) from intronic TE counts
  repo: https://github.com/mcgilldinglab/MATES
provenance:
  source_task_ids:
  - task_003
  - task_002
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:b05c4d248704daeb692a982da63c60fc3365b70f24b90a34559bd414f06abe59
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mcgilldinglab/MATES
  repo_url: https://github.com/mcgilldinglab/MATES
  tools:
  - MATES
  - MATES_model
  - SoloTE
  - TE_quantifier
  - TE_quantifier_Intronic
  - TE_quantifier_LongRead
  - bam_processor
  - data_processor
  - scTE
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# single-cell-te-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify transposable element (TE) expression/accessibility at locus-specific resolution in single-cell sequencing data using MATES, a deep neural network that probabilistically allocates multi-mapping reads to individual TE loci based on flanking unique-read coverage context, improving over subfamily-level methods like scTE and SoloTE.

## When to use

Use when single-cell short-read data (10X scRNA, Smart-seq2 scRNA, scATAC-seq, Visium, multi-omics) requires TE quantification where multi-mapping reads from repetitive TEs must be accurately assigned to specific loci rather than collapsed to subfamily level.

## When NOT to use

- Data lacks sufficient multi-mapping TE reads — use MATES simple mode / version >=0.1.8 fallback instead of full locus-specific inference
- Target TEs are long autonomously expressed elements like L1HS spanning several thousand bases — flanking-context-based multi-mapping estimation is inaccurate for TEs longer than 2001 bp
- Only bulk-level TE quantification is needed — bulk tools such as TEtranscripts or Telescope are designed for non-single-cell inputs

## Inputs

- STAR/STARsolo-aligned BAM files from single-cell RNA-seq, scATAC-seq, Visium, or multi-omics data (10X format: one BAM per sample; Smart_seq format: one BAM per cell)
- Barcode list files (10X data, bc_ind e.g. CB/CR)
- TE reference BED file (RepeatMasker-derived, e.g. TE_nooverlap.bed for exclusive or TE_full.bed for inclusive mode) and gene reference GTF
- Sample list and BAM path list files

## Outputs

- Per-cell locus-specific TE count matrix (URi + ceil(alpha_i x MRi) per locus)
- Subfamily-level TE count matrix
- Locus-specific multi-mapping rate estimates (alpha_i) from the MLP regressor
- BigWig coverage files with multi-mapping coverage adjusted per base pair by alpha_i
- Cell clusters and ARI/NMI agreement metrics between TE-based and gene-based clusterings
- Marker TE associations per cell population (e.g. MERVL-int and MT2_Mm for 2CLC)

## How to apply

Map raw reads with STAR/STARsolo, then use MATES.bam_processor to segregate unique- vs multi-mapping reads and construct unique (Vu) and multi (Vm) coverage vectors spanning 2001 bp (1000 bp up- and downstream of each TE start). Classify bins as multi-dominant (M) or unique-dominant (U) using bin size W and threshold P, downsample training samples to at most 2500 per TE family, drop families with fewer than 50 samples, and truncate TE loci longer than 1000 bp to the first 1000 bp. Train the AutoEncoder (MATES_model) on unique-read coverage vectors with TE-family one-hot encoding, then estimate the locus-specific multi-mapping rate alpha_i with the MLP regressor. Compute per-cell per-locus counts as URi + ceil(alpha_i x MRi) with MATES.TE_quantifier, then cluster cells on the TE-only matrix and compare against gene-based clustering using ARI/NMI, and identify marker TEs per cell population.

## Related tools

- **MATES** (Core deep-learning pipeline: bam_processor, data_processor, MATES_model (AutoEncoder + MLP), and TE_quantifier modules for locus-specific multi-mapping read allocation and TE counting) — https://github.com/mcgilldinglab/MATES
- **scTE** (Baseline single-cell TE quantification method for benchmarking correlation against long-read ground truth (R2 = 0.6499 vs MATES 0.7531 on nanopore single-cell))
- **SoloTE** (Baseline single-cell TE quantification method for benchmarking (R2 = 0.6841 vs MATES 0.7531 on nanopore single-cell))
- **bam_processor** (MATES module that partitions BAMs per cell, splits unique vs multi-mapping reads, and constructs TE-specific coverage vectors) — https://github.com/mcgilldinglab/MATES
- **TE_quantifier** (MATES module using the trained MLP regressor to estimate alpha_i and compute locus-level per-cell TE counts) — https://github.com/mcgilldinglab/MATES
- **TE_quantifier_LongRead** (MATES module for TE quantification on long-read (nanopore/PacBio) single-cell data) — https://github.com/mcgilldinglab/MATES
- **TE_quantifier_Intronic** (MATES module applying intronic-mode correction by subtracting Velocyto-identified unspliced reads (N'=N-s) from intronic TE counts) — https://github.com/mcgilldinglab/MATES

## Examples

```
from MATES import MATES_pipeline; mates = MATES_pipeline(TE_mode, data_mode, sample_list_file, bam_path_file); mates.preprocessing(); mates.run()
```

## Evaluation signals

- TE-only clustering agreement with gene-based clustering reproduces reported medians: ARI 0.397 / NMI 0.496 (mouse 2CLC 10X), ARI 0.105 / NMI 0.161 (Smart-seq2 glioblastoma), ARI 0.309 / NMI 0.438 (10X scATAC mouse brain)
- Locus-level clustering ARI significantly exceeds subfamily-level ARI (P = 5.48e-7 for glioblastoma, P = 2.55e-34 for scATAC)
- Against nanopore long-read ground truth, subfamily R2 for MATES exceeds scTE and SoloTE in all settings (0.7531 single-cell, 0.9276 pseudo-bulk, 0.5096 PacBio) with one-sided F-test p < 0.05
- Known marker TEs are recovered in expected cell populations: MERVL-int and MT2_Mm enriched flanking Zscan4c/Zscan4d in 2CLC (binomial test); RMER16_Mm/RLTR44B in macrophages; MamRep434/MER124 in astrocytes
- Count matrix schema check: every locus count equals URi + ceil(alpha_i x MRi); training-set filters respected (max 2500 samples/family, families < 50 samples dropped, loci > 1000 bp truncated)

## Limitations

- MATES typically requires more computational resources such as GPUs than scTE and SoloTE
- Multi-mapping rate estimates may be inaccurate for TEs longer than 2001 bp due to incomplete flanking coverage; long autonomously expressed TEs like L1HS are hard to quantify
- Identical TE copies with high sequence similarity (e.g. HERVK isoforms) cannot be distinguished when flanking regions lack uniquely mappable reads
- Trimming TE-gene overlaps (exclusive mode) may unintentionally distort TE quantification; no quantitative guidance on when to opt out
- Velocyto does not identify which specific reads are unspliced, requiring custom source-adapted code for the intronic mode
- Total runtime including preprocessing and mapping increases linearly with data size

## Evidence

- [readme] simple mode pipeline: [evidence span withheld — anchor status: external_doc]
