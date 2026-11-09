---
name: te-locus-coverage-vector-construction
description: Use when you need per-locus flanking read-context features for TE quantification
  from STAR/STARsolo-mapped single-cell BAMs, e.g. to train MATES' AE/MLP or predict
  locus-specific multi-mapping ratios alpha.
when_to_use_negative:
- The TE of interest is a long autonomously expressed element such as L1HS spanning
  several thousand bases, since vectors cover only the start-centered 2001 bp window
  and the multi-mapping rate estimate becomes inaccurate due to incomplete flanking
  coverage
- TE loci whose copies are near-identical (e.g. HERVK isoforms) with no uniquely mappable
  flanking regions — coverage context cannot distinguish them
- You need TE quantification only at subfamily level from an existing feature count
  matrix; no per-locus read context is required
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: MATES
  role: Provides the bam_processor module that partitions single-cell BAMs into per-cell
    sub-BAMs, segregates unique vs multi-mapping reads, and builds TE-specific coverage
    vectors Vu/Vm with U/M bin labels
  repo: https://github.com/mcgilldinglab/MATES
- name: STAR/STARsolo
  role: Upstream aligner whose unique/multi-mapping read annotations are segregated
    by bam_processor before coverage vector construction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:1b4ea8ed05103f29e8b4911c66e47e6de59470ace11e94b1907e8e6f17339292
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# te-locus-coverage-vector-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs unique-read (Vu) and multi-mapping-read (Vm) coverage vectors spanning 2001 bp centered on each TE start position from single-cell BAM files, then bins them as U-bin/M-bin, providing the training and prediction inputs for MATES' AutoEncoder and MLP multi-mapping ratio model.

## When to use

Use when you need per-locus flanking read-context features for TE quantification from STAR/STARsolo-mapped single-cell BAMs, e.g. to train MATES' AE/MLP or predict locus-specific multi-mapping ratios alpha.

## When NOT to use

- The TE of interest is a long autonomously expressed element such as L1HS spanning several thousand bases, since vectors cover only the start-centered 2001 bp window and the multi-mapping rate estimate becomes inaccurate due to incomplete flanking coverage
- TE loci whose copies are near-identical (e.g. HERVK isoforms) with no uniquely mappable flanking regions — coverage context cannot distinguish them
- You need TE quantification only at subfamily level from an existing feature count matrix; no per-locus read context is required

## Inputs

- Sorted single-cell BAM file(s) mapped with STAR/STARsolo (10X-format or Smart-seq2 format)
- TE reference BED/CSV (default 'TE_full.bed'/'TE_full.csv' for inclusive mode, 'TE_nooverlap.bed'/'TE_nooverlap.csv' for exclusive mode)
- Barcode list files and sample list files (for 10X-format data)
- Parameters: bin size W, multi-dominant threshold P, TE_mode (exclusive/inclusive)

## Outputs

- Per-cell, per-TE-locus unique-read coverage vectors Vu
- Per-cell, per-TE-locus multi-mapping-read coverage vectors Vm
- U-bin/M-bin labels per locus (bin labels used for the continuity loss)
- Training samples for the MATES AutoEncoder (at most 2500 per TE family, families with <50 samples removed)

## How to apply

Run MATES' bam_processor on the sorted single-cell BAM (10X-format multi-cell BAMs or Smart_seq one-cell-per-BAM) to segregate unique- from multi-mapping reads after STAR/STARsolo alignment. For each TE locus, build coverage vectors Vu (unique reads) and Vm (multi-mapping reads) spanning 2001 bp centered on the TE start (1000 bp each side); truncate TE loci longer than 1000 bp to their initial 1000 bp because flanking context beyond that is not used. Slice each vector into W-bp bins and label a bin M-bin (multi-dominant) if the percentage of multi-mapping reads exceeds threshold P, otherwise U-bin; these labels later supervise the continuity loss. Inclusive/exclusive mode controls whether TE instances overlapping genes are retained ('TE_full' vs 'TE_nooverlap' references, optionally trimmed with bedtools). The resulting per-cell, per-locus coverage vectors are the AutoEncoder training inputs, downsampled to at most 2500 samples per TE family with families under 50 samples removed.

## Related tools

- **MATES** (Provides the bam_processor module that partitions single-cell BAMs into per-cell sub-BAMs, segregates unique vs multi-mapping reads, and builds TE-specific coverage vectors Vu/Vm with U/M bin labels) — https://github.com/mcgilldinglab/MATES
- **STAR/STARsolo** (Upstream aligner whose unique/multi-mapping read annotations are segregated by bam_processor before coverage vector construction)

## Examples

```
from MATES import bam_processor; bam_processor.count_coverage_vec(TE_mode, data_mode, threads_num, sample_list_file)
```

## Evaluation signals

- Coverage vectors span exactly 2001 bp centered on each TE start position (1000 bp upstream + 1000 bp downstream), with long TE loci truncated to the initial 1000 bp
- Every W-bp bin carries a U or M label consistent with the rule: M-bin iff multi-mapping read percentage exceeds threshold P
- Unique-mapping and multi-mapping read sets partition the cell's BAM-aligned reads as reported by STAR/STARsolo NH tags
- Training set per TE family contains at most 2500 samples and no TE family with fewer than 50 samples remains
- Downstream spot-check: locus counts in te_locus_counts.tsv satisfy count = UR + ceil(alpha x MR), confirming vectors fed the alpha predictor correctly

## Limitations

- For TEs longer than 2001 bp the estimated multi-mapping rate may not be very accurate due to incomplete flanking coverage; adjusting vector length is not a definitive fix
- Identical TE copies (e.g. HERVK isoforms) with high sequence similarity and no uniquely mappable flanking regions cannot be distinguished
- Trimming TE-gene overlaps (exclusive mode) could unintentionally distort TE quantification for TEs intersecting neighboring genes

## Evidence

- [readme] bam_processor segregates reads and builds coverage vectors: [evidence span withheld — anchor status: external_doc]
- [readme] bam_processor constructs TE-specific coverage vectors: [evidence span withheld — anchor status: external_doc]
