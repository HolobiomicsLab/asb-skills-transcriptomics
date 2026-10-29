---
name: single-cell-transposable-element-quantification
description: Use when single-cell scRNA-seq, scATAC-seq, multi-omics, Visium, or long-read
  data need locus-specific TE count matrices, especially when multi-mapping reads
  from repetitive TE sequences must be resolved beyond subfamily-level or random/-map
  assignment.
when_to_use_negative:
- The target is long autonomously expressed TEs such as L1HS spanning several thousand
  bases, which exceed the 2001 bp coverage vector and yield inaccurate multi-mapping
  rate estimates
- The TEs of interest are nearly identical copies (e.g., HERVK isoforms) lacking uniquely
  mappable flanking regions, so loci cannot be distinguished
- Only bulk TE abundance is needed — bulk-level methods such as Squire, TEtranscripts,
  or Telescope are the appropriate choice
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: MATES
  role: Primary deep learning model (AutoEncoder + MLP) that builds 2001 bp coverage
    vectors and allocates multi-mapping reads to specific TE loci per cell
  repo: https://github.com/mcgilldinglab/MATES
- name: scTE
  role: Benchmark single-cell TE quantification baseline compared against MATES in
    clustering ARI/NMI tests
- name: SoloTE
  role: Benchmark single-cell TE quantification baseline compared against MATES; in
    locus-level comparisons MATES achieved ~10.52% higher ARI with one-sided t-test
    P ≈ 2.60e-12
- name: STAR/STARsolo
  role: Aligns raw reads and produces BAM files segregated into unique- and multi-mapping
    reads for MATES input
- name: bedtools
  role: Trims the TE reference against the gene reference in 'exclusive' TE_mode to
    remove TE-gene overlaps
- name: samtools
  role: BAM processing dependency installed alongside MATES
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:e653a898305fbf270e5bfec10e75f2deb9cc13ade418c416eb16bf514e963776
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
  - BEDTools
  - MATES
  - Pandas
  - Pysam
  - STAR/STARsolo
  - Samtools
  - SoloTE
  - bedtools
  - samtools
  - scTE
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# single-cell-transposable-element-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Locus-specific quantification of transposable elements (TEs) in single-cell sequencing data using MATES, a deep learning model that allocates multi-mapping reads to specific TE loci using coverage vectors of flanking unique-read context, outperforming scTE and SoloTE in downstream cell clustering (ARI/NMI).

## When to use

Use when single-cell scRNA-seq, scATAC-seq, multi-omics, Visium, or long-read data need locus-specific TE count matrices, especially when multi-mapping reads from repetitive TE sequences must be resolved beyond subfamily-level or random/best-map assignment.

## When NOT to use

- The target is long autonomously expressed TEs such as L1HS spanning several thousand bases, which exceed the 2001 bp coverage vector and yield inaccurate multi-mapping rate estimates
- The TEs of interest are nearly identical copies (e.g., HERVK isoforms) lacking uniquely mappable flanking regions, so loci cannot be distinguished
- Only bulk TE abundance is needed — bulk-level methods such as Squire, TEtranscripts, or Telescope are the appropriate choice

## Inputs

- BAM files of aligned single-cell reads (10X format with multiple cells per BAM, or Smart-seq format with one BAM per cell)
- Cell barcode files (e.g., CB/CR barcode fields in BAM)
- TE reference (RepeatMasker-based BED, e.g., TE_nooverlap.bed or TE_full.bed) and gene reference (GTF) for Human/Mouse or custom species
- sample_list_file and bam_path_file text files mapping sample IDs to BAM paths

## Outputs

- Cell-by-TE-locus count matrices with multi-mapping reads probabilistically allocated (URi + ⌈αi × MRi⌉)
- Subfamily-level and locus-level TE quantification tables
- BigWig coverage files with multi-mapping read coverage adjusted per base pair by αi
- Downstream clustering metrics (ARI/NMI) and marker TE lists (e.g., MT2, MERVL-int, ATCopia66LTR, GYPSY12_LTR)

## How to apply

