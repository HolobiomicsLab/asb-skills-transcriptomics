---
name: differential-gene-expression-analysis-of-t-cell-subsets
description: Use when you have clustered an mvTCR joint embedding of paired scRNA-seq/TCR
  data into antigen-specific and bystander T cell clusters and need to characterize
  their transcriptional differences, e.g. effector CD8+ T cells from SARS-CoV-2 PBMCs.
when_to_use_negative:
- No paired TCR sequences are available — mvTCR was applied mainly to paired data
  (gene expression plus α- and β-CDR3), so the specific/bystander cluster separation
  underlying this DGE comparison cannot be reproduced from gene expression alone.
- You already have explicit specificity annotations and want to train a predictor
  — use mvTCR's training-with-prediction-task workflow (tutorials/03_training_prediction.ipynb)
  instead of cluster-level DGE.
- The dataset is not T cell immune profiling data (e.g. it is a bulk RNA-seq or non-immune
  single-cell study) — cluster labels will not correspond to antigen-specific vs bystander
  T cells.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Scanpy
  role: Provides rank_genes_groups for differential gene expression between specific
    and bystander clusters and hosts the clustered mvTCR embedding in Scanpy-format
- name: mvTCR
  role: Generates the shared cell-level transcriptome–TCR embedding whose clusters
    are the basis of the DGE comparison
  repo: https://github.com/SchubertLab/mvTCR
- name: scirpy
  role: Used for TCR preprocessing/annotation required before embedding and cluster
    labeling
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:603dfa441bf714efdc627aaff04d1c2fbe0584338b9277b0ba322ef5a53be757
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - PyTorch
  - Scanpy
  - mvTCR
  - scirpy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# differential-gene-expression-analysis-of-t-cell-subsets

## Summary

Differential gene expression analysis between antigen-specific and bystander T cell clusters identified in an mvTCR joint transcriptome–TCR embedding, using Scanpy rank_genes_groups to characterize marker gene signatures of SARS-CoV-2-specific versus bystander CD8+ T cells.

## When to use

Use when you have clustered an mvTCR joint embedding of paired scRNA-seq/TCR data into antigen-specific and bystander T cell clusters and need to characterize their transcriptional differences, e.g. effector CD8+ T cells from SARS-CoV-2 PBMCs.

## When NOT to use

- No paired TCR sequences are available — mvTCR was applied mainly to paired data (gene expression plus α- and β-CDR3), so the specific/bystander cluster separation underlying this DGE comparison cannot be reproduced from gene expression alone.
- You already have explicit specificity annotations and want to train a predictor — use mvTCR's training-with-prediction-task workflow (tutorials/03_training_prediction.ipynb) instead of cluster-level DGE.
- The dataset is not T cell immune profiling data (e.g. it is a bulk RNA-seq or non-immune single-cell study) — cluster labels will not correspond to antigen-specific vs bystander T cells.

## Inputs

- Clustered mvTCR joint embedding of paired scRNA-seq/TCR data in Scanpy-format AnnData (e.g. 103,761-cell SARS-CoV-2 PBMC dataset)
- Cluster specificity labels (SARS-CoV-2-specific vs bystander) derived from IEDB TCR-epitope query with MHCFlurry HLA-binding filtering
- Cell type annotation (IFN-response-enriched effector CD8+ T cells)

## Outputs

- Per-cluster / per-group marker gene ranking from Scanpy rank_genes_groups between specific and bystander clusters
- Marker gene direction signatures (KLRD1, GZMB up in bystander; CD8A down, IL7R and MT-CO genes up in specific clusters)

## How to apply

First train mvTCR (v0.1.3 with PyTorch v1.8.0 on a CUDA-capable GPU) to fuse transcriptome and TCR into a shared cell-level embedding, then cluster this embedding with Scanpy and identify the IFN-response-enriched effector CD8+ T cell clusters. Label clusters as antigen-specific or bystander by querying TCRs to the IEDB with MHCFlurry HLA-type filtering against donor HLAs. Run Scanpy rank_genes_groups between the specific and bystander clusters to obtain marker genes for each group. Verify the result by checking marker gene directions: KLRD1 and GZMB should be upregulated in bystander clusters while CD8A is down and IL7R and MT-CO genes are up in specific clusters.

## Related tools

- **Scanpy** (Provides rank_genes_groups for differential gene expression between specific and bystander clusters and hosts the clustered mvTCR embedding in Scanpy-format)
- **mvTCR** (Generates the shared cell-level transcriptome–TCR embedding whose clusters are the basis of the DGE comparison) — https://github.com/SchubertLab/mvTCR
- **scirpy** (Used for TCR preprocessing/annotation required before embedding and cluster labeling)

## Examples

```
import scanpy as sc; sc.tl.rank_genes_groups(adata, groupby='specific_vs_bystander', method='wilcoxon')
```

## Evaluation signals

- Marker direction check: KLRD1 and GZMB are upregulated in bystander clusters while CD8A is down and IL7R and MT-CO genes are up in specific clusters
- Cluster count check: analysis reproduces five SARS-CoV-2-specific and three bystander clusters among IFN-response-enriched effector CD8+ T cells
- DGE is computed on clusters from the joint mvTCR embedding, not from unimodal gene-expression clusters (specific/bystander cells are unidentifiable in unimodal analysis)
- rank_genes_groups is run within the standard Scanpy workflow over the Scanpy-format embedding (mvTCR's representation seamlessly integrates into standard single-cell analysis workflows)
- Setup matches the reproducibility environment: mvTCR v0.1.3 with PyTorch v1.8.0 in an isolated conda environment

## Limitations

- mvTCR was applied mainly to fully paired data (gene expression plus α- and β-CDR3); handling of partially missing TCR data remains insufficiently addressed, which limits DGE comparisons on incomplete datasets.
- Tuning the contribution of each modality requires repeated model retraining with an additional hyperparameter search, so the embedding (and hence cluster definitions feeding the DGE) can vary with hardware and CUDA version.
- Only the CDR3 sequence is used for TCR representation; VDJ-gene encoding and pre-trained TCR embeddings are not incorporated, which may limit specificity labeling of clusters prior to DGE.

## Evidence

- [discussion] mvTCR seamlessly integrates into standard single-cell analysis workflows over the Scanpy-format: "mvTCR’s representation seamlessly integrates into standard single-cell analysis workflows over the Scanpy-format51"
- [discussion] mvTCR well-suited for identifying clusters of related cells within a repertoire: "makes it particularly well-suited for identifying clusters of related cells within a repertoire."
- [abstract] mvTCR distinguishes T cell subpopulations binding to SARS-CoV-2 antigens from bystander cells: "we use mvTCR to distinguish T cell subpopulations binding to SARS-CoV-2 antigens from bystander cells"
- [readme] Reproducibility repo pins mvTCR v0.1.3 with PyTorch v1.8.0: "The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0."
