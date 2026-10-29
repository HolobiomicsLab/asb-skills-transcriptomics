---
name: louvain-clustering-nmi-optimization
description: Use when evaluating integrated scRNA-seq embeddings (e.g., scDREAMER
  vs. scVI/Scanorama/Harmony baselines) and you need per-method cluster assignments
  that are resolution-optimized before computing NMI, ARI, or ASW metrics.
when_to_use_negative:
- When the goal is batch-correction-only assessment (batch ASW, PCR batch, kBET, graph
  connectivity) that does not require cluster assignments
- When no ground-truth cell-type labels exist to compute NMI, since the resolution
  cannot be selected by NMI maximization
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Louvain
  role: Graph-based clustering algorithm applied to the integrated embedding with
    a resolution swept to maximize NMI against ground-truth cell labels
- name: Scanpy
  role: Framework in which the AnnData embeddings and clustering/metric computation
    are performed
  repo: https://github.com/Zafar-Lab/scDREAMER-reproducibility
- name: scDREAMER
  role: Produces the adversarial-VAE latent embeddings whose clusters are computed;
    one method among twelve compared
  repo: https://github.com/Zafar-Lab/scDREAMER
provenance:
  source_task_ids:
  - task_008
  - task_005
  - task_003
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43590-8
    title: scDREAMER for atlas-level integration of single-cell datasets using deep
      generative model paired with adversarial classifier
  source_content_hash: sha256:d080943aee0fa3ece20af15ce4052228b1012c41c70473d80b847e2177bbdd66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Zafar-Lab/scDREAMER-reproducibility
  tools:
  - ADAM optimizer
  - BBKNN
  - DESC
  - Harmony
  - INSCT
  - LIGER
  - Louvain
  - Louvain clustering
  - Scanorama
  - Scanpy
  - Seurat
  - iMAP
  - scANVI
  - scDML
  - scDREAMER
  - scGEN
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# Louvain clustering with NMI-maximizing resolution

## Summary

Clusters each integrated single-cell embedding with the Louvain algorithm, sweeping resolution to find the value that maximizes NMI against ground-truth cell-type labels, so downstream bio-conservation metrics (NMI, ARI, cell-type ASW) are computed on a fair, optimized clustering per method.

## When to use

Use when evaluating integrated scRNA-seq embeddings (e.g., scDREAMER vs. scVI/Scanorama/Harmony baselines) and you need per-method cluster assignments that are resolution-optimized before computing NMI, ARI, or ASW metrics.

## When NOT to use

- When the goal is batch-correction-only assessment (batch ASW, PCR batch, kBET, graph connectivity) that does not require cluster assignments
- When no ground-truth cell-type labels exist to compute NMI, since the resolution cannot be selected by NMI maximization

## Inputs

- Integrated low-dimensional embedding (latent space) from scDREAMER or a competing method (e.g., scVI, INSCT, Scanorama)
- Ground-truth cell-type annotations for the cells (Scanpy AnnData object with labels)

## Outputs

- Cluster assignments per cell at the NMI-maximizing Louvain resolution
- NMI and ARI values against ground-truth labels, and cell-type ASW (scaled (ASW+1)/2) computed from those clusters, feeding per-method metrics tables such as pancreas_integration_metrics.csv

## How to apply

After obtaining each method's integrated latent embedding, run the Louvain clustering algorithm and sweep the resolution parameter, selecting the resolution that maximizes the NMI value computed against the ground-truth cell-type annotations. Reuse these optimized cluster assignments for all subsequent bio-conservation metrics (NMI, ARI, cell-type average silhouette width scaled as (ASW+1)/2) so that comparisons across methods such as scDREAMER, scVI, Scanorama, Harmony, Seurat, BBKNN, INSCT, LIGER, iMAP, scANVI, scGEN, and scDML are not biased by a fixed, suboptimal resolution. This resolution-optimization step is applied identically per method on each dataset (pancreas, lung, human immune, heart, macaque retina, 1M-cell cross-species). Finally, min-max normalize each metric across methods before averaging into composite bio-conservation and batch-correction scores.

## Related tools

- **Louvain** (Graph-based clustering algorithm applied to the integrated embedding with a resolution swept to maximize NMI against ground-truth cell labels)
- **Scanpy** (Framework in which the AnnData embeddings and clustering/metric computation are performed) — https://github.com/Zafar-Lab/scDREAMER-reproducibility
- **scDREAMER** (Produces the adversarial-VAE latent embeddings whose clusters are computed; one method among twelve compared) — https://github.com/Zafar-Lab/scDREAMER

## Examples

```
sc.pp.neighbors(adata, use_rep='latent'); sc.tl.louvain(adata, resolution=res_grid[np.argmax([metrics.adjusted_mutual_info(adata.obs['celltype'], sc.tl.louvain(adata, resolution=r, return_idents=True)) for r in res_grid])])
```

## Evaluation signals

- The selected Louvain resolution is the one that maximizes NMI against the ground-truth cell-type labels for that embedding
- Computed NMI and ARI values fall in [0, 1] and scaled cell-type ASW = (ASW+1)/2 lies in [0, 1]
- The same resolution-optimization and cluster assignments are reused consistently for NMI, ARI, and cell-type ASW of a given method
- Every competing method (scVI, Scanorama, Harmony, Seurat, BBKNN, INSCT, LIGER, iMAP, scANVI, scGEN, scDML, scDREAMER) yields a computable set of metrics so the composite-score table is complete

## Limitations

- Resolution search is supervised: it requires ground-truth cell-type annotations, so it cannot be used on fully unlabeled datasets
- Louvain clustering results can be sensitive to the kNN graph underlying the integrated embedding; low-quality embeddings may prevent any resolution from recovering rare cell types
- For very large datasets (e.g., the ~1M-cell cross-species task), kBET and other graph computations may hit computational limitations, though Louvain itself remained tractable

## Evidence

- [methods] Louvain clustering at NMI-maximizing resolution applied per method: "We have used the Louvain clustering algorithm to compute the clusters with a resolution that maximizes the NMI value."
- [supplementary] Metrics derived from Louvain clustering feed the bio-conservation composite scores: [evidence span withheld — anchor status: not_found]
- [readme] Readme points to reproducibility notebooks for the analyses: [evidence span withheld — anchor status: external_doc]
