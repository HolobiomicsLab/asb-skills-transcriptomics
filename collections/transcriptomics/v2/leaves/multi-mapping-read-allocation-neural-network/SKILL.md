---
name: multi-mapping-read-allocation-neural-network
description: Use when quantifying transposable element expression at individual locus
  resolution in single-cell scRNA-seq/scATAC-seq data where repetitive TE sequences
  generate multi-mapping reads that alignment-only tools (scTE, SoloTE, Telescope)
  cannot resolve per-locus.
when_to_use_negative:
- Quantifying long autonomously expressed TEs such as L1HS spanning several thousand
  bases, where flanking context within the 2001 bp window is incomplete and αi estimates
  become inaccurate
- Distinguishing highly identical TE copies (e.g., HERVK isoforms) that lack uniquely
  mappable flanking regions — the model cannot separate these loci
- 'When computational resources lack GPUs: MATES typically requires more resources
  such as GPUs than scTE or SoloTE'
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: MATES
  role: Deep learning framework (AutoEncoder + MLP regressor) that estimates locus-specific
    multi-mapping rates and produces locus-level TE count matrices
  repo: https://github.com/mcgilldinglab/MATES
- name: bam_processor
  role: MATES module that partitions BAMs per cell, segregates unique vs multi-mapping
    reads, and constructs TE-specific coverage vectors
  repo: https://github.com/mcgilldinglab/MATES
- name: data_processor
  role: MATES module that generates training/prediction data, classifies bins as M
    or U using bin size W and threshold P
  repo: https://github.com/mcgilldinglab/MATES
- name: MATES_model
  role: MLP regressor module that predicts the multi-mapping ratio αi from latent
    embeddings and TE-family one-hot encodings
  repo: https://github.com/mcgilldinglab/MATES
- name: TE_quantifier
  role: MATES module that computes final locus-level TE counts as URi + ceil(αi ×
    MRi) and exports BigWig files
  repo: https://github.com/mcgilldinglab/MATES
- name: TE_quantifier_Intronic
  role: MATES module applying intronic-mode correction N′ = N − s by subtracting Velocyto-identified
    unspliced reads
  repo: https://github.com/mcgilldinglab/MATES
- name: scTE
  role: Comparison baseline single-cell TE quantification method (subfamily-level,
    alignment-based)
- name: SoloTE
  role: Comparison baseline single-cell TE quantification method (subfamily-level,
    alignment-based)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:3b712d34fa2ad2720bd31648c878c8dd29d89c6e4d3d24b9a22fddea43352739
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
  - bam_processor
  - data_processor
  - scTE
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# multi-mapping-read-allocation-neural-network

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A deep learning skill (MATES) that probabilistically allocates multi-mapping reads to specific transposable element loci in single-cell data by learning locus-specific multi-mapping rates from unique-read coverage context flanking each TE start, rather than assigning reads to best- or random-mapped loci.

## When to use

Use when quantifying transposable element expression at individual locus resolution in single-cell scRNA-seq/scATAC-seq data where repetitive TE sequences generate multi-mapping reads that alignment-only tools (scTE, SoloTE, Telescope) cannot resolve per-locus.

## When NOT to use

- Quantifying long autonomously expressed TEs such as L1HS spanning several thousand bases, where flanking context within the 2001 bp window is incomplete and αi estimates become inaccurate
- Distinguishing highly identical TE copies (e.g., HERVK isoforms) that lack uniquely mappable flanking regions — the model cannot separate these loci
- When computational resources lack GPUs: MATES typically requires more resources such as GPUs than scTE or SoloTE

## Inputs

- STAR/STARsolo-aligned BAM files from 10x scRNA-seq (barcoded, multi-cell per BAM) or Smart-seq2 (one BAM per cell)
- TE reference bed file (RepeatMasker-derived, e.g., TE_nooverlap.bed for exclusive mode or TE_full.bed for inclusive mode)
- Gene reference GTF (optionally trimmed against TE reference with bedtools)
- Barcode list files matching each BAM (bc_path_file) for 10X-format data
- Optional: Velocyto splicing output for intronic-mode correction via TE_quantifier_Intronic

## Outputs

- Locus-level TE count matrix (cells × TE loci) with counts computed as URi + ceil(αi × MRi)
- Subfamily-level TE count matrix
- Locus-specific multi-mapping rate estimates αi per cell/locus
- BigWig coverage files with multi-mapping coverage scaled per base pair by αi
- Per-cell unique (Vu) and multi-mapping (Vm) coverage vectors and U/M bin classifications

