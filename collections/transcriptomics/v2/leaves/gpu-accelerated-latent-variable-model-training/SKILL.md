---
name: gpu-accelerated-latent-variable-model-training
description: Use when a large single-cell dataset (e.g. Heart Cell Atlas, ~500,000
  cells) must undergo covariate-removal correction via a latent-variable model, or
  when wall-clock runtimes of scVI vs SiFT vs regress_out must be measured across
  increasing cell counts.
when_to_use_negative:
- When the goal is biological signal quality of the correction rather than scalability
  — the article notes no ground truth reference is available to evaluate computational
  filtering results, and integration metrics may be application-specific.
- When the dataset is small (e.g. thousands of cells) such that CPU regression baselines
  are cheap enough that GPU latent-variable training adds no practical runtime benefit.
- When the signal to remove requires cell-gene-expression-level correction rather
  than a joint latent representation — the article notes 'most integration methods,
  including scVI, provide a correction at the level of a joint latent representation
  of the cells'.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scVI (scvi-tools)
  role: The GPU-accelerated latent-variable model trained for covariate-removal correction;
    the method whose runtime is benchmarked (36 min on full Heart Cell Atlas)
  repo: https://github.com/scverse/scvi-tools
- name: SiFT (sift-sc)
  role: Fastest comparator method (12 min on full dataset); kernel-based projection
    filtering at the gene expression level, run with GPU via sift.SiFT with a KnnKernel
    built from the covariate mappings
  repo: https://github.com/nitzanlab/sift-sc
- name: scanpy regress_out
  role: CPU baseline that regresses the 10 covariates out of the expression matrix;
    slowest method (1 h 26 min on full dataset)
- name: pip
  role: Installation channel for sift-sc (pip install sift-sc) within the Python environment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:77e861da4a37b2b5928be2918fd30b5bbf2f20cce565902b45f4260edd891f86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/scvi-tools
  tools:
  - Python
  - SiFT
  - SiFT (sift-sc)
  - pip
  - scVI (scvi-tools)
  - scanpy regress_out
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# gpu-accelerated-latent-variable-model-training

## Summary

Trains scVI (scverse scvi-tools), a variational latent-variable model operating on a joint latent representation of cells, with GPU acceleration to perform covariate correction at scale — here benchmarked against SiFT (gene-expression-level filtering) and scanpy regress_out on Heart Cell Atlas subsamples up to ~486,134 cells.

## When to use

Use when a large single-cell dataset (e.g. Heart Cell Atlas, ~500,000 cells) must undergo covariate-removal correction via a latent-variable model, or when wall-clock runtimes of scVI vs SiFT vs regress_out must be measured across increasing cell counts.

## When NOT to use

- When the goal is biological signal quality of the correction rather than scalability — the article notes no ground truth reference is available to evaluate computational filtering results, and integration metrics may be application-specific.
- When the dataset is small (e.g. thousands of cells) such that CPU regression baselines are cheap enough that GPU latent-variable training adds no practical runtime benefit.
- When the signal to remove requires cell-gene-expression-level correction rather than a joint latent representation — the article notes 'most integration methods, including scVI, provide a correction at the level of a joint latent representation of the cells'.

## Inputs

- 8 Heart Cell Atlas subsamples as AnnData objects, ranging from 5,000 to 486,134 cells (full dataset ~500,000 cells)
- 10 continuous random covariates to be removed as nuisance signals
- GPU-enabled environment for scVI (PyTorch) and SiFT

## Outputs

- Per-method, per-subsample wall-clock runtime table (fig2c_runtimes.csv)
- Runtime versus number of cells plot reproducing Fig. 2c ordering: SiFT (12 min) < scVI (36 min) < regress_out (1 h 26 min) on the full Heart Cell Atlas

## How to apply

Load the 8 Heart Cell Atlas subsamples (5,000, 10,000, ..., 486,134 cells) together with the 10 continuous random covariates to be removed as nuisance signals into AnnData objects. Set up and train the scVI model (scverse scvi-tools, PyTorch/PyTorch Lightning backend) for the covariate-removal setting with GPU support, recording wall-clock time per subsample. Run the comparators on identical subsamples under identical hardware conventions: SiFT (sift-sc, e.g. sift.SiFT with a KnnKernel built from the covariate mappings, on GPU) and scanpy regress_out on CPU. Assemble per-method, per-subsample timings into fig2c_runtimes.csv and plot runtime versus number of cells as in Fig. 2c. The key decision point is hardware parity: in the reference benchmark SiFT and scVI run with GPU support while regress_out runs on CPU, which is why the full-dataset ordering is 12 min (SiFT) < 36 min (scVI) < 1 h 26 min (regress_out).

## Related tools

- **scVI (scvi-tools)** (The GPU-accelerated latent-variable model trained for covariate-removal correction; the method whose runtime is benchmarked (36 min on full Heart Cell Atlas)) — https://github.com/scverse/scvi-tools
- **SiFT (sift-sc)** (Fastest comparator method (12 min on full dataset); kernel-based projection filtering at the gene expression level, run with GPU via sift.SiFT with a KnnKernel built from the covariate mappings) — https://github.com/nitzanlab/sift-sc
- **scanpy regress_out** (CPU baseline that regresses the 10 covariates out of the expression matrix; slowest method (1 h 26 min on full dataset))
- **pip** (Installation channel for sift-sc (pip install sift-sc) within the Python environment)

## Examples

```
import scvi; scvi.model.SCVI.setup_anndata(adata); model = scvi.model.SCVI(adata); model.train(accelerator="gpu"); print(model.adata.n_obs)
```

## Evaluation signals

- Full-dataset (≈500,000 cells) runtimes reproduce the reported ordering: SiFT ≈12 min < scVI ≈36 min < regress_out ≈1 h 26 min
- fig2c_runtimes.csv contains one wall-clock row per method (SiFT, scVI, regress_out) per each of the 8 subsamples (5,000 through 486,134 cells)
- Runtime-versus-cell-count curves are monotonically increasing for all three methods, as in Fig. 2c
- scVI and SiFT timings were recorded with GPU support while regress_out timing is recorded on CPU, matching the reported hardware convention
- The same 10 continuous random covariates are removed by every method before timing comparison

## Limitations

- Runtime comparison is hardware-convention dependent: SiFT and scVI ran with GPU support while regress_out is CPU-bound, so absolute numbers are not a like-for-like methodological comparison.
- SiFT modifies gene expression so data is no longer properly log-transformed, a property expected in certain downstream analysis tasks.
- No ground truth reference exists to evaluate the filtering results; integration metrics used for evaluation may be application-specific.
- SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal.

## Evidence

- [other] scvi runtime finding on Heart Cell Atlas: [evidence span withheld — anchor status: not_found]
- [intro] scVI latent representation: "most integration methods, including scVI, provide a correction at the level of a joint latent representation of the cells"
- [readme] scvi-tools GPU acceleration: [evidence span withheld — anchor status: not_found]
- [abstract] SiFT scalability claim: "SiFT performs the correction at the gene expression level, can scale to large datasets, and compares favorably to state-of-the-art methods"
- [methods] SiFT installation via pip: "Install `SiFT` in one of the following ways: Through **pip**:: pip install sift-sc"
