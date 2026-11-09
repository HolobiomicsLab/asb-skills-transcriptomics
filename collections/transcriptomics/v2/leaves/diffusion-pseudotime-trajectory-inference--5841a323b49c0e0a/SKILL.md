---
name: diffusion-pseudotime-trajectory-inference
description: Use when you have cell-level feature embeddings (e.g., tGPT features
  in AnnData) plus cell-type labels and need to resolve developmental branches or
  lineage order from the representation space.
when_to_use_negative:
- Input is raw count matrices or gene expression levels rather than precomputed tGPT
  feature representations — the skill operates on embeddings, not expression
- The biological question concerns batch integration or clustering performance rather
  than developmental ordering/lineage inference
- Data are bulk tissue transcriptomes, where tGPT features are instead used for genomic
  alteration/prognosis association analyses, not pseudo-time
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: tGPT
  role: Provides the pretrained transformer feature representations (rank-based top-expressing
    gene sequences) on which diffusion pseudo-time is computed
  repo: https://github.com/deeplearningplus/tGPT
- name: Pegasus v1.4.3
  role: Constructs the diffusion pseudo-time maps on the tGPT feature representations
  repo: https://github.com/lilab-bcb/pegasus
- name: Scanpy v1.6.0
  role: Loads feature embeddings and cell-type labels into AnnData objects for downstream
    trajectory analysis and visualization
  repo: https://github.com/theislab/scanpy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:b4b716866a3c3ad177d2b764a48e0e9acac88f1f50799348da1829f34c1ca978
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# diffusion-pseudotime-trajectory-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs diffusion pseudo-time developmental trajectories from tGPT-derived feature embeddings of large-scale single-cell datasets (HCA bone marrow, n=282,588; HCL, 586,135 cells) using Pegasus v1.4.3 and visualizes branches with force-directed layout embedding (FLE) to show whether learned representations resolve known lineage structure.

## When to use

Use when you have cell-level feature embeddings (e.g., tGPT features in AnnData) plus cell-type labels and need to resolve developmental branches or lineage order from the representation space.

## When NOT to use

- Input is raw count matrices or gene expression levels rather than precomputed tGPT feature representations — the skill operates on embeddings, not expression
- The biological question concerns batch integration or clustering performance rather than developmental ordering/lineage inference
- Data are bulk tissue transcriptomes, where tGPT features are instead used for genomic alteration/prognosis association analyses, not pseudo-time

## Inputs

- tGPT-derived feature embedding matrix (mean of last_hidden_state over top-expressing gene tokens) for each cell
- Cell type annotation labels (e.g., cell_labels text/tabular files) per cell
- AnnData objects built with Scanpy from the embeddings and labels (HCA bone marrow n=282,588; HCL 586,135 cells)

## Outputs

- Diffusion pseudo-time map computed on tGPT feature representations
- Force-directed layout embedding (FLE) trajectory figures exported as PNG files showing developmental branches (e.g., Figure 4A-4K organization)

## How to apply

Load the tGPT-derived feature embedding matrices and cell-type annotation labels into AnnData objects with Scanpy (HCA bone marrow, n=282,588 cells; HCL, 586,135 cells). Construct diffusion pseudo-time maps using Pegasus v1.4.3 computed directly on the tGPT feature representations, then visualize the cell trajectories with the force-directed layout embedding (FLE) algorithm and export diffusion map figures (PNG). Decision point: compare the resulting branch structure against known cell labels — on HCA this should resolve HSC to erythroid cells or DCs/monocytes, naive T cells to cytotoxic T cells and NK cells, and pro-B to pre-B through memory/plasma cells; on HCL, fetal mesenchymal progenitors to fibroblast/enterocyte/astrocyte-oligodendrocyte branches. The rationale is that tGPT's rank-based, autoregressively pretrained representations align clusters and lineage trajectories with known cell states, so pseudo-time computed on them is a validation of representation quality rather than a de novo annotation.

## Related tools

- **tGPT** (Provides the pretrained transformer feature representations (rank-based top-expressing gene sequences) on which diffusion pseudo-time is computed) — https://github.com/deeplearningplus/tGPT
- **Pegasus v1.4.3** (Constructs the diffusion pseudo-time maps on the tGPT feature representations) — https://github.com/lilab-bcb/pegasus
- **Scanpy v1.6.0** (Loads feature embeddings and cell-type labels into AnnData objects for downstream trajectory analysis and visualization) — https://github.com/theislab/scanpy

## Examples

```
import scanpy as sc; adata = sc.AnnData(features); adata.obs['celltype'] = celltype; sc.pp.neighbors(adata, n_neighbors=20); sc.tl.umap(adata); sc.pl.umap(adata, color=['celltype'])
```

## Evaluation signals

- Diffusion pseudo-time branch structure derived from tGPT features is highly aligned with known cell labels and states
- HCA branches reproduce the expected organization: HSC to erythroid/DC-monocyte, naive T to cytotoxic T/NK, pro-B to pre-B to naive B to memory/plasma cells
- HCL branches show fetal mesenchymal progenitors to fibroblast/enterocyte/astrocyte-oligodendrocyte, matching the Figure 4A-4K organization
- Diffusion map figures export successfully as PNG files with FLE layouts placing contiguous cell types along pseudo-time

## Limitations

- tGPT uses only the top-expressing genes and may miss information represented in low-expressing genes
- tGPT uses gene expression rankings, not actual expression levels; fold changes among genes are neglected and this can affect biological interpretation
- The language modeling objective predicts gene rankings, which is not directly related to biological issues; further study is required

## Evidence
