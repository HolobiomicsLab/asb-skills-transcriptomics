---
name: single-cell-unwanted-variation-filtering
description: Use when single-cell RNA-seq data contain nuisance biological signals
  such as cell cycle or sex effects that must be removed before downstream analysis
  while preserving biological signal (e.g., genes of interest in corrected HVGs).
when_to_use_negative:
- When a fully accurate signal mapping is unavailable — an inaccurate or coarse-grained
  mapping may lead to loss of signal, since SiFT strongly relies on the prior knowledge
  used to define the cell-cell similarity kernel
- When downstream analysis requires properly log-transformed expression, since SiFT-corrected
  data are no longer properly log-transformed
- When a ground-truth reference for filtering quality is required — no such ground
  truth exists and evaluation must rely on application-specific integration metrics
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SiFT (sift-sc)
  role: Primary filtering tool; computes a KnnKernel from cell-cycle and sex marker
    genes, projects expression onto the signal, and deducts the projection to remove
    unwanted variation at the gene expression level
  repo: https://github.com/nitzanlab/sift-sc
- name: scVI
  role: Baseline batch-correction method; trained on the same counts with cell-cycle
    and sex scores supplied as continuous covariates for latent-space removal
  repo: https://github.com/scverse/scvi-tools
- name: Python
  role: Implementation language for the SiFT, regress_out, scVI, and iLISI evaluation
    workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:3cd5456c5f6b76c68b6559f82cd20d827024acab6319637fbd1dcbbcdfb24ec0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - Python
  - SiFT
  - SiFT (sift-sc)
  - scVI
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# single-cell-unwanted-variation-filtering

## Summary

A skill for removing unwanted biological variation (e.g., cell cycle, sex) from single-cell RNA-seq data using SiFT's kernel-based projection at the gene expression level, benchmarked against Scanpy regress_out and scVI with continuous covariates.

## When to use

Use when single-cell RNA-seq data contain nuisance biological signals such as cell cycle or sex effects that must be removed before downstream analysis while preserving biological signal (e.g., genes of interest in corrected HVGs).

## When NOT to use

- When a fully accurate signal mapping is unavailable — an inaccurate or coarse-grained mapping may lead to loss of signal, since SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel
- When downstream analysis requires properly log-transformed expression, since SiFT-corrected data are no longer properly log-transformed
- When a ground-truth reference for filtering quality is required — no such ground truth exists and evaluation must rely on application-specific integration metrics

## Inputs

- Single-cell RNA-seq count matrix (AnnData/.h5ad), e.g. Drosophila wing disc development data
- Curated cell-cycle marker gene list
- Curated sex marker gene list
- Genes of interest list for gene-preservation evaluation
- Cell-cycle and sex covariate scores (for regress_out and scVI baselines)

## Outputs

- Filtered (corrected) gene expression count matrix from SiFT knn-kernel projection
- regress_out-corrected and scVI (continuous-covariate) corrected matrices for comparison
- Graph iLISI scores per method for cell-cycle-only and cell-cycle+sex settings (Fig. 2b style plot)
- Summary table of % genes of interest retained in corrected HVGs per method

## How to apply

Load the scRNA-seq count matrix (e.g., Drosophila wing disc data, AnnData/.h5ad) together with curated cell-cycle and sex marker gene lists. Compute a cell-cell similarity kernel with respect to the signal to filter — for marker-gene-based filtering use sift.kernels.KnnKernel built from the cell-cycle and sex markers. Project the expression count matrix onto the signal kernel via matrix multiplication and deduct this projection to obtain the filtered count matrix. As baselines, run scanpy's regress_out on cell-cycle and sex covariates, and train scVI supplying cell-cycle and sex scores as continuous covariates. Evaluate each method with graph iLISI scores for the cell-cycle-only and cell-cycle+sex label settings, and quantify the overlap between genes of interest and corrected HVGs; the reference result is ~40% gene-of-interest overlap after SiFT filtering versus ~30% for regress_out (Fig. 2b).

## Related tools

- **SiFT (sift-sc)** (Primary filtering tool; computes a KnnKernel from cell-cycle and sex marker genes, projects expression onto the signal, and deducts the projection to remove unwanted variation at the gene expression level) — https://github.com/nitzanlab/sift-sc
- **scVI** (Baseline batch-correction method; trained on the same counts with cell-cycle and sex scores supplied as continuous covariates for latent-space removal) — https://github.com/scverse/scvi-tools
- **Python** (Implementation language for the SiFT, regress_out, scVI, and iLISI evaluation workflow)

## Examples

```
pip install sift-sc; import sift; sifter = sift.SiFT(data, kernel=sift.kernels.KnnKernel(n_neighbors=30)); sifter.filter(signal_genes=cellcycle_and_sex_markers)
```

## Evaluation signals

- Graph iLISI scores for SiFT compare favorably to regress_out and scVI for both cell-cycle-only and cell-cycle+sex label settings (reproducing Fig. 2b)
- ~40% of genes of interest remain in the corrected HVG set after SiFT filtering versus only ~30% for regress_out
- Filtered count matrix is produced at the gene expression level and same dimensions as the input
- Kernel construction uses the curated cell-cycle and sex marker genes (sift.kernels.KnnKernel) rather than arbitrary features

## Limitations

- No ground truth reference is available to evaluate computational filtering results; integration metrics used may be application-specific
- SiFT modifies gene expression so data are no longer properly log-transformed, a property expected in certain downstream analysis tasks
- Statistical implications of applying commonly used analysis tools to corrected gene expression data are unexplored
- SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel; inaccurate or coarse-grained mapping may lead to loss of signal

## Evidence

- [methods] knn kernel from markers: "sift.kernels.RBFKernel sift.kernels.LaplacianKernel sift.kernels.KnnKernel sift.kernels.MappingKernel"
- [intro] drosophila filter step: "applying SiFT to filter nuisance signals in Drosophila wing disc development single-cell data and removing cell cycle effects"
- [discussion] kernel projection rationale: "This kernel is then used to obtain a projection of gene expression onto the signal, which is then removed from the original expression."
- [abstract] comparison favorability: "SiFT performs the correction at the gene expression level, can scale to large datasets, and compares favorably to state-of-the-art methods"
- [readme] pip install: "Install SiFT via PyPI by running::

    pip install sift-sc"
- [intro] downstream analysis: "Now, diverse analysis tools can be applied to study the filtered data and explore the underlying biological signals it encodes"
