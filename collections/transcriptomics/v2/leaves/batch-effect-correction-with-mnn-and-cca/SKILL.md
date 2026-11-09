---
name: batch-effect-correction-with-mnn-and-cca
description: Use when multi-individual droplet-based scRNA-seq UMI count matrices
  show batch effects (e.g., same-batch samples clustering together in t-SNE of 50
  PCs) and you need corrected/aligned data as input to downstream clustering baselines.
when_to_use_negative:
- The analysis goal is principled multi-individual clustering of raw UMI counts that
  accounts for heterogeneity and unbalanced sequencing depth — use a model-based approach
  like BAMM-SC instead, since naive correction discards information (Cell Ranger downsampling
  causes 'substantial information loss')
- Only a single individual/library is present — there is no between-batch effect to
  correct
- Genes have not been filtered for excessive zeros; lowly-expressed genes should be
  removed before correction and clustering
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scran
  role: Implementation of mutual nearest neighbors (MNN) batch correction applied
    before downstream clustering
- name: mclust
  role: Provides adjustedRandIndex to score clusterings obtained on MNN-corrected
    or CCA-aligned data against truth labels
- name: BAMMSC
  role: Multi-individual Bayesian Dirichlet multinomial mixture used as the model-based
    comparison target that avoids explicit batch correction
  repo: github.com/lichen-lab/BAMMSC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:3061fd6fb84e9faf406d8133d2e58983dd52a3f15fe552af5ddd7c8460cd4287
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BAMM-SC
  - BAMMSC
  - BAMMSC R package
  - DIMM-SC
  - SC3
  - TSCAN
  - mclust
  - scran
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# batch-effect-correction-with-mnn-and-cca

## Summary

Applies mutual nearest neighbors (MNN) batch correction implemented in scran, or CCA-based alignment, to scRNA-seq count matrices from multiple individuals before clustering, as batch-effect-removal baselines against which model-based multi-individual clustering (e.g., BAMM-SC) can be benchmarked via adjusted Rand index.

## When to use

Use when multi-individual droplet-based scRNA-seq UMI count matrices show batch effects (e.g., same-batch samples clustering together in t-SNE of 50 PCs) and you need corrected/aligned data as input to downstream clustering baselines.

## When NOT to use

- The analysis goal is principled multi-individual clustering of raw UMI counts that accounts for heterogeneity and unbalanced sequencing depth — use a model-based approach like BAMM-SC instead, since naive correction discards information (Cell Ranger downsampling causes 'substantial information loss')
- Only a single individual/library is present — there is no between-batch effect to correct
- Genes have not been filtered for excessive zeros; lowly-expressed genes should be removed before correction and clustering

## Inputs

- List of gene-by-cell UMI count matrices, one per individual/batch (genes shared across individuals, cell numbers may differ)
- Marker-gene-based approximated truth labels for ARI evaluation

## Outputs

- Batch-corrected (MNN) or aligned (CCA) expression matrix suitable for K-means/TSCAN/SC3/Seurat clustering
- Per-run cluster memberships across all individuals and adjusted Rand index values (aggregated as mean, SD, and range over replicates)

## How to apply

First confirm batch effect by producing a t-SNE plot based on the first 50 principal components of all cells pooled across donors; if samples from the same batch cluster together, correction is warranted. For the MNN route, apply mutual nearest neighbors batch correction as implemented in scran, then cluster the corrected expression matrix (e.g., with K-means, TSCAN, SC3, or Seurat). For the CCA route, align individuals via canonical correlation analysis and then apply the chosen clustering method on the aligned data. Run each correction-plus-clustering combination multiple times (the benchmark used ten replicates) and compute mclust::adjustedRandIndex against marker-gene-based approximated truth labels on the union of cell memberships across individuals to compare against model-based alternatives like BAMM-SC, which instead models batch effect directly in a Bayesian hierarchical Dirichlet multinomial mixture on raw counts.

## Related tools

- **scran** (Implementation of mutual nearest neighbors (MNN) batch correction applied before downstream clustering)
- **mclust** (Provides adjustedRandIndex to score clusterings obtained on MNN-corrected or CCA-aligned data against truth labels)
- **BAMMSC** (Multi-individual Bayesian Dirichlet multinomial mixture used as the model-based comparison target that avoids explicit batch correction) — github.com/lichen-lab/BAMMSC

## Examples

```
library(BAMMSC); library(mclust); result=BAMMSC(X=data_BAMMSC,K=4); adjustedRandIndex(unlist(result$mem),unlist(Z))
```

## Evaluation signals

- Exploratory t-SNE on the first 50 PCs no longer shows samples from the same batch clustering together after correction
- ARI computed with mclust::adjustedRandIndex on the union of cell memberships across individuals is reproducible across ten replicates (report mean, SD, and range)
- Benchmarked mean ARI is comparable to reported baseline levels, e.g., BAMM-SC achieved 0.487 (human PBMC), 0.882 (mouse lung), 0.762 (human skin) as the reference to beat
- Corrected matrices retain the same gene dimensionality across individuals so the same clustering method can be applied uniformly

## Limitations

- Downsampling-style depth equalization (e.g., Cell Ranger aggregation) causes substantial information loss for individuals with higher sequencing depth; MNN/CCA correction does not recover this
- As a two-step approach, correction-then-clustering does not explicitly model heterogeneity among multiple individuals, unlike a unified Bayesian hierarchical model on raw counts
- Genes with excessive zeros are filtered prior to analysis, which the authors note 'may be problematic for rare cell type identification'

## Evidence

- [intro] mutual nearest neighbors16 (MNN) (implemented in scran): "mutual nearest neighbors16 (MNN) (implemented in scran)"
- [intro] batch effect in t-SNE: "observed a clear batch effect: samples from the same batch tend to cluster together"
- [intro] Cell Ranger downsampling information loss: "reads from different libraries are downsampled such that all libraries have the same sequencing depth, leading to substantial information loss"
- [discussion] gene filtering prior to analysis: "we filtered out genes with excessive zeros from the analysis under the assumption that lowly-expressed genes do not contribute much to clustering"
- [readme] ARI evaluation from README example: "adjustedRandIndex(unlist(result$mem),unlist(Z))"
