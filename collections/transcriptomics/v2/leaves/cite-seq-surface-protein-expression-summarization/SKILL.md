---
name: cite-seq-surface-protein-expression-summarization
description: Use when you have scMM output from CITE-seq (transcriptome & surface
  protein) data and need cluster-level multimodal feature heatmaps to interpret a
  high-resolution latent-space clustering, e.g. reproducing the PBMC ~54-cluster figure.
when_to_use_negative:
- Input is raw unnormalized counts with no trained scMM model/output — the skill summarizes
  the joint latent representation, not raw matrices
- Modalities other than transcriptome/surface protein CITE-seq pairs where the NB
  protein decoder and Protein_count.mtx structure do not apply (e.g. chromatin accessibility
  requires the ATAC-seq folder/peak.tsv layout instead)
- The goal is coarse cell-type annotation only, not the ~54-cluster high-resolution
  regime the paper reports
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scMM
  role: Provides the joint transcriptome-protein latent representation and NB-distribution
    surface protein sampling used for cluster-level summarization
  repo: https://github.com/kodaim1115/scMM
- name: R
  role: Downstream analysis environment; Rphenograph clustering, umap embedding, and
    pheatmap rendering of scMM outputs per R/tutorial.R
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:21c66c4c84525d66298668169c0715285afa1fcafb17c6b3f3e36d0dba215ac2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kodaim1115/scMM
  repo_url: https://github.com/kodaim1115/scMM
  tools:
  - R
  - scMM
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cite-seq-surface-protein-expression-summarization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Summarize CITE-seq surface protein expression per cell cluster by sampling protein measurements from scMM's negative binomial distributions and rendering cluster-level z-scored transcriptome and protein heatmap panels. This turns the scMM joint latent representation into fine-grained, interpretable cluster characterizations (~54 clusters on PBMC CITE-seq data).

## When to use

Use when you have scMM output from CITE-seq (transcriptome & surface protein) data and need cluster-level multimodal feature heatmaps to interpret a high-resolution latent-space clustering, e.g. reproducing the PBMC ~54-cluster figure.

## When NOT to use

- Input is raw unnormalized counts with no trained scMM model/output — the skill summarizes the joint latent representation, not raw matrices
- Modalities other than transcriptome/surface protein CITE-seq pairs where the NB protein decoder and Protein_count.mtx structure do not apply (e.g. chromatin accessibility requires the ATAC-seq folder/peak.tsv layout instead)
- The goal is coarse cell-type annotation only, not the ~54-cluster high-resolution regime the paper reports

## Inputs

- scMM output directory for the PBMC analysis (latent representations)
- gene.tsv feature file
- barcode.tsv single-cell barcode file
- CITE-seq Protein_count.mtx surface protein count matrix (MatrixMarket format)

## Outputs

- Rphenograph cluster assignment per cell (~54 clusters)
- UMAP embedding of scMM latent variables colored by cluster
- Side-by-side transcriptome and surface-protein heatmap panels (pheatmap, cluster-level z-scores)
- Per-cluster mean feature tables for genes and surface proteins

## How to apply

Load the scMM output directory in R, reading gene.tsv / barcode.tsv and the CITE-seq Protein_count.mtx matrix. Cluster cells in the scMM latent space with Rphenograph (kNN graph plus Louvain community detection at the tutorial-default neighborhood size) to recover the paper's high-resolution partition (~54 clusters). Embed latent variables with umap in R to visualize the joint transcriptome-protein latent space colored by Rphenograph cluster. Compute per-cluster mean values for genes and surface proteins, scale each feature across clusters to z-scores, and select marker features per modality; scMM samples protein measurements from its NB distributions for each cell for this crossmodal summarization. Render the z-score panels with pheatmap as side-by-side transcriptome and surface-protein heatmap panels to demonstrate cluster-level interpretability.

## Related tools

- **scMM** (Provides the joint transcriptome-protein latent representation and NB-distribution surface protein sampling used for cluster-level summarization) — https://github.com/kodaim1115/scMM
- **R** (Downstream analysis environment; Rphenograph clustering, umap embedding, and pheatmap rendering of scMM outputs per R/tutorial.R)

## Examples

```
Rscript R/tutorial.R
```

## Evaluation signals

- Clustering reaches the paper's reported high-resolution regime of ~54 clusters on PBMC CITE-seq data
- Heatmap panels show coherent, cluster-specific multimodal marker patterns matching the published figure
- Transcriptome and surface-protein panels are z-scored (scaled across clusters) before rendering
- UMAP embedding of the latent space colored by Rphenograph cluster shows well-separated, fine-grained communities
- Output figures reproduce the side-by-side (transcriptome | surface protein) panel layout of the paper

## Limitations

- Cluster count depends on the Rphenograph k-nearest-neighbor neighborhood size; deviating from the tutorial default may not recover the ~54-cluster regime
- Protein heatmap values are sampled from scMM's NB distributions (crossmodal generation), not directly observed counts, so sampled realizations vary
- The README tutorial data are randomly subsampled 15,000-cell BMNC toy data; full PBMC-scale runs may need different resource settings

## Evidence

- [abstract] Analysis of recently produced datasets validated that scMM facilitates high-resolution clustering with rich interpretability.: "Analysis of recently produced datasets validated that scMM facilitates high-resolution clustering with rich interpretability."
- [readme] Tutorial on downstream analysisfor scMM outputs can be found at `R/tutorial.R`: "Tutorial on downstream analysisfor scMM outputs can be found at `R/tutorial.R`"
- [readme] CITE-seq folder structure with Protein_count.mtx, protein.tsv and barcode.tsv: "single-cell barcode stored in `barcode.tsv` should be included"
- [intro] scMM is a novel deep generative model-based framework for the extraction of interpretable joint representations and cross-modal generation for single-cell multiomics data (e.g. transcriptome & chromatin accessibility, transcriptome & surface proteins).: "scMM is a novel deep generative model-based framework for the extraction of interpretable joint representations and cross-modal generation for single-cell multiomics data"
