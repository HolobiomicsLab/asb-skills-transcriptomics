---
name: over-correction-scoring-from-umap-embeddings
description: Use when benchmarking single-cell batch-correction methods (scCobra,
  Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) for over-correction risk on
  data containing a known biological difference between batches, e.g., an implanted
  perturbation in GSE149614 immune cells.
when_to_use_negative:
- When evaluating integration quality metrics other than over-correction (e.g., batch
  mixing or bio-conservation via the scIB Leiden-clustering pipeline at resolutions
  0.1–2.0)
- When the dataset has no known or implanted biological difference between batches
  — over-correction cannot be detected without a signal that should be preserved
- When a distribution-free, embedding-independent over-correction measure is required;
  these scores are derived from 2D UMAP embeddings per the SCALEX methodology
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0621
tools:
- name: scCobra
  role: Method under evaluation; achieved lowest over-correction (0.043 all cell types;
    0.038 CD4+ T subset)
  repo: https://github.com/mcgilldinglab/scCobra
- name: scVI
  role: Benchmark method (scvi-tools v0.11) whose over-correction scores (0.086 all
    cell types; 0.158 CD4+ T subset) are compared against scCobra
- name: Scanorama
  role: Benchmark method; lowest over-correction among benchmarks for all cell types
    (0.012) in the reference table
- name: scDML
  role: Benchmark method; highest over-correction on the CD4+ T subset (0.278)
- name: Seurat
  role: Benchmark method run with default parameters for comparison
- name: Harmony
  role: Benchmark method run with default parameters for comparison
- name: scDREAMER
  role: Benchmark method run with default parameters for comparison
- name: Scanpy
  role: Preprocessing (filtering, 10,000-count normalization with log1p, 2000 HVG
    selection) and UMAP embedding computation
- name: scIB
  role: Benchmarking pipeline context (Leiden clustering across resolutions 0.1–2.0)
    for integrated outputs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:82a9b94235a7707f38028a29fc6dfcf722ccaab3a20d1d5adf1376de551f6c63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scCobra
  tools:
  - Harmony
  - Scanorama
  - Scanpy
  - Seurat
  - scCobra
  - scDML
  - scDREAMER
  - scIB
  - scVI
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# over-correction-scoring-from-umap-embeddings

## Summary

Quantify how much a batch-correction method erases genuine biological signal by computing an over-correction score from two-dimensional UMAP embeddings of integrated outputs, following the SCALEX methodology, on data where an implanted perturbation (e.g., 'Defense response to Virus' genes up-shifted in CD4+ T cells of the 10X batch) must be preserved.

## When to use

Use when benchmarking single-cell batch-correction methods (scCobra, Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) for over-correction risk on data containing a known biological difference between batches, e.g., an implanted perturbation in GSE149614 immune cells.

## When NOT to use

- When evaluating integration quality metrics other than over-correction (e.g., batch mixing or bio-conservation via the scIB Leiden-clustering pipeline at resolutions 0.1–2.0)
- When the dataset has no known or implanted biological difference between batches — over-correction cannot be detected without a signal that should be preserved
- When a distribution-free, embedding-independent over-correction measure is required; these scores are derived from 2D UMAP embeddings per the SCALEX methodology

## Inputs

- Perturbed scRNA-seq count matrix (e.g., GSE149614 immune dataset, 33506 cells, 16 cell types, 10 batches) as .h5ad after Scanpy filtering (cells <200 genes removed, genes in <3 cells removed, mitochondrial fraction >5% removed, 10,000-count normalization with log1p, 2000 HVGs)
- Perturbation spec: GOBP term ('Defense response to Virus') gene set, target cell type (CD4+ T cells), and target batch ('10X')
- Integrated embeddings from each batch-correction method (scCobra, Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER), repeated over 5 random seeds

## Outputs

- overcorrection_scores.csv with normalized over-correction scores per method, averaged across 5 seeds, for all cell types and the CD4+ T cell subset
- Two-dimensional UMAP embeddings of each method's integrated output used for score computation

## How to apply

First preprocess the dataset (GSE149614) per the scCobra protocol: drop cells with <200 expressed genes, genes in <3 cells, cells with mitochondrial fraction >5%, normalize to 10,000 counts per cell with log1p, and select 2000 highly variable genes with Scanpy. Implant the biological signal (add 1 to expression of genes annotated to the 'Defense response to Virus' GOBP term in CD4+ T cells of the '10X' batch), then run each batch-correction method with default parameters over 5 random seeds. From each integrated output, compute a two-dimensional UMAP embedding and derive the over-correction score following the SCALEX methodology, both for all cell types and for the CD4+ T cell subset. Normalize the scores and average across the 5 seeds; lower scores indicate less over-correction. Validate against reference values from Fig. 3c (all cell types: scCobra 0.043, scVI 0.086, Scanorama 0.012; CD4+ T subset: scCobra 0.038, scVI 0.158, scDML 0.278), confirming scCobra exhibits the lowest over-correction.

## Related tools

- **scCobra** (Method under evaluation; achieved lowest over-correction (0.043 all cell types; 0.038 CD4+ T subset)) — https://github.com/mcgilldinglab/scCobra
- **scVI** (Benchmark method (scvi-tools v0.11) whose over-correction scores (0.086 all cell types; 0.158 CD4+ T subset) are compared against scCobra)
- **Scanorama** (Benchmark method; lowest over-correction among benchmarks for all cell types (0.012) in the reference table)
- **scDML** (Benchmark method; highest over-correction on the CD4+ T subset (0.278))
- **Seurat** (Benchmark method run with default parameters for comparison)
- **Harmony** (Benchmark method run with default parameters for comparison)
- **scDREAMER** (Benchmark method run with default parameters for comparison)
- **Scanpy** (Preprocessing (filtering, 10,000-count normalization with log1p, 2000 HVG selection) and UMAP embedding computation)
- **scIB** (Benchmarking pipeline context (Leiden clustering across resolutions 0.1–2.0) for integrated outputs)

## Examples

```
git clone https://github.com/mcgilldinglab/scCobra.git; pip install scanpy scib; # then run_integration.ipynb to obtain integrated embeddings, compute 2D UMAPs, and score over-correction per SCALEX across 5 seeds into overcorrection_scores.csv
```

## Evaluation signals

- Reproduced normalized over-correction scores match Fig. 3c reference values within tolerance: scCobra ≈ 0.043 and Scanorama ≈ 0.012 for all cell types
- CD4+ T cell subset scores match references: scCobra ≈ 0.038, scVI ≈ 0.158, scDML ≈ 0.278
- scCobra exhibits the lowest over-correction among all seven methods on the perturbed immune dataset
- Score variance across the 5 random seeds is small after averaging; per-seed runs are deterministic given the seed
- Perturbation check: 'Defense response to Virus' gene expression is shifted by +1 only in CD4+ T cells of the '10X' batch before integration

## Limitations

- Over-correction scores are derived from two-dimensional UMAP embeddings, following the SCALEX methodology, introducing dependence on UMAP
- Benchmark methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) were run with default parameters, which may not reflect their tuned performance
- Evaluation relies on a simulated implanted perturbation, which may not resemble real scRNA-seq variation between batches
