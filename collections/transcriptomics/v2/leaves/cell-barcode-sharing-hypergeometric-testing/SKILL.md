---
name: cell-barcode-sharing-hypergeometric-testing
description: Use when multiplexed 10x Genomics droplet scRNA-seq samples (e.g. HiSeq
  4000-sequenced libraries) must be tested pairwise for more shared cell barcodes
  than expected by chance, to flag barcode-swapping artefacts.
when_to_use_negative:
- Plate-based scRNA-seq datasets where sample identity is carried by sample barcodes
  rather than shared droplet cell barcodes (use the read-swapping regression approach
  instead)
- Datasets where libraries were sequenced only on non-patterned flow-cell instruments
  (e.g. HiSeq 2500), which serve as negative controls with no expected excess sharing
- Analyses aiming to correct swapping per molecule — use cell exclusion or molecule
  exclusion (same UMI, cell barcode, and aligned gene combination) rather than the
  sharing test
tools:
- name: DropletUtils
  role: Reads the 10x Genomics droplet count matrices, calls cells with emptyDrops,
    and supports removal of barcode-swapped pseudo-cells
  repo: https://github.com/MarioniLab/DropletUtils
- name: R
  role: Language in which the count matrices are loaded, cell barcodes are compared
    pairwise, and the hypergeometric test is computed
- name: HiSeq 4000
  role: Patterned flow-cell instrument whose multiplexed 10x libraries show excess
    barcode sharing that the test detects
- name: HiSeq 2500
  role: Non-patterned instrument used to sequence the negative-control libraries where
    no excess sharing is expected
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:3340fa0c6191bf5686257194dfa9580a58c69ea84f342c8c8a05280c2bd58437
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - 10x Genomics
  - DropletUtils
  - HiSeq 2500
  - HiSeq 4000
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cell-barcode-sharing-hypergeometric-testing

## Summary

Reproduces the hypergeometric test from Griffiths et al. (2018) that detects excess sharing of called 10x Genomics cell barcodes between multiplexed droplet samples, the signature of barcode swapping on patterned flow-cell instruments such as the HiSeq 4000.

## When to use

Use when multiplexed 10x Genomics droplet scRNA-seq samples (e.g. HiSeq 4000-sequenced libraries) must be tested pairwise for more shared cell barcodes than expected by chance, to flag barcode-swapping artefacts.

## When NOT to use

- Plate-based scRNA-seq datasets where sample identity is carried by sample barcodes rather than shared droplet cell barcodes (use the read-swapping regression approach instead)
- Datasets where libraries were sequenced only on non-patterned flow-cell instruments (e.g. HiSeq 2500), which serve as negative controls with no expected excess sharing
- Analyses aiming to correct swapping per molecule — use cell exclusion or molecule exclusion (same UMI, cell barcode, and aligned gene combination) rather than the sharing test

## Inputs

- Multiplexed 10x Genomics droplet count matrices in 10x matrix format (breast tumour and mouse epithelial from HiSeq 4000; mouse embryonic and resequenced epithelial from HiSeq 2500)
- Called cell barcodes per sample (from emptyDrops cell calling at FDR 0.01, minimum 1000 UMIs)

## Outputs

- Per-sample-pair table of observed shared barcode counts, expected counts under the null, and hypergeometric p-values
- barcode_sharing_tests.csv containing the full per-pair results

## How to apply

Load the multiplexed 10x Genomics droplet count matrices (breast tumour and mouse epithelial cells from the HiSeq 4000 run; mouse embryonic and resequenced epithelial cells from the HiSeq 2500 control) in R via DropletUtils-compatible reading of the 10x matrix format. Call cells per sample with emptyDrops at an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs, then extract each sample's set of called cell barcodes. For every pair of samples within a multiplexed set, count the number of shared barcodes and test each pair for excess overlap beyond random barcode sampling using a hypergeometric test (full details in Supplementary Note 5 of the barcode-swapping study). Record observed shared counts, null-expected counts, and hypergeometric p-values per pair, and compare instruments: significant excess sharing should appear only in HiSeq 4000-sequenced pairs, with HiSeq 2500 pairs as negative controls (~2.5% of reads were mislabelled on the HiSeq 4000, the phenomenon this test detects).

## Related tools

- **DropletUtils** (Reads the 10x Genomics droplet count matrices, calls cells with emptyDrops, and supports removal of barcode-swapped pseudo-cells) — https://github.com/MarioniLab/DropletUtils
- **R** (Language in which the count matrices are loaded, cell barcodes are compared pairwise, and the hypergeometric test is computed)
- **HiSeq 4000** (Patterned flow-cell instrument whose multiplexed 10x libraries show excess barcode sharing that the test detects)
- **HiSeq 2500** (Non-patterned instrument used to sequence the negative-control libraries where no excess sharing is expected)

## Examples

```
sce <- DropletUtils::read10xCounts("sample1/matrix.mtx"); p <- phyper(length(intersect(bc1,bc2))-1, length(bc1), Nbarcodes-length(bc1), length(bc2), lower.tail=FALSE)
```

## Evaluation signals

- barcode_sharing_tests.csv exists with one row per sample pair containing observed shared count, null-expected count, and hypergeometric p-value
- Every pair of samples within each multiplexed set is represented exactly once in the results table
- Significant excess sharing (small hypergeometric p-values) is restricted to HiSeq 4000-sequenced sample pairs
- HiSeq 2500-sequenced pairs show no excess sharing, matching the paper's negative-control expectation
- Cell barcode sets used are reproducible: cells called with emptyDrops at FDR 0.01 and minimum cell size of 1000 UMIs

## Limitations

- Full hypergeometric test details are in Supplementary Note 5, which is unavailable from the main text ('No changelog found.')
- The paper's cell exclusion strategy (removing all barcodes shared between samples) can incorrectly discard genuine cell libraries — the fraction incorrectly discarded was counted in their simulation
- Estimates ~2.5% of reads mislabelled on the HiSeq 4000, lower than previous reports, so effect sizes may be modest and detectability depends on sample multiplexing depth

## Evidence

- [discussion] Hypergeometric test details deferred to Supplementary Note 5: "We tested for an excessive sharing of cell barcodes using a hypergeometric test (see Supplementary Note 5)."
- [discussion] Cell calling parameters: "calling cells using emptyDrops18 using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs"
- [abstract] HiSeq 4000 mislabelling rate: "approximately 2.5% of reads were mislabelled between samples on the HiSeq 4000, which is lower than previous reports"
- [discussion] Cell exclusion caveat: "We applied cell exclusion by removing all barcodes that were shared between samples, and counted the fraction of cell libraries that were incorrectly discarded."
- [readme] DropletUtils capabilities: "identification of cells from empty droplets, removal of barcode-swapped pseudo-cells, and downsampling of the count matrix."
