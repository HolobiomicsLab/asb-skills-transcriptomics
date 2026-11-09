---
name: droplet-scrna-seq-cell-barcode-extraction
description: Use when multiplexed 10x Genomics droplet count matrices must be reduced
  to per-sample sets of called cell barcodes (e.g. to test for excess inter-sample
  barcode sharing on HiSeq 4000 vs HiSeq 2500).
when_to_use_negative:
- Input is plate-based Smart-seq2 scRNA-seq data, where barcodes are sample-specific
  and the droplet emptyDrops cell-calling step does not apply
- Input is already a list of called cell barcodes rather than a raw 10x count matrix
- The goal is quantifying the swapped fraction of reads via the regression-gradient
  approach on plate data rather than calling droplet cell barcodes
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DropletUtils
  role: R/Bioconductor package used to read the 10x Genomics count matrices and call
    cells from empty droplets (emptyDrops, FDR 0.01, min 1000 UMIs) to obtain called
    cell barcodes
  repo: https://github.com/MarioniLab/DropletUtils
- name: 10x Genomics
  role: Droplet-based scRNA-seq platform whose count matrices and barcode structure
    provide the input barcodes
- name: R
  role: Implementation language for loading matrices with DropletUtils and downstream
    barcode overlap analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:883f4dede161317eb175e5ac26c21469c13c8520900f22c6425b6336f58eec7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - 10x Genomics
  - DropletUtils
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# droplet-scrna-seq-cell-barcode-extraction

## Summary

Extracts the set of called cell barcodes from 10x Genomics droplet scRNA-seq count matrices (e.g. HiSeq 4000- and HiSeq 2500-sequenced multiplexed samples) using DropletUtils/emptyDrops, so that barcode overlap between samples can be quantified for barcode-swapping artefact detection.

## When to use

Use when multiplexed 10x Genomics droplet count matrices must be reduced to per-sample sets of called cell barcodes (e.g. to test for excess inter-sample barcode sharing on HiSeq 4000 vs HiSeq 2500).

## When NOT to use

- Input is plate-based Smart-seq2 scRNA-seq data, where barcodes are sample-specific and the droplet emptyDrops cell-calling step does not apply
- Input is already a list of called cell barcodes rather than a raw 10x count matrix
- The goal is quantifying the swapped fraction of reads via the regression-gradient approach on plate data rather than calling droplet cell barcodes

## Inputs

- 10x Genomics droplet scRNA-seq count matrices (10x matrix format: matrix.mtx, barcodes.tsv, genes.tsv) for each sample in a multiplexed set
- Multiplexed samples sequenced on HiSeq 4000 (breast tumour, mouse epithelial) and HiSeq 2500 (mouse embryonic, resequenced epithelial) (ArrayExpress E-MTAB-6854)

## Outputs

- Per-sample vector of called cell barcodes passing emptyDrops FDR <= 0.01 and minimum 1000 UMIs
- Per-pair shared barcode counts used as input to the hypergeometric sharing test (barcode_sharing_tests.csv context)

## How to apply

Load each 10x Genomics droplet count matrix (10x matrix format, DropletUtils-compatible) for every sample in the multiplexed set — e.g. breast tumour and mouse epithelial from the HiSeq 4000 run, mouse embryonic and resequenced epithelial from the HiSeq 2500 run. Call cells with emptyDrops using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs, as in the barcode-swapping study. Retain the barcode sequences of cells passing these thresholds as each sample's called cell barcode set. Decision point: the emptyDrops FDR cutoff and minimum UMI count control the stringency of cell calling and thus which barcodes enter downstream pairwise overlap counts; for the swapping analysis these sets feed a hypergeometric test for excessive sharing of cell barcodes between sample pairs (details deferred to Supplementary Note 5).

## Related tools

- **DropletUtils** (R/Bioconductor package used to read the 10x Genomics count matrices and call cells from empty droplets (emptyDrops, FDR 0.01, min 1000 UMIs) to obtain called cell barcodes) — https://github.com/MarioniLab/DropletUtils
- **10x Genomics** (Droplet-based scRNA-seq platform whose count matrices and barcode structure provide the input barcodes)
- **R** (Implementation language for loading matrices with DropletUtils and downstream barcode overlap analysis)

## Examples

```
library(DropletUtils); counts <- read10xCounts("sample1_matrix"); e <- emptyDrops(counts, lower=100); called <- rownames(e)[which(e$FDR <= 0.01 & e$Total >= 1000)]
```

## Evaluation signals

- Cell calling uses the stated thresholds: emptyDrops FDR threshold 0.01 and minimum cell size of 1000 UMIs
- Extracted barcode sets are restricted to barcodes present in the 10x matrix barcodes.tsv of the corresponding sample
- Excess barcode sharing detected by the downstream hypergeometric test is restricted to HiSeq 4000-sequenced pairs, with HiSeq 2500 pairs as the negative control
- Called cell counts are consistent with a droplet experiment (thousands of cells per sample), not empty droplets

## Limitations

- Details of the hypergeometric test for excessive cell-barcode sharing are deferred to Supplementary Note 5, which is unavailable in this context ('No changelog found.')
- Cell exclusion by removing all barcodes shared between samples can incorrectly discard genuine cell libraries; the fraction incorrectly discarded was counted in simulation
- Excess sharing is expected only for patterned flow-cell instruments (HiSeq 4000, HiSeq X, NovaSeq); HiSeq 2500 data serves as the negative control

## Evidence

- [discussion] Cell calling with emptyDrops at FDR threshold 0.01 and minimum cell size 1000 UMIs: "calling cells using emptyDrops18 using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs"
- [discussion] Hypergeometric test for barcode sharing: "We tested for an excessive sharing of cell barcodes using a hypergeometric test (see Supplementary Note 5)."
- [readme] DropletUtils provides cell calling and barcode swap removal: "This includes data loading from count matrices or molecule information files, identification of cells from empty droplets, removal of barcode-swapped pseudo-cells"
