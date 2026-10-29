---
name: single-nucleus-rna-seq-cell-calling
description: Use when you have a raw unfiltered count matrix (mtx or h5) from a droplet-based
  snRNA-seq experiment and must decide which barcodes are nucleus-containing droplets
  versus cell-free droplets with ambient RNA.
when_to_use_negative:
- Gene expression values need ambient-RNA correction — EmptyNN does not estimate corrected
  profiles; apply SoupX instead
- Low-quality cells with high mitochondrial read fraction need removal — that is downstream
  filtering, not cell calling
- Droplets already classified by HTO hashing or demuxlet genetic ground truth — use
  those labels directly for evaluation rather than re-calling cells
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: EmptyNN
  role: 'Primary cell-calling method: PU-learning neural network classifying barcodes
    as cell-free or cell-containing (threshold=100, k=10, 10 iterations)'
  repo: https://github.com/lkmklsmn/empty_nn
- name: Cell Ranger v.2
  role: Baseline cell caller using a total-UMI-count-distribution cutoff (barcodes
    with total UMI more than m/10 considered cells)
- name: EmptyDrops
  role: Baseline cell caller using a Dirichlet-multinomial model to infer the cell-free
    transcriptome profile and flag significant deviations
- name: Cell Ranger v.3
  role: Baseline cell caller that integrates EmptyDrops into its cell-calling algorithm
- name: CellBender
  role: Neural-network baseline for filtering barcodes; expected-cells and total-droplets-included
    chosen per dataset via barcode rank plot
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:db8a6bd9fadd945e33d7e850bbff7ca8f2b2e67b015237bbcd7c5a16efbf3540
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# single-nucleus-rna-seq-cell-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compare cell-calling methods (EmptyNN, Cell Ranger v.2, EmptyDrops, Cell Ranger v.3, CellBender) on a raw unfiltered snRNA-seq count matrix to determine how many nucleus-containing droplets each retains, and assess cytoplasmic contamination via the fraction of spliced reads. Grounded in the EmptyNN paper's adult mouse brain snRNA-seq analysis.

## When to use

Use when you have a raw unfiltered count matrix (mtx or h5) from a droplet-based snRNA-seq experiment and must decide which barcodes are nucleus-containing droplets versus cell-free droplets with ambient RNA.

## When NOT to use

- Gene expression values need ambient-RNA correction — EmptyNN does not estimate corrected profiles; apply SoupX instead
- Low-quality cells with high mitochondrial read fraction need removal — that is downstream filtering, not cell calling
- Droplets already classified by HTO hashing or demuxlet genetic ground truth — use those labels directly for evaluation rather than re-calling cells

## Inputs

- Raw unfiltered count matrix in mtx or h5 format (e.g., via Read10X_h5)
- Barcode rank plot for CellBender parameter inspection
- Genome annotation of spliced/unspliced reads for contamination assessment

## Outputs

- Boolean vector of cell-free vs cell-containing droplet predictions per method
- Per-barcode prediction probabilities for barcodes in set U (EmptyNN)
- Per-method counts of retained nucleus-containing droplets
- Fraction of spliced reads per method with statistical comparison (p-value)
- cell_calling_mouse_brain_results.csv summary table

## How to apply

Obtain the raw unfiltered count matrix, transpose it so rows are droplets, and run EmptyNN with threshold = 100, k = 10 cross-validation folds, and 10 training iterations; only barcodes with total counts greater than 10 enter the analysis. In parallel, run Cell Ranger v.2 (which sets a cutoff from the total-count distribution), EmptyDrops (Dirichlet-multinomial model of the cell-free transcriptome), Cell Ranger v.3 (which integrates EmptyDrops), and CellBender, choosing the expected-cells / total-droplets-included parameter by inspecting the barcode rank plot for the dataset. Tabulate retained nucleus-containing droplets per method (mouse brain targets: EmptyNN 2,222; Cell Ranger v.2 2,371; EmptyDrops 2,181; Cell Ranger v.3 2,566; CellBender 4,386). Compute the fraction of spliced reads among retained droplets per method and compare statistically (reported p = 3.29e-5, EmptyNN lower than Cell Ranger v.3), writing the combined summary table to cell_calling_mouse_brain_results.csv.

## Related tools

- **EmptyNN** (Primary cell-calling method: PU-learning neural network classifying barcodes as cell-free or cell-containing (threshold=100, k=10, 10 iterations)) — https://github.com/lkmklsmn/empty_nn
- **Cell Ranger v.2** (Baseline cell caller using a total-UMI-count-distribution cutoff (barcodes with total UMI more than m/10 considered cells))
- **EmptyDrops** (Baseline cell caller using a Dirichlet-multinomial model to infer the cell-free transcriptome profile and flag significant deviations)
- **Cell Ranger v.3** (Baseline cell caller that integrates EmptyDrops into its cell-calling algorithm)
- **CellBender** (Neural-network baseline for filtering barcodes; expected-cells and total-droplets-included chosen per dataset via barcode rank plot)

## Examples

```
nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE)
```

## Evaluation signals

- Number of retained droplets matches expected values for the adult mouse brain dataset: EmptyNN 2,222; Cell Ranger v.2 2,371; EmptyDrops 2,181; Cell Ranger v.3 2,566; CellBender 4,386
- Fraction of spliced reads is significantly lower in EmptyNN- versus Cell Ranger v.3-filtered nuclei (p = 3.29e-5)
- EmptyNN output contains a boolean keep vector plus per-barcode probabilities only for barcodes in set U (total counts > 10)
- Summary CSV cell_calling_mouse_brain_results.csv has one row per method with droplet counts and spliced-read fractions
- CellBender total-droplets-included parameter is justified by inspection of the dataset-specific barcode rank plot

## Limitations

- The assumption that low-total-count barcodes are cell-free may fail when genuine low-count cells exist; the T threshold must be manually adjusted
- Number of cross-validation folds k and threshold T must be specified manually; no hyperparameter selection approach is implemented
- Retained droplets may have high mitochondrial read fraction and require separate downstream quality filtering
- EmptyNN does not correct ambient RNA in expression estimates; external tools such as SoupX are needed
- CellBender's expected number of cells and total-droplets-included must be chosen manually per dataset via the barcode rank plot

## Evidence
