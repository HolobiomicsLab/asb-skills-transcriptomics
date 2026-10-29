---
name: autoencoder-training-on-coverage-vectors
description: 'Use when quantifying transposable elements at locus level in scRNA-seq
  or scATAC-seq data: multi-mapping reads must be allocated to specific TE loci using
  unique-read flanking context before counting.'
when_to_use_negative:
- TEs longer than 2001 bp (e.g., L1HS or long autonomously expressed elements) where
  flanking coverage vectors are incomplete and alpha estimates may be inaccurate
- Highly identical TE copies (e.g., HERVK isoforms) lacking uniquely mappable flanking
  regions, where the AE cannot distinguish loci
- Datasets without sufficient multi-mapping TE reads or TE families with fewer than
  50 training samples
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3308
tools:
- name: MATES
  role: Provides the MATES_model AutoEncoder and MLP regressor trained on Vu coverage
    vectors with TE-family one-hot encoding to estimate locus-specific multi-mapping
    ratios
  repo: https://github.com/mcgilldinglab/MATES
- name: bam_processor
  role: Segregates unique vs multi-mapping reads from BAMs and constructs the TE-specific
    Vu/Vm coverage vectors used as AE training input
  repo: https://github.com/mcgilldinglab/MATES
- name: TE_quantifier
  role: Applies the trained MLP to estimate alpha_i and computes per-locus counts
    as UR_i + ceil(alpha_i x MR_i)
  repo: https://github.com/mcgilldinglab/MATES
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:f99af6fe0237d1c0bc74cef78a552dbe075b500cd78c5c584ecfc99146043118
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
  - TE_quantifier
  - bam_processor
  - data_processor
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# autoencoder-training-on-coverage-vectors

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a MATES AutoEncoder on unique-mapping-read coverage vectors (Vu) spanning 2001 bp around each TE start, with one-hot TE-family encoding, to learn latent local-read-context embeddings used to predict locus-specific multi-mapping ratios in single-cell data.

## When to use

Use when quantifying transposable elements at locus level in scRNA-seq or scATAC-seq data: multi-mapping reads must be allocated to specific TE loci using unique-read flanking context before counting.

## When NOT to use

- TEs longer than 2001 bp (e.g., L1HS or long autonomously expressed elements) where flanking coverage vectors are incomplete and alpha estimates may be inaccurate
- Highly identical TE copies (e.g., HERVK isoforms) lacking uniquely mappable flanking regions, where the AE cannot distinguish loci
- Datasets without sufficient multi-mapping TE reads or TE families with fewer than 50 training samples

## Inputs

- Sorted single-cell BAM files (10X multi-cell format or Smart-seq one-cell-per-file) mapped with STAR/STARsolo
- TE reference bed/csv (e.g., TE_nooverlap.bed or TE_full.bed from build_reference.py)
- Unique-read coverage vectors Vu and multi-mapping coverage vectors Vm per TE locus (2001 bp)
- TE family one-hot encoding vectors
- U/M bin labels with bin size W and threshold P

## Outputs

- Trained AutoEncoder model producing latent embeddings (Zm) of local read context
- Trained MLP regressor predicting per-locus multi-mapping ratio alpha_i
- Per-locus TE count matrix (te_locus_counts.tsv) with counts computed as UR_i + ceil(alpha_i x MR_i)

## How to apply

After MATES bam_processor segregates unique and multi-mapping reads, construct unique-read coverage vectors Vu (and multi-mapping Vm) spanning 2001 bp — 1000 bp upstream and downstream of each TE start — truncating TE loci longer than 1000 bp to their first 1000 bp. Slice vectors into W-bp bins and label each bin M-bin (multi-dominant) or U-bin (unique-dominant) based on whether multi-mapping read percentage exceeds threshold P. Train the AutoEncoder (MATES_model module) on Vu with a one-hot TE-family input vector and MSE reconstruction loss, downsampling to at most 2500 training samples per TE family and removing families with fewer than 50 samples. The learned latent embedding is combined with the TE-family embedding and fed to an MLP regressor optimizing a combined loss L1 (grand loss) + lambda*L2 (bin-label continuity loss) to predict the locus-specific multi-mapping ratio alpha.

## Related tools

- **MATES** (Provides the MATES_model AutoEncoder and MLP regressor trained on Vu coverage vectors with TE-family one-hot encoding to estimate locus-specific multi-mapping ratios) — https://github.com/mcgilldinglab/MATES
- **bam_processor** (Segregates unique vs multi-mapping reads from BAMs and constructs the TE-specific Vu/Vm coverage vectors used as AE training input) — https://github.com/mcgilldinglab/MATES
- **TE_quantifier** (Applies the trained MLP to estimate alpha_i and computes per-locus counts as UR_i + ceil(alpha_i x MR_i)) — https://github.com/mcgilldinglab/MATES

## Examples

```
import MATES; from MATES import bam_processor, MATES_model; bam_processor.count_coverage_vec('exclusive', '10X', 4, sample_list_file, bc_path_file=bc_path_file)
```

## Evaluation signals

- Spot-check loci in te_locus_counts.tsv satisfy count = UR + ceil(alpha x MR) using stored UR, alpha, and MR values
- Training data respects filter thresholds: no TE family has more than 2500 training samples and families with fewer than 50 samples are removed
- Reconstruction loss (MSE) on Vu coverage vectors decreases and converges during AE training
- Downstream locus-level clustering on the resulting counts reproduces published benchmarks (median ARI 0.309, NMI 0.438 for 10x adult mouse brain scATAC-seq)
- Loci longer than 1000 bp are truncated to the initial 1000 bp in the training inputs

## Limitations

- MATES requires more computational resources such as GPUs than counterparts scTE and SoloTE for quantifying TEs
- Multi-mapping rate estimation may be inaccurate for TEs longer than 2001 bp due to incomplete flanking coverage
- Coverage vector length is fixed at 2001 bp, so long autonomously expressed TEs like L1HS spanning several thousand bases are poorly quantified; parameter adjustments are not definitive solutions

## Evidence
