---
name: cell-lineage-branch-structure-validation
description: Use when pretrained-model feature embeddings (e.g., tGPT hidden-state
  features) must be checked against known cell type labels and states by reconstructing
  developmental lineage trajectories and branch structure.
when_to_use_negative:
- Input is raw gene expression counts rather than pretrained-model feature embeddings
  — extract tGPT features first (rank-based top-expressing gene input, sequence length
  64 or 128)
- The goal is de novo lineage discovery without known cell labels to validate against
  — alignment to known labels and states is the core validation signal here
- Batch-effect integration or clustering benchmarking is the goal — use the Scanpy/Pegasus/scVI
  benchmarking workflow with batch-correction tools instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tGPT
  role: Source of the feature representations on which diffusion pseudo-time and lineage
    branches are computed
  repo: https://github.com/deeplearningplus/tGPT
- name: Pegasus v1.4.3
  role: Constructs the diffusion pseudo-time maps on the tGPT feature representations
  repo: https://github.com/lilab-bcb/pegasus
- name: Scanpy
  role: Assembles AnnData objects with embeddings and cell type annotations, computes
    neighbor graphs, and visualizes trajectories
  repo: https://github.com/theislab/scanpy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:f57706563ed79af77a5ab2229ad81a852567ee94a08dfb5d7106f1a371a96000
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
  - tGPT
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cell-lineage-branch-structure-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that feature representations learned by tGPT resolve biologically correct developmental branches by constructing diffusion pseudo-time maps with Pegasus (v1.4.3) and visualizing trajectories with force-directed layout embedding (FLE), checking branch structure against known cell type labels.

## When to use

Use when pretrained-model feature embeddings (e.g., tGPT hidden-state features) must be checked against known cell type labels and states by reconstructing developmental lineage trajectories and branch structure.

## When NOT to use

- Input is raw gene expression counts rather than pretrained-model feature embeddings — extract tGPT features first (rank-based top-expressing gene input, sequence length 64 or 128)
- The goal is de novo lineage discovery without known cell labels to validate against — alignment to known labels and states is the core validation signal here
- Batch-effect integration or clustering benchmarking is the goal — use the Scanpy/Pegasus/scVI benchmarking workflow with batch-correction tools instead

## Inputs

- tGPT-derived feature embedding matrices (hidden-state features from the embedding layer and 8 transformer layers)
- Cell type annotation labels (e.g., celltype column in AnnData .obs)
- HCA Human Cell Atlas Census of Immune Cells bone marrow dataset (n=282,588 cells)
- Human Cell Landscape (HCL) dataset (586,135 cells)
- AnnData objects assembled with Scanpy

## Outputs

- Diffusion pseudo-time maps computed on tGPT feature representations
- Force-directed layout embedding (FLE) trajectory visualizations
- PNG figures of resolved developmental branches (HCA and HCL)

## How to apply

Load the tGPT-derived feature embedding matrices together with cell type annotation labels into AnnData objects with Scanpy — e.g., HCA bone marrow cells (n=282,588 from 64 healthy donors) and the HCL dataset (586,135 human cells from a Chinese Han population). Construct diffusion pseudo-time maps on the tGPT feature representations using the Pegasus package (v1.4.3), then visualize cell trajectories with the force-directed layout embedding (FLE) algorithm. Decision point: confirm the resolved branches match known biology — on HCA, HSCs to erythroid cells or DCs/monocytes, naive T cells to cytotoxic T cells and NK cells, and pro-B to pre-B to naive B to memory/plasma cells; on HCL, fetal mesenchymal progenitors to fibroblast/enterocyte/astrocyte-oligodendrocyte lineages. Export the diffusion map figures as PNG files and compare branch organization to the paper's Figure 4A-4K.

## Related tools

- **tGPT** (Source of the feature representations on which diffusion pseudo-time and lineage branches are computed) — https://github.com/deeplearningplus/tGPT
- **Pegasus v1.4.3** (Constructs the diffusion pseudo-time maps on the tGPT feature representations) — https://github.com/lilab-bcb/pegasus
- **Scanpy** (Assembles AnnData objects with embeddings and cell type annotations, computes neighbor graphs, and visualizes trajectories) — https://github.com/theislab/scanpy

## Examples

```
import scanpy as sc; adata = sc.AnnData(features); adata.obs['celltype'] = celltype; sc.pp.neighbors(adata, n_neighbors=20); sc.tl.umap(adata); sc.pl.umap(adata, color=['celltype'])
```

## Evaluation signals

- Branch structure from tGPT features is highly aligned with known cell labels and states (paper's abstract-level validation claim)
- HCA branches reproduce: HSC to erythroid cells or DCs/monocytes; naive T to cytotoxic T and NK; pro-B to pre-B to naive B to memory/plasma cells
- HCL branches reproduce: fetal mesenchymal progenitors to fibroblast/enterocyte/astrocyte-oligodendrocyte
- Diffusion map branch organization matches the paper's Figure 4A-4K when exported as PNGs
- Pseudo-time ordering is monotonic along each branch from progenitor to terminal cell states

## Limitations

- tGPT uses only the top-expressing genes (62 or 126), so information represented in low-expressing genes may be missed in the embeddings feeding the trajectories
- tGPT uses gene expression rankings, not actual expression levels, so fold changes among genes are neglected and this can affect biological interpretation
- The language modeling objective predicts gene rankings and is not directly related to biological issues, so branch structure may require further study to interpret
- Pretrained tGPT weights/checkpoint availability for reuse is not stated beyond the code repository link and Hugging Face checkpoint names

## Evidence
