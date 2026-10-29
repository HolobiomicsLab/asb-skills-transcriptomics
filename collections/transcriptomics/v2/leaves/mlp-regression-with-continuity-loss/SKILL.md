---
name: mlp-regression-with-continuity-loss
description: 'Use when single-cell reads must be allocated to individual TE loci:
  unique/multi coverage vectors (Vu/Vm) with U/M bin labels are available and per-locus
  α estimates are needed for read counting.'
when_to_use_negative:
- Quantifying long autonomously expressed TEs such as L1HS spanning several thousand
  bases — the 2001-bp coverage window gives incomplete flanking coverage and inaccurate
  α estimates
- Distinguishing identical TE copies such as HERVK isoforms that lack uniquely mappable
  flanking regions
- Data without sufficient multi-mapping TE reads is better handled by MATES v0.1.8's
  simple mode rather than full α regression
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: MATES
  role: Provides the AutoEncoder + MLP regressor (MATES_model, data_processor) that
    encodes Vu coverage vectors with TE family one-hot inputs and predicts locus-specific
    multi-mapping ratio α
  repo: https://github.com/mcgilldinglab/MATES
- name: bam_processor
  role: MATES module that splits BAMs into per-cell sub-BAMs, segregates unique vs
    multi-mapping reads, and constructs TE-specific coverage vectors feeding the model
  repo: https://github.com/mcgilldinglab/MATES
- name: data_processor
  role: MATES module for generating training and prediction data (U/M binning of coverage
    vectors) used by the AE/MLP training
  repo: https://github.com/mcgilldinglab/MATES
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:69fc2578a60cebe2a92e5b14b25b873a67a7dccd24502aba6a44d305a6d8e669
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
  - bam_processor
  - data_processor
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mlp-regression-with-continuity-loss

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a multilayer perceptron regressor on AutoEncoder-encoded unique-read coverage vectors (Vu), combined with TE family one-hot embeddings, to predict a locus-specific multi-mapping ratio (α) used by MATES to probabilistically allocate multi-mapping reads to specific TE loci.

## When to use

Use when single-cell reads must be allocated to individual TE loci: unique/multi coverage vectors (Vu/Vm) with U/M bin labels are available and per-locus α estimates are needed for read counting.

## When NOT to use

- Quantifying long autonomously expressed TEs such as L1HS spanning several thousand bases — the 2001-bp coverage window gives incomplete flanking coverage and inaccurate α estimates
- Distinguishing identical TE copies such as HERVK isoforms that lack uniquely mappable flanking regions
- Data without sufficient multi-mapping TE reads is better handled by MATES v0.1.8's simple mode rather than full α regression

## Inputs

- Sorted single-cell BAM file(s) mapped with STAR/STARsolo, segregated into unique-mapping and multi-mapping read sets via MATES bam_processor
- Unique-read coverage vectors Vu and multi-mapping coverage vectors Vm spanning 2001 bp centered on each TE start position
- W-bp bin labels (U-bin / M-bin) determined by threshold P on multi-mapping read percentage
- One-hot encoded TE family vectors

## Outputs

- Per-locus multi-mapping ratio α_i predicted by the MLP regressor
- Per-locus total read counts per cell (UR_i + ceil(α_i × MR_i)), written to te_locus_counts.tsv
- Multi-mapping coverage per base pair adjusted by α_i, exportable as BigWig files

## How to apply

First train an AutoEncoder on unique-read coverage vectors Vu (2001 bp windows centered on each TE start, sliced into W-bp bins labeled M-bin if multi-mapping read percentage exceeds threshold P, else U-bin), using a one-hot TE-family input vector and MSE reconstruction loss; downsample to at most 2500 training samples per TE family and drop families with fewer than 50 samples. Then train an MLP regressor on the AE latent embeddings (Vu encoding) to predict the locus-specific multi-mapping ratio α, optimizing the combined loss L1 + lambda*L2 where L1 is the grand loss and L2 is a continuity loss enforcing smooth α transitions across adjacent bin labels. The predicted α is applied as total locus count = UR_i + ceil(α_i × MR_i). Long TE loci (>1000 bp) are truncated to their initial 1000 bp before vector construction.

## Related tools

- **MATES** (Provides the AutoEncoder + MLP regressor (MATES_model, data_processor) that encodes Vu coverage vectors with TE family one-hot inputs and predicts locus-specific multi-mapping ratio α) — https://github.com/mcgilldinglab/MATES
- **bam_processor** (MATES module that splits BAMs into per-cell sub-BAMs, segregates unique vs multi-mapping reads, and constructs TE-specific coverage vectors feeding the model) — https://github.com/mcgilldinglab/MATES
- **data_processor** (MATES module for generating training and prediction data (U/M binning of coverage vectors) used by the AE/MLP training) — https://github.com/mcgilldinglab/MATES

## Examples

```
from MATES import MATES_model, data_processor
```

## Evaluation signals

- te_locus_counts.tsv exists and contains one row per TE locus with a numeric count column
- Spot-check loci satisfy count = UR_i + ceil(α_i × MR_i) using the stored UR, α, and MR values
- Training data per TE family does not exceed 2500 samples and families with fewer than 50 samples are excluded
- Predicted α shows continuity (smoothness) across adjacent bins, reflecting the L2 continuity loss
- MATES validation against long-read data (scNanoRNAseq, PacBio) shows improved locus-specific TE quantification over scTE and SoloTE

## Limitations

- MATES requires more computational resources (GPUs) than scTE and SoloTE
- Multi-mapping rate estimation may be inaccurate for TEs longer than 2001 bp due to incomplete flanking coverage
- Identical TE copies (e.g., HERVK isoforms) are hard to distinguish due to high sequence similarity and lack of uniquely mappable flanking regions
- Lengthening the coverage vector is not a definitive fix for long-TE limitations; users should remain cautious

## Evidence

- [readme] An autoEncoder model is employed to create latent embeddings (Zm) capturing local read context and is combined with TE family information (Tk).: [evidence span withheld — anchor status: external_doc]
