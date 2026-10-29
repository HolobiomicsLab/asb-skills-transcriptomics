---
name: trajectory-differential-expression-and-go-enrichment
description: Use when a trained LatentVelo (or similar latent-dynamics) model has
  produced latent trajectories and you need the genes driving each lineage path plus
  GO term interpretation, e.g. endothelial/erythroid/mesenchyme/neural gastrulation
  trajectories.
when_to_use_negative:
- The model has not yet been trained and latent trajectories have not been computed
  — trajectory DE and GO enrichment require ltv.cell_trajectories output first
- Sparse cell clusters with few observed transient cells connect the trajectories
  — LatentVelo's inferred trajectories may incorrectly start from disconnected clusters
  (e.g. microglia in dentate gyrus), invalidating downstream DE ordering
- The input is a gene count matrix without spliced/unspliced layers or a fitted dynamics
  model — GO enrichment alone does not need this skill's trajectory machinery
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: LatentVelo
  role: Provides the trained model and ltv.cell_trajectories to extract latent developmental
    trajectories along which DE genes are ranked
  repo: https://github.com/Spencerfar/LatentVelo
- name: GSEApy
  role: Performs GO gene set enrichment analysis on the differentially expressed gene
    lists per trajectory
  repo: https://github.com/zqfang/GSEApy
- name: scanpy
  role: AnnData handling and differential expression/ranking utilities along trajectory-ordered
    cells
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100581
    title: Inferring single-cell transcriptomic dynamics with structured latent gene
      expression dynamics
  source_content_hash: sha256:67927333ff1768ec071261235432658af9d657e6bf85ae6e5f1385db36e32dd7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Spencerfar/LatentVelo
  repo_url: https://github.com/Spencerfar/LatentVelo
  tools:
  - GSEApy
  - LatentVelo
  - scanpy
  - scib
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# trajectory-differential-expression-and-go-enrichment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts cell trajectories from a trained LatentVelo model with ltv.cell_trajectories, identifies genes differentially expressed along inferred developmental paths, and runs GO enrichment with GSEApy to give biological meaning to predicted latent trajectories.

## When to use

Use when a trained LatentVelo (or similar latent-dynamics) model has produced latent trajectories and you need the genes driving each lineage path plus GO term interpretation, e.g. endothelial/erythroid/mesenchyme/neural gastrulation trajectories.

## When NOT to use

- The model has not yet been trained and latent trajectories have not been computed — trajectory DE and GO enrichment require ltv.cell_trajectories output first
- Sparse cell clusters with few observed transient cells connect the trajectories — LatentVelo's inferred trajectories may incorrectly start from disconnected clusters (e.g. microglia in dentate gyrus), invalidating downstream DE ordering
- The input is a gene count matrix without spliced/unspliced layers or a fitted dynamics model — GO enrichment alone does not need this skill's trajectory machinery

## Inputs

- Trained LatentVelo model (ltv.models.AnnotVAE or ltv.models.VAE with saved parameters)
- AnnData object (h5ad) with spliced/unspliced layers, batch_key and celltype_key prepared via ltv.utils.anvi_clean_recipe
- Latent-space AnnData from ltv.output_results with 'spliced_velocity' key
- Gene set databases / GO term annotation files for GSEApy enrichment

## Outputs

- z_traj and times arrays from ltv.cell_trajectories describing latent trajectories
- Per-trajectory differentially expressed gene lists (endothelial, erythroid, mesenchyme, neural)
- GO enrichment results from GSEApy (enriched terms, p-values, NES)
- Composite figure panels combining trajectory DE and GO enrichment with benchmark metrics

## How to apply

After training the AnnotVAE (e.g. latent_dim=70, zr_dim=6, batch_correction=True, batches=3 on the 20,000-cell mouse gastrulation AnnData), call ltv.cell_trajectories(model, adata) to obtain the latent trajectory z_traj and associated times, then plot the paths on the latent-space UMAP. For each inferred trajectory (e.g. endothelial, erythroid, mesenchyme, neural), identify genes differentially expressed along the path using scanpy ranking utilities on the trajectory-ordered cells. Run GO enrichment on each gene list with GSEApy, which implements a Rust port of fgsea's multilevel algorithm for gene set enrichment. Assemble the enrichment results as panels (e.g. the GO-enrichment panel of figure5_panels.png) alongside transition-score, batch-mixing, and cosine-similarity metrics.

## Related tools

- **LatentVelo** (Provides the trained model and ltv.cell_trajectories to extract latent developmental trajectories along which DE genes are ranked) — https://github.com/Spencerfar/LatentVelo
- **GSEApy** (Performs GO gene set enrichment analysis on the differentially expressed gene lists per trajectory) — https://github.com/zqfang/GSEApy
- **scanpy** (AnnData handling and differential expression/ranking utilities along trajectory-ordered cells)

## Examples

```
z_traj, times = ltv.cell_trajectories(model, adata); sc.pl.umap(latent_adata, color='celltype'); gseapy.enrichr(gene_list=de_genes, gene_sets='GO_Biological_Process_2021', organism='Mouse')
```

## Evaluation signals

- ltv.cell_trajectories returns non-empty z_traj and times arrays with a trajectory for each requested lineage/initial-cell state
- Trajectories plotted on the latent UMAP follow the expected developmental transitions (e.g. toward endothelial, erythroid, mesenchyme, neural fates) rather than starting from a disconnected cluster
- Each trajectory yields a non-trivial DE gene list that can be supplied to GSEApy without empty results
- GSEApy enrichment returns significant GO terms consistent with the known biology of each lineage (validated against the paper's Figure 5 GO-enrichment panel)

## Limitations

- LatentVelo assumes a single initial latent state from which all trajectories start, and every cell must be reachable by a continuous trajectory; violations distort trajectory ordering used for DE
- In sparse cell clusters with few observed transient cells connecting trajectories, latent trajectories may not start from expected initial states (e.g. dentate gyrus microglia misassigned lowest latent times)
- LatentVelo hyperparameters may need adjustment (larger latent state dimension, cell-type annotated AnnotVAE, or gene subsetting) before trajectories are reliable enough for DE/GO interpretation

## Evidence

- [other] Task workflow combining trajectories, DE, and GSEApy GO enrichment on gastrulation lineages: [evidence span withheld — anchor status: not_found]
- [readme] GSEApy citation and role as Python GSEA implementation: [evidence span withheld — anchor status: external_doc]
