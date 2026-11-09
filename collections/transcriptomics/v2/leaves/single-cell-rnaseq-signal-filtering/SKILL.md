---
name: single-cell-rnaseq-signal-filtering
description: Use when scRNA-seq count data contain a nuisance or confounding signal
  (cell cycle, sex, spatial zonation, batch, healthy reference) that must be removed
  at the gene-expression level while preserving the biological attribute of interest.
when_to_use_negative:
- When no prior knowledge is available to define the cell-cell similarity kernel —
  SiFT strongly relies on this prior knowledge, and an inaccurate or coarse-grained
  mapping may lead to loss of signal
- When downstream tasks require properly log-transformed data — filtering modifies
  gene expression so the corrected matrix is no longer properly log-transformed
- When a dedicated integration model at the latent-representation level is preferred
  (e.g., scVI-style joint latent correction) rather than gene-expression-level correction
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SiFT (sift-sc)
  role: 'Primary framework: constructs the cell-cell similarity kernel, projects expression
    onto the nuisance signal, and deducts the projection'
  repo: https://github.com/nitzanlab/sift-sc
- name: scVI
  role: Baseline and scalability comparison method for covariate removal at the joint
    latent representation level
  repo: https://github.com/scverse/scvi-tools
- name: Cyclum
  role: Provides cell-cycle phase bins and circular pseudotime distances used to build
    SiFT kernels, and serves as the upper-reference baseline for cell-cycle removal
  repo: https://github.com/KChen-lab/Cyclum
- name: pip
  role: Installation channel for the sift-sc package
- name: Python
  role: Language in which the filtering pipeline and downstream evaluation are implemented
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:2d83a882082f14f85b26b967f9c39d177c1427020b797efdd26c48d5d9864455
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - Cyclum
  - Python
  - SiFT
  - SiFT (sift-sc)
  - pip
  - scVI
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# single-cell-rnaseq-signal-filtering

## Summary

Kernel-based probabilistic filtering of biological signals from single-cell RNA-seq data using SiFT: define a cell-cell similarity kernel for the signal to remove, project expression onto that signal, and deduct the projection to expose hidden biological processes such as subclones, circadian rhythms, or disease dynamics.

## When to use

Use when scRNA-seq count data contain a nuisance or confounding signal (cell cycle, sex, spatial zonation, batch, healthy reference) that must be removed at the gene-expression level while preserving the biological attribute of interest.

## When NOT to use

- When no prior knowledge is available to define the cell-cell similarity kernel — SiFT strongly relies on this prior knowledge, and an inaccurate or coarse-grained mapping may lead to loss of signal
- When downstream tasks require properly log-transformed data — filtering modifies gene expression so the corrected matrix is no longer properly log-transformed
- When a dedicated integration model at the latent-representation level is preferred (e.g., scVI-style joint latent correction) rather than gene-expression-level correction

## Inputs

- Gene-by-cell expression count matrix (e.g., 279-cell mESC virtual tumor count matrix, Heart Cell Atlas subsamples up to ~486,134 cells)
- Per-cell annotations of the signal to filter (cell-cycle phase labels, Cyclum bins or distances, cell-cycle gene scores, spatial coordinates, healthy/disease sample labels)
- Continuous covariate matrices (e.g., 10 random covariates for the Heart Cell Atlas scalability benchmark)

## Outputs

- Filtered gene expression count matrix with the nuisance signal's projection deducted
- Quantitative removal and bio-conservation scores (e.g., fig3d_scores.csv comparing SiFT kernel variants against Seurat regress-out, ccRemover, and Cyclum)
- Runtime measurements per method and dataset size (e.g., fig2c_runtimes.csv for the SiFT vs. scVI vs. regress_out scalability benchmark)

## How to apply

