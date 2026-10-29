---
name: cell-calling-method-benchmarking
description: Use when a raw unfiltered gene-by-barcode count matrix (mtx or h5) from
  droplet-based scRNA/snRNA-seq needs cell-calling, and multiple callers must be compared
  against ground-truth labels (HTO singlet/doublet or demuxlet genotypes).
when_to_use_negative:
- The input is an already-filtered cell-by-gene matrix from a filtered Cell Ranger
  output — cell calling requires the raw unfiltered matrix including cell-free droplets
- The goal is ambient RNA correction of gene expression rather than cell calling —
  EmptyNN does not estimate corrected expression profiles and tools like SoupX are
  needed
- Droplet-based data is not applicable (e.g., plate-based or FACS-sorted scRNA-seq)
  since there are no cell-free droplets to distinguish
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: EmptyNN
  role: Primary PU-learning neural-network cell caller (threshold T=100, k=10 folds,
    N=10 iterations) applied to the transposed raw count matrix
  repo: https://github.com/lkmklsmn/empty_nn
- name: Cell Ranger v.2
  role: Baseline cell caller using a total-count distribution cutoff (threshold 200
    in the original cell hashing study)
- name: EmptyDrops
  role: Baseline caller using a Dirichlet-multinomial model to infer the cell-free
    transcriptome profile and call significant deviations
- name: Cell Ranger v.3
  role: Baseline caller integrating EmptyDrops into its cell-calling algorithm
- name: CellBender
  role: Neural-network baseline caller; expected number of cells and total-droplets-included
    chosen via barcode rank plot inspection
- name: demuxlet
  role: Infers per-barcode genotype labels and singlet probabilities from a merged
    VCF, providing ground truth in the multiplexed PBMC dataset
  repo: https://github.com/yelabucsf/demuxlet_paper_code/
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:2197d407b303a761d80b463e7a8f5f2952c4bbba1132bdf80ad28d9d2fbc4d11
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lkmklsmn/empty_nn
  repo_url: https://github.com/lkmklsmn/empty_nn
  tools:
  - Cell Ranger v.2
  - Cell Ranger v.3
  - CellBender
  - EmptyDrops
  - EmptyNN
  - demuxlet
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cell-calling-method-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark EmptyNN against Cell Ranger v.2, EmptyDrops, Cell Ranger v.3, and CellBender on raw unfiltered scRNA-seq count matrices, evaluating AUROC against ground-truth labels (HTO-derived or demuxlet-derived) or comparing retained droplet counts.

## When to use

Use when a raw unfiltered gene-by-barcode count matrix (mtx or h5) from droplet-based scRNA/snRNA-seq needs cell-calling, and multiple callers must be compared against ground-truth labels (HTO singlet/doublet or demuxlet genotypes).

## When NOT to use

- The input is an already-filtered cell-by-gene matrix from a filtered Cell Ranger output — cell calling requires the raw unfiltered matrix including cell-free droplets
- The goal is ambient RNA correction of gene expression rather than cell calling — EmptyNN does not estimate corrected expression profiles and tools like SoupX are needed
- Droplet-based data is not applicable (e.g., plate-based or FACS-sorted scRNA-seq) since there are no cell-free droplets to distinguish

## Inputs

- Raw unfiltered gene-by-barcode count matrix in mtx or h5 format (e.g., neurons_900_raw.h5)
- HTO (hashtag oligo) counts for singlet/doublet/negative ground-truth labels (cell hashing dataset)
- Genome-aligned BAM file (e.g., SRR5398237) and merged VCF file for demuxlet-derived ground truth
- Barcode rank plot for choosing CellBender's expected-number-of-cells and total-droplets-included parameters

## Outputs

- Per-method AUROC values against ground-truth labels (e.g., auroc_results.csv, auroc_comparison.csv)
- Per-method counts of retained cell-containing droplets (e.g., cell_calling_mouse_brain_results.csv)
- EmptyNN boolean keep vector and prediction probabilities for barcodes in set U

## How to apply

Load the raw unfiltered count matrix (Read10X_h5 for h5) and transpose it so rows are droplets; run EmptyNN with threshold = 100, k = 10 cross-validation folds, and N = 10 iterations to classify barcodes in set U as cell-free or cell-containing. Run baselines on the same matrix: Cell Ranger v.2 (total-count cutoff), EmptyDrops (Dirichlet-multinomial model of the cell-free transcriptome), Cell Ranger v.3 (EmptyDrops-integrated), and CellBender, choosing CellBender's expected-number-of-cells / total-droplets-included by inspecting the barcode rank plot. For ground truth, integrate HTO counts (singlet/doublet/negative labels) or run demuxlet with a merged VCF on a genome-aligned BAM to infer per-barcode genotype labels. Compute AUROC per method against the ground-truth labels (e.g., via pROC) and, for datasets without ground truth, tabulate the number of retained droplets per method. Validate against reported benchmarks: EmptyNN 96.30% (multiplexed PBMC, 6,025 retained droplets) and 94.73% (cell hashing), with baselines such as Cell Ranger v.2 82.94%, EmptyDrops 86.43%, Cell Ranger v.3 83.03%, CellBender 58.02%.

## Related tools

- **EmptyNN** (Primary PU-learning neural-network cell caller (threshold T=100, k=10 folds, N=10 iterations) applied to the transposed raw count matrix) — https://github.com/lkmklsmn/empty_nn
- **Cell Ranger v.2** (Baseline cell caller using a total-count distribution cutoff (threshold 200 in the original cell hashing study))
- **EmptyDrops** (Baseline caller using a Dirichlet-multinomial model to infer the cell-free transcriptome profile and call significant deviations)
- **Cell Ranger v.3** (Baseline caller integrating EmptyDrops into its cell-calling algorithm)
- **CellBender** (Neural-network baseline caller; expected number of cells and total-droplets-included chosen via barcode rank plot inspection)
- **demuxlet** (Infers per-barcode genotype labels and singlet probabilities from a merged VCF, providing ground truth in the multiplexed PBMC dataset) — https://github.com/yelabucsf/demuxlet_paper_code/

## Examples

```
counts <- t(Read10X_h5("./data/raw.h5")); nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE)
```

## Evaluation signals

- EmptyNN AUROC reproduces 94.73% on the cell hashing dataset and 96.30% on the multiplexed PBMC dataset within a reasonable tolerance
- Baseline AUROCs fall in the reported ranges (Cell Ranger v.2 82.94%, EmptyDrops 86.43%, Cell Ranger v.3 83.03%, CellBender 58.02% on multiplexed PBMC; baselines 79.87%–90.16% on cell hashing)
- Retained droplet counts match targets on the mouse brain snRNA-seq dataset: EmptyNN 2,222; Cell Ranger v.2 2,371; EmptyDrops 2,181; Cell Ranger v.3 2,566; CellBender 4,386
- Results tables (auroc_results.csv, auroc_comparison.csv, cell_calling_mouse_brain_results.csv) are written with one row per method
- On cell hashing, EmptyNN classifies 19,009 of 39,842 barcodes (47.7%) as cell-containing and recovers 885 barcodes below the original 200-count threshold

## Limitations

- The key assumption that low-total-count barcodes are cell-free may fail when genuine low-count cells exist; the T threshold must be manually adjusted
- Parameter selection (cross-validation folds k and threshold T) is manual with no automated hyperparameter selection implemented, though the method is claimed robust to various k and T
- Retained droplets may have a high fraction of mitochondrial reads, requiring separate low-quality-cell removal downstream
- CellBender's expected-number-of-cells and total-droplets-included depend on each dataset and must be chosen by inspecting the barcode rank plot

## Evidence
