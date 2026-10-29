---
name: auroc-computation-for-barcode-classification
description: Use when a raw unfiltered gene-by-barcode count matrix has independent
  per-barcode ground-truth labels (e.g., demuxlet singlet probabilities or HTO singlet/doublet/negative
  calls) and you need to quantify how well competing cell-calling methods separate
  cell-free from cell-containing droplets.
when_to_use_negative:
- The dataset has no independent ground truth (no HTO hashing counts and no genetic
  variation/demuxlet labels), so AUROC against true cell-free/cell-containing status
  cannot be computed
- The goal is downstream biological analysis rather than method benchmarking — EmptyNN
  does not correct ambient RNA contamination and low-quality cells must still be removed
  separately
- Input is an already-filtered count matrix; AUROC benchmarking requires the raw unfiltered
  matrix so all barcodes, including cell-free droplets, are evaluated
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3170
tools:
- name: EmptyNN
  role: Method under benchmark; predicts cell-free vs cell-containing droplets with
    default T = 100, k = 10, N = 10
  repo: https://github.com/lkmklsmn/empty_nn
- name: Cell Ranger v.2
  role: Baseline cell-calling method using a total-count distribution cutoff; also
    regenerates the raw count matrix from the BAM
- name: EmptyDrops
  role: Baseline cell-calling method using a Dirichlet-multinomial model to infer
    the cell-free transcriptome profile
- name: Cell Ranger v.3
  role: Baseline cell-calling method integrating EmptyDrops into its algorithm
- name: CellBender
  role: Baseline neural-network-based cell-calling method; parameters set from barcode
    rank plot inspection
- name: demuxlet
  role: Infers per-barcode genotype labels and singlet probabilities from genetic
    variation, serving as ground truth for AUROC computation
  repo: https://github.com/yelabucsf/demuxlet_paper_code/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:3db30069e9dc1f6dd2fb1c36623abd09c454e95f2fd46ea93e09d38ece4115cf
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# auroc-computation-for-barcode-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark cell-calling methods on droplet-based scRNA-seq data by computing AUROC of cell-free vs cell-containing droplet predictions against ground-truth labels (HTO-derived or demuxlet-derived genetic variation labels), as done to show EmptyNN reached 96.30% AUROC on the multiplexed PBMC dataset versus Cell Ranger v.2 (82.94%), EmptyDrops (86.43%), Cell Ranger v.3 (83.03%), and CellBender (58.02%).

## When to use

Use when a raw unfiltered gene-by-barcode count matrix has independent per-barcode ground-truth labels (e.g., demuxlet singlet probabilities or HTO singlet/doublet/negative calls) and you need to quantify how well competing cell-calling methods separate cell-free from cell-containing droplets.

## When NOT to use

- The dataset has no independent ground truth (no HTO hashing counts and no genetic variation/demuxlet labels), so AUROC against true cell-free/cell-containing status cannot be computed
- The goal is downstream biological analysis rather than method benchmarking — EmptyNN does not correct ambient RNA contamination and low-quality cells must still be removed separately
- Input is an already-filtered count matrix; AUROC benchmarking requires the raw unfiltered matrix so all barcodes, including cell-free droplets, are evaluated

## Inputs

- Raw unfiltered gene-by-barcode count matrix (mtx or h5) from Cell Ranger v.2 on SRR5398237
- Genome-aligned BAM file from Sequence Read Archive (SRR5398237)
- Merged VCF file from the demuxlet paper-code repository
- Per-barcode ground-truth labels and singlet probabilities from demuxlet
- Cell/cell-free predictions and probabilities from each cell-calling method (EmptyNN, Cell Ranger v.2, EmptyDrops, Cell Ranger v.3, CellBender)

## Outputs

- AUROC value per cell-calling method (e.g., written to auroc_results.csv)
- Comparison table showing EmptyNN 96.30%, EmptyDrops 86.43%, Cell Ranger v.3 83.03%, Cell Ranger v.2 82.94%, CellBender 58.02%

