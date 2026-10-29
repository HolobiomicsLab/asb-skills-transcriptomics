---
name: cell-type-purity-evaluation
description: Use when a joint latent embedding (e.g. scConfluence latent in mdata.obsm['latent'])
  must be quantitatively validated against known cell-type labels, as in diagonal-integration
  benchmarks or hyperparameter sensitivity sweeps.
when_to_use_negative:
- The data have no reliable cell-type labels (e.g. unannotated modality-specific populations),
  where label-free metrics like FOSCTTM or graph connectivity are needed instead
- The goal is to assess cross-modality nearest-neighbor consistency rather than label
  homogeneity — FOSCTTM is the appropriate metric there
- Comparing embeddings whose neighborhood-graph construction (k, representation) differs,
  since purity values are not comparable across different graph settings
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scConfluence
  role: Produces the joint latent embeddings whose cell-type purity is being scored
    during benchmarking and m-parameter sensitivity analysis
  repo: https://github.com/cantinilab/scconfluence
- name: Scanpy
  role: Builds the neighborhood graph (sc.pp.neighbors) and UMAP on the 'latent' representation
    used for purity (kNN label agreement) computation
- name: Muon
  role: Holds the MuData object whose obsm field stores the embeddings and whose obs
    stores cell-type labels used in purity scoring
- name: Seurat v4.3.0
  role: Baseline method whose joint embeddings are scored with the same purity metric
    for comparison
- name: rliger v1.0.0
  role: Baseline method (Liger) whose joint embeddings are scored with the same purity
    metric for comparison
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:5ecf1b19a319a9ac8d08aea3f2524911f25421ecb177cfebbdb8ac5a40a400b8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cantinilab/scconfluence
  repo_url: https://github.com/cantinilab/scconfluence
  tools:
  - Liger
  - MultiMAP
  - Muon
  - POT
  - Scanpy
  - Seurat
  - Seurat v4.3.0
  - Uniport
  - rliger v1.0.0
  - scConfluence
  - scGLUE
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cell-type-purity-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores a shared latent embedding from single-cell multimodal integration by cell-type purity: for each cell, check whether its nearest neighbors in the joint space carry the same cell-type label. It is one of the five benchmark metrics used to show scConfluence outperforms Seurat, Liger, MultiMAP, Uniport and scGLUE.

## When to use

Use when a joint latent embedding (e.g. scConfluence latent in mdata.obsm['latent']) must be quantitatively validated against known cell-type labels, as in diagonal-integration benchmarks or hyperparameter sensitivity sweeps.

## When NOT to use

- The data have no reliable cell-type labels (e.g. unannotated modality-specific populations), where label-free metrics like FOSCTTM or graph connectivity are needed instead
- The goal is to assess cross-modality nearest-neighbor consistency rather than label homogeneity — FOSCTTM is the appropriate metric there
- Comparing embeddings whose neighborhood-graph construction (k, representation) differs, since purity values are not comparable across different graph settings

## Inputs

- Joint cell embedding (e.g. mdata.obsm['latent'] from scConfluence's model.get_latent())
- Per-cell cell-type annotations from the MuData/AnnData obs metadata (QC-filtered PBMC 10X, OP Multiome, BMCITE, OP Cite, or scCATseq data)

## Outputs

- Per-method, per-dataset purity values (rows in benchmark_metrics.csv with columns method, dataset, metric)
- Purity-vs-m curve values for sensitivity sweeps (rows in sensitivity_curves.csv)

## How to apply

Build a neighborhood graph on the joint embedding with Scanpy (sc.pp.neighbors on the 'latent' representation, optionally followed by sc.tl.umap), then measure the fraction of each cell's k nearest neighbors sharing its annotated cell-type label (kNN label agreement). Apply the same purity computation uniformly across methods (scConfluence, Seurat v4.3.0, rliger v1.0.0, MultiMAP, Uniport, scGLUE) and datasets so scores are comparable, and aggregate per-method, per-dataset values into a benchmark_metrics.csv table. When sweeping unbalanced-OT parameters, compute purity at each m value (e.g. m = 0.3, 0.4, 0.5, 0.6, 0.75) alongside FOSCTTM and graph connectivity to detect flat, robust regions; the default m=0.5 (ot.partial.partial_wasserstein) is expected to be near-optimal. Acceptance is qualitative consistency with the paper's reported pattern — scConfluence's purity should exceed all five state-of-the-art baselines on CITE-seq and Multiome benchmarks.

## Related tools

- **scConfluence** (Produces the joint latent embeddings whose cell-type purity is being scored during benchmarking and m-parameter sensitivity analysis) — https://github.com/cantinilab/scconfluence
- **Scanpy** (Builds the neighborhood graph (sc.pp.neighbors) and UMAP on the 'latent' representation used for purity (kNN label agreement) computation)
- **Muon** (Holds the MuData object whose obsm field stores the embeddings and whose obs stores cell-type labels used in purity scoring)
- **Seurat v4.3.0** (Baseline method whose joint embeddings are scored with the same purity metric for comparison)
- **rliger v1.0.0** (Baseline method (Liger) whose joint embeddings are scored with the same purity metric for comparison)

## Examples

```
import scanpy as sc; sc.pp.neighbors(mdata, use_rep='latent'); sc.tl.umap(mdata)
```

## Evaluation signals

- Purity is computed identically for all methods on the same neighbor-graph settings, yielding comparable values in benchmark_metrics.csv
- scConfluence's purity should be higher than Seurat, Liger, MultiMAP, Uniport and scGLUE on the CITE-seq (BMCITE, OP Cite) and Multiome (PBMC 10X, OP Multiome) benchmarks
- In sensitivity sweeps, purity at the default m=0.5 lies in a near-optimal, flat region of the purity-vs-m curve, reproducing the qualitative pattern of Supplementary Fig. 13
- Latent embeddings used for purity are present in mdata.obsm['latent'] and labels come from QC-filtered cells (mitochondrial proportion, expressed genes, total counts filtering already applied)
- Purity is reported jointly with complementary metrics (FOSCTTM, transfer accuracy, graph connectivity) rather than in isolation

## Limitations

- Rare cell populations are hard to detect and under-represented in parameter estimation, especially with mini-batch gradient descent, which can distort purity estimates for rare types
- Cell-type proportions differ between modalities in unpaired data (documented in Supplementary Figs. 15–18), so purity depends on the labeling available in each modality
- Purity requires trusted annotations; for modality-specific populations absent from a reference label set it cannot be computed meaningfully

## Evidence

- [methods] Compute the five integration metrics (purity, transfer accuracy, connectivity, FOSCTTM, cell type FOSCTTM) on the joint embeddings from each method and each dataset.: [evidence span withheld — anchor status: not_found]
- [methods] score the shared latent space with cell-type purity (kNN label agreement), FOSCTTM (cross-modality nearest-neighbor consistency) and graph connectivity: [evidence span withheld — anchor status: not_found]
- [discussion] We set by default m=0.5 which produces robust performances when prior information about the level of unbalanceness of the data is not available: [evidence span withheld — anchor status: not_found]
- [readme] These embeddings can then be used for visualization and clustering, useful for discovering subpopulations of cells, and for imputation of features across modalities.: "These embeddings can then be used for visualization and clustering, useful for discovering subpopulations of cells, and for imputation of features across modalities."
