---
name: multi-mapping-read-allocation-to-te-loci
description: Use when single-cell RNA-seq or scATAC-seq alignments contain multi-mapping
  reads over repetitive TE loci and locus-specific (not subfamily-level) TE quantification
  or marker discovery is the goal.
when_to_use_negative:
- The TE of interest is a long autonomously expressed element such as L1HS spanning
  several thousand bases, where flanking unique-read coverage within the 2001 bp window
  is incomplete
- The TE copies are near-identical (e.g., HERVK isoforms) lacking uniquely mappable
  flanking regions, so αi estimation cannot distinguish loci
- Quantification at the subfamily level is sufficient and GPU resources are constrained,
  since MATES requires more compute (GPUs) than scTE or SoloTE
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0622
tools:
- name: MATES
  role: Deep-learning framework (bam_processor, data_processor, MATES_model, TE_quantifier
    modules) that segregates reads, trains the AutoEncoder/MLP, and computes locus-specific
    TE counts
  repo: https://github.com/mcgilldinglab/MATES
- name: bam_processor
  role: Partitions BAMs into per-cell unique/multi-mapping sub-BAMs and constructs
    TE-specific Vu/Vm coverage vectors
  repo: https://github.com/mcgilldinglab/MATES
- name: TE_quantifier
  role: MLP regressor estimating locus-specific multi-mapping rate αi and quantifying
    per-cell TE counts
  repo: https://github.com/mcgilldinglab/MATES
- name: bedtools
  role: Optionally trims the TE reference against the gene reference to remove overlaps
    (exclusive TE_mode)
- name: samtools
  role: BAM manipulation dependency installed in the MATES conda environment
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:a07ab2f4d05c53849a693ae1d0bb8ba4c91eca723f4f1906374ea3823f077c27
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
  - bedtools
  - data_processor
  - samtools
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# multi-mapping-read-allocation-to-te-loci

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MATES-based allocation of multi-mapping reads to specific transposable element loci in single-cell data, using unique-read coverage context flanking each TE start to train an AutoEncoder + MLP that estimates a locus-specific multi-mapping rate αi, yielding locus-level TE counts URi + ⌈αi × MRi⌉ per cell.

## When to use

Use when single-cell RNA-seq or scATAC-seq alignments contain multi-mapping reads over repetitive TE loci and locus-specific (not subfamily-level) TE quantification or marker discovery is the goal.

## When NOT to use

- The TE of interest is a long autonomously expressed element such as L1HS spanning several thousand bases, where flanking unique-read coverage within the 2001 bp window is incomplete
- The TE copies are near-identical (e.g., HERVK isoforms) lacking uniquely mappable flanking regions, so αi estimation cannot distinguish loci
- Quantification at the subfamily level is sufficient and GPU resources are constrained, since MATES requires more compute (GPUs) than scTE or SoloTE

## Inputs

- STAR/STARsolo-aligned BAM files of single-cell data (10X multi-cell BAMs or Smart-seq2 per-cell BAMs)
- Cell barcode lists with sample-to-BAM/barcode-path mapping files (10X format, bc_ind e.g. CB)
- TE reference bed/CSV ('TE_nooverlap.bed'/'TE_full.bed' for exclusive/inclusive modes) and gene reference for optional overlap trimming
- RepeatMasker TE annotation (e.g., custom reference built with build_reference.py for non-human/mouse species)

## Outputs

- Locus-specific per-cell TE count matrix (URi + ⌈αi × MRi⌉) with TE loci keyed as chromosome|start|end (e.g., chr3|104522003|104522491)
- Estimated locus-specific multi-mapping rates αi and adjusted multi-mapping coverage BigWig files
- Subfamily-level TE counts and downstream clustering metrics (median ARI/NMI) and cell-type-specific TE markers

## How to apply