## How to apply

First establish ground truth: for the Kang et al. multiplexed PBMC dataset (SRR5398237), download the genome-aligned BAM, regenerate the raw matrix with Cell Ranger v.2, and run demuxlet with the merged VCF from the yelabucsf/demuxlet_paper_code repository to assign per-barcode genotype labels and singlet probabilities. Then run each method on the identical raw count matrix — EmptyNN with default parameters (threshold T = 100, k = 10 cross-validation folds, N = 10 iterations), plus Cell Ranger v.2, EmptyDrops, Cell Ranger v.3, and CellBender (setting CellBender's expected-number-of-cells and total-droplets-included by inspecting the barcode rank plot). Finally, compute the AUROC of each method's predictions against the demuxlet-derived cell-containing/cell-free labels and record values (e.g., in auroc_results.csv). Rationale: AUROC is threshold-free and directly comparable across methods whose cell/non-cell cutoffs differ in nature (total-count cutoff, Dirichlet-multinomial significance test, or neural-network posterior probability).

## Related tools

- **EmptyNN** (Method under benchmark; predicts cell-free vs cell-containing droplets with default T = 100, k = 10, N = 10) — https://github.com/lkmklsmn/empty_nn
- **Cell Ranger v.2** (Baseline cell-calling method using a total-count distribution cutoff; also regenerates the raw count matrix from the BAM)
- **EmptyDrops** (Baseline cell-calling method using a Dirichlet-multinomial model to infer the cell-free transcriptome profile)
- **Cell Ranger v.3** (Baseline cell-calling method integrating EmptyDrops into its algorithm)
- **CellBender** (Baseline neural-network-based cell-calling method; parameters set from barcode rank plot inspection)
- **demuxlet** (Infers per-barcode genotype labels and singlet probabilities from genetic variation, serving as ground truth for AUROC computation) — https://github.com/yelabucsf/demuxlet_paper_code/

## Examples

```
R: nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE); then compute AUROC of nn.res$probabilities against demuxlet labels and write auroc_results.csv
```

## Evaluation signals

- Computed AUROC on the multiplexed PBMC dataset reproduces the reported values within reasonable tolerance: EmptyNN 96.30%, Cell Ranger v.2 82.94%, EmptyDrops 86.43%, Cell Ranger v.3 83.03%, CellBender 58.02%
- EmptyNN retains 6,025 cell-containing droplets in the multiplexed PBMC dataset when demuxlet labels are used
- AUROC results are persisted in a machine-readable file (auroc_results.csv) with one row per method
- All methods are evaluated on the same raw unfiltered count matrix and the same demuxlet-derived ground-truth label set (no per-method barcode subsets)
- CellBender parameters (expected-number-of-cells, total-droplets-included) are documented as derived from barcode rank plot inspection

## Limitations

- Ground truth is limited to cell hashing or genetic variation labels; because cell-calling classifications across methods overlap highly, differences may have limited biological interpretation
- EmptyNN's assumption that low total-count barcodes are cell-free may fail when genuine low-count cells exist; the T threshold must be adjusted manually
- CellBender's expected-number-of-cells and total-droplets-included parameters must be chosen manually per dataset via barcode rank plot inspection, introducing subjective tuning

## Evidence

- [results] EmptyNN retained 6,025 cell-containing droplets and achieved an AUROC of 96.30%: [evidence span withheld — anchor status: not_found]
- [results] demuxlet applied to infer barcode labels: "we applied demuxlet to infer the label of each barcode and evaluated the performance of the cell-calling methods."
- [discussion] merged VCF from demuxlet repo: "The merged VCF file containing all samples was downloaded from the demuxlet GitHub repository (https://github.com/yelabucsf/demuxlet_paper_code/)"
- [readme] EmptyNN default parameters: "nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE)"
- [discussion] CellBender parameter selection: "The expected number of cells depends on each dataset. The barcode rank plot for each dataset was examined to determine the optimal parameter total-droplets-included"
