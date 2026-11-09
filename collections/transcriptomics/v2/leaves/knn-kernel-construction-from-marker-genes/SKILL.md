---
name: knn-kernel-construction-from-marker-genes
description: Use when a scRNA-seq dataset contains unwanted biological variation (e.g.,
  cell cycle, sex) that can be encoded by curated marker gene lists, and you need
  a cell-cell similarity kernel for SiFT gene-level filtering or covariate scores
  for regress_out/scVI baselines.
when_to_use_negative:
- The signal to filter cannot be encoded by prior-knowledge marker genes or a known
  mapping — SiFT strongly relies on the prior knowledge used to define the cell-cell
  similarity kernel, and an inaccurate or coarse-grained mapping may lead to a loss
  of signal.
- You only need latent-space batch integration without gene-level correction — scVI-style
  joint latent representation may be more appropriate since most integration methods
  correct at the latent level.
- Downstream analysis strictly requires properly log-transformed expression — SiFT
  modifies gene expression so data is no longer properly log-transformed.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SiFT (sift-sc)
  role: Python package providing sift.kernels.KnnKernel used to build the marker-gene-defined
    kNN similarity kernel and perform projection-based filtering
  repo: https://github.com/nitzanlab/sift-sc
- name: scVI
  role: 'Baseline comparison: trained on the same counts with cell-cycle and sex scores
    supplied as continuous covariates for latent-space correction'
  repo: https://github.com/scverse/scvi-tools
- name: Scanpy
  role: Provides regress_out baseline on cell-cycle and sex covariates and downstream
    analysis/HVG computation on the filtered data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:3e39759277b2583f94947144bf9ffc725ba1355863cd5db92f25af2a17ac1111
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - Python
  - Scanpy
  - SiFT
  - SiFT (sift-sc)
  - scVI
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# knn-kernel-construction-from-marker-genes

## Summary

Constructs a probabilistic k-nearest-neighbor cell-cell similarity kernel using marker gene lists (e.g., cell-cycle and sex markers) as the signal encoding, for use as the kernel argument in SiFT's projection-based filtering of unwanted variation from scRNA-seq count matrices.

## When to use

Use when a scRNA-seq dataset contains unwanted biological variation (e.g., cell cycle, sex) that can be encoded by curated marker gene lists, and you need a cell-cell similarity kernel for SiFT gene-level filtering or covariate scores for regress_out/scVI baselines.

## When NOT to use

- The signal to filter cannot be encoded by prior-knowledge marker genes or a known mapping — SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel, and an inaccurate or coarse-grained mapping may lead to a loss of signal.
- You only need latent-space batch integration without gene-level correction — scVI-style joint latent representation may be more appropriate since most integration methods correct at the latent level.
- Downstream analysis strictly requires properly log-transformed expression — SiFT modifies gene expression so data is no longer properly log-transformed.

## Inputs

- scRNA-seq count matrix (e.g., Drosophila wing disc dataset, AnnData/h5ad)
- Curated cell-cycle marker gene list
- Curated sex marker gene list
- Choice of kernel type (sift.kernels.KnnKernel) and kNN parameters

## Outputs

- Probabilistic cell-cell similarity kNN kernel encoding the marker-defined signal
- Filtered gene expression count matrix (projection deducted from original counts)
- Downstream iLISI graph scores and gene-of-interest vs HVG overlap statistics

## How to apply

Take the curated marker gene lists (e.g., cell-cycle and sex marker genes for the Drosophila wing disc data) and use them to encode the signal to be filtered; SiFT uses this mapping to compute a cell-cell similarity kernel, here via sift.kernels.KnnKernel. The kernel captures similarity between cells with respect to the marker-defined signal, so kNN neighbors reflect cell-cycle/sex state rather than global expression similarity. The kernel is then used to obtain a projection of the gene expression count matrix onto the signal, which is deducted from the original expression to yield a filtered count matrix that can be compared against Scanpy's regress_out and scVI (with covariates supplied as continuous latent covariates). Evaluate the kernel's effectiveness downstream by computing graph iLISI scores on the filtered data and measuring the fraction of genes of interest that remain in the corrected HVG set (reported ~40% after SiFT vs ~30% after regress_out).

## Related tools

- **SiFT (sift-sc)** (Python package providing sift.kernels.KnnKernel used to build the marker-gene-defined kNN similarity kernel and perform projection-based filtering) — https://github.com/nitzanlab/sift-sc
- **scVI** (Baseline comparison: trained on the same counts with cell-cycle and sex scores supplied as continuous covariates for latent-space correction) — https://github.com/scverse/scvi-tools
- **Scanpy** (Provides regress_out baseline on cell-cycle and sex covariates and downstream analysis/HVG computation on the filtered data)

## Examples

```
pip install sift-sc; from sift import SiFT; s = SiFT(data=adata); s.fit(kernel={'type': 'knn', 'key': 'cell_cycle_sex_markers'}); s.filter()
```

## Evaluation signals

- Graph iLISI scores computed on the filtered data for cell-cycle-only and cell-cycle+sex label settings should show SiFT comparing favorably to regress_out and scVI (Fig. 2b).
- Overlap of genes of interest with the corrected top variable gene (HVG) set should land near ~40% after SiFT filtering vs ~30% for regress_out.
- Kernel neighbors should reflect marker-gene-defined state (cell cycle/sex), i.e., iLISI mixing across cell-cycle/sex labels increases after filtering.
- Filtered count matrix retains biological signal: preserved genes of interest should remain detectable in downstream HVGs rather than being removed alongside the nuisance signal.

## Limitations

- SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal.
- No ground truth reference is available to evaluate computational filtering results; integration metrics used for evaluation may be application-specific.
- SiFT modifies gene expression so data is no longer properly log-transformed, a property expected in certain downstream analysis tasks.
- Statistical implications of applying commonly used analysis tools to corrected gene expression data are unexplored.

## Evidence

- [intro] kernel from marker genes: "we compute a probabilistic cell–cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter"
- [methods] kernel types in methods: "sift.kernels.RBFKernel
    sift.kernels.LaplacianKernel
    sift.kernels.KnnKernel
    sift.kernels.MappingKernel"
- [intro] filter cell cycle and sex: "applying SiFT to filter nuisance signals in Drosophila wing disc development single-cell data"
- [intro] projection and deduction: "we obtain a projection of the expression count matrix onto the signal we seek to filter using matrix multiplication"
- [discussion] prior-knowledge reliance: "it is important to note that SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel."
- [readme] install via pip: "Install SiFT via PyPI by running::

    pip install sift-sc"
