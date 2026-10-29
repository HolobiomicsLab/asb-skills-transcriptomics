---
name: multi-mapping-read-segregation-from-bam
description: Use when you have single-cell BAM files with multi-mapping reads to repetitive
  TE loci and need per-cell, per-locus coverage vectors and unique/multi read segregation
  before training a MATES model.
when_to_use_negative:
- Input is already a feature-count matrix or h5ad gene expression table — segregation
  requires alignment-level BAM input
- Quantifying long autonomously expressed TEs such as L1HS spanning several thousand
  bases — the 2001-bp window centered on the TE start yields incomplete flanking coverage
- TE loci (e.g., HERVK isoforms) with high sequence similarity and no uniquely mappable
  flanking regions — flanking context carries no signal for segregation
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: MATES
  role: Provides the bam_processor (and data_processor) modules used to split BAMs
    per cell, segregate unique vs multi-mapping reads, count coverage vectors, and
    determine U/M bins
  repo: https://github.com/mcgilldinglab/MATES
- name: STAR/STARsolo
  role: Upstream mapper whose output BAMs supply the unique vs multi-mapping alignment
    tags that bam_processor segregates on
- name: samtools
  role: Dependency installed alongside MATES for BAM manipulation in the environment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:ada73e7ba596b5570e772657962313182771b5f85096020599f40acdb04cfd5e
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
  - samtools
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# multi-mapping-read-segregation-from-bam

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Segregate reads from a sorted single-cell BAM (mapped by STAR/STARsolo) into unique-mapping and multi-mapping sets per cell, then build per-locus 2001-bp coverage vectors (Vu for unique reads, Vm for multi-mapping reads) centered on TE start positions, binning them as U-bin or M-bin — the preprocessing substrate for MATES locus-specific TE quantification.

## When to use

Use when you have single-cell BAM files with multi-mapping reads to repetitive TE loci and need per-cell, per-locus coverage vectors and unique/multi read segregation before training a MATES model.

## When NOT to use

- Input is already a feature-count matrix or h5ad gene expression table — segregation requires alignment-level BAM input
- Quantifying long autonomously expressed TEs such as L1HS spanning several thousand bases — the 2001-bp window centered on the TE start yields incomplete flanking coverage
- TE loci (e.g., HERVK isoforms) with high sequence similarity and no uniquely mappable flanking regions — flanking context carries no signal for segregation

## Inputs

- Sorted single-cell BAM file(s) mapped with STAR/STARsolo (10X multi-cell BAM or Smart-seq per-cell BAM)
- Cell barcode list files (bc_path_file) with barcode field indicator such as CB
- TE reference BED/CSV file (TE_nooverlap.bed / TE_full.bed, or a custom ref_path)
- Sample list file mapping sample IDs to BAM paths

## Outputs

- Per-cell sub-BAM files split into unique-mapping and multi-mapping read sets
- Per-locus coverage vectors Vu (unique reads) and Vm (multi-mapping reads) spanning 2001 bp around each TE start
- U-bin/M-bin labels per W-bp bin used for the continuity loss in MATES training

## How to apply

Load the sorted single-cell BAM with the MATES bam_processor module and split it into per-cell sub-BAMs, segregating reads into unique-mapping (single-locus alignments) and multi-mapping sets produced by STAR/STARsolo. For each TE locus, construct a coverage vector spanning 2001 bp (1000 bp upstream and downstream of the TE start); truncate TE loci longer than 1000 bp to their initial 1000 bp. Slice each vector into W-bp bins and label each bin as an M-bin if the percentage of multi-mapping reads exceeds threshold P, otherwise as a U-bin; these labels feed the downstream continuity loss (L2) of the MATES MLP regressor. Use data_mode '10X' for multi-cell BAMs (prefer split_count_10X_data) and 'Smart_seq' for single-cell-per-BAM data, selecting TE_mode 'exclusive' or 'inclusive' to control whether TE instances overlapping genes are removed.

## Related tools

- **MATES** (Provides the bam_processor (and data_processor) modules used to split BAMs per cell, segregate unique vs multi-mapping reads, count coverage vectors, and determine U/M bins) — https://github.com/mcgilldinglab/MATES
- **STAR/STARsolo** (Upstream mapper whose output BAMs supply the unique vs multi-mapping alignment tags that bam_processor segregates on)
- **samtools** (Dependency installed alongside MATES for BAM manipulation in the environment)

## Examples

```
from MATES import bam_processor; bam_processor.split_count_10X_data(TE_mode, sample_list_file, bam_path_file, bc_path_file, bc_ind='CB', num_threads=1)
```

## Evaluation signals

- Unique-read count plus multi-read count per cell equals the total reads originally present in the per-cell sub-BAM (no reads lost during split_count_10X_data)
- Every coverage vector spans exactly 2001 bp (or 1000 bp for TE loci longer than 1000 bp after truncation to the initial 1000 bp)
- Each bin is labeled M-bin only when the multi-mapping read percentage exceeds threshold P; verify on a spot-check locus
- Downstream invariant: spot-check loci in te_locus_counts.tsv satisfy count = UR + ceil(alpha × MR) using the stored UR, alpha, and MR values
- import MATES succeeds in the conda environment and the per-cell sub-BAM files exist for every barcode in bc_path_file

## Limitations

- MATES requires more computational resources such as GPUs than scTE and SoloTE for quantifying TEs
- Multi-mapping rate estimation may be inaccurate for TEs longer than 2001 bp due to incomplete coverage of flanking unique-read context
- Identical TE copies such as HERVK isoforms are hard to distinguish due to high sequence similarity and lack of uniquely mappable flanking regions

## Evidence

- [readme] The bam_processor module efficiently manages input BAM files by partitioning them into sub-BAM files for individual cells, distinguishing unique mapping from multi mapping reads.: [evidence span withheld — anchor status: external_doc]