## How to apply

Map raw reads with STAR/STARsolo, then use MATES.bam_processor to segregate unique- and multi-mapping reads aligned to TE loci. Build 2001 bp coverage vectors (1000 bp up-/downstream of each TE start) with MATES.data_processor, dividing each vector into bins classified as multi-dominant (M) or unique-dominant (U) based on whether the multi-mapping read percentage exceeds threshold P (bin size W). Train the AutoEncoder on unique-read coverage vectors (Vu) with TE-family one-hot encoding (Tk), limiting training samples to at most 2500 per TE family and dropping families with fewer than 50 samples; truncate TE loci longer than 1000 bp to the first 1000 bp. Use the MATES_model MLP regressor on the latent embeddings (Zm) plus family encoding to estimate the locus-specific multi-mapping rate αi, then compute per-locus total counts as URi + ceil(αi × MRi) with MATES.TE_quantifier. Downstream, cluster cells on the MATES TE count matrix (e.g., Louvain/Leiden) and identify marker TEs — e.g., MERVL-int and MT2_Mm defining the 2CLC population — with locus-level clustering agreement assessed against gene-based clustering via ARI (reported median 0.397) and NMI (0.496).

## Related tools

- **MATES** (Deep learning framework (AutoEncoder + MLP regressor) that estimates locus-specific multi-mapping rates and produces locus-level TE count matrices) — https://github.com/mcgilldinglab/MATES
- **bam_processor** (MATES module that partitions BAMs per cell, segregates unique vs multi-mapping reads, and constructs TE-specific coverage vectors) — https://github.com/mcgilldinglab/MATES
- **data_processor** (MATES module that generates training/prediction data, classifies bins as M or U using bin size W and threshold P) — https://github.com/mcgilldinglab/MATES
- **MATES_model** (MLP regressor module that predicts the multi-mapping ratio αi from latent embeddings and TE-family one-hot encodings) — https://github.com/mcgilldinglab/MATES
- **TE_quantifier** (MATES module that computes final locus-level TE counts as URi + ceil(αi × MRi) and exports BigWig files) — https://github.com/mcgilldinglab/MATES
- **TE_quantifier_Intronic** (MATES module applying intronic-mode correction N′ = N − s by subtracting Velocyto-identified unspliced reads) — https://github.com/mcgilldinglab/MATES
- **scTE** (Comparison baseline single-cell TE quantification method (subfamily-level, alignment-based))
- **SoloTE** (Comparison baseline single-cell TE quantification method (subfamily-level, alignment-based))

## Examples

```
from MATES import MATES_pipeline; mates = MATES_pipeline(TE_mode, data_mode, sample_list_file, bam_path_file); mates.preprocessing(); mates.run()
```

## Evaluation signals

- 2CLC cluster from TE-only clustering co-expresses Zscan4c/Zscan4d and is enriched for MERVL-int and MT2_Mm reads flanking the Zscan4c/Zscan4d loci (binomial test)
- Locus-level clustering agreement with gene-based clustering reaches reported medians ARI 0.397 and NMI 0.496 in the 10x mouse chemical reprogramming dataset
- Predicted multi-mapping allocations agree with long-read ground truth (scNanoRNAseq nanopore and PacBio postnatal mouse brain data)
- Locus-level ARI exceeds subfamily-level ARI across clustering resolutions (Fig. 2h box plot comparison)
- Training set passes filter checks: ≤2500 training samples per TE family retained and families with <50 samples removed

## Limitations

- MATES typically requires more computational resources such as GPUs than scTE and SoloTE for quantifying TEs
- Limited ability to quantify long autonomously expressed TEs like L1HS spanning several thousand bases; adjusting coverage vector length is not a definitive solution and multi-mapping rate estimates may be inaccurate for TEs longer than 2001 bp
- Identical TE copies (e.g., HERVK isoforms) with high sequence similarity and no uniquely mappable flanking regions cannot be distinguished between loci
- Trimming TE-gene overlaps (exclusive mode) may unintentionally distort TE quantification; no quantitative guidance on when to opt out is provided
- Velocyto does not identify which specific reads are unspliced, requiring custom source-adapted code (version 0.17) for the intronic mode
- Total runtime including preprocessing and mapping increases linearly with data size

## Evidence

- [methods] bam_processor 10X counting: [evidence span withheld — anchor status: external_doc]
