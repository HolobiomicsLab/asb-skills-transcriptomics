---
name: mca-cell-gene-coembedding
description: Use when starting any GSDensity pathway-centric analysis on a normalized
  cell-by-gene matrix (scRNA-seq or ST) — the MCA co-embedding is the required first
  step before compute.kld or compute.nn.edges.
when_to_use_negative:
- Targeted gene panels with limited numbers of genes — the article states GSDensity
  'is not suitable for targeted gene data having limited numbers of genes'; imputation
  is suggested but not implemented
- Data that has not been normalized, or with known over-/under-corrected batch effects,
  since improper correction introduces bias into the embedding
- MacOS with M1 chip, where installation fails due to a CelliD/scuttle dependency
  issue
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GSDensity
  role: Provides compute.mca, which performs the MCA co-embedding of cells and genes;
    install via remotes::install_github
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: Origin of the MCA co-embedding approach; GSDensity reuses CelliD's MCA implementation
    and requires an updated CelliD version as a dependency
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- name: Seurat
  role: Loads and normalizes the expression data (e.g., pbmc3k, Visium datasets) before
    the MCA embedding is computed
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_005
  - task_007
  - task_008
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:03e92e063b998ea52a9f750ca7b2c5f023a0a27a66b40e0a106fce794641a454
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - AUCell
  - CelliD
  - GSDensity
  - RANN
  - Seurat
  - SeuratData
  - VAM
  - future.apply
  - igraph
  - monocle3
  - msigdbr
  - ssGSEA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# MCA cell-gene co-embedding

## Summary

Co-embeds cells and genes from a single-cell or spatial transcriptomics dataset into a shared low-dimensional latent space using multiple correspondence analysis (compute.mca from gsdensity, following CelliD), enabling downstream density-based gene set coordination testing and cell-gene nearest-neighbor graph construction.

## When to use

Use when starting any GSDensity pathway-centric analysis on a normalized cell-by-gene matrix (scRNA-seq or ST) — the MCA co-embedding is the required first step before compute.kld or compute.nn.edges.

## When NOT to use

- Targeted gene panels with limited numbers of genes — the article states GSDensity 'is not suitable for targeted gene data having limited numbers of genes'; imputation is suggested but not implemented
- Data that has not been normalized, or with known over-/under-corrected batch effects, since improper correction introduces bias into the embedding
- MacOS with M1 chip, where installation fails due to a CelliD/scuttle dependency issue

## Inputs

- Normalized cell-by-gene expression matrix as a Seurat object (e.g., pbmc3k after NormalizeData)
- Single-cell scRNA-seq counts (e.g., pbmc3k via SeuratData, GSE148673 TNBC, GSE153162 mouse brain) or 10X Visium spatial transcriptomics data

## Outputs

- A co-embedding object (coembed) containing MCA coordinates for both cells and genes in a shared latent space, consumable by compute.kld, compute.nn.edges, and compute.spatial.kld.df

## How to apply

Normalize the expression matrix first (e.g., Seurat NormalizeData), then call gsdensity::compute.mca(object = <Seurat object>) to project both cells and genes into the same MCA space; gene positions reflect each gene's relationship to cells and other genes, which is what later allows compute.kld to compare gene set density against all-gene background density and compute.nn.edges to build a shared cell-gene nearest-neighbor graph. Restrict downstream analyses to the intersection of co-embedding features with the original gene set (e.g., genes.use = intersect(rownames(ce), rownames(pbmc3k))) so that rows correspond to genes, not cells. For datasets with batch effects, evaluate correction quality beforehand, since over- or under-correction introduces bias into the embedding. This step dominates memory use (~25 Gb for a 40,000-cell dataset), so provision accordingly.

## Related tools

- **GSDensity** (Provides compute.mca, which performs the MCA co-embedding of cells and genes; install via remotes::install_github) — https://github.com/qingnanl/gsdensity
- **CelliD** (Origin of the MCA co-embedding approach; GSDensity reuses CelliD's MCA implementation and requires an updated CelliD version as a dependency) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- **Seurat** (Loads and normalizes the expression data (e.g., pbmc3k, Visium datasets) before the MCA embedding is computed)

## Examples

```
pbmc3k <- NormalizeData(pbmc3k); ce <- compute.mca(object = pbmc3k)
```

## Evaluation signals

- The co-embed object row/feature names should include both cells and genes; genes.use for downstream compute.kld must be set to intersect(rownames(ce), rownames(pbmc3k)) to select genes only
- Downstream sanity check: compute.kld on known marker gene sets should yield adjusted p < 0.05 for marker sets and near-zero significance for size-matched random gene sets (e.g., across the 8 benchmark datasets)
- Memory footprint should be budgeted at roughly 25 Gb for a 40,000-cell dataset (hcabm40k scale); failure to allocate this indicates the embedding step was misconfigured
- The embedding must precede graph construction with identical parameters — compute.nn.edges(coembed = ce, nn.use = 300) consumes the same co-embed object

## Limitations

- MCA embedding is the largest memory cost, about 25 Gb for a medium-sized 40,000-cell dataset (hcabm40k, 17,369 genes)
- Batch over- or under-correction may bias the embedding; the article recommends carefully evaluating batch correction outcome before pathway analysis
- Not suitable for targeted gene data with limited gene numbers; imputation of unmeasured genes is suggested but not implemented
- Installation fails on MacOS M1 due to a CelliD/scuttle dependency; the 'anticlust' dependency must be an older version (e.g., 0.6.1)

## Evidence

- [intro] MCA co-embedding rationale: "GSDensity uses multiple correspondence analysis14 (MCA) to co-embed cells and genes into a latent space"
- [readme] README core idea: "we use multiple correspondence analsysis (MCA) to co-embed cells and genes (thanks to the CelliD package)"
- [methods] Concrete invocation: "compute.mca(object = pbmc3k)"
- [discussion] Memory limitation: "The largest memory cost was in the MCA embedding step, with about 25 Gb for a medium sized dataset"
