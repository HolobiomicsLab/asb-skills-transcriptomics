---
name: hto-demultiplexing-ground-truth-labeling
description: Use when benchmarking cell-calling/filtering methods on a multiplexed
  (cell hashing) scRNA-seq dataset where HTO counts provide singlet, doublet, and
  negative ground-truth labels for each raw barcode.
when_to_use_negative:
- The dataset lacks HTO-based multiplexing or other independent ground-truth labels
  (e.g., genetic variation from demuxlet), so no reliable singlet/doublet/negative
  labels can be assigned
- The goal is to correct ambient RNA contamination in gene expression estimates rather
  than benchmark cell calling — the paper notes EmptyNN does not estimate corrected
  profiles and SoupX should be applied instead
- The matrix is already a filtered cell-by-gene table, since cell-calling evaluation
  requires the raw unfiltered barcode space
tools:
- name: EmptyNN
  role: Cell-calling method under evaluation; its cell-free/cell-containing predictions
    on the raw cell hashing matrix are scored against HTO-derived labels (achieved
    AUROC 94.73%)
  repo: https://github.com/lkmklsmn/empty_nn
- name: Cell Ranger v.2
  role: Baseline caller using a total-count cutoff (threshold 200 in the original
    study) evaluated against HTO-derived labels
- name: Cell Ranger v.3
  role: Baseline caller with EmptyDrops integrated into its cell-calling algorithm,
    evaluated against HTO-derived labels
- name: EmptyDrops
  role: Baseline caller employing a Dirichlet-multinomial model to infer the cell-free
    droplet profile, evaluated against HTO-derived labels
- name: CellBender
  role: Neural-network-based baseline cell caller; expected number of cells set via
    the barcode rank plot (total-droplets-included), evaluated against HTO-derived
    labels
- name: pROC
  role: R procedure used to compute the AUROC of each cell-calling method against
    the HTO-derived singlet/doublet/negative labels
- name: demuxlet
  role: 'Alternative ground-truth source: infers singlet/doublet/negative labels from
    genetic variation in the multiplexed PBMC dataset (SRR5398237)'
  repo: https://github.com/yelabucsf/demuxlet_paper_code/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:a55dea3e5c0550eab7e90b6d14056a51cdff93a2acf5ec20370841bee49da1c9
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
  - pROC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# hto-demultiplexing-ground-truth-labeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derives per-barcode ground-truth labels (singlet, doublet, negative) from hashtag oligo (HTO) counts in the Stoeckius cell hashing dataset, then benchmarks cell-calling methods such as EmptyNN against these labels via AUROC computed with pROC.

## When to use

Use when benchmarking cell-calling/filtering methods on a multiplexed (cell hashing) scRNA-seq dataset where HTO counts provide singlet, doublet, and negative ground-truth labels for each raw barcode.

## When NOT to use

- The dataset lacks HTO-based multiplexing or other independent ground-truth labels (e.g., genetic variation from demuxlet), so no reliable singlet/doublet/negative labels can be assigned
- The goal is to correct ambient RNA contamination in gene expression estimates rather than benchmark cell calling — the paper notes EmptyNN does not estimate corrected profiles and SoupX should be applied instead
- The matrix is already a filtered cell-by-gene table, since cell-calling evaluation requires the raw unfiltered barcode space

## Inputs

- Raw unfiltered count matrix of the Stoeckius cell hashing dataset in mtx or h5 format
- HTO (hashtag oligo) counts per barcode for demultiplexing
- Method-specific outputs: EmptyNN predictions/probabilities, Cell Ranger v.2/v.3 calls, EmptyDrops calls, CellBender calls

## Outputs

- Per-barcode ground-truth labels (singlet, doublet, negative) from HTO counts
- Per-method AUROC values written to auroc_comparison.csv (expected: EmptyNN 94.73%, baselines 79.87%-90.16%)

## How to apply

Load the raw unfiltered count matrix of the Stoeckius cell hashing dataset with Read10X_h5, then integrate information derived from the HTO counts, which provide singlet, doublet, and negative labels for each barcode. Run the candidate callers on the same raw matrix: EmptyNN (transposed count matrix, threshold=100, k=10, iteration=10), Cell Ranger v.2 (total-count cutoff of 200 as used in the original study), Cell Ranger v.3 (EmptyDrops-integrated), standalone EmptyDrops (Dirichlet-multinomial model), and CellBender (expected number of cells chosen from the barcode rank plot via total-droplets-included). Compute the AUROC of each method's cell-free/cell-containing predictions against the HTO-derived labels using the pROC procedure. Rationale: HTO-derived labels supply independent ground truth so that method rankings reflect true droplet composition rather than count-distribution artifacts. Correct application yields EmptyNN at 94.73% AUROC with baselines in the 79.87%-90.16% range, confirming EmptyNN's superiority.

## Related tools

- **EmptyNN** (Cell-calling method under evaluation; its cell-free/cell-containing predictions on the raw cell hashing matrix are scored against HTO-derived labels (achieved AUROC 94.73%)) — https://github.com/lkmklsmn/empty_nn
- **Cell Ranger v.2** (Baseline caller using a total-count cutoff (threshold 200 in the original study) evaluated against HTO-derived labels)
- **Cell Ranger v.3** (Baseline caller with EmptyDrops integrated into its cell-calling algorithm, evaluated against HTO-derived labels)
- **EmptyDrops** (Baseline caller employing a Dirichlet-multinomial model to infer the cell-free droplet profile, evaluated against HTO-derived labels)
- **CellBender** (Neural-network-based baseline cell caller; expected number of cells set via the barcode rank plot (total-droplets-included), evaluated against HTO-derived labels)
- **pROC** (R procedure used to compute the AUROC of each cell-calling method against the HTO-derived singlet/doublet/negative labels)
- **demuxlet** (Alternative ground-truth source: infers singlet/doublet/negative labels from genetic variation in the multiplexed PBMC dataset (SRR5398237)) — https://github.com/yelabucsf/demuxlet_paper_code/

## Examples

```
counts <- t(Read10X_h5("./data/cell_hashing_raw.h5")); nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE); auroc <- pROC::roc(hto_labels, nn.res$nn.prob)
```

## Evaluation signals

- Computed EmptyNN AUROC is approximately 94.73% on the cell hashing dataset
- Each baseline (Cell Ranger v.2, Cell Ranger v.3, EmptyDrops, CellBender) falls within the reported 79.87%-90.16% range
- Per-method AUROC values are written to auroc_comparison.csv with EmptyNN ranked above all baselines
- Every barcode in the 39,842-barcode raw matrix receives a singlet, doublet, or negative label from the HTO counts
- EmptyNN retains 19,009 (47.7%) and removes 20,833 (52.3%) of the 39,842 barcodes when run with default parameters (threshold=100, k=10, iteration=10)

## Limitations

- Benchmarking ground truth is limited to cell hashing or genetic variation labels; high overlap in cell-calling classifications across methods may limit biological interpretation of method differences
- Low-count genuine cells can confound cell-free assumptions; the T threshold (default 100) may need manual adjustment when cell-containing droplets with very low total counts exist
- For CellBender, the expected number of cells (total-droplets-included) must be chosen manually per dataset by inspecting the barcode rank plot

## Evidence
