---
name: scatac-seq-peak-plus-te-feature-construction
description: Use when analyzing 10x scATAC-seq (or multi-omics) data and you want
  clustering/benchmarking feature matrices that combine accessible-chromatin peaks
  with locus-specific TE counts from MATES, alongside TE-only and multi-mapping-TE-only
  perspectives.
when_to_use_negative:
- Input is already a processed cell-by-feature count matrix — this skill covers quantification
  from aligned BAMs, not downstream re-clustering of existing matrices
- Quantifying long autonomously expressed TEs such as L1HS spanning several thousand
  bases, where MATES' 2001 bp coverage vector gives inaccurate multi-mapping rates
- TE copies lacking uniquely mappable flanking regions (e.g., highly identical HERVK
  isoforms), where locus-specific allocation is unreliable
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3168
tools:
- name: MATES
  role: Quantifies locus-specific TE counts in scATAC-seq data by allocating multi-mapping
    reads using flanking unique-read context; provides bam_processor and data_processor
    modules for 10X-format data
  repo: https://github.com/mcgilldinglab/MATES
- name: scTE
  role: Baseline single-cell TE quantification method benchmarked against MATES on
    Peak+TE matrices
- name: SoloTE
  role: Baseline single-cell TE quantification method benchmarked against MATES on
    Peak+TE matrices
- name: bedtools
  role: Optionally trims the TE reference against the gene/peak reference to remove
    overlaps (exclusive TE_mode)
- name: STARsolo
  role: Maps raw scATAC-seq reads and segregates unique- vs multi-mapping alignments
    feeding into MATES
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:a42887ad8353af0e4c0db88f8964d69e1fe68829821c20941f338496fa0eb961
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
  - STARsolo
  - SoloTE
  - bedtools
  - scTE
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# scatac-seq-peak-plus-te-feature-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds combined Peak+TE cell-by-feature matrices for 10x scATAC-seq data, where MATES-quantified locus-specific transposable element counts are appended to standard ATAC peak counts so downstream clustering can leverage both open-chromatin peaks and TE loci.

## When to use

Use when analyzing 10x scATAC-seq (or multi-omics) data and you want clustering/benchmarking feature matrices that combine accessible-chromatin peaks with locus-specific TE counts from MATES, alongside TE-only and multi-mapping-TE-only perspectives.

## When NOT to use

- Input is already a processed cell-by-feature count matrix — this skill covers quantification from aligned BAMs, not downstream re-clustering of existing matrices
- Quantifying long autonomously expressed TEs such as L1HS spanning several thousand bases, where MATES' 2001 bp coverage vector gives inaccurate multi-mapping rates
- TE copies lacking uniquely mappable flanking regions (e.g., highly identical HERVK isoforms), where locus-specific allocation is unreliable

## Inputs

- BAM files from STAR/STARsolo alignment of 10x scATAC-seq reads (unique- and multi-mapping reads)
- 10x barcode and sample list files (sample_list_file, bam_path_file, bc_path_file)
- TE reference bed file (RepeatMasker-derived; default 'TE_full.bed' for inclusive / 'TE_nooverlap.bed' for exclusive TE_mode)
- ATAC peak reference / peak count matrix
- Known cell-type labels for ARI/NMI evaluation

## Outputs

- Cell-by-(Peak+TE) combined count matrix for the scATAC-seq dataset
- Cell-by-TE-locus and multi-mapping-TE-only count matrices from the same MATES run
- Cell cluster assignments (Louvain/Leiden) with ARI/NMI scores against known labels over multiple seeds
- BigWig coverage files with multi-mapping reads adjusted by αi per base pair

## How to apply

Map raw 10x scATAC-seq reads (e.g., adult mouse brain) with STARsolo and segregate unique- vs multi-mapping reads. Run MATES, which allocates multi-mapping reads to specific TE loci using a 2001 bp coverage vector centered on each TE start position, trained via an AutoEncoder on unique-read coverage vectors (Vu) with an MLP regressor estimating the locus-specific multi-mapping rate αi, giving total counts URi + ceil(αi × MRi). Build the feature matrix under the Peak+TE perspective by concatenating standard ATAC peak counts with MATES TE-locus counts (parallel TE-only and multi-mapping-TE-only matrices can be built from the same run). Choose 'exclusive' or 'inclusive' TE_mode in MATES to remove or retain TE instances overlapping genes/peaks; optionally trim the TE reference against the gene reference with bedtools. Cluster cells (e.g., Louvain/Leiden in Scanpy) and score against known labels with ARI/NMI over N=10 random seeds, comparing methods with one-sided Student's t-tests.

## Related tools

- **MATES** (Quantifies locus-specific TE counts in scATAC-seq data by allocating multi-mapping reads using flanking unique-read context; provides bam_processor and data_processor modules for 10X-format data) — https://github.com/mcgilldinglab/MATES
- **scTE** (Baseline single-cell TE quantification method benchmarked against MATES on Peak+TE matrices)
- **SoloTE** (Baseline single-cell TE quantification method benchmarked against MATES on Peak+TE matrices)
- **bedtools** (Optionally trims the TE reference against the gene/peak reference to remove overlaps (exclusive TE_mode))
- **STARsolo** (Maps raw scATAC-seq reads and segregates unique- vs multi-mapping alignments feeding into MATES)

## Examples

```
from MATES import bam_processor; bam_processor.split_count_10X_data('inclusive', sample_list_file, bam_path_file, bc_path_file, bc_ind='CB')
```

## Evaluation signals

- MATES should rank first across the three perspectives (Gene/Peak+TE, TE-only, multi-mapping-TE-only) in ARI/NMI clustering benchmarks as in Fig. 6
- Locus-level MATES vs SoloTE comparison should show ~10.52% higher ARI for MATES with one-sided t-test P ≈ 2.60e-12
- ARI/NMI distributions should be computed over N=10 random seeds to confirm stability of clustering gains
- bam_processor output should split each 10X BAM into per-cell sub-BAMs with unique vs multi-mapping reads segregated and TE-specific coverage vectors spanning 2001 bp around TE starts
- Total TE counts per locus should equal URi + ceil(αi × MRi) with αi in [0, 1] from the MLP regressor

## Limitations

- MATES requires more computational resources (GPUs) than scTE and SoloTE for quantifying TEs
- Multi-mapping rate estimation may be inaccurate for TEs longer than 2001 bp due to incomplete flanking coverage
- Trimming TE-gene overlaps (exclusive mode) may unintentionally distort TE quantification, and no quantitative guidance exists for when to opt out
- Velocyto does not identify which specific reads are unspliced, requiring custom source-adapted code for the intronic mode
- Total runtime including preprocessing and mapping increases linearly with data size

## Evidence

- [readme] Exclusive TE_mode semantics: [evidence span withheld — anchor status: external_doc]
- [readme] scATAC downstream analysis tutorial: [evidence span withheld — anchor status: external_doc]
- [other] Benchmarking workflow: [evidence span withheld — anchor status: not_found]
