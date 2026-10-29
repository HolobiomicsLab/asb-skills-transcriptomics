---
name: covariate-regression-from-gene-expression
description: Use when comparing the wall-clock scalability of nuisance-covariate removal
  methods (SiFT, scVI, scanpy regress_out) on large scRNA-seq data, e.g. continuous
  covariates regressed from the expression matrix across subsamples up to ~500,000
  cells.
when_to_use_negative:
- The dataset is too small for a scalability benchmark (runtimes are dominated by
  overhead rather than scaling to ~500,000 cells), so the SiFT < scVI < regress_out
  ordering cannot be meaningfully tested
- The goal is batch-effect integration in a joint latent representation rather than
  removal of covariates at the gene-expression level — scVI's latent correction, not
  expression-level regression, is the appropriate target
- No nuisance covariates or prior knowledge defining the signal-to-filter mapping
  are available, since SiFT's cell-cell similarity kernel cannot be constructed without
  such a mapping
tools:
- name: SiFT (sift-sc)
  role: 'Fastest method under test: filters the 10 continuous covariates via a KnnKernel
    cell-cell similarity kernel built from the covariate mappings, run on GPU'
  repo: https://github.com/nitzanlab/sift-sc
- name: scVI (scvi-tools)
  role: Latent-space integration model trained per subsample for the same covariate-removal
    setting with GPU support; serves as the middle runtime reference
  repo: https://github.com/scverse/scvi-tools
- name: scanpy regress_out
  role: Baseline that regresses the 10 covariates directly out of the expression matrix
    per subsample; slowest method and therefore the upper bound of the runtime ordering
- name: pip
  role: Installation channel for the sift-sc package before benchmarking
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:d26baa30749f37d617c7288e13ec77065692e1d9afe8d331f1e2663663eaa9b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - Python
  - SiFT
  - SiFT (sift-sc)
  - pip
  - scVI (scvi-tools)
  - scanpy regress_out
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# covariate-regression-from-gene-expression

## Summary

Benchmarks the runtime scalability of covariate-removal methods that operate directly on the gene expression matrix (SiFT filtering, scanpy regress_out) versus latent-space correction (scVI) on increasingly large subsamples of a single-cell dataset, testing whether the reported SiFT < scVI < regress_out runtime ordering holds on the ~500,000-cell Heart Cell Atlas.

## When to use

Use when comparing the wall-clock scalability of nuisance-covariate removal methods (SiFT, scVI, scanpy regress_out) on large scRNA-seq data, e.g. continuous covariates regressed from the expression matrix across subsamples up to ~500,000 cells.

## When NOT to use

- The dataset is too small for a scalability benchmark (runtimes are dominated by overhead rather than scaling to ~500,000 cells), so the SiFT < scVI < regress_out ordering cannot be meaningfully tested
- The goal is batch-effect integration in a joint latent representation rather than removal of covariates at the gene-expression level — scVI's latent correction, not expression-level regression, is the appropriate target
- No nuisance covariates or prior knowledge defining the signal-to-filter mapping are available, since SiFT's cell-cell similarity kernel cannot be constructed without such a mapping

## Inputs

- 8 Heart Cell Atlas subsamples ranging from 5,000 to 486,134 cells (AnnData-format single-cell expression data)
- 10 continuous random covariates to be removed as nuisance signals
- GPU-enabled Python environment (for SiFT and scVI runs)

## Outputs

- fig2c_runtimes.csv: per-method, per-subsample wall-clock runtime table (SiFT, scVI, regress_out x 8 subsamples)
- Runtime-versus-number-of-cells plot reproducing Fig. 2c
- Verification (or refutation) of the full-dataset runtime ordering SiFT (~12 min) < scVI (~36 min) < regress_out (~1 h 26 min)

## How to apply

Load the 8 Heart Cell Atlas subsamples (5,000 to 486,134 cells) together with the 10 continuous random covariates designated as nuisance signals. Run SiFT (sift-sc, e.g. sift.SiFT with a KnnKernel built from the covariate mappings) on each subsample with GPU support, then train scVI (scverse scvi-tools) on each subsample for the same covariate-removal setting with GPU support, and run scanpy regress_out on each subsample to regress the 10 covariates out of the expression matrix — recording wall-clock time per method per subsample. Assemble the per-method, per-subsample measurements into fig2c_runtimes.csv and plot runtime versus number of cells as in Fig. 2c. The decision point is whether the measured full-dataset runtimes reproduce the reported ordering of ~12 min (SiFT) < 36 min (scVI) < 1 h 26 min (regress_out); the rationale is that SiFT and regress_out correct at the gene-expression level while scVI corrects only at a joint latent representation, so SiFT should scale best.

## Related tools

- **SiFT (sift-sc)** (Fastest method under test: filters the 10 continuous covariates via a KnnKernel cell-cell similarity kernel built from the covariate mappings, run on GPU) — https://github.com/nitzanlab/sift-sc
- **scVI (scvi-tools)** (Latent-space integration model trained per subsample for the same covariate-removal setting with GPU support; serves as the middle runtime reference) — https://github.com/scverse/scvi-tools
- **scanpy regress_out** (Baseline that regresses the 10 covariates directly out of the expression matrix per subsample; slowest method and therefore the upper bound of the runtime ordering)
- **pip** (Installation channel for the sift-sc package before benchmarking)

## Examples

```
pip install sift-sc
```

## Evaluation signals

- The full-dataset runtimes should land near the reported values: ~12 min for SiFT, ~36 min for scVI, ~1 h 26 min for regress_out, preserving the ordering SiFT < scVI < regress_out
- fig2c_runtimes.csv contains one wall-clock measurement for each of the 3 methods across each of the 8 subsamples (5,000 to 486,134 cells) — no missing cells in the table
- Runtime should increase monotonically (or near-monotonically) with the number of cells for every method, as visualized in the Fig. 2c-style plot
- SiFT and scVI runs must be executed with GPU support to match the reported benchmark conditions; regress_out runs on the expression matrix with the 10 covariates as regression keys

## Limitations

- SiFT and scVI were benchmarked with GPU support, so cross-method runtime comparisons are confounded by hardware acceleration availability; regress_out has no GPU pathway in this setup
- scVI corrects at the level of a joint latent representation of cells, not at the gene-expression level, so it is not a strictly equivalent covariate-removal operation to SiFT or regress_out
- Regressing covariates from expression modifies the gene expression so that the data is no longer properly log-transformed, which downstream tasks may expect
- SiFT's cell-cell similarity kernel strongly relies on prior knowledge; an inaccurate or coarse-grained mapping (e.g. covariate mappings) may lead to a loss of signal

## Evidence

- [other] Runtime ordering finding: [evidence span withheld — anchor status: not_found]
- [abstract] Expression-level correction and scalability: "SiFT performs the correction at the gene expression level, can scale to large datasets, and compares favorably to state-of-the-art methods"
- [intro] scVI latent-space correction: "most integration methods, including scVI, provide a correction at the level of a joint latent representation of the cells"
- [discussion] Open-source scaling claim: "We have made SiFT available as an open-source python package along with documentation and tutorials and ensured it can efficiently scale"
- [discussion] Log-transformation caveat: "it modifies the gene expression so that data is no longer properly log-transformed, a property that is expected in certain downstream analysis tasks"
- [methods] SiFT installation: "Install `SiFT` in one of the following ways: Through **pip**:: pip install sift-sc"
- [readme] scvi-tools description: [evidence span withheld — anchor status: not_found]
