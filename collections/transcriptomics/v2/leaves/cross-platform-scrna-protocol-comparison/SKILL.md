---
name: cross-platform-scrna-protocol-comparison
description: Use when scRNA-seq FASTQ data from multiple UMI-based platforms (Chromium,
  Drop-seq, ICELL8, SmartSeq3) has been processed by both UniverSC and its vendor
  pipeline and you need to quantify concordance of the resulting GBMs.
when_to_use_negative:
- Input is 5' scRNA-seq data from a technology other than Chromium, which UniverSC
  does not support
- Data is not from a UMI-based single-cell platform, since UniverSC only supports
  UMI-based protocols
- You need to change data content rather than compare pipelines — the barcodes kept
  and genes padded are chosen only to make the pair comparable, not for biological
  inference
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: UniverSC
  role: Universal wrapper around Cell Ranger that processes FASTQs from any UMI-based
    platform with per-technology pre-set parameters (Table 1 of 40 technologies)
  repo: https://github.com/minoda-lab/universc
- name: 10x Genomics bamtofastq
  role: Converts Cell Ranger BAM output back to FASTQ so the same reads can be reprocessed
    by UniverSC
  repo: https://github.com/10XGenomics/bamtofastq
- name: Seurat
  role: Clustering analysis of each adjusted GBM pair (version 4.1.1 in R 4.1.2) prior
    to ARI computation
- name: clues
  role: R package (version 0.6.2.2) used to compute the Adjusted Rand Index between
    the two clustering outcomes per platform
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34681-z
    title: A flexible cross-platform single-cell data processing pipeline
  source_content_hash: sha256:64fc0b56ffc706b32b641cd804c1e98fa7515c2078e1743cd98d4a7bb756d78d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/minoda-lab/universc
  tools:
  - 10x Genomics bamtofastq
  - R
  - Seurat
  - UniverSC
  - clues
  - stats
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cross-platform-scrna-protocol-comparison

## Summary

Benchmark a universal scRNA-seq processor (UniverSC) against platform-specific pipelines (Cell Ranger, dropSeqPipe, CogentAP, zUMIs) by clustering matched gene-barcode matrices (GBMs) and scoring similarity with Adjusted Rand Index and Pearson correlation.

## When to use

Use when scRNA-seq FASTQ data from multiple UMI-based platforms (Chromium, Drop-seq, ICELL8, SmartSeq3) has been processed by both UniverSC and its vendor pipeline and you need to quantify concordance of the resulting GBMs.

## When NOT to use

- Input is 5' scRNA-seq data from a technology other than Chromium, which UniverSC does not support
- Data is not from a UMI-based single-cell platform, since UniverSC only supports UMI-based protocols
- You need to change data content rather than compare pipelines — the barcodes kept and genes padded are chosen only to make the pair comparable, not for biological inference

## Inputs

- Matched pairs of gene-barcode matrices (GBMs) from UniverSC and platform-specific pipelines (Cell Ranger for Chromium, dropSeqPipe for Drop-seq, CogentAP for ICELL8, zUMIs for SmartSeq3)
- UniverSC-processed FASTQ inputs generated from platform BAMs via 10x Genomics bamtofastq
- Reference genome used for all processing (human GRCh38 chr21 or mouse GRCm39)

## Outputs

- ari_results.csv: table of ARI values (plus Pearson correlations) for the four platform pairs
- Per-platform clustering outcomes from Seurat for each member of every GBM pair

## How to apply

First adjust each UniverSC/counterpart pair of GBMs to have matching sets of barcodes and genes: keep only barcodes found in both GBMs and pad genes missing from one matrix with 0 UMIs. Then run clustering on each adjusted GBM with the R package Seurat (version 4.1.1) in R 4.1.2 and compute the Adjusted Rand Index between the two clustering outcomes using R packages stats (4.1.2) and clues (0.6.2.2). In parallel, compute the Pearson correlation between each GBM pair, treating each gene-by-cell UMI-count pair as one data point unless both counts are zero. Assemble the four ARI values (with Pearson correlations as supporting metrics) into a single results table (ari_results.csv). The rationale is that a universal wrapper should yield biologically equivalent GBMs to vendor pipelines when fed the same FASTQs against the same reference (e.g., human GRCh38 chr21 or full mouse GRCm39).

## Related tools

- **UniverSC** (Universal wrapper around Cell Ranger that processes FASTQs from any UMI-based platform with per-technology pre-set parameters (Table 1 of 40 technologies)) — https://github.com/minoda-lab/universc
- **10x Genomics bamtofastq** (Converts Cell Ranger BAM output back to FASTQ so the same reads can be reprocessed by UniverSC) — https://github.com/10XGenomics/bamtofastq
- **Seurat** (Clustering analysis of each adjusted GBM pair (version 4.1.1 in R 4.1.2) prior to ARI computation)
- **clues** (R package (version 0.6.2.2) used to compute the Adjusted Rand Index between the two clustering outcomes per platform)

## Examples

```
library(Seurat); c1 <- Idents(seurat_universc); c2 <- Idents(seurat_counterpart); ARI <- clues::adjustedRand(c1, c2, indices="ARI")
```

## Evaluation signals

- ARI per UniverSC/counterpart pair reproduces the reported benchmark: approximately 1.0 for Chromium, 0.78 for Drop-seq, 0.87 for ICELL8, and 0.78 for SmartSeq3 (Fig. 2)
- Each GBM pair contains exactly matching sets of barcodes and genes after adjustment (shared barcodes kept; missing genes padded with 0 UMIs)
- Pearson correlations are computed excluding gene-cell pairs where both counts are zero
- Results table contains exactly four rows, one for each platform comparison (Chromium, Drop-seq, ICELL8, SmartSeq3)

## Limitations

- UniverSC pins Cell Ranger version 3.0.2 due to licensing; later Cell Ranger versions add scATAC-seq/TCR/BCR features but do not majorly affect scRNA-seq processing
- Technologies for 5' scRNA-seq other than Chromium are not supported at the time of writing
- Only reads mapping to chromosome 21 were kept (human datasets) and the ICELL8 dataset was downsampled to 250 K reads with seqtk (fixed random seed), so results are specific to these reduced datasets

## Evidence

- [methods] The adjusted pair of GBMs was then used to carry out clustering analysis with an R package Seurat: "The adjusted pair of GBMs was then used to carry out clustering analysis with an R package Seurat"
- [methods] Barcode/gene harmonisation before comparison: "only barcodes found in both GBMs were kept, and genes only found in one GBM were added to the other with 0 UMIs assigned."
- [methods] Concordance metrics: "the Pearson correlation between the GBMs and ARI between the two clustering outcomes were calculated"
- [readme] Scope of universal processing: [evidence span withheld — anchor status: external_doc]
- [methods] Cross-platform test scope: "Test datasets were prepared for Chromium16, Drop-seq6, ICELL87, and SmartSeq38,10"
