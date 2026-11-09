---
name: 2clc-population-marker-identification
description: Use when analyzing 10x mouse chemical reprogramming scRNA-seq to find
  a 2CLC population and its defining TE markers using locus-specific TE quantification
  with MATES.
when_to_use_negative:
- The goal is quantification of long autonomously expressed TEs such as L1HS that
  span several thousand bases, exceeding MATES' 2001 bp coverage-vector design
- TE copies are near-identical with no uniquely mappable flanking regions (e.g., HERVK
  isoforms), where locus-specific multi-mapping assignment cannot be resolved
- Input is bulk RNA-seq or an existing gene-expression feature matrix and the question
  does not require locus-specific TE quantification at single-cell level
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MATES
  role: Deep-learning pipeline for locus-specific TE quantification that allocates
    multi-mapping reads using flanking unique-read coverage context
  repo: https://github.com/mcgilldinglab/MATES
- name: bam_processor
  role: Partitions 10X BAM files per cell, segregates unique vs multi-mapping reads,
    and builds TE-specific coverage vectors
  repo: https://github.com/mcgilldinglab/MATES
- name: data_processor
  role: Builds 2001 bp coverage vectors and classifies bins as M or U using bin size
    W and threshold P
  repo: https://github.com/mcgilldinglab/MATES
- name: MATES_model
  role: AutoEncoder + MLP regressor that estimates locus-specific multi-mapping rates
    αi
  repo: https://github.com/mcgilldinglab/MATES
- name: TE_quantifier
  role: Computes per-locus total counts as URi + ceil(αi × MRi) for downstream clustering
  repo: https://github.com/mcgilldinglab/MATES
- name: TE_quantifier_Intronic
  role: Optional intronic mode correcting TE counts by subtracting Velocyto-identified
    unspliced reads (N′ = N − s)
  repo: https://github.com/mcgilldinglab/MATES
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-53114-7
    title: 'MATES: a deep learning-based model for locus-specific quantification of
      transposable elements in single cell'
  source_content_hash: sha256:d84444206267494c737cf44f3aa0bb1aeb77899394a4785865843e77deaa0749
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
  - TE_quantifier_Intronic
  - bam_processor
  - data_processor
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# 2clc-population-marker-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies the 2-cell-like cell (2CLC) subpopulation in mouse chemical reprogramming 10x scRNA-seq by clustering on MATES locus-specific transposable element counts and detecting the signature TE markers MT2_Mm and MERVL-int, which are enriched flanking the Zscan4c/Zscan4d loci.

## When to use

Use when analyzing 10x mouse chemical reprogramming scRNA-seq to find a 2CLC population and its defining TE markers using locus-specific TE quantification with MATES.

## When NOT to use

- The goal is quantification of long autonomously expressed TEs such as L1HS that span several thousand bases, exceeding MATES' 2001 bp coverage-vector design
- TE copies are near-identical with no uniquely mappable flanking regions (e.g., HERVK isoforms), where locus-specific multi-mapping assignment cannot be resolved
- Input is bulk RNA-seq or an existing gene-expression feature matrix and the question does not require locus-specific TE quantification at single-cell level

## Inputs

- Raw 10x mouse chemical reprogramming scRNA-seq reads (mapped to a reference genome with STARsolo)
- BAM files with cell barcodes (CB/CR fields) for 10X-format data
- Mouse TE reference (RepeatMasker-derived TE_full.bed / TE_nooverlap.bed) and gene reference GTF

## Outputs

- Locus-level TE count matrix (URi + ceil(αi × MRi)) per cell from MATES.TE_quantifier
- Cell clusters computed on the TE count matrix with the 2CLC population annotated by Zscan4c/Zscan4d positivity
- Binomial-test enrichment statistics and bar plots for MT2_Mm and MERVL-int reads flanking Zscan4c/Zscan4d loci
- Clustering-agreement metrics (adjusted Rand index and normalized mutual information) between TE-only and gene-based clusterings; expected medians ARI 0.397, NMI 0.496
- Box plot comparing locus-level vs subfamily-level ARI across clustering resolutions

## How to apply

