---
name: 10x-count-matrix-handling-with-dropletutils
description: Use when droplet-based 10x Genomics count matrices (raw gene-barcode
  matrices) from multiplexed samples must be read into R and called cells extracted
  before barcode-swapping or sample-sharing analyses.
when_to_use_negative:
- Data are plate-based scRNA-seq (e.g. Smart-seq2 plates or the Nestorowa/Richard
  plate datasets), where barcode swapping is quantified by linear regression on swapped
  reads rather than droplet barcode sharing
- The count matrix is already filtered to a final cell set, so emptyDrops cell calling
  and empty-droplet modelling are not applicable
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: DropletUtils
  role: R/Bioconductor package used to load 10x count matrices, call cells with emptyDrops,
    and remove barcode-swapped pseudo-cells
  repo: https://github.com/MarioniLab/DropletUtils
- name: R
  role: Analysis environment in which the droplet count matrices are read and the
    hypergeometric sharing tests are run
- name: 10x Genomics
  role: Source of the droplet-based single-cell RNA-seq data whose matrix format is
    loaded by DropletUtils
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-05083-x
    title: Detection and removal of barcode swapping in single-cell RNA-seq data
  source_content_hash: sha256:f9e60353ca8bb3b712d51c363106d84e17d4fd94ad2abd193b24dc4906ae9624
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MarioniLab/DropletUtils
  tools:
  - 10x Genomics
  - DropletUtils
  - R
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# 10x-count-matrix-handling-with-dropletutils

## Summary

Load and pre-process 10x Genomics droplet count matrices with the DropletUtils R package in the context of the barcode-swapping study (Griffiths et al. 2018), including emptyDrops cell calling (FDR 0.01, minimum 1000 UMIs) so that per-sample cell-barcode sets can be compared for excess sharing on HiSeq 4000 vs HiSeq 2500 data.

## When to use

Use when droplet-based 10x Genomics count matrices (raw gene-barcode matrices) from multiplexed samples must be read into R and called cells extracted before barcode-swapping or sample-sharing analyses.

## When NOT to use

- Data are plate-based scRNA-seq (e.g. Smart-seq2 plates or the Nestorowa/Richard plate datasets), where barcode swapping is quantified by linear regression on swapped reads rather than droplet barcode sharing
- The count matrix is already filtered to a final cell set, so emptyDrops cell calling and empty-droplet modelling are not applicable

## Inputs

- 10x Genomics droplet count matrices (matrix.mtx / barcodes.tsv / genes.tsv) from multiplexed HiSeq 4000 (human breast tumour, mouse epithelial) and HiSeq 2500 (mouse embryonic, resequenced epithelial) runs
- Empty-droplet barcode counts for emptyDrops cell calling

## Outputs

- Per-sample sets of called cell barcodes (emptyDrops FDR < 0.01, >= 1000 UMIs)
- Per-pair observed shared barcode counts, expected counts under the null, and hypergeometric p-values written to barcode_sharing_tests.csv

## How to apply

Read the 10x matrix (matrix.mtx, barcodes.tsv, genes.tsv) into R with DropletUtils-compatible functions to build a count matrix for each sample in each multiplexed set. Call cells using emptyDrops with an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs, which is the parameterisation used in the paper's droplet analyses of HiSeq 2500- and HiSeq 4000-sequenced mouse epithelial cells. Extract the set of called cell barcodes per sample, then compute pairwise shared-barcode counts and test for excess sharing with a hypergeometric test (details in Supplementary Note 5). Record observed and expected shared counts and p-values for all sample pairs in barcode_sharing_tests.csv, verifying that significant excess sharing is restricted to HiSeq 4000-sequenced samples with HiSeq 2500 pairs as the negative control.

## Related tools

- **DropletUtils** (R/Bioconductor package used to load 10x count matrices, call cells with emptyDrops, and remove barcode-swapped pseudo-cells) — https://github.com/MarioniLab/DropletUtils
- **R** (Analysis environment in which the droplet count matrices are read and the hypergeometric sharing tests are run)
- **10x Genomics** (Source of the droplet-based single-cell RNA-seq data whose matrix format is loaded by DropletUtils)

## Examples

```
library(DropletUtils); sce <- read10xCounts("raw_gene_bc_matrices/mm10/"); called <- emptyDrops(counts(sce), filter.empty=FALSE); cells <- rownames(called[which(called$FDR <= 0.01 & called$Total >= 1000),])
```

## Evaluation signals

- emptyDrops was run with FDR threshold 0.01 and minimum cell size 1000 UMIs, matching the paper's droplet parameterisation
- Loaded matrices reconstruct the correct genes x barcodes dimensions from the 10x matrix format for each sample
- barcode_sharing_tests.csv contains observed shared counts, expected counts under the null, and hypergeometric p-values for every sample pair within each multiplexed set
- Significant excess barcode sharing is restricted to HiSeq 4000-sequenced sample pairs, with HiSeq 2500 pairs showing no excess sharing (negative control behaviour)

## Limitations

- Hypergeometric test details are deferred to Supplementary Note 5, which is unavailable in the provided document ('No changelog found.'), so the exact null parameterisation must be recovered from the supplementary report repo
- Cell exclusion (removing all barcodes shared between samples) can incorrectly discard genuine cell libraries; the paper counted the fraction incorrectly discarded in simulations
- Barcode swapping was only detected on patterned flow-cell instruments (HiSeq 4000, HiSeq X, NovaSeq); HiSeq 2500 data showed no excess sharing, so this pipeline adds little on non-patterned flow-cell data

## Evidence

- [discussion] Cell calling with emptyDrops at FDR threshold 0.01 and minimum cell size 1000 UMIs: "calling cells using emptyDrops18 using an FDR threshold of 0.01 and a minimum cell size of 1000 UMIs"
- [discussion] Hypergeometric test details deferred to Supplementary Note 5: "We tested for an excessive sharing of cell barcodes using a hypergeometric test (see Supplementary Note 5)."
- [readme] DropletUtils provides utilities for droplet single-cell data from 10X Genomics: "The DropletUtils package provides a number of utility functions for handling single-cell (RNA-seq) data from droplet technologies such as 10X Genomics."
- [intro] Barcode swapping restricted to patterned flow-cell instruments: "patterned flow-cell Illumina sequencing machines, including the HiSeq 4000, HiSeq X, and NovaSeq"