Map raw reads with STAR/STARsolo, then use MATES.bam_processor to segregate unique- and multi-mapping reads and build unique (Vu) and multi (Vm) coverage vectors spanning 2001 bp (1000 bp up- and downstream of each TE start). Classify bins as multi-dominant (M) or unique-dominant (U) using bin size W and threshold P, downsample training samples to at most 2500 per TE family, drop families with fewer than 50 samples, and truncate TE loci longer than 1000 bp to the first 1000 bp. Train the MATES_model AutoEncoder on Vu vectors with TE family one-hot encoding, then use the TE_quantifier MLP regressor to estimate the locus-specific multi-mapping rate αi and compute per-cell counts as URi + ⌈αi × MRi⌉. Optionally run in exclusive or inclusive TE_mode depending on whether TE instances overlapping genes are removed (via bedtools trimming). Validate by clustering cells on the locus-level TE matrix versus gene/peak matrices and reporting median ARI/NMI (e.g., glioblastoma Smart-seq2: ARI 0.105, NMI 0.161; mouse brain scATAC: ARI 0.309, NMI 0.438) and the locus- vs subfamily-level ARI improvement.

## Related tools

- **MATES** (Deep-learning framework (bam_processor, data_processor, MATES_model, TE_quantifier modules) that segregates reads, trains the AutoEncoder/MLP, and computes locus-specific TE counts) — https://github.com/mcgilldinglab/MATES
- **bam_processor** (Partitions BAMs into per-cell unique/multi-mapping sub-BAMs and constructs TE-specific Vu/Vm coverage vectors) — https://github.com/mcgilldinglab/MATES
- **TE_quantifier** (MLP regressor estimating locus-specific multi-mapping rate αi and quantifying per-cell TE counts) — https://github.com/mcgilldinglab/MATES
- **bedtools** (Optionally trims the TE reference against the gene reference to remove overlaps (exclusive TE_mode))
- **samtools** (BAM manipulation dependency installed in the MATES conda environment)

## Examples

```
from MATES import MATES_pipeline; mates = MATES_pipeline(TE_mode, data_mode, sample_list_file, bam_path_file); mates.preprocessing(); mates.run()
```

## Evaluation signals

- Median ARI/NMI between TE-only and gene-expression-based cell clustering match published values (Smart-seq2 glioblastoma: ARI 0.105, NMI 0.161; 10x mouse brain scATAC: ARI 0.309, NMI 0.438)
- Locus-level TE clustering significantly outperforms subfamily-level clustering (glioblastoma P = 5.48e-7, Fig. 3j; mouse brain scATAC P = 2.55e-34, Fig. 4i)
- Known cell-type TE markers recovered: neoplastic (EGFR, HUERS-P1-int, HERVK-int) and immune (CD74, LTR2B, LTR40A1) in glioblastoma; RMER16_Mm/RLTR44B in macrophages, MamRep434/MER124 in astrocytes, MURVY-LTR/MamRep1527 in oligodendrocytes in mouse brain
- Training-set filters enforced: ≤2500 samples per TE family, families with <50 samples dropped, loci >1000 bp truncated to first 1000 bp
- Import of MATES modules succeeds and version supports data without sufficient multi-mapping TE reads (v0.1.8+)

## Limitations

- MATES requires more computational resources such as GPUs than scTE and SoloTE
- Limited ability to quantify long autonomously expressed TEs like L1HS spanning several thousand bases; parameter adjustments (e.g., coverage vector length) are not definitive solutions
- Identical TE copies (e.g., HERVK isoforms) with high sequence similarity and no uniquely mappable flanking regions are hard to distinguish, and αi may be inaccurate for TEs longer than 2001 bp due to incomplete flanking coverage
- Trimming TE-gene overlaps may unintentionally distort TE quantification, with no quantitative evaluation of when to opt out

## Evidence

- [readme] README module description: [evidence span withheld — anchor status: external_doc]
- [other] Benchmark statistic: [evidence span withheld — anchor status: not_found]
