---
name: single-cell-visualization
description: Use when you have tGPT feature embeddings (or another single-cell representation)
  plus cell type annotations in AnnData and need to render cluster/cell-type UMAPs
  or developmental trajectory maps (HSC→erythroid/DC-monocyte branches).
when_to_use_negative:
- Input is already a rendered embedding or image rather than a feature matrix — plotting
  adds no new representation
- You need quantitative clustering benchmarks rather than visual trajectories (use
  the grid search of n_neighbors 5–100 and resolution 0.1–2 for metrics instead)
- Data are bulk tissue transcriptomes where single-cell trajectory layout (FLE/diffusion
  maps) is not applicable
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Scanpy v1.6.0 (README tested with 1.9.1)
  role: Builds AnnData from tGPT features, computes the neighbor graph (n_neighbors=20),
    runs Leiden and UMAP, and renders PNG plots colored by celltype/leiden
  repo: github.com/theislab/scanpy
- name: Pegasus v1.4.3
  role: Constructs diffusion pseudo-time maps on the tGPT feature representations
    whose cell trajectories are visualized with force-directed layout embedding (FLE)
  repo: github.com/lilab-bcb/pegasus
- name: tGPT
  role: Supplies the pretrained transformer feature embeddings (last_hidden_state
    means) that the visualizations are computed on
  repo: github.com/deeplearningplus/tGPT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:f8f2d226cc001e88db06fe2ef7deee309a9d92f94f3273d871228a42295bd740
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deeplearningplus/tGPT
  tools:
  - Pegasus
  - Pegasus v1.4.3
  - Scanpy
  - Scanpy v1.6.0 (README tested with 1.9.1)
  - tGPT
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# single-cell-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualize tGPT-derived feature embeddings as UMAP cluster/cell-type plots and diffusion pseudo-time trajectories: Scanpy for neighbor graph + UMAP rendering, and Pegasus (v1.4.3) diffusion maps rendered with force-directed layout embedding (FLE) to resolve developmental lineages on HCA and HCL datasets.

## When to use

Use when you have tGPT feature embeddings (or another single-cell representation) plus cell type annotations in AnnData and need to render cluster/cell-type UMAPs or developmental trajectory maps (HSC→erythroid/DC-monocyte branches).

## When NOT to use

- Input is already a rendered embedding or image rather than a feature matrix — plotting adds no new representation
- You need quantitative clustering benchmarks rather than visual trajectories (use the grid search of n_neighbors 5–100 and resolution 0.1–2 for metrics instead)
- Data are bulk tissue transcriptomes where single-cell trajectory layout (FLE/diffusion maps) is not applicable

## Inputs

- tGPT feature embedding matrix (e.g. mean of last_hidden_state over non-pad tokens, from lixiangchun/transcriptome-gpt-1024-8-16-64 checkpoint)
- Cell type annotation file (e.g. gzipped text of labels like ./data/Muris_cell_labels.txt.gz) read into adata.obs['celltype']
- AnnData objects for HCA bone marrow (n=282,588 cells) and HCL (586,135 cells) with cell type annotations

## Outputs

- PNG UMAP figures colored by cell type and by Leiden clusters
- PNG diffusion pseudo-time / FLE trajectory figures showing HCA and HCL developmental branches (reproducing Figure 4A-4K branch organization)

## How to apply

Load the tGPT-derived feature embedding matrix and cell type labels into a Scanpy AnnData object, marking 'celltype' as a categorical obs column. Compute a neighbor graph with sc.pp.neighbors(adata, n_neighbors=20), run Leiden clustering (e.g. resolution=0.6), and render UMAPs colored by both 'celltype' and 'leiden' via sc.pl.umap, exporting PNG figures. For lineage structure, construct diffusion pseudo-time maps with Pegasus (v1.4.3) computed on the tGPT feature representations and visualize the cell trajectory with the force-directed layout embedding (FLE) algorithm. The rationale is that the article showed tGPT-derived single-cell clusters and lineage trajectories are highly aligned with known cell labels and states, so a correctly rendered plot should reproduce the HCA branches (HSC to erythroid or DC-monocyte; naive T to cytotoxic T/NK; pro-B to pre-B to naive B to memory/plasma cells) and HCL branches (fetal mesenchymal progenitors to fibroblast/enterocyte/astrocyte-oligodendrocyte).

## Related tools

- **Scanpy v1.6.0 (README tested with 1.9.1)** (Builds AnnData from tGPT features, computes the neighbor graph (n_neighbors=20), runs Leiden and UMAP, and renders PNG plots colored by celltype/leiden) — github.com/theislab/scanpy
- **Pegasus v1.4.3** (Constructs diffusion pseudo-time maps on the tGPT feature representations whose cell trajectories are visualized with force-directed layout embedding (FLE)) — github.com/lilab-bcb/pegasus
- **tGPT** (Supplies the pretrained transformer feature embeddings (last_hidden_state means) that the visualizations are computed on) — github.com/deeplearningplus/tGPT

## Examples

```
sc.pp.neighbors(adata,n_neighbors=20); sc.tl.leiden(adata,resolution=0.6); sc.tl.umap(adata); sc.pl.umap(adata, color=["celltype"], show=True)
```

## Evaluation signals

- FLE/diffusion pseudo-time branch structure is highly aligned with known cell labels and states
- HCA plot resolves the expected branches: HSC to erythroid cells or DCs and monocytes, naive T cells to cytotoxic T cells and NK cells, pro-B cells to pre-B cells (Figure 4A-4K organization)
- HCL plot resolves fetal mesenchymal progenitors to fibroblast/enterocyte/astrocyte-oligodendrocyte branches
- UMAP colored by 'celltype' vs colored by 'leiden' shows concordant cluster/cell-type organization
- Exported PNG figures render without padding-token contamination (features are means over token positions 1:eos_idx, excluding padding)

## Limitations

- tGPT uses only the top-expressing genes, so visualized trajectories may miss information represented within low-expressing genes
- tGPT uses gene expression rankings, not actual expression levels; fold changes among genes are neglected, which can affect biological interpretation of plotted structure
- The language-modeling pretraining objective is not directly related to cell clustering, so the last transformer layer representation is not guaranteed to give rise to better clustering/visual separation

## Evidence
