---
name: leiden-clustering-resolution-sweep
description: Use when comparing single-cell integration or batch-correction methods
  (e.g., scCobra vs. Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) and you need
  resolution-robust ARI/NMI/ASW metrics on each method's embedding.
when_to_use_negative:
- The goal is routine cell-type annotation or downstream biological interpretation
  of a single embedding, not a method benchmark — a single fixed Leiden resolution
  is sufficient and the sweep adds no information.
- 'You need distribution-free or full-dimensional evaluation: the scCobra paper notes
  scores were derived from two-dimensional UMAP embeddings following SCALEX, so results
  are dependent on that UMAP projection.'
- No ground-truth cell-type labels or batch labels are available for the cells, since
  ARI, NMI, CellType ASW, and batch-removal metrics all require them.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: scIB
  role: Evaluation pipeline used to apply Leiden clustering across the 0.1–2.0 resolution
    range and compute batch-removal and biological-conservation metrics
  repo: https://scib.readthedocs.io/en/latest/index.html
- name: Scanpy
  role: Preprocessing and Leiden clustering backend on the AnnData objects holding
    each method's integrated embedding
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:1f14081508f386681bb82a3a87d5623b089084fc6beaedbdb180ed43dbdb4b6a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - MAESTRO
  - PyTorch
  - Scanorama
  - Scanpy
  - Seurat
  - scCobra
  - scDML
  - scDREAMER
  - scIB
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# leiden-clustering-resolution-sweep

## Summary

Benchmarks integrated single-cell embeddings by running Leiden clustering over a resolution sweep (0.1–2.0 in 0.1 increments) via the scIB pipeline, so batch-removal and biological-conservation metrics (ARI, NMI, Batch ASW, CellType ASW, Graph Connectivity) are not artifacts of a single clustering granularity.

## When to use

Use when comparing single-cell integration or batch-correction methods (e.g., scCobra vs. Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) and you need resolution-robust ARI/NMI/ASW metrics on each method's embedding.

## When NOT to use

- The goal is routine cell-type annotation or downstream biological interpretation of a single embedding, not a method benchmark — a single fixed Leiden resolution is sufficient and the sweep adds no information.
- You need distribution-free or full-dimensional evaluation: the scCobra paper notes scores were derived from two-dimensional UMAP embeddings following SCALEX, so results are dependent on that UMAP projection.
- No ground-truth cell-type labels or batch labels are available for the cells, since ARI, NMI, CellType ASW, and batch-removal metrics all require them.

## Inputs

- Integrated cell embedding (e.g., AnnData obsm embedding from scCobra, Seurat, Harmony, Scanorama, scVI, scDML, or scDREAMER)
- Per-cell batch labels for each dataset
- Ground-truth cell-type annotations for biological conservation metrics
- Optional two-dimensional UMAP embedding of the integrated data (SCALEX-style scoring)

## Outputs

- Per-resolution Leiden cluster assignments (0.1 to 2.0, increments of 0.1) for each method's embedding
- Metric table of ARI, NMI, Batch ASW, CellType ASW, Graph Connectivity, and Batch mixing score per method per resolution
- Aggregated benchmark table (e.g., benchmark_metrics.csv, batch_integration_metrics.csv, or hypothalamus_integration_benchmark.csv) reproducing article figure rankings

## How to apply

After each integration method produces a cell embedding, load it into an AnnData object together with batch labels and ground-truth cell-type labels. Run Leiden clustering across resolutions from 0.1 to 2.0 in increments of 0.1, as the scCobra paper does with the scIB pipeline. For each resolution, compute batch-removal metrics (Batch mixing score, Batch ASW) and biological-conservation metrics (ARI, NMI, CellType ASW, Graph Connectivity); in the scCobra benchmark, over-correction scores were derived from two-dimensional UMAP embeddings following the SCALEX methodology. Sweep the full resolution range per method rather than fixing one resolution, then aggregate scores (e.g., the summed ARI+NMI+CellType ASW score) into a per-method comparison table such as benchmark_metrics.csv or batch_integration_metrics.csv. Correct application means the sweep covers all 20 resolution values for every method on identical inputs, so rankings are comparable.

## Related tools

- **scIB** (Evaluation pipeline used to apply Leiden clustering across the 0.1–2.0 resolution range and compute batch-removal and biological-conservation metrics) — https://scib.readthedocs.io/en/latest/index.html
- **Scanpy** (Preprocessing and Leiden clustering backend on the AnnData objects holding each method's integrated embedding)

## Examples

```
import scanpy as sc; [sc.tl.leiden(adata, resolution=round(r,1), key_added=f'leiden_{round(r,1)}') for r in [i/10 for i in range(1,21)]]
```

## Evaluation signals

- The resolution sweep contains exactly 20 values: 0.1, 0.2, ..., 2.0 (increments of 0.1), applied identically to every method's embedding.
- ARI and NMI values fall in [0, 1]; CellType ASW and Graph Connectivity in [0, 1]; Batch ASW differences should decrease as batch mixing improves.
- The best-performing resolution may differ per method, so verify the aggregated score (e.g., summed ARI+NMI+CellType ASW) selects the optimum per method before ranking.
- The output metric table reproduces the expected figure ranking — e.g., scCobra attains the highest biological-conservation score on the lung atlas, immune, and pancreas datasets (Fig. 2c-f), and the multi-omic PBMC ranking matches Fig. 4b/4c.
- Clustering is run on the integrated embedding (not raw counts), so re-clustering raw data should yield clearly different (worse) batch metrics.

## Limitations

- Benchmarking methods were run with default parameters, which may not reflect their tuned performance; metric rankings reflect defaults.
- Scores are derived from two-dimensional UMAP embeddings following the SCALEX methodology, introducing dependence on the UMAP projection.
- No universal clustering resolution is optimal for all datasets; results at a single resolution can misrepresent a method's integration quality, which is why the sweep is required.

## Evidence

- [discussion] Resolution range via scIB: "The scIB pipeline applied Leiden clustering across a range of resolutions from 0.1 to 2.0 (in increments of 0.1)"
- [abstract] Rationale for benchmarking embeddings: "scCobra effectively mitigates batch effects, minimizes over-correction, and ensures biologically meaningful data integration"
- [discussion] Dependence on UMAP embeddings: "The scores were derived from two-dimensional UMAP embeddings, following the methodology described in the original SCALEX26 paper"
- [readme] scIB installation context: [evidence span withheld — anchor status: external_doc]
