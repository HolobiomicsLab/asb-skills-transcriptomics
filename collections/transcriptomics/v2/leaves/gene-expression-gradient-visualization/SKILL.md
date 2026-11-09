---
name: gene-expression-gradient-visualization
description: Use when you have cross-modality latent embeddings (e.g. from scConfluence's
  diagonal integration of morphology and scRNA-seq) and need to inspect whether a
  marker gene such as Fezf2 shows a graded pattern along a biological manifold (e.g.
  apical-dendrite organization of IT neurons).
when_to_use_negative:
- No shared latent embedding exists yet and the modalities have not been integrated
  (e.g. electrophysiology vs gene expression, where cross-modality feature connections
  are unavailable)
- The goal is quantitative gradient testing rather than qualitative visualization
  of expression along a manifold
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: scConfluence
  role: Produces the shared low-dimensional latent embeddings that the UMAP/gradient
    visualization is built on
  repo: https://github.com/cantinilab/scconfluence
- name: scanpy
  role: sc.pp.neighbors and sc.tl.umap compute the graph and 2D embedding; sc.pl.umap
    renders expression-gradient overlays
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:c9c49fee8b5df48b3c06b1671d52131e3f2203bddb9a9f0b44f16e8135829f79
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
  - scConfluence
  - scanpy
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 3
---

# gene-expression-gradient-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualize gene expression gradients over a shared low-dimensional cell embedding by computing a neighbor graph and UMAP on scConfluence latent embeddings and coloring cells by marker expression (e.g. Fezf2) and cell-type labels to reveal molecular-morphological gradients.

## When to use

Use when you have cross-modality latent embeddings (e.g. from scConfluence's diagonal integration of morphology and scRNA-seq) and need to inspect whether a marker gene such as Fezf2 shows a graded pattern along a biological manifold (e.g. apical-dendrite organization of IT neurons).

## When NOT to use

- No shared latent embedding exists yet and the modalities have not been integrated (e.g. electrophysiology vs gene expression, where cross-modality feature connections are unavailable)
- The goal is quantitative gradient testing rather than qualitative visualization of expression along a manifold

## Inputs

- MuData object with latent embeddings in mdata.obsm['latent'] (from scConfluence model.get_latent())
- Cell-level metadata in mdata.obs (e.g. tufted/untufted IT neuron labels)
- Normalized gene expression matrix containing the marker gene (e.g. Fezf2) in the scRNA-seq modality

## Outputs

- Neighbor graph stored in mdata (sc.pp.neighbors on use_rep='latent')
- UMAP coordinates in mdata.obsm (sc.tl.umap)
- UMAP scatter plots colored by cell-type labels and by marker gene expression (Fig. 6b-d analog)

## How to apply

After scConfluence training, retrieve shared latent embeddings with model.get_latent() and write them into mdata.obsm['latent'], aligned to mdata.obs_names. Compute the kNN graph on this representation with sc.pp.neighbors(mdata, use_rep='latent') and embed with sc.tl.umap(mdata). Then render the UMAP colored by cell-type labels (tufted vs untufted IT neurons) and by the marker gene's expression (Fezf2) to reproduce Fig. 6b-d style panels. The correct result is a spatially organized manifold where the marker signal changes gradually rather than in isolated patches — for the Patch-seq data this means tufted and untufted Intra Telencephalic neurons arranged along the apical-dendrite axis with a graded Fezf2 pattern, highest in tufted layer-5 IT neurons.

## Related tools

- **scConfluence** (Produces the shared low-dimensional latent embeddings that the UMAP/gradient visualization is built on) — https://github.com/cantinilab/scconfluence
- **scanpy** (sc.pp.neighbors and sc.tl.umap compute the graph and 2D embedding; sc.pl.umap renders expression-gradient overlays)

## Examples

```
sc.pp.neighbors(mdata, use_rep='latent'); sc.tl.umap(mdata); sc.pl.umap(mdata, color=['Fezf2'])
```

## Evaluation signals

- mdata.obsm['latent'] row count and order match mdata.obs_names before computing neighbors
- UMAP embedding exists in mdata.obsm after sc.tl.umap and the neighbor graph was computed with use_rep='latent'
- Tufted and untufted IT neurons form organized, non-random structure on the UMAP rather than intermixing randomly
- Fezf2 expression shows a graded (continuous) pattern along the embedding aligned with the apical-dendrite manifold, highest in tufted layer-5 IT neurons
- Reproduced panels are visually consistent with Fig. 6b-d of the scConfluence paper

## Limitations

- scConfluence and downstream visualization handle rare cell populations poorly; rare populations may be under-represented or invisible in the embedding
- The gradient is a qualitative visualization signal, not a statistical test of association
- Requires a GPU is not required but strongly recommended for scConfluence upstream; CPU-only runs may be slow on large datasets

## Evidence

- [discussion] neighbor graph and UMAP on latent: [evidence span withheld — anchor status: not_found]
