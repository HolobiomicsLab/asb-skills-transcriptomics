---
name: hvg-overlap-quantification
description: Use when comparing unwanted-variation removal methods (SiFT, scanpy regress_out,
  scVI) on single-cell RNA-seq data and you need a read-out of preserved biological
  signal, e.g. whether curated cell-cycle, sex or disease gene lists remain among
  corrected HVGs.
when_to_use_negative:
- When no curated gene list or prior biological annotation exists — the metric requires
  genes of interest to compare against corrected HVGs (SiFT itself 'strongly relies
  on the prior knowledge used to define the cell–cell similarity kernel').
- When the correction is applied only in latent space and per-gene expression is never
  modified — if corrected HVGs cannot be recomputed in gene space, the overlap metric
  is undefined (this is why it favors gene-level correction methods).
- As a stand-alone quality verdict — integration metrics 'in general may be application-specific',
  so it should complement, not replace, mixing scores like iLISI.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SiFT (sift-sc)
  role: Gene-level filtering whose corrected HVG set is scored for gene-of-interest
    overlap; run with sift.kernels.KnnKernel built from cell-cycle and sex marker
    genes
  repo: https://github.com/nitzanlab/sift-sc
- name: scVI
  role: Latent-space correction baseline (continuous cell-cycle and sex covariates)
    whose output is scored against the same gene-overlap metric
  repo: https://github.com/scverse/scvi-tools
- name: Python
  role: Implementation environment for recomputing HVGs, set-overlap computation,
    and plotting the Fig. 2b comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:ac9968f1d7806e15739a5651ae22a13872b31111cce42ed59a2dc81f24c754ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - Python
  - SiFT
  - SiFT (sift-sc)
  - scVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# hvg-overlap-quantification

## Summary

Quantifies how many genes of interest survive in the corrected highly variable gene (HVG) set after a batch-correction/filtering method, serving as a signal-preservation benchmark. In the SiFT Drosophila wing disc benchmark it distinguishes gene-level correction (SiFT: ~40% overlap) from latent-space correction (regress_out: ~30%).

## When to use

Use when comparing unwanted-variation removal methods (SiFT, scanpy regress_out, scVI) on single-cell RNA-seq data and you need a read-out of preserved biological signal, e.g. whether curated cell-cycle, sex or disease gene lists remain among corrected HVGs.

## When NOT to use

- When no curated gene list or prior biological annotation exists — the metric requires genes of interest to compare against corrected HVGs (SiFT itself 'strongly relies on the prior knowledge used to define the cell–cell similarity kernel').
- When the correction is applied only in latent space and per-gene expression is never modified — if corrected HVGs cannot be recomputed in gene space, the overlap metric is undefined (this is why it favors gene-level correction methods).
- As a stand-alone quality verdict — integration metrics 'in general may be application-specific', so it should complement, not replace, mixing scores like iLISI.

## Inputs

- Corrected/filtered single-cell count matrices (one per method: SiFT knn-kernel-filtered, regress_out-corrected, scVI-corrected)
- Curated list of genes of interest (cell cycle and sex marker genes for the Drosophila wing disc dataset)
- Per-cell covariate labels for the nuisance signal (cell-cycle-only and cell-cycle+sex settings)

## Outputs

- Overlap fraction of genes of interest retained in the corrected HVG set per method (e.g. ~40% SiFT vs ~30% regress_out)
- Summary table of gene-overlap scores per method and per covariate setting
- Combined iLISI + gene-overlap figure reproducing Fig. 2b

## How to apply

After running each correction method (SiFT with a sift.kernels.KnnKernel built from curated cell-cycle and sex marker genes, scanpy's regress_out on cell-cycle and sex covariates, and scVI with continuous covariates) on the Drosophila wing disc count matrix, recompute HVGs on each corrected dataset. Compute the set overlap between the curated genes of interest and the corrected HVGs for both the cell-cycle-only and cell-cycle+sex covariate settings. Report the overlap fraction per method — in the source benchmark, SiFT retains ~40% of genes of interest in corrected HVGs versus ~30% for regress_out — alongside graph iLISI scores (Fig. 2b) that measure how well the nuisance signal (cell cycle, sex) was actually mixed away. Judge success by the combination: high iLISI (variation removed) plus higher gene-of-interest overlap (biological signal preserved) favors the method; also account for the fact that SiFT modifies expression so the output is no longer properly log-transformed before recomputing HVGs.

## Related tools

- **SiFT (sift-sc)** (Gene-level filtering whose corrected HVG set is scored for gene-of-interest overlap; run with sift.kernels.KnnKernel built from cell-cycle and sex marker genes) — https://github.com/nitzanlab/sift-sc
- **scVI** (Latent-space correction baseline (continuous cell-cycle and sex covariates) whose output is scored against the same gene-overlap metric) — https://github.com/scverse/scvi-tools
- **Python** (Implementation environment for recomputing HVGs, set-overlap computation, and plotting the Fig. 2b comparison)

## Examples

```
import scanpy as sc; hvg = set(adata[:, goi].var_names[sc.pp.highly_variable_genes(adata, n_top_genes=2000, inplace=False)['highly_variable']]); overlap = len(set(goi) & hvg) / len(goi)
```

## Evaluation signals

- SiFT gene-of-interest overlap with corrected HVGs lands near the reported ~40%; regress_out lands near ~30% (Fig. 2b).
- Overlap fraction is computed for both covariate settings (cell-cycle-only and cell-cycle+sex) and the ranking of methods is consistent across them.
- High iLISI scores accompany low overlap of nuisance marker genes, confirming the removed signal is the unwanted one rather than biological signal.
- The corrected expression is checked for log-transformation state before HVG recomputation, since SiFT output 'is no longer properly log-transformed'.

## Limitations

- No ground truth reference is available to evaluate computational filtering results; evaluation depends on application-specific integration metrics.
- SiFT modifies gene expression so the data is no longer properly log-transformed, which affects downstream tasks like HVG selection.
- SiFT strongly relies on prior knowledge used to define the cell–cell similarity kernel; inaccurate or coarse-grained mapping may lead to a loss of signal (inflating the overlap metric spuriously).
- Statistical implications of applying commonly used analysis tools to corrected gene expression data are unexplored.

## Evidence

- [abstract] SiFT performs the correction at the gene expression level, can scale to large datasets, and compares favorably to state-of-the-art methods: "SiFT performs the correction at the gene expression level, can scale to large datasets, and compares favorably to state-of-the-art methods"
- [discussion] it modifies the gene expression so that data is no longer properly log-transformed: "it modifies the gene expression so that data is no longer properly log-transformed, a property that is expected in certain downstream analysis tasks"
- [discussion] we suggest using integration metrics (which in general may be application-specific): "we suggest using integration metrics (which in general may be application-specific), as has been used throughout this manuscript"
- [discussion] SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel: "it is important to note that SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel."
- [methods] sift.kernels.RBFKernel sift.kernels.LaplacianKernel sift.kernels.KnnKernel sift.kernels.MappingKernel: "sift.kernels.RBFKernel
    sift.kernels.LaplacianKernel
    sift.kernels.KnnKernel
    sift.kernels.MappingKernel"
