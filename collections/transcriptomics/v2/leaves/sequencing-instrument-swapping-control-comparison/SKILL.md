---
name: sequencing-instrument-swapping-control-comparison
description: Use when multiplexed 10x Genomics droplet libraries have been sequenced
  on both a patterned-flowcell instrument (HiSeq 4000/HiSeq X/NovaSeq) and a non-patterned
  control (HiSeq 2500), and you need to quantify or rule out excess inter-sample cell-barcode
  sharing.
when_to_use_negative:
- Data derive from plate-based scRNA-seq (e.g. Smart-seq2) where sample identity is
  carried by well plate barcodes rather than 10x cell barcodes; use the regression-gradient
  swapping-fraction estimation instead
- Only a single sequencing instrument is available, so no HiSeq 2500 negative-control
  comparison can be constructed
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DropletUtils
  role: R package used to load 10x Genomics count matrices, call cells (emptyDrops),
    and remove barcode-swapped pseudo-cells from droplet data
  repo: https://github.com/MarioniLab/DropletUtils
- name: R
  role: Analysis environment in which the hypergeometric barcode-sharing tests and
    per-pair results table are computed
- name: HiSeq 4000
  role: Patterned-flowcell instrument on which swapping-affected multiplexed droplet
    libraries were sequenced
- name: HiSeq 2500
  role: Non-patterned control instrument; resequencing the same libraries on HiSeq
    2500 provides the negative control showing no excess barcode sharing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:6189f5611733cd1742406686f70d3d916f2070a6a7668c12b6ace52fc4e72cee
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# sequencing-instrument-swapping-control-comparison

## Summary

A control-comparison design for detecting barcode swapping artefacts in 10x Genomics droplet scRNA-seq by testing whether multiplexed samples sequenced on patterned-flowcell instruments (HiSeq 4000) share more cell barcodes than expected by chance, using the same libraries sequenced on HiSeq 2500 as a negative control.

## When to use

Use when multiplexed 10x Genomics droplet libraries have been sequenced on both a patterned-flowcell instrument (HiSeq 4000/HiSeq X/NovaSeq) and a non-patterned control (HiSeq 2500), and you need to quantify or rule out excess inter-sample cell-barcode sharing.

## When NOT to use

- Data derive from plate-based scRNA-seq (e.g. Smart-seq2) where sample identity is carried by well plate barcodes rather than 10x cell barcodes; use the regression-gradient swapping-fraction estimation instead
- Only a single sequencing instrument is available, so no HiSeq 2500 negative-control comparison can be constructed

## Inputs

- Multiplexed 10x Genomics droplet count matrices (matrix.mtx / barcodes / genes format) for HiSeq 4000-sequenced samples (human breast tumour, mouse epithelial cells)
- Matching HiSeq 2500-sequenced control count matrices (mouse embryonic, resequenced mouse epithelial cells)
- Sample-to-multiplex-set mapping defining which samples share a droplet run

## Outputs

- barcode_sharing_tests.csv with per-pair observed shared barcode counts, expected counts under the hypergeometric null, and p-values
- Instrument-level comparison showing significant excess sharing restricted to HiSeq 4000 samples

## How to apply

Load the 10x Genomics droplet count matrices with DropletUtils-compatible readers and call cells using emptyDrops at an FDR threshold of 0.01 with a minimum cell size of 1000 UMIs. For every pair of samples within each multiplexed set, count shared cell barcodes and test for excess sharing with a hypergeometric test (details in Supplementary Note 5 of the barcode-swapping study). Verify that significant excess sharing is restricted to HiSeq 4000-sequenced samples (human breast tumour and mouse epithelial cells), while the HiSeq 2500-sequenced pairs (mouse embryonic and resequenced epithelial) show no excess sharing, mirroring the paper's control design in which the same libraries were sequenced on both instruments. Record observed shared counts, expected counts under the null, and p-values for all pairs in barcode_sharing_tests.csv. If swapping is confirmed, follow-up with cell exclusion (removing barcodes shared between samples) or molecule exclusion (removing molecules sharing the same UMI, cell barcode, and aligned gene combination), accepting that cell exclusion may incorrectly discard some genuine cell libraries.

## Related tools

- **DropletUtils** (R package used to load 10x Genomics count matrices, call cells (emptyDrops), and remove barcode-swapped pseudo-cells from droplet data) — https://github.com/MarioniLab/DropletUtils
- **R** (Analysis environment in which the hypergeometric barcode-sharing tests and per-pair results table are computed)
- **HiSeq 4000** (Patterned-flowcell instrument on which swapping-affected multiplexed droplet libraries were sequenced)
- **HiSeq 2500** (Non-patterned control instrument; resequencing the same libraries on HiSeq 2500 provides the negative control showing no excess barcode sharing)

## Examples

```
library(DropletUtils); counts <- read10xCounts("raw_gene_bc_matrices/mm10"); called <- emptyDrops(counts, lower=1000); cells <- rownames(called)[which(called$FDR <= 0.01)]
```

## Evaluation signals

- Excess barcode sharing (hypergeometric p-values below the chosen significance threshold) should be observed only among HiSeq 4000-sequenced sample pairs, not HiSeq 2500 pairs
- The results table barcode_sharing_tests.csv must contain, for every within-set sample pair, an observed shared count, an expected count under the null, and a hypergeometric p-value
- Cell calling via emptyDrops must use FDR 0.01 and minimum 1000 UMIs, and the resulting called-barcode sets should have plausible sizes for droplet data
- In the positive control, swapped artefactual cell libraries should be evident among HiSeq 4000 samples; after cell exclusion, all barcodes shared between samples should be removed

## Limitations

- Hypergeometric test details are deferred to Supplementary Note 5, which is unavailable in the extracted material ('No changelog found.'), so exact parameterisation must be recovered from the BarcodeSwapping2017 repository report
- Cell exclusion (removing all barcodes shared between samples) can incorrectly discard genuine cell libraries; the simulation in the paper counted this incorrect-discard fraction
- The swapping rate is instrument-specific: ~2.5% of reads mislabelled on HiSeq 4000, so results do not generalise to non-patterned flow-cell instruments where no excess sharing was detected

## Evidence

- [discussion] Excess sharing restricted to HiSeq 4000 droplet samples: "We tested for an excessive sharing of cell barcodes using a hypergeometric test (see Supplementary Note 5)."
- [discussion] HiSeq 2500 control resequencing: "As a control, we repeated this procedure with the same libraries sequenced on HiSeq 2500."
- [discussion] emptyDrops cell-calling thresholds: "calling cells using emptyDrops18 using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs"
- [abstract] Instrument context: "approximately 2.5% of reads were mislabelled between samples on the HiSeq 4000, which is lower than previous reports"
- [readme] DropletUtils scope: "removal of barcode-swapped pseudo-cells, and downsampling of the count matrix"