Map raw reads with STAR/STARsolo and segregate unique- vs multi-mapping reads; build unique (Vu) and multi-mapping (Vm) coverage vectors spanning 2001 bp centered on each TE start position (1000 bp upstream/downstream). Classify bins as multi-dominant (M) or unique-dominant (U) using bin size W and threshold P, train an AutoEncoder on unique-read coverage vectors with TE family one-hot encoding (downsampling to at most 2500 training samples per family, dropping families with fewer than 50 samples, truncating TEs longer than 1000 bp to the first 1000 bp), then estimate the locus-specific multi-mapping rate αi with an MLP regressor and compute total counts as URi + ⌈αi × MRi⌉. Run in simple mode with MATES_pipeline or stepwise via the bam_processor, data_processor, MATES_model, and TE_quantifier modules; choose 'exclusive' or 'inclusive' TE_mode depending on whether TEs overlapping genes are trimmed with bedtools. Downstream, build cell-by-TE matrices (TE-only, Gene/Peak+TE, multi-mapping-TE-only), cluster with Louvain/Leiden in Scanpy, and score against reference labels with ARI/NMI.

## Related tools

- **MATES** (Primary deep learning model (AutoEncoder + MLP) that builds 2001 bp coverage vectors and allocates multi-mapping reads to specific TE loci per cell) — https://github.com/mcgilldinglab/MATES
- **scTE** (Benchmark single-cell TE quantification baseline compared against MATES in clustering ARI/NMI tests)
- **SoloTE** (Benchmark single-cell TE quantification baseline compared against MATES; in locus-level comparisons MATES achieved ~10.52% higher ARI with one-sided t-test P ≈ 2.60e-12)
- **STAR/STARsolo** (Aligns raw reads and produces BAM files segregated into unique- and multi-mapping reads for MATES input)
- **bedtools** (Trims the TE reference against the gene reference in 'exclusive' TE_mode to remove TE-gene overlaps)
- **samtools** (BAM processing dependency installed alongside MATES)

## Examples

```
from MATES import MATES_pipeline; mates = MATES_pipeline(TE_mode, data_mode, sample_list_file, bam_path_file); mates.preprocessing(); mates.run()
```

## Evaluation signals

- Reproduced ARI values match reported figures within tolerance: MATES Arabidopsis TE-only 0.3514 / Gene+TE 0.6668, SoloTE 0.3200 / 0.5916, scTE 0.3315 / 0.5775; Drosophila MATES 0.3188 vs scTE 0.3078
- In mammalian benchmarks (TE-only and multi-mapping-TE-only perspectives), MATES ranks first across Gene/Peak+TE, TE-only, and multi-mapping-TE-only comparisons as in Fig. 6a, b, with one-sided Student's t-tests over N=10 random seeds
- Marker TEs recovered include MT2 and MERVL-int enriched flanking Zscan4c/Zscan4d in 2-cell-like cells, ATCopia66LTR and ATCopia41LTR in Arabidopsis, and GYPSY12_LTR in Drosophila
- Verification against long-read ground truth (scNanoRNAseq melanoma brain metastasis nanopore data and PacBio postnatal mouse brain data) shows more accurate allocation than alignment-based baselines
- Installation sanity check: `import MATES` succeeds in the conda environment with python=3.9, samtools, and bedtools installed

## Limitations

- MATES requires more computational resources such as GPUs than scTE and SoloTE for quantifying TEs
- Limited ability to quantify long autonomously expressed TEs like L1HS spanning several thousand bases; multi-mapping rate estimation may be inaccurate for TEs longer than 2001 bp due to incomplete flanking coverage
- Identical TE copies such as HERVK isoforms are hard to distinguish due to high sequence similarity and lack of uniquely mappable flanking regions; adjusting parameters like coverage vector length is not a definitive solution
- Trimming TE-gene overlaps ('exclusive' mode) could unintentionally distort TE quantification where TEs intersect neighboring genes, and no quantitative guidance on when to opt out is provided
- Intronic mode requires custom code adapted from Velocyto (version 0.17) because Velocyto does not identify which specific reads are unspliced

## Evidence

- [discussion] MLP multi-mapping rate and count formula: [evidence span withheld — anchor status: not_found]
- [readme] Simple-mode API: [evidence span withheld — anchor status: external_doc]
- [readme] bam_processor role: [evidence span withheld — anchor status: external_doc]
