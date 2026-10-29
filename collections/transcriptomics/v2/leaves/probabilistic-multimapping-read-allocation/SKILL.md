---
name: probabilistic-multimapping-read-allocation
description: Use when single-cell (scRNA, scATAC, Multi-Omics, Visium, or long-read)
  BAM files mapped by STAR/STARsolo contain multi-mapping reads over repetitive TE
  loci and locus-specific (not subfamily-level) TE counts are needed.
when_to_use_negative:
- Quantifying long autonomously expressed TEs such as L1HS that span several thousand
  bases — flanking coverage within the 2001 bp window is incomplete and α estimates
  unreliable
- Distinguishing highly identical TE copies (e.g., HERVK isoforms) that lack uniquely
  mappable flanking regions
- Subfamily-level TE quantification alone is sufficient, or data lack sufficient multi-mapping
  TE reads (though MATES ≥ v0.1.8 adds a fallback mode)
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: MATES
  role: Deep learning framework whose AutoEncoder + MLP predicts locus-specific multi-mapping
    ratio α from unique-read coverage context; modules bam_processor, data_processor,
    MATES_model, TE_quantifier implement the pipeline
  repo: https://github.com/mcgilldinglab/MATES
- name: STAR/STARsolo
  role: Upstream aligner producing the BAM whose reads are segregated into unique-
    and multi-mapping sets
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:a90e0ad7fc0abcb3df3c47ec2430f4ffa2d61ae5e3cf01cd0e5bdb7c3dd91608
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
  - STAR/STARsolo
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# probabilistic-multimapping-read-allocation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MATES-based deep learning procedure that predicts a locus-specific multi-mapping ratio (α) for each transposable element locus from unique-read flanking coverage context, then probabilistically allocates multi-mapping reads as UR_i + ceil(α_i × MR_i) for single-cell, locus-level TE quantification.

## When to use

Use when single-cell (scRNA, scATAC, Multi-Omics, Visium, or long-read) BAM files mapped by STAR/STARsolo contain multi-mapping reads over repetitive TE loci and locus-specific (not subfamily-level) TE counts are needed.

## When NOT to use

- Quantifying long autonomously expressed TEs such as L1HS that span several thousand bases — flanking coverage within the 2001 bp window is incomplete and α estimates unreliable
- Distinguishing highly identical TE copies (e.g., HERVK isoforms) that lack uniquely mappable flanking regions
- Subfamily-level TE quantification alone is sufficient, or data lack sufficient multi-mapping TE reads (though MATES ≥ v0.1.8 adds a fallback mode)

## Inputs

- Sorted single-cell BAM file(s) mapped with STAR/STARsolo (10X-format multi-cell BAM or Smart-seq one-cell-per-BAM)
- Cell barcode list file (bc_path_file) with barcode field indicator (e.g., CB) for 10X-format data
- TE reference bed/csv file (e.g., 'TE_nooverlap.bed' for exclusive or 'TE_full.bed' for inclusive mode); default references generated via build_reference.py

## Outputs

- Per-locus TE count matrix (te_locus_counts.tsv) with one row per TE locus and numeric count column satisfying count = UR + ceil(alpha × MR)
- Locus-specific multi-mapping ratios (α_i) per TE locus
- Coverage vectors Vu and Vm per TE locus, plus U/M bin labels
- BigWig coverage files with multi-mapping read coverage per base pair scaled by α_i
- Subfamily-level and locus-level TE quantification matrices

## How to apply

Segregate reads into unique- and multi-mapping sets with the MATES bam_processor module, then build coverage vectors Vu (unique) and Vm (multi) spanning 2001 bp centered on each TE start (1000 bp each side), truncating loci longer than 1000 bp to their initial 1000 bp. Slice vectors into W-bp bins and label each bin M-bin if the multi-mapping read percentage exceeds threshold P, else U-bin. Train an AutoEncoder on the Vu vectors with a one-hot TE-family vector (MSE reconstruction loss), downsampling to at most 2500 samples per family and dropping families with fewer than 50 samples; the latent embedding is combined with the TE-family encoding and fed to an MLP regressor predicting α per locus, optimized with combined loss L1 + λ·L2 (grand loss plus continuity loss over bin labels). Compute per-cell locus counts as UR_i + ceil(α_i × MR_i) and multiply multi-mapping coverage per base pair by α_i for BigWig export.

## Related tools

- **MATES** (Deep learning framework whose AutoEncoder + MLP predicts locus-specific multi-mapping ratio α from unique-read coverage context; modules bam_processor, data_processor, MATES_model, TE_quantifier implement the pipeline) — https://github.com/mcgilldinglab/MATES
- **STAR/STARsolo** (Upstream aligner producing the BAM whose reads are segregated into unique- and multi-mapping sets)

## Examples

```
from MATES import MATES_pipeline; mates = MATES_pipeline(TE_mode, data_mode, sample_list_file, bam_path_file); mates.preprocessing(); mates.run()
```

## Evaluation signals

- te_locus_counts.tsv exists, contains one row per TE locus with a numeric count column
- Spot-check loci satisfy count = UR + ceil(α × MR) using stored UR, α, and MR values
- AE training filters applied: no TE family retained with fewer than 50 samples, and at most 2500 training samples per family
- Coverage vectors span exactly 2001 bp (1000 bp up/downstream of TE start), with loci > 1000 bp truncated to the initial 1000 bp
- Downstream validation: MATES quantification improves cell clustering and marker TE identification versus scTE/SoloTE, verifiable against long-read (scNanoRNAseq, PacBio) ground truth as in the paper

## Limitations

- MATES requires more computational resources (GPUs) than scTE and SoloTE
- Multi-mapping rate estimation may be inaccurate for TEs longer than 2001 bp due to incomplete flanking coverage
- Identical TE copies (e.g., HERVK isoforms) with high sequence similarity and no uniquely mappable flanking regions are hard to distinguish
- Trimming TE-gene overlaps (exclusive mode) may unintentionally distort TE quantification; no quantitative guidance on when to opt out
- Total runtime including preprocessing and mapping increases linearly with data size

## Evidence

- [readme] bam_processor role: [evidence span withheld — anchor status: external_doc]
