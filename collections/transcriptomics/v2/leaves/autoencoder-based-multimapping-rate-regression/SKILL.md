---
name: autoencoder-based-multimapping-rate-regression
description: Use when Smart-seq2 or 10X single-cell RNA-seq/ATAC data must have multi-mapping
  reads accurately allocated to individual TE loci, rather than collapsed at subfamily
  level.
when_to_use_negative:
- TEs longer than the 2001 bp coverage window (e.g. long autonomously expressed L1HS)
  — flanking coverage is incomplete and alpha_i estimates unreliable
- Nearly identical TE copies with no uniquely mappable flanking context (e.g. HERVK
  isoforms) — the model cannot distinguish loci
- Input is already a gene expression or feature count table with no alignment-level
  BAM data to derive coverage vectors from
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MATES
  role: 'Deep learning framework: AutoEncoder plus MLP regressor for estimating locus-specific
    multi-mapping rates'
  repo: https://github.com/mcgilldinglab/MATES
- name: bam_processor
  role: Partitions STAR-aligned BAMs into unique/multi-mapping read sets and constructs
    TE-specific coverage vectors
  repo: https://github.com/mcgilldinglab/MATES
- name: data_processor
  role: Builds Vu/Vm coverage vectors and M/U bin classifications feeding model training
  repo: https://github.com/mcgilldinglab/MATES
- name: MATES_model
  role: Trains the AutoEncoder on Vu with TE family one-hot encoding and the MLP regressor
    for alpha_i
  repo: https://github.com/mcgilldinglab/MATES
- name: TE_quantifier
  role: Applies predicted alpha_i to compute locus-level counts URi + ceil(alpha_i
    x MRi)
  repo: https://github.com/mcgilldinglab/MATES
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:dae0ba6244327baaa496b1033f39a4e7cf8850c8d8221925c3d120e765eb6ee7
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# autoencoder-based-multimapping-rate-regression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MATES's deep learning core: an AutoEncoder trained on unique-read coverage vectors (Vu, with TE family one-hot encoding) produces latent embeddings that an MLP regressor uses to estimate locus-specific multi-mapping rates (alpha_i), enabling locus-specific TE quantification as URi + ceil(alpha_i x MRi) per cell.

## When to use

Use when Smart-seq2 or 10X single-cell RNA-seq/ATAC data must have multi-mapping reads accurately allocated to individual TE loci, rather than collapsed at subfamily level.

## When NOT to use

- TEs longer than the 2001 bp coverage window (e.g. long autonomously expressed L1HS) — flanking coverage is incomplete and alpha_i estimates unreliable
- Nearly identical TE copies with no uniquely mappable flanking context (e.g. HERVK isoforms) — the model cannot distinguish loci
- Input is already a gene expression or feature count table with no alignment-level BAM data to derive coverage vectors from

## Inputs

- STAR-aligned single-cell BAM files (Smart-seq2 one BAM per cell, or 10X-format BAMs with barcodes)
- TE reference (RepeatMasker-derived BED, e.g. TE_nooverlap.bed or TE_full.bed)
- Unique-read coverage vectors Vu (2001 bp windows around TE starts)
- Multi-read coverage vectors Vm and M/U bin classifications (bin size W, threshold P)

## Outputs

- Locus-specific multi-mapping rates alpha_i per TE locus per cell
- Locus-level TE count matrix (counts = URi + ceil(alpha_i x MRi))
- Adjusted multi-mapping coverage (optionally exported as BigWig)

## How to apply

After STAR alignment, segregate unique- and multi-mapping reads with MATES.bam_processor, then use MATES.data_processor to build 2001 bp coverage vectors (1000 bp up- and downstream of each TE start). Classify bins as multi-dominant (M) or unique-dominant (U) using bin size W and threshold P; the M/U bin labels provide supervision for the multi-mapping rate. Train the MATES_model AutoEncoder on unique-read coverage vectors Vu with TE family one-hot encoding (T_k) to obtain latent embeddings Z_m, cap training at 2500 samples per TE family, and drop families with fewer than 50 samples. The MLP regressor then predicts the locus-specific multi-mapping ratio alpha_i, and MATES.TE_quantifier computes per-cell locus counts as URi + ceil(alpha_i x MRi). Rationale: flanking unique-read context is informative about which locus multi-mapping reads truly originate from, and the model is trained with a global loss combining reconstruction and read-coverage continuity terms.

## Related tools

- **MATES** (Deep learning framework: AutoEncoder plus MLP regressor for estimating locus-specific multi-mapping rates) — https://github.com/mcgilldinglab/MATES
- **bam_processor** (Partitions STAR-aligned BAMs into unique/multi-mapping read sets and constructs TE-specific coverage vectors) — https://github.com/mcgilldinglab/MATES
- **data_processor** (Builds Vu/Vm coverage vectors and M/U bin classifications feeding model training) — https://github.com/mcgilldinglab/MATES
- **MATES_model** (Trains the AutoEncoder on Vu with TE family one-hot encoding and the MLP regressor for alpha_i) — https://github.com/mcgilldinglab/MATES
- **TE_quantifier** (Applies predicted alpha_i to compute locus-level counts URi + ceil(alpha_i x MRi)) — https://github.com/mcgilldinglab/MATES

## Examples

```
from MATES import MATES_pipeline; mates = MATES_pipeline(TE_mode, data_mode, sample_list_file, bam_path_file); mates.preprocessing(); mates.run()
```

## Evaluation signals

- Clustering cells on the resulting TE-only matrix against the gene-expression matrix should recover median ARI ~0.105 and median NMI ~0.161 on the Smart-seq2 glioblastoma benchmark
- Locus-level clustering should significantly outperform subfamily-level clustering (ARI improvement, P = 5.48e-7, Fig. 3j)
- Training set integrity: no TE family exceeds 2500 training samples and families with fewer than 50 samples are dropped
- Predicted counts are integers equal to URi plus a rounded-up fraction of multi-mapping reads MRi (URi + ceil(alpha_i x MRi))
- Marker TE associations (e.g. HUERS-P1-int/HERVK-int with neoplastic markers EGFR, LTR2B near CD166) are recovered in the output table

## Limitations

- MATES requires more computational resources (GPUs) than scTE and SoloTE
- Multi-mapping rate estimation is inaccurate for TEs longer than 2001 bp because flanking coverage is incomplete
- Identical TE copies such as HERVK isoforms cannot be distinguished due to high sequence similarity and lack of uniquely mappable flanking regions
- Trimming TE-gene overlaps (exclusive mode) may unintentionally distort TE quantification and no quantitative guidance on opting out is provided

## Evidence

- [discussion] AutoEncoder training on unique-read vectors: "we employ coverage vectors constructed from unique mapping reads Vu to train the AutoEncoder(AE) model"
- [discussion] Count formula: "The total read number of a given cell at TE locus i is then calculated by URi + ⌈αi × MRi⌉"
- [discussion] Coverage vector definition: "we define the coverage vector to span 2001 bp, extending 1000 bp both upstream and downstream from the TE start position"
- [discussion] Training set caps: "we limited training samples for each TE family not exceeding 2500 inputs"
- [readme] AutoEncoder embeddings with TE family info: [evidence span withheld — anchor status: not_found]
- [results] Benchmark statistics: [evidence span withheld — anchor status: not_found]
