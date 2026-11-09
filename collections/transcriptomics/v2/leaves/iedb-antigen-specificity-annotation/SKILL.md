---
name: iedb-antigen-specificity-annotation
description: Use when paired scRNA-seq + TCR data from patients with known epitope
  exposure (e.g. SARS-CoV-2 PBMCs) and donor HLA types are available, and clusters
  in a mvTCR joint embedding must be labeled antigen-specific vs bystander.
when_to_use_negative:
- TCR sequences are unpaired or partially missing (mvTCR was applied mainly to fully
  paired α/β-CDR3 + gene expression data)
- Donor HLA types are unavailable, so MHCFlurry donor-specific binding filtering cannot
  be performed
- Data are BCR/antibody repertoires — the paper notes no in-depth benchmark exists
  for B-cell receptors due to lack of large-scale B cell datasets with specificity
  annotation
tools:
- name: IEDB
  role: Database queried for TCR-epitope pairings to nominate SARS-CoV-2-specific
    TCRs
- name: MHCFlurry
  role: Predicts epitope binding to donor HLAs to filter IEDB hits
- name: mvTCR
  role: Produces the joint transcriptome+TCR embedding whose clusters receive the
    specificity labels
  repo: https://github.com/SchubertLab/mvTCR
- name: Scanpy
  role: Clusters the embedding and runs rank_genes_groups for marker-gene validation
- name: scirpy
  role: Preprocesses and annotates the paired TCR data prior to embedding
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:64922938e020440c320d6b5115c22a96540df0edc0bc4be957471052871317e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - IEDB
  - MHCFlurry
  - PyTorch
  - Scanpy
  - mvTCR
  - scirpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# IEDB antigen-specificity annotation

## Summary

Labels T cell clusters as antigen-specific or bystander by querying TCR sequences against IEDB for epitope pairings and retaining only hits whose restricting epitope is predicted (MHCFlurry) to bind the donor's HLAs, then validating labels via differential gene expression.

## When to use

Use when paired scRNA-seq + TCR data from patients with known epitope exposure (e.g. SARS-CoV-2 PBMCs) and donor HLA types are available, and clusters in a mvTCR joint embedding must be labeled antigen-specific vs bystander.

## When NOT to use

- TCR sequences are unpaired or partially missing (mvTCR was applied mainly to fully paired α/β-CDR3 + gene expression data)
- Donor HLA types are unavailable, so MHCFlurry donor-specific binding filtering cannot be performed
- Data are BCR/antibody repertoires — the paper notes no in-depth benchmark exists for B-cell receptors due to lack of large-scale B cell datasets with specificity annotation

## Inputs

- Paired single-cell RNA-seq + TCR (scRNA/TCR) data, e.g. the 103,761-cell SARS-CoV-2 PBMC cohort
- Donor HLA types for MHCFlurry binding prediction
- Cluster assignments from the mvTCR joint embedding (Scanpy clustering)
- Processed data annotated per tutorials/01_preprocessing.ipynb (Scanpy/Scirpy)

## Outputs

- Per-cluster labels: SARS-CoV-2-specific (5 clusters reported) vs bystander (3 clusters reported) among IFN-response-enriched effector CD8+ T cells
- TCR-epitope pairing table filtered by MHCFlurry donor-HLA binding
- rank_genes_groups results confirming marker-gene directions (KLRD1/GZMB up in bystander; CD8A down, IL7R/MT-CO up in specific)

## How to apply

After clustering the mvTCR cell-level embedding (Scanpy) and identifying IFN-response-enriched effector CD8+ clusters, query the TCR sequences to the Immune Epitope Database (IEDB) to retrieve TCR-epitope pairings. Filter candidate pairings with MHCFlurry so that only epitopes predicted to bind the donor's HLA types are retained, then use the surviving pairings to label clusters containing matching TCRs as SARS-CoV-2-specific and clusters without such matches as bystander. Validate labels by differential expression (Scanpy rank_genes_groups) between the two cluster sets: bystander clusters should show upregulated KLRD1 and GZMB, while specific clusters should show downregulated CD8A and upregulated IL7R and MT-CO genes. The reproducibility setup uses mvTCR v0.1.3 with PyTorch v1.8.0 on a CUDA-capable GPU, as in tutorials/02_training_analysis.ipynb.

## Related tools

- **IEDB** (Database queried for TCR-epitope pairings to nominate SARS-CoV-2-specific TCRs)
- **MHCFlurry** (Predicts epitope binding to donor HLAs to filter IEDB hits)
- **mvTCR** (Produces the joint transcriptome+TCR embedding whose clusters receive the specificity labels) — https://github.com/SchubertLab/mvTCR
- **Scanpy** (Clusters the embedding and runs rank_genes_groups for marker-gene validation)
- **scirpy** (Preprocesses and annotates the paired TCR data prior to embedding)

## Evaluation signals

- Analysis reproduces five SARS-CoV-2-specific and three bystander clusters among IFN-response-enriched effector CD8+ T cells
- Scanpy rank_genes_groups shows KLRD1 and GZMB upregulated in bystander clusters
- rank_genes_groups shows CD8A downregulated and IL7R and MT-CO genes upregulated in specific clusters
- Every IEDB-derived specificity call is supported by an epitope with MHCFlurry-predicted binding to the corresponding donor HLA
- Specific vs bystander separation is not achievable from a unimodal (transcriptome-only or TCR-only) analysis

## Limitations

- mvTCR was mainly applied to fully paired data (gene expression with α- and β-CDR3); handling of partially missing TCR data is insufficiently addressed
- Adjusting the contribution of each modality requires repeated retraining, ideally with an additional hyperparameter search
- Only the CDR3 sequence is used for the TCR representation; VDJ-gene encoding and pre-trained TCR embedding models are not incorporated
- Re-trained models may diverge from the published ones due to hardware/CUDA differences; pre-trained models are provided on Zenodo (doi 10.5281/zenodo.7215447)

## Evidence

- [methods] IEDB query filtered by donor HLA: [evidence span withheld — anchor status: not_found]
- [abstract] mvTCR separates antigen-specific from bystander cells: "we use mvTCR to distinguish T cell subpopulations binding to SARS-CoV-2 antigens from bystander cells"
- [discussion] SARS-CoV-2 dataset result: "In a large-scale SARS-CoV-2 dataset, mvTCR effectively distinguished between bystander and antigen-specific T cells."
- [methods] Training with specificity annotation tutorial: "tutorials/03_training_prediction.ipynb : training the model, when you have specificity annotation."
- [readme] Reproducibility pinning: [evidence span withheld — anchor status: external_doc]
