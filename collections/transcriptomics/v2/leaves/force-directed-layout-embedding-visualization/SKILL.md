---
name: force-directed-layout-embedding-visualization
description: Use when tGPT-derived feature embeddings and cell type annotations (e.g.
  AnnData objects loaded with Scanpy) must be ordered along developmental trajectories
  and the branch structure rendered as a trajectory visualization.
when_to_use_negative:
- The task is clustering or batch-effect benchmarking rather than developmental ordering
  — tGPT feature representations are also used with KNN/Leiden and Scanpy/UMAP pipelines,
  but FLE trajectory visualization addresses lineage inference specifically
- Only raw count matrices are available with no pretrained tGPT feature representations;
  the diffusion pseudo-time analysis depends on the learned embeddings from the transformer
  layers or embedding layer
- The dataset has no known cell type labels or expected lineage structure to validate
  the branch organization against
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: tGPT
  role: Source of the feature representations on which diffusion pseudo-time is computed;
    pretrained checkpoint lixiangchun/transcriptome-gpt-1024-8-16-64 (62 top-expressing
    genes, max_len 64)
  repo: https://github.com/deeplearningplus/tGPT
- name: Pegasus v1.4.3
  role: Constructs the diffusion pseudo-time maps and provides the force-directed
    layout embedding (FLE) algorithm for trajectory visualization
  repo: https://github.com/lilab-bcb/pegasus
- name: Scanpy v1.6.0
  role: Loads feature embeddings and cell type labels into AnnData objects and supports
    neighbor graph computation and embedding visualization prior to FLE rendering
  repo: https://github.com/theislab/scanpy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:f925ff8f0762b47e2942c65ab81c838e4ef9247b1c140fa134c04dd4d4282036
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deeplearningplus/tGPT
  repo_url: https://github.com/deeplearningplus/tGPT
  tools:
  - Pegasus
  - Pegasus v1.4.3
  - Scanpy
  - Scanpy v1.6.0
  - tGPT
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# force-directed-layout-embedding-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualize cell trajectories from diffusion pseudo-time maps computed on tGPT feature representations using the force-directed layout embedding (FLE) algorithm from Pegasus v1.4.3, resolving developmental branches such as HSC-to-erythroid/DC-monocyte in the HCA bone marrow dataset (n=282,588) and fetal mesenchymal progenitor branches in the HCL dataset (586,135 cells).

## When to use

Use when tGPT-derived feature embeddings and cell type annotations (e.g. AnnData objects loaded with Scanpy) must be ordered along developmental trajectories and the branch structure rendered as a trajectory visualization.

## When NOT to use

- The task is clustering or batch-effect benchmarking rather than developmental ordering — tGPT feature representations are also used with KNN/Leiden and Scanpy/UMAP pipelines, but FLE trajectory visualization addresses lineage inference specifically
- Only raw count matrices are available with no pretrained tGPT feature representations; the diffusion pseudo-time analysis depends on the learned embeddings from the transformer layers or embedding layer
- The dataset has no known cell type labels or expected lineage structure to validate the branch organization against

## Inputs

- tGPT-derived feature embedding matrices (e.g. mean of last_hidden_state over ranked top-expressing gene tokens, length 64 or 128)
- Cell type annotation labels (e.g. celltype column in AnnData.obs, as category)
- AnnData objects prepared with Scanpy for HCA bone marrow (n=282,588) and HCL (586,135 cells) datasets

## Outputs

- Diffusion pseudo-time maps computed with Pegasus v1.4.3 on tGPT features
- Force-directed layout embedding (FLE) trajectory figures exported as PNG files
- Developmental branch assignments (e.g. HSC to erythroid/DC-monocyte, naive T to cytotoxic T/NK, pro-B to pre-B/naive B/memory/plasma)

## How to apply

Load the tGPT-derived feature embedding matrices and cell type annotation labels (e.g. HCA bone marrow, n=282,588 from 64 healthy donors; HCL, 586,135 cells) into AnnData objects with Scanpy. Construct diffusion pseudo-time maps on the tGPT feature representations using Pegasus v1.4.3, then visualize the resulting cell trajectories with the force-directed layout embedding (FLE) algorithm. Interpret the layout as developmental branches: in HCA, HSCs to erythroid cells or DCs and monocytes, naive T cells to cytotoxic T cells and NK cells, and pro-B cells through pre-B/naive B to memory/plasma cells; in HCL, fetal mesenchymal progenitors to fibroblast, enterocyte, and astrocyte-oligodendrocyte branches. Export the diffusion map figures as PNG files and validate that branch organization reproduces Figure 4A-4K. The rationale is that tGPT feature representations capture lineage structure well enough that pseudo-time ordering aligns with known cell labels and states without manual curation.

## Related tools

- **tGPT** (Source of the feature representations on which diffusion pseudo-time is computed; pretrained checkpoint lixiangchun/transcriptome-gpt-1024-8-16-64 (62 top-expressing genes, max_len 64)) — https://github.com/deeplearningplus/tGPT
- **Pegasus v1.4.3** (Constructs the diffusion pseudo-time maps and provides the force-directed layout embedding (FLE) algorithm for trajectory visualization) — https://github.com/lilab-bcb/pegasus
- **Scanpy v1.6.0** (Loads feature embeddings and cell type labels into AnnData objects and supports neighbor graph computation and embedding visualization prior to FLE rendering) — https://github.com/theislab/scanpy

## Examples

```
import scanpy as sc; adata = sc.AnnData(features); adata.obs['celltype'] = celltype; sc.pp.neighbors(adata, n_neighbors=20); sc.tl.leiden(adata, resolution=0.6); sc.tl.umap(adata); sc.pl.umap(adata, color=['celltype'])
```

## Evaluation signals

- Diffusion pseudo-time branch structure derived from tGPT features is highly aligned with the known cell type labels and states
- HCA branches resolve as HSC to erythroid or DC/monocyte, naive T to cytotoxic T/NK, and pro-B to pre-B/naive B to memory/plasma cells
- HCL branches resolve as fetal mesenchymal progenitors to fibroblast/enterocyte/astrocyte-oligodendrocyte
- Exported diffusion map figures reproduce the branch organization of Figure 4A-4K
- Leiden clustering and cell type coloring on the same AnnData (sc.tl.leiden resolution 0.6; neighbors n_neighbors=20) show coherent separation consistent with the FLE trajectory branches

## Limitations

- tGPT uses only the top-expressing genes (62 or 126) and may miss information specifically represented in low-expressing genes, which could distort branch structure
- tGPT uses gene expression rankings rather than actual expression levels, so fold changes among genes are neglected, which can affect biological interpretation of the trajectories
- The pretrained tGPT weights/checkpoint availability beyond the code repository link is not stated, so reproducing the exact feature representations may depend on the published checkpoint (transcriptome-gpt-1024-8-16-64)

## Evidence