Install the sift-sc package with pip and import it as 'import sift'. First choose and construct a cell-cell similarity kernel encoding the signal to filter: sift.kernels.MappingKernel when per-cell labels or mappings exist (e.g., ground-truth cell-cycle mapping, Cyclum bins, Seurat cell-cycle labels, healthy-sample reference mapping), sift.kernels.KnnKernel when only per-cell scores or distances are available (e.g., cell-cycle gene scores, Cyclum distances), or RBFKernel/LaplacianKernel for continuous covariates. Then run sift.SiFT to project the expression count matrix onto the signal kernel and deduct the projection, yielding a filtered count matrix. Finally, apply downstream analysis (clustering, trajectory inference, integration metrics) to the filtered data to assess whether the nuisance signal was removed and the biological signal of interest was conserved; compare against baselines such as Seurat regress-out, ccRemover, scVI, or scanpy regress_out using cell-cycle removal and bio-conservation scores.

## Related tools

- **SiFT (sift-sc)** (Primary framework: constructs the cell-cell similarity kernel, projects expression onto the nuisance signal, and deducts the projection) — https://github.com/nitzanlab/sift-sc
- **scVI** (Baseline and scalability comparison method for covariate removal at the joint latent representation level) — https://github.com/scverse/scvi-tools
- **Cyclum** (Provides cell-cycle phase bins and circular pseudotime distances used to build SiFT kernels, and serves as the upper-reference baseline for cell-cycle removal) — https://github.com/KChen-lab/Cyclum
- **pip** (Installation channel for the sift-sc package)
- **Python** (Language in which the filtering pipeline and downstream evaluation are implemented)

## Examples

```
pip install sift-sc; import sift; kernel = sift.kernels.MappingKernel(mapping=cellcycle_labels); sft = sift.SiFT(kernel=kernel); X_filtered = sft.filter(X)
```

## Evaluation signals

- Cell-cycle removal score on the corrected matrix is quantitatively higher for SiFT kernel variants than for Seurat regress-out, ccRemover, and regress_out-style baselines
- Bio-conservation score on the corrected matrix is higher when filtering with ground-truth or Cyclum-based mappings, indicating subclone separability is preserved or enhanced
- Runtime ordering on the full Heart Cell Atlas (~500,000 cells) is SiFT (~12 min) < scVI (~36 min) < regress_out (~1 h 26 min)
- Filtered matrix has the same cells and genes as the input count matrix (projection was deducted, not a latent embedding substituted)
- All method scores assemble into the expected table schema (e.g., fig3d_scores.csv with one row per SiFT kernel variant and per baseline)

## Limitations

- Lacks a ground truth reference for evaluating filtering results in real datasets; integration metrics used for evaluation may be application-specific
- Filtering modifies gene expression so data are no longer properly log-transformed, a property expected in certain downstream tasks; statistical implications of applying standard tools to corrected expression are unexplored
- Strongly relies on the prior knowledge used to define the cell-cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal

## Evidence

- [abstract] Kernel-based projection filtering rationale: "we present SiFT, a kernel-based projection method for filtering biological signals in single-cell data"
- [intro] Projection and deduction step: "we obtain a projection of the expression count matrix onto the signal we seek to filter using matrix multiplication"
- [intro] Kernel captures signal-specific similarity: "we compute a probabilistic cell–cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter"
- [methods] Available kernel classes: "sift.kernels.RBFKernel
sift.kernels.LaplacianKernel
sift.kernels.KnnKernel
sift.kernels.MappingKernel"
- [methods] Installation via pip: "Install `SiFT` in one of the following ways: Through **pip**:: pip install sift-sc"
- [discussion] Projection onto the signal then removed from expression: "This kernel is then used to obtain a projection of gene expression onto the signal, which is then removed from the original expression."
- [discussion] Package and scalability claim: "We have made SiFT available as an open-source python package along with documentation and tutorials and ensured it can efficiently scale"
- [readme] Install via PyPI: "Install SiFT via PyPI by running::

    pip install sift-sc"
