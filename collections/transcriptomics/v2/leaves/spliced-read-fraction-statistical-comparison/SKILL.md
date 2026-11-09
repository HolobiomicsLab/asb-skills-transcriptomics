---
name: spliced-read-fraction-statistical-comparison
description: Use when benchmarking cell-calling methods on single-nucleus RNA-seq
  raw unfiltered count matrices and you need a contamination-sensitive quality metric
  (spliced-read fraction) to compare retained barcodes across methods.
when_to_use_negative:
- The input is already a filtered feature-by-cell matrix or an h5ad with cell barcodes
  pre-called — cell calling must be run on the raw unfiltered matrix for the comparison
  to be valid
- The data are standard scRNA-seq (whole cells) rather than snRNA-seq — the spliced-read
  fraction is a cytoplasmic-contamination signal specific to nuclei; whole-cell data
  do not support this interpretation
- Only one cell-calling method is being used — the metric is a comparative benchmark
  across methods, not a standalone QC pass/fail filter
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: EmptyNN
  role: PU-learning neural-network cell calling (threshold 100, k=10 folds, 10 iterations)
    that yields the nuclei with the lower spliced-read fraction
  repo: https://github.com/lkmklsmn/empty_nn
- name: Cell Ranger v.2
  role: Baseline cell calling via total-count-distribution cutoff on the same raw
    matrix
- name: EmptyDrops
  role: Cell calling using a Dirichlet-multinomial model to infer the cell-free transcriptome
    profile
- name: Cell Ranger v.3
  role: Cell calling that integrates EmptyDrops; its retained nuclei are compared
    against EmptyNN for spliced-read fraction (p = 3.29e-5)
- name: CellBender
  role: Neural-network-based cell calling with expected number of cells / total-droplets-included
    parameters set from barcode rank plot inspection
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:09eb397c1fb72d23f6647e3c8fd0dc48781f778c3de8311df0eba0c26867c1ec
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
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spliced-read-fraction-statistical-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies cytoplasmic contamination in snRNA-seq cell-calling by computing the fraction of spliced reads among retained nucleus-containing droplets per method (EmptyNN, Cell Ranger v.2, EmptyDrops, Cell Ranger v.3, CellBender) and statistically testing the differences; in the EmptyNN paper's adult mouse brain dataset, EmptyNN- versus Cell Ranger v.3-filtered nuclei showed a significantly lower spliced-read fraction (p = 3.29e-5).

## When to use

Use when benchmarking cell-calling methods on single-nucleus RNA-seq raw unfiltered count matrices and you need a contamination-sensitive quality metric (spliced-read fraction) to compare retained barcodes across methods.

## When NOT to use

- The input is already a filtered feature-by-cell matrix or an h5ad with cell barcodes pre-called — cell calling must be run on the raw unfiltered matrix for the comparison to be valid
- The data are standard scRNA-seq (whole cells) rather than snRNA-seq — the spliced-read fraction is a cytoplasmic-contamination signal specific to nuclei; whole-cell data do not support this interpretation
- Only one cell-calling method is being used — the metric is a comparative benchmark across methods, not a standalone QC pass/fail filter

## Inputs

- Raw unfiltered snRNA-seq count matrix in mtx or h5 format (adult mouse brain dataset)
- Cell-calling assignments per barcode from EmptyNN, Cell Ranger v.2, EmptyDrops, Cell Ranger v.3, and CellBender
- Read-level annotations (spliced vs. unspliced) per barcode, e.g. from a genome-aligned BAM processed with velocyto/STARsolo-style exon-intron annotation

## Outputs

- Per-method counts of retained nucleus-containing droplets
- Per-barcode and per-method fraction of spliced reads
- Statistical comparison of spliced-read fractions across methods (e.g. p = 3.29e-5 for EmptyNN vs. Cell Ranger v.3)
- cell_calling_mouse_brain_results.csv summary table

## How to apply

Obtain the raw unfiltered count matrix (mtx or h5) for the adult mouse brain snRNA-seq dataset, transpose it so rows are droplets, and run each cell-calling method on the same matrix: EmptyNN (threshold = 100, k = 10 cross-validation folds, 10 training iterations), Cell Ranger v.2 (total-count-distribution cutoff), EmptyDrops (Dirichlet-multinomial model of the cell-free transcriptome), Cell Ranger v.3 (integrates EmptyDrops), and CellBender (expected-cells and total-droplets-included chosen by inspecting the barcode rank plot). Tabulate retained nucleus-containing droplets per method (expected: EmptyNN 2,222; Cell Ranger v.2 2,371; EmptyDrops 2,181; Cell Ranger v.3 2,566; CellBender 4,386). For each method's retained barcodes, compute the fraction of spliced reads — in nuclei, cytoplasmic RNA contamination inflates spliced reads, so lower fractions indicate cleaner nuclear profiles. Statistically compare the spliced-read fractions across methods (reported p = 3.29e-5 for EmptyNN vs. Cell Ranger v.3) and write the combined summary table to cell_calling_mouse_brain_results.csv.

## Related tools

- **EmptyNN** (PU-learning neural-network cell calling (threshold 100, k=10 folds, 10 iterations) that yields the nuclei with the lower spliced-read fraction) — https://github.com/lkmklsmn/empty_nn
- **Cell Ranger v.2** (Baseline cell calling via total-count-distribution cutoff on the same raw matrix)
- **EmptyDrops** (Cell calling using a Dirichlet-multinomial model to infer the cell-free transcriptome profile)
- **Cell Ranger v.3** (Cell calling that integrates EmptyDrops; its retained nuclei are compared against EmptyNN for spliced-read fraction (p = 3.29e-5))
- **CellBender** (Neural-network-based cell calling with expected number of cells / total-droplets-included parameters set from barcode rank plot inspection)

## Examples

```
counts <- t(Read10X_h5('./data/raw_feature_bc_matrix.h5')); nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE)
```

## Evaluation signals

- Retained droplet counts match the published values: EmptyNN 2,222; Cell Ranger v.2 2,371; EmptyDrops 2,181; Cell Ranger v.3 2,566; CellBender 4,386
- EmptyNN's retained nuclei show a significantly lower fraction of spliced reads than Cell Ranger v.3's (p = 3.29e-5)
- All methods are run on the identical raw unfiltered count matrix (mtx or h5) transposed so rows are droplets/barcodes
- EmptyNN parameters match the defaults: threshold = 100, k = 10 cross-validation folds, 10 training iterations
- The combined per-method summary table is written to cell_calling_mouse_brain_results.csv

## Limitations

- Low total count barcodes are assumed cell-free; this assumption may not hold when genuine cell-containing droplets with very low total counts exist, and the user must manually adjust the T threshold
- Parameter selection (cross-validation folds k and threshold T) must be specified manually; no hyperparameter selection approach is implemented
- The expected number of cells for CellBender and total-droplets-included depend on each dataset and must be chosen by examining the barcode rank plot
- Retained cell-containing droplets may have a high fraction of mitochondrial reads; low-quality cells must be removed separately in downstream analysis

## Evidence

- [results] EmptyNN- vs Cell Ranger v.3 spliced-read fraction difference: [evidence span withheld — anchor status: not_found]
