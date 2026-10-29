---
name: scanpy-covariate-regression
description: Use when a scRNA-seq count matrix has continuous nuisance covariate scores
  (e.g., cell cycle or sex scores) that must be regressed out as a pre-processing
  baseline before clustering or trajectory analysis.
when_to_use_negative:
- When the goal is to preserve maximum biological signal at gene-expression level
  — regress_out retained only 30% of genes of interest in corrected HVGs versus 40%
  for SiFT knn-kernel filtering
- When correction is needed only in latent space rather than on gene expression —
  scVI performs correction at the level of a joint latent representation
- When no reliable marker gene lists or covariate scores exist to define the nuisance
  signal to regress
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Scanpy
  role: Provides regress_out, the covariate-regression baseline that removes cell-cycle
    and sex scores from gene expression
- name: SiFT (sift-sc)
  role: Comparison method whose knn-kernel filtering is benchmarked against regress_out
    on iLISI and gene-preservation metrics
  repo: https://github.com/nitzanlab/sift-sc
- name: scVI
  role: Latent-space baseline trained with continuous cell-cycle and sex covariates
    for comparison against regress_out
  repo: https://github.com/scverse/scvi-tools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:1946ff05e87448727b5f68673a54b0b85ea6bf5c561a5b479c885a8d24b54454
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
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# scanpy-covariate-regression

## Summary

Baseline approach for removing unwanted variation (e.g., cell cycle, sex) from single-cell RNA-seq data by regressing covariate scores out of gene expression with Scanpy's regress_out, evaluated here as a comparison baseline against SiFT and scVI on Drosophila wing disc data.

## When to use

Use when a scRNA-seq count matrix has continuous nuisance covariate scores (e.g., cell cycle or sex scores) that must be regressed out as a pre-processing baseline before clustering or trajectory analysis.

## When NOT to use

- When the goal is to preserve maximum biological signal at gene-expression level — regress_out retained only 30% of genes of interest in corrected HVGs versus 40% for SiFT knn-kernel filtering
- When correction is needed only in latent space rather than on gene expression — scVI performs correction at the level of a joint latent representation
- When no reliable marker gene lists or covariate scores exist to define the nuisance signal to regress

## Inputs

- Single-cell RNA-seq count matrix (AnnData/.h5ad) from Drosophila wing disc or comparable tissue
- Curated cell-cycle and sex marker gene lists used to compute per-cell covariate scores

## Outputs

- Covariate-corrected gene expression matrix
- iLISI scores quantifying residual cell-cycle and sex variation
- Overlap fraction of genes of interest with the corrected HVG set (reported ~30% for regress_out vs ~40% for SiFT)

## How to apply

Load the single-cell expression matrix (e.g., .h5ad) and score cells for each nuisance signal, such as curated cell-cycle and sex marker gene sets. Run scanpy's regress_out with the cell-cycle and sex scores as covariates to produce corrected gene expression. Evaluate the correction with graph iLISI scores computed for cell-cycle-only and cell-cycle+sex label settings, and quantify preservation of biological signal by measuring the overlap of genes of interest with the corrected HVG set. In the SiFT paper's benchmark (Fig. 2b), regress_out retained only ~30% of genes of interest in corrected HVGs versus ~40% for SiFT's knn-kernel filtering, so it should be treated as a baseline whose signal loss is expected and must be checked.

## Related tools

- **Scanpy** (Provides regress_out, the covariate-regression baseline that removes cell-cycle and sex scores from gene expression)
- **SiFT (sift-sc)** (Comparison method whose knn-kernel filtering is benchmarked against regress_out on iLISI and gene-preservation metrics) — https://github.com/nitzanlab/sift-sc
- **scVI** (Latent-space baseline trained with continuous cell-cycle and sex covariates for comparison against regress_out) — https://github.com/scverse/scvi-tools

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('wing_disc.h5ad'); sc.pp.regress_out(adata, ['cell_cycle_score', 'sex_score']); sc.pp.highly_variable_genes(adata)
```

## Evaluation signals

- Graph iLISI scores computed for cell-cycle-only and cell-cycle+sex label settings show reduced nuisance variation after regression
- Overlap of genes of interest with corrected HVGs is near the reported ~30% for regress_out (vs ~40% for SiFT)
- Corrected data reproduces the comparative ordering of methods shown in Fig. 2b of the SiFT paper
- Downstream clustering/trajectories no longer separate by cell-cycle or sex scores

## Limitations

- Correction at gene-expression level can remove biological signal: only 30% of genes of interest remain in corrected HVGs versus 40% for SiFT
- No ground truth reference is available to evaluate computational filtering results; evaluation relies on application-specific integration metrics
- Corrected gene expression is no longer properly log-transformed, a property expected in certain downstream analysis tasks

## Evidence

- [results] SiFT compares favorably to regress_out and scVI on graph iLISI scores (Fig. 2b), and preserves more biological variation: 40% of genes of interest remain in the corrected HVG set after SiFT filtering versus only 30% for regress_out.: [evidence span withheld — anchor status: not_found]
- [intro] most integration methods, including scVI, provide a correction at the level of a joint latent representation of the cells: "most integration methods, including scVI, provide a correction at the level of a joint latent representation of the cells"
- [methods] Run scanpy's regress_out on the same dataset with cell-cycle and sex covariates as the batch-correction baseline.: [evidence span withheld — anchor status: not_found]
- [discussion] it modifies the gene expression so that data is no longer properly log-transformed, a property that is expected in certain downstream analysis tasks: "it modifies the gene expression so that data is no longer properly log-transformed, a property that is expected in certain downstream analysis tasks"
