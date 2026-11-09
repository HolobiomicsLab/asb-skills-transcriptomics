---
name: single-cell-feature-embedding-analysis
description: Use when tGPT feature embedding matrices plus cell type annotations (e.g.
  HCA bone marrow n=282,588 or HCL 586,135 cells) are available and the goal is pseudo-temporal
  developmental lineage inference.
when_to_use_negative:
- Input is raw single-cell count data rather than tGPT-derived feature embeddings
  — the diffusion pseudo-time analysis is computed on tGPT feature representations,
  not expression matrices
- Goal is batch-effect correction or clustering of embeddings — benchmarking/batch
  tasks use separate pipelines (Scanpy 1.6.0, Pegasus 1.4.3, scVI 0.13.0 with MNN/ComBat/Harmony/etc.),
  not the pseudo-time skill
- The dataset is a bulk tumor tissue transcriptome — the lineage trajectory analysis
  targets single-cell atlases such as HCA and HCL
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: tGPT
  role: Generative pretraining model producing the feature embeddings (from the embedding
    layer and 8 transformer layers) on which pseudo-time is computed
  repo: https://github.com/deeplearningplus/tGPT
- name: Pegasus v1.4.3
  role: Constructs diffusion pseudo-time maps from the tGPT feature representations
  repo: https://github.com/lilab-bcb/pegasus
- name: Scanpy
  role: Loads features and cell type labels into AnnData, computes neighbor graphs
    and visualizations (e.g. sc.pp.neighbors, sc.tl.umap)
  repo: https://github.com/theislab/scanpy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2023.106536
    title: Generative pretraining from large-scale transcriptomes for single-cell
      deciphering
  source_content_hash: sha256:41f1f93e7d1f16eb36d9c4591cec360cda8f9a557b4606ed9152fdc5fd5033e8
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# single-cell-feature-embedding-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct developmental lineage trajectories from tGPT-derived feature embeddings by constructing diffusion pseudo-time maps with Pegasus (v1.4.3) and visualizing them with force-directed layout embedding (FLE), validating branch structure against known cell type labels.

## When to use

Use when tGPT feature embedding matrices plus cell type annotations (e.g. HCA bone marrow n=282,588 or HCL 586,135 cells) are available and the goal is pseudo-temporal developmental lineage inference.

## When NOT to use

- Input is raw single-cell count data rather than tGPT-derived feature embeddings — the diffusion pseudo-time analysis is computed on tGPT feature representations, not expression matrices
- Goal is batch-effect correction or clustering of embeddings — benchmarking/batch tasks use separate pipelines (Scanpy 1.6.0, Pegasus 1.4.3, scVI 0.13.0 with MNN/ComBat/Harmony/etc.), not the pseudo-time skill
- The dataset is a bulk tumor tissue transcriptome — the lineage trajectory analysis targets single-cell atlases such as HCA and HCL

## Inputs

- tGPT-derived feature embedding matrix for HCA bone marrow (n=282,588 cells from 64 healthy donors)
- tGPT-derived feature embedding matrix for HCL (586,135 human cells)
- Cell type annotation labels (e.g. celltype column / annotation text files loaded into adata.obs)
- AnnData objects compatible with Scanpy/Pegasus

## Outputs

- Diffusion pseudo-time map computed on tGPT feature representations
- Force-directed layout embedding (FLE) trajectory visualization (PNG figures)
- Resolved developmental branches (e.g. HSC to erythroid/DC-monocyte, naive T to cytotoxic T/NK, pro-B to pre-B lineage paths)
- Alignment assessment of pseudo-time branches against known cell labels and states

## How to apply

Load the tGPT-derived feature embedding matrices and cell type annotation labels (e.g. HCA bone marrow with 282,588 cells, HCL with 586,135 cells) into AnnData objects with Scanpy. Construct diffusion pseudo-time maps using Pegasus (v1.4.3) computed on the tGPT feature representations, and visualize the cell trajectory with the force-directed layout embedding (FLE) algorithm. Decision point: the analysis is run on tGPT feature representations extracted from the embedding layer and 8 transformer layers, where the last_hidden_state is mean-pooled over token positions up to each cell's end-of-sequence index (eos_idx = attention_mask.sum(dim=1) - 1). Validation requires confirming that the diffusion pseudo-time branch structure is highly aligned with known cell labels — HCA branches from HSCs to erythroid cells or DCs and monocytes, naive T cells to cytotoxic T cells and NK cells, and pro-B to pre-B to naive B to memory/plasma cells; HCL branches from fetal mesenchymal progenitors to fibroblast/enterocyte/astrocyte-oligodendrocyte. Export the diffusion map figures as PNG files for the branch structures comparable to the article's Figure 4A-4K.

## Related tools

- **tGPT** (Generative pretraining model producing the feature embeddings (from the embedding layer and 8 transformer layers) on which pseudo-time is computed) — https://github.com/deeplearningplus/tGPT
- **Pegasus v1.4.3** (Constructs diffusion pseudo-time maps from the tGPT feature representations) — https://github.com/lilab-bcb/pegasus
- **Scanpy** (Loads features and cell type labels into AnnData, computes neighbor graphs and visualizations (e.g. sc.pp.neighbors, sc.tl.umap)) — https://github.com/theislab/scanpy

## Examples

```
adata = sc.AnnData(features); adata.obs['celltype'] = pd.read_csv(celltype_path, header=None)[0].astype('category'); sc.pp.neighbors(adata, n_neighbors=20); sc.tl.umap(adata)
```

## Evaluation signals

- The diffusion pseudo-time branch structure derived from tGPT features is highly aligned with known cell labels and states (the article's stated validation criterion)
- HCA branches reproduce the expected organization: HSC to erythroid/DC-monocyte, naive T to cytotoxic T/NK, and pro-B to pre-B to naive B to memory/plasma cells (Figure 4A-4K organization)
- HCL branches resolve fetal mesenchymal progenitors to fibroblast/enterocyte/astrocyte-oligodendrocyte
- AnnData objects contain both the feature embedding (X) and categorical celltype in adata.obs before Pegasus/FLE is run
- Diffusion map figures are exported as PNG files with branch structure matching the source card workflow

## Limitations

- tGPT uses only top-expressing genes; information specifically represented in low-expressing genes may be missed
- tGPT uses gene expression rankings, not actual expression levels, so fold changes among genes are neglected and this can affect biological interpretation
- The language modeling objective predicts gene rankings, which is not directly related to biological issues; further study is required
- This does not guarantee that the feature representation from the last transformer layer gives rise to better clustering

## Evidence
