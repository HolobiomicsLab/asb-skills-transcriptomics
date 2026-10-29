---
name: cell-cycle-variation-removal
description: Use when single-cell count matrices show cell-cycle-driven confounding
  (per-cell cell-cycle scores/phases available) that must be removed while preserving
  biological signal such as subclone or cell-type separability.
when_to_use_negative:
- When the goal is batch-effect correction or sample integration rather than removing
  a biological signal like cell cycle — integration methods (bbknn, Harmony, scVI)
  operate on joint latent representations instead
- When no prior knowledge of the signal to filter exists — SiFT strongly relies on
  the prior knowledge used to define the cell-cell similarity kernel, and an inaccurate
  or coarse-grained mapping may lead to a loss of signal
- When downstream analysis requires properly log-transformed expression, since SiFT-corrected
  data is no longer properly log-transformed
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SiFT
  role: Kernel-based projection method used to filter the cell-cycle signal from expression
    counts; provides sift.kernels.MappingKernel/KnnKernel and sift.SiFT
  repo: https://github.com/nitzanlab/sift-sc
- name: Cyclum
  role: Autoencoder-based cell-cycle phase inference used to derive binned phases
    and distances for kernel construction, and as the upper-reference baseline
  repo: https://github.com/KChen-lab/Cyclum
- name: ccRemover
  role: Baseline method for cell-cycle removal compared against SiFT on the semi-synthetic
    benchmark
- name: Seurat
  role: Source of cell-cycle labels/scores used for a MappingKernel and for the Seurat
    regress-out baseline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:1d209fe63422d933b8a249258b6696830f6b50753baaf835eb1ae4c5799ba46a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - Cyclum
  - Python
  - Seurat
  - SiFT
  - SiFT (sift-sc)
  - ccRemover
  - pip
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# cell-cycle-variation-removal

## Summary

Removes cell-cycle-driven unwanted variation from single-cell RNA-seq data by projecting expression onto a cell-cycle signal via a cell-cell similarity kernel (SiFT) and deducting the projection, benchmarked against dedicated cell-cycle removal baselines such as Seurat regression, ccRemover, and Cyclum.

## When to use

Use when single-cell count matrices show cell-cycle-driven confounding (per-cell cell-cycle scores/phases available) that must be removed while preserving biological signal such as subclone or cell-type separability.

## When NOT to use

- When the goal is batch-effect correction or sample integration rather than removing a biological signal like cell cycle — integration methods (bbknn, Harmony, scVI) operate on joint latent representations instead
- When no prior knowledge of the signal to filter exists — SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel, and an inaccurate or coarse-grained mapping may lead to a loss of signal
- When downstream analysis requires properly log-transformed expression, since SiFT-corrected data is no longer properly log-transformed

## Inputs

- Single-cell gene expression count matrix (e.g. 279-cell mESC virtual tumor dataset)
- Per-cell cell-cycle annotations (ground-truth cell-cycle mapping, Cyclum-binned phases, Cyclum distances, or Seurat cell-cycle labels)
- Cell-cycle gene scores for KNN kernel construction

## Outputs

- Filtered expression count matrix with cell-cycle signal removed (one per kernel variant)
- Table of cell-cycle removal scores and bio-conservation scores for all methods (e.g. fig3d_scores.csv)

## How to apply

Install SiFT via pip (pip install sift-sc) and import it as 'import sift'. Construct a cell-cell similarity kernel capturing similarity with respect to the cell-cycle signal — e.g. sift.kernels.MappingKernel using ground-truth cell-cycle labels or Cyclum-binned phases, or sift.kernels.KnnKernel on cell-cycle gene scores or Cyclum distances. Run sift.SiFT to project the expression count matrix onto the cell-cycle signal and deduct this projection, yielding a filtered count matrix. For benchmarking, compute a cell-cycle removal score and a bio-conservation score on the corrected data for each SiFT kernel variant and for baselines (Seurat regress-out of cell-cycle scores, ccRemover, covariate regression, with Cyclum as upper reference), then assemble all scores into a comparison table (e.g. fig3d_scores.csv).

## Related tools

- **SiFT** (Kernel-based projection method used to filter the cell-cycle signal from expression counts; provides sift.kernels.MappingKernel/KnnKernel and sift.SiFT) — https://github.com/nitzanlab/sift-sc
- **Cyclum** (Autoencoder-based cell-cycle phase inference used to derive binned phases and distances for kernel construction, and as the upper-reference baseline) — https://github.com/KChen-lab/Cyclum
- **ccRemover** (Baseline method for cell-cycle removal compared against SiFT on the semi-synthetic benchmark)
- **Seurat** (Source of cell-cycle labels/scores used for a MappingKernel and for the Seurat regress-out baseline)

## Examples

```
pip install sift-sc; import sift; sft = sift.SiFT(kernel=sift.kernels.MappingKernel(mapping=cellcycle_labels)); sft.filter(data=count_matrix)
```

## Evaluation signals

- Cell-cycle removal score on the corrected data is quantitatively higher than baseline methods (Seurat regress-out, ccRemover, covariate regression)
- Bio-conservation score on corrected data is higher when using ground truth or Cyclum-based mappings for filtering
- Subclone separability is preserved or enhanced in the filtered matrix on the two-subclone virtual tumor dataset
- Six kernel variants (mapping on ground-truth labels, KNN on gene scores, KNN on Cyclum phases, Cyclum distances, mapping on Cyclum bins, mapping on Seurat labels) each yield a corrected matrix with finite values of the same shape as the input counts

## Limitations

- SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal
- Lacks a ground truth reference for evaluation — integration metrics used for evaluation may be application-specific
- Corrected data is no longer properly log-transformed, a property expected in certain downstream analysis tasks
- Statistical implications of applying commonly used analysis tools to corrected gene expression data remain unexplored

## Evidence

- [intro] we compute a probabilistic cell–cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter: "we compute a probabilistic cell–cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter"
- [intro] we obtain a projection of the expression count matrix onto the signal we seek to filter using matrix multiplication: "we obtain a projection of the expression count matrix onto the signal we seek to filter using matrix multiplication"
- [methods] SiFT - Signal FilTering ... Install `SiFT` in one of the following ways: Through **pip**:: pip install sift-sc: "Install `SiFT` in one of the following ways: Through **pip**:: pip install sift-sc"
- [readme] Signal FilTering is a tool for uncovering hidden biological processes in single-cell data: "Signal FilTering is a tool for uncovering hidden biological processes in single-cell data."
- [discussion] it is important to note that SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel: "it is important to note that SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel."
