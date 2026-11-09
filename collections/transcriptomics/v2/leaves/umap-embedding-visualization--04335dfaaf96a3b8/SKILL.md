---
name: umap-embedding-visualization
description: Use when an integrated scRNA-seq embedding (AnnData with an integrated
  latent representation and batch/cell-type annotations) needs a 2D UMAP visualization
  to qualitatively verify batch mixing and cell-type separation after integration.
when_to_use_negative:
- When only quantitative metrics are needed for the benchmark table and no visual
  inspection or publication figure is required
- When the input AnnData has no integrated embedding yet — run the integration method
  (e.g., scDREAMER) first; a UMAP of raw/pre-integration PCA would misrepresent batch
  mixing
- When the goal is clustering or metric computation itself (e.g., Louvain clustering
  at NMI-maximizing resolution) rather than visualization
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Scanpy
  role: computes the UMAP layout and renders the visualization of the integrated embedding
    stored in the AnnData object
- name: scDREAMER
  role: produces the integrated latent embedding whose UMAP is visualized
  repo: https://github.com/Zafar-Lab/scDREAMER
- name: scDREAMER-reproducibility
  role: provides the notebooks/scripts that generate the manuscript's UMAP figures
  repo: https://github.com/Zafar-Lab/scDREAMER-reproducibility
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:7e77727295bfd5ca09c9e62016226884079da5c250e5e7e346923be434934bbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER
  tools:
  - ADAM
  - BBKNN
  - Harmony
  - INSCT
  - LIGER
  - Scanorama
  - Scanpy
  - Seurat
  - iMAP
  - scANVI
  - scDML
  - scDREAMER
  - scDREAMER-Sup
  - scDREAMER-reproducibility
  - scGEN
  - scVI
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# umap-embedding-visualization

## Summary

Generates UMAP projections of integrated single-cell embeddings (e.g., the scDREAMER latent space of the 486,134-cell healthy heart atlas with 147 batches) so that batch mixing and biological structure can be visually inspected alongside quantitative metrics.

## When to use

Use when an integrated scRNA-seq embedding (AnnData with an integrated latent representation and batch/cell-type annotations) needs a 2D UMAP visualization to qualitatively verify batch mixing and cell-type separation after integration.

## When NOT to use

- When only quantitative metrics are needed for the benchmark table and no visual inspection or publication figure is required
- When the input AnnData has no integrated embedding yet — run the integration method (e.g., scDREAMER) first; a UMAP of raw/pre-integration PCA would misrepresent batch mixing
- When the goal is clustering or metric computation itself (e.g., Louvain clustering at NMI-maximizing resolution) rather than visualization

## Inputs

- Scanpy AnnData object containing the integrated embedding (e.g., scDREAMER latent coordinates) for cells of the healthy heart atlas (486,134 cells, 147 batches)
- Batch and cell-type annotation columns in the AnnData metadata for coloring the UMAP

## Outputs

- UMAP 2D coordinates added to the AnnData object
- UMAP scatter plot of the scDREAMER integrated embedding colored by batch and cell type
- Figure accompanying heart_atlas_integration_metrics.csv in the benchmark output

## How to apply

Take the integrated embedding produced by the integration method (e.g., the scDREAMER adversarial-VAE latent space, trained with the ADAM optimizer and batch size 128 on the top 2000 highly variable genes selected via scanpy.pp.highly_variable_genes with flavor='seurat'), attach it back to the Scanpy AnnData object, and compute a UMAP layout with Scanpy. Color the UMAP by batch label and cell-type annotation so batch-correction quality (e.g., graph connectivity, kBET on the kNN graph) and biological conservation (Louvain clusters at the NMI-maximizing resolution, NMI/ARI) can be inspected visually. In the scDREAMER heart-atlas benchmark, the final UMAP accompanied the per-method metric table written to heart_atlas_integration_metrics.csv. The visualization is a presentation/inspection layer: it must be generated from the already-computed integrated embedding, not replace the quantitative composite bio-conservation, batch-correction, and combined scores.

## Related tools

- **Scanpy** (computes the UMAP layout and renders the visualization of the integrated embedding stored in the AnnData object)
- **scDREAMER** (produces the integrated latent embedding whose UMAP is visualized) — https://github.com/Zafar-Lab/scDREAMER
- **scDREAMER-reproducibility** (provides the notebooks/scripts that generate the manuscript's UMAP figures) — https://github.com/Zafar-Lab/scDREAMER-reproducibility

## Examples

```
import scanpy as sc; sc.pp.neighbors(adata, use_rep='integrated_latent'); sc.tl.umap(adata); sc.pl.umap(adata, color=['batch','cell_type'])
```

## Evaluation signals

- The UMAP is computed on the integrated embedding (not the pre-integration HVG matrix) of the AnnData object
- The plot shows cells colored by both batch and cell-type annotation so batch mixing and biological separation are both checkable
- Visual batch mixing on the UMAP is consistent with quantitative batch-correction metrics (graph connectivity, kBET with k=50 on the kNN graph, PCR batch) reported for the same method
- Visual cell-type clustering is consistent with Louvain clusters at the resolution maximizing NMI, and with NMI/ARI and cell-type ASW scaled as (ASW+1)/2
- The figure corresponds to the method's row in heart_atlas_integration_metrics.csv (e.g., scDREAMER's highest combined composite score)

## Limitations

- UMAP is a qualitative, stochastic projection — it can visually exaggerate or obscure batch mixing and should not substitute for the composite batch-correction and bio-conservation scores
- For the healthy heart atlas (486,134 cells, 147 batches), downstream computations on the embedding neighborhood graph (e.g., kBET) failed for some methods due to computational limitations, so the corresponding UMAPs cannot be cross-validated against kBET for those methods

## Evidence

- [other] workflow: ... 5. Generate a UMAP visualization of the scDREAMER integrated embedding and write the full per-method metric table to heart_atlas_integration_metrics.csv.: [evidence span withheld — anchor status: not_found]
- [methods] We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value.: "We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value."
- [supplementary] The source code and usage tutorial for scDREAMER are freely available at https://github.com/Zafar-Lab/scDREAMER: "The source code and usage tutorial for scDREAMER are freely available at https://github.com/Zafar-Lab/scDREAMER"
- [readme] README describes the framework whose embedding is visualized: "scDREAMER is a single-cell data integration framework that employs a novel adversarial variational autoencoder for learning lower-dimensional cellular embeddings"