Map raw 10x reads with STARsolo, then run MATES.bam_processor to segregate unique- and multi-mapping reads aligned to TE loci and build 2001 bp coverage vectors (1000 bp up-/downstream of each TE start). Classify bins as multi-dominant (M) or unique-dominant (U) with bin size W and multi-mapping threshold P, train the AutoEncoder on unique-read coverage vectors with TE-family one-hot encoding, and use the MATES_model MLP regressor to estimate locus-specific multi-mapping rates αi, computing per-locus counts as URi + ceil(αi × MRi) with MATES.TE_quantifier. Cluster cells (Louvain/Leiden) on the resulting TE count matrix, flag the 2CLC cluster as the one scoring positive for Zscan4c/Zscan4d, and test enrichment of MT2_Mm and MERVL-int reads flanking the Zscan4c/Zscan4d loci with a binomial test. Validate by confirming cluster co-expression of Zscan4c/Zscan4d, significant enrichment of the two markers, and agreement between TE-only and gene-based clusterings targeting reported medians ARI 0.397 and NMI 0.496 at locus-level resolution.

## Related tools

- **MATES** (Deep-learning pipeline for locus-specific TE quantification that allocates multi-mapping reads using flanking unique-read coverage context) — https://github.com/mcgilldinglab/MATES
- **bam_processor** (Partitions 10X BAM files per cell, segregates unique vs multi-mapping reads, and builds TE-specific coverage vectors) — https://github.com/mcgilldinglab/MATES
- **data_processor** (Builds 2001 bp coverage vectors and classifies bins as M or U using bin size W and threshold P) — https://github.com/mcgilldinglab/MATES
- **MATES_model** (AutoEncoder + MLP regressor that estimates locus-specific multi-mapping rates αi) — https://github.com/mcgilldinglab/MATES
- **TE_quantifier** (Computes per-locus total counts as URi + ceil(αi × MRi) for downstream clustering) — https://github.com/mcgilldinglab/MATES
- **TE_quantifier_Intronic** (Optional intronic mode correcting TE counts by subtracting Velocyto-identified unspliced reads (N′ = N − s)) — https://github.com/mcgilldinglab/MATES

## Examples

```
from MATES import bam_processor; bam_processor.split_count_10X_data('exclusive', sample_list_file, bam_path_file, bc_path_file, bc_ind='CB')
```

## Evaluation signals

- The identified 2CLC cluster co-expresses Zscan4c and Zscan4d
- Binomial test shows significant enrichment of MT2_Mm and MERVL-int reads flanking the Zscan4c/Zscan4d loci
- Locus-level clustering agreement with gene-based clustering reaches reported medians ARI 0.397 and NMI 0.496
- Locus-level ARI exceeds subfamily-level ARI in the Fig. 2h resolution-sweep box plot
- TE count matrix dimensions match the number of cells passing QC and TE loci in the reference, with per-locus counts ≥ unique read counts (URi)

## Limitations

- MATES requires more computational resources (GPUs) than scTE and SoloTE for quantifying TEs
- Multi-mapping rate estimation may be inaccurate for TEs longer than 2001 bp due to incomplete flanking coverage
- Identical TE copies (e.g., HERVK isoforms) cannot be distinguished due to high sequence similarity and lack of uniquely mappable flanking regions
- Trimming TE-gene overlaps in exclusive mode could unintentionally distort quantification of TEs intersecting neighboring genes

## Evidence

- [discussion] 2CLC markers identified via locus-level TE quantification: "markers such as MT2 and MERVL-int were enriched in the Zscan4 gene cluster, particularly flanking the Zscan4c and Zscan4d genes in 2CLC cells"
- [discussion] Count formula and flanking-context rationale: "The total read number of a given cell at TE locus i is then calculated by URi + ⌈αi × MRi⌉"
- [discussion] Coverage vector design: "we define the coverage vector to span 2001 bp, extending 1000 bp both upstream and downstream from the TE start position"
- [abstract] Improved quantification and marker discovery claim: "MATES shows improved performance over existing methods, enhancing the accuracy of TE quantification and aiding in the identification of marker TEs for identified cell populations"
- [readme] README API for 10X preprocessing: [evidence span withheld — anchor status: external_doc]
