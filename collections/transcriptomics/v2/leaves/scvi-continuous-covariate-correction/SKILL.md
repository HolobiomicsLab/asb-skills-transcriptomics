---
name: scvi-continuous-covariate-correction
description: Use when single-cell RNA-seq data contain quantifiable continuous nuisance
  covariates (e.g., cell-cycle or sex scores) and a latent-space batch-correction
  baseline is needed to compare against gene-level filtering methods.
when_to_use_negative:
- When correction must occur at the gene expression level (e.g., downstream tools
  require corrected genes or properly log-transformed expression) — scVI corrects
  only in a joint latent representation
- When nuisance variation cannot be encoded as continuous per-cell covariate scores
- When no integration metrics appropriate to the application are available to evaluate
  whether unwanted variation was actually removed
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scVI (scvi-tools)
  role: Variational inference model trained with continuous covariates (cell-cycle
    and sex scores) to remove unwanted variation in latent space as a baseline in
    the SiFT comparison
  repo: https://github.com/scverse/scvi-tools
- name: SiFT (sift-sc)
  role: Gene-level kernel-based filtering method that scVI is compared against on
    Drosophila wing disc data using iLISI and gene-preservation scores
  repo: https://github.com/nitzanlab/sift-sc
- name: Python
  role: Implementation language for the scVI workflow and downstream iLISI/HVG-overlap
    evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:5a186cb39abc116ba921a854364cc9b2c033ee4045d0f3755edf3b893a6b2b58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/scvi-tools
  tools:
  - Python
  - SiFT
  - SiFT (sift-sc)
  - scVI
  - scVI (scvi-tools)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# scvi-continuous-covariate-correction

## Summary

Trains an scVI variational autoencoder with continuous covariates (e.g., cell-cycle and sex scores) to remove unwanted biological variation in a joint latent representation, serving as a deep-learning baseline for gene-level filtering methods like SiFT.

## When to use

Use when single-cell RNA-seq data contain quantifiable continuous nuisance covariates (e.g., cell-cycle or sex scores) and a latent-space batch-correction baseline is needed to compare against gene-level filtering methods.

## When NOT to use

- When correction must occur at the gene expression level (e.g., downstream tools require corrected genes or properly log-transformed expression) — scVI corrects only in a joint latent representation
- When nuisance variation cannot be encoded as continuous per-cell covariate scores
- When no integration metrics appropriate to the application are available to evaluate whether unwanted variation was actually removed

## Inputs

- AnnData object (adata) with raw single-cell RNA-seq count matrix
- Continuous covariate scores per cell (e.g., cell-cycle score, sex score) stored in adata.obs
- Optional: list of genes of interest and HVG set for preservation evaluation

## Outputs

- Trained scVI SCVI model
- Latent cell representation with cell-cycle and sex variation removed (get_latent_representation)
- iLISI graph scores per covariate setting (cell-cycle-only and cell-cycle+sex)
- Overlap statistics of genes of interest with the corrected top-variable-gene set

## How to apply

Load the single-cell count matrix into AnnData and attach continuous covariate scores (e.g., cell-cycle and sex scores curated from marker gene lists) to the obs table. Register these via scvi.model.SCVI.setup_anndata with continuous_covariate_keys so the encoder conditions on them, then train the SCVI model and extract the latent representation in which the covariate-driven variation is removed. In the SiFT paper this workflow was applied to Drosophila wing disc single-cell data with cell-cycle and sex covariates, and evaluated with graph iLISI scores for cell-cycle-only and cell-cycle+sex settings. Because scVI corrects only in latent space, also quantify how many genes of interest overlap the top variable genes after correction, since methods that operate at the gene expression level (e.g., SiFT via sift.kernels.KnnKernel) preserve more biological signal (~40% of genes of interest retained in corrected HVGs versus ~30% for regress_out).

## Related tools

- **scVI (scvi-tools)** (Variational inference model trained with continuous covariates (cell-cycle and sex scores) to remove unwanted variation in latent space as a baseline in the SiFT comparison) — https://github.com/scverse/scvi-tools
- **SiFT (sift-sc)** (Gene-level kernel-based filtering method that scVI is compared against on Drosophila wing disc data using iLISI and gene-preservation scores) — https://github.com/nitzanlab/sift-sc
- **Python** (Implementation language for the scVI workflow and downstream iLISI/HVG-overlap evaluation)

## Examples

```
import scvi; scvi.model.SCVI.setup_anndata(adata, continuous_covariate_keys=['cell_cycle_score','sex_score']); model = scvi.model.SCVI(adata); model.train(); latent = model.get_latent_representation()
```

## Evaluation signals

- Graph iLISI scores improve for cell-cycle-only and cell-cycle+sex label settings after conditioning on the continuous covariates
- Latent representation shows mixing of cells across cell-cycle phases and sexes (e.g., via kNN-graph iLISI)
- Overlap fraction of genes of interest with the corrected HVG set is quantified and compared to gene-level methods (SiFT ~40% vs regress_out ~30% reported; scVI typically lower since correction is latent)
- setup_anndata registers the continuous_covariate_keys without errors and training converges (validation loss stabilizes)

## Limitations

- scVI provides a correction at the level of a joint latent representation of the cells, so corrected gene expression is not produced and downstream gene-level analyses are limited
- Evaluation of unwanted-variation removal lacks a ground truth reference; results depend on application-specific integration metrics such as iLISI
- In the Drosophila wing disc benchmark, latent methods like scVI preserved less biological variation than gene-level filtering (SiFT retained ~40% of genes of interest in corrected HVGs)

## Evidence

- [intro] scVI corrects in latent space: "most integration methods, including scVI, provide a correction at the level of a joint latent representation of the cells"
- [readme] scvi-tools package description: [evidence span withheld — anchor status: not_found]
- [discussion] Evaluation relies on application-specific integration metrics: "To mitigate this, we suggest using integration metrics (which in general may be application-specific), as has been used throughout this manuscript"
- [methods] Continuous covariate setup in the benchmark workflow: "taken from https://github.com/scverse/scvi-tools/blob/master/docs/_templates/autosummary/class.rst"
