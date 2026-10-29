---
name: mhc-peptide-binding-prediction-filtering
description: Use when TCR repertoires (paired scRNA-seq + TCR-seq, e.g. PBMCs from
  SARS-CoV-2 patients) are clustered and clusters must be annotated as antigen-specific
  vs bystander using epitope database hits filtered by donor HLA binding.
when_to_use_negative:
- Specificity annotations already exist (e.g. training mvTCR's prediction task with
  known specificity labels via tutorials/03_training_prediction.ipynb) — no IEDB/HLA
  filtering needed
- Donor HLA types are unknown, since MHCFlurry filtering cannot be applied and IEDB
  hits cannot be narrowed to patient-specific binders
- Data are unpaired or lack CDR3 sequences, since mvTCR is mainly applied to paired
  data with gene expression plus alpha- and beta-CDR3
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: IEDB
  role: Epitope database queried for SARS-CoV-2 TCR-epitope pairings
- name: MHCFlurry
  role: Predicts peptide binding to donor HLAs to filter IEDB hits before labeling
    clusters
- name: mvTCR
  role: Provides the joint transcriptome-TCR embedding whose clusters receive the
    antigen-specific/bystander labels
  repo: https://github.com/SchubertLab/mvTCR
- name: Scanpy
  role: Cluster the mvTCR embedding and run rank_genes_groups to verify marker gene
    directions between labeled clusters
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:c3550485fc12a74aeeab8c7a79a0d53eaa4164423cf90c592b14d8aae34dd3ac
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# MHC-peptide binding prediction filtering

## Summary

Labels T-cell clusters as antigen-specific or bystander by querying TCRs against IEDB for epitope pairings and retaining only hits whose peptide is predicted (e.g. with MHCFlurry) to bind the donor's HLAs. This HLA-aware filtering converts generic epitope matches into patient-specific specificity annotations for downstream mvTCR cluster interpretation.

## When to use

Use when TCR repertoires (paired scRNA-seq + TCR-seq, e.g. PBMCs from SARS-CoV-2 patients) are clustered and clusters must be annotated as antigen-specific vs bystander using epitope database hits filtered by donor HLA binding.

## When NOT to use

- Specificity annotations already exist (e.g. training mvTCR's prediction task with known specificity labels via tutorials/03_training_prediction.ipynb) — no IEDB/HLA filtering needed
- Donor HLA types are unknown, since MHCFlurry filtering cannot be applied and IEDB hits cannot be narrowed to patient-specific binders
- Data are unpaired or lack CDR3 sequences, since mvTCR is mainly applied to paired data with gene expression plus alpha- and beta-CDR3

## Inputs

- Per-cluster TCR sequences (CDR3 clonotypes) from mvTCR-embedded, Scanpy/scirpy-processed single-cell data
- Donor HLA type assignments
- SARS-CoV-2 TCR-epitope pairings from the IEDB database

## Outputs

- Cluster-level labels: antigen-specific vs bystander T-cell clusters
- List of HLA-binding-filtered TCR-epitope matches per cluster

## How to apply

After clustering the mvTCR joint transcriptome-TCR embedding (e.g. identifying IFN-response-enriched effector CD8+ clusters), query the cluster TCRs against IEDB for SARS-CoV-2 TCR-epitope pairings. For each hit, predict peptide binding to the donor's HLA types (e.g. with MHCFlurry) and keep only peptides predicted to bind the donor HLAs, so that unbindable epitope matches are discarded. Clusters enriched for HLA-filtered hits are labeled SARS-CoV-2-specific; matched clusters lacking such binding support are labeled bystander. Validate by differential gene expression between the two label groups — in the reference SARS-CoV-2 PBMC cohort this yields five specific and three bystander clusters, with KLRD1 and GZMB up in bystander clusters while CD8A is down and IL7R and MT-CO genes are up in specific clusters.

## Related tools

- **IEDB** (Epitope database queried for SARS-CoV-2 TCR-epitope pairings)
- **MHCFlurry** (Predicts peptide binding to donor HLAs to filter IEDB hits before labeling clusters)
- **mvTCR** (Provides the joint transcriptome-TCR embedding whose clusters receive the antigen-specific/bystander labels) — https://github.com/SchubertLab/mvTCR
- **Scanpy** (Cluster the mvTCR embedding and run rank_genes_groups to verify marker gene directions between labeled clusters)

## Evaluation signals

- Reference result reproduced: five SARS-CoV-2-specific and three bystander clusters among IFN-response-enriched effector CD8+ T cells in the 103,761-cell PBMC cohort
- Marker gene directions match published signatures: KLRD1 and GZMB upregulated in bystander clusters; CD8A down and IL7R and MT-CO genes up in specific clusters (Scanpy rank_genes_groups)
- Every retained IEDB hit has predicted binding to at least one donor HLA type (MHCFlurry filter applied, no unfiltered epitope labels)
- Specific-vs-bystander labels align with cluster structure in the mvTCR joint embedding, not with gene-expression-only clusters

## Limitations

- mvTCR was mainly applied to paired data (gene expression with alpha- and beta-CDR3); partially missing TCR data is not well handled, restricting which cells can be assigned specificity labels
- mvTCR currently uses only the CDR3 sequence for TCR representation, so specificity calls rest on CDR3 matching alone
- Specificity labels are database-dependent (IEDB coverage) and HLA-prediction-dependent (MHCFlurry accuracy); unreported epitope-HLA combinations yield no annotation

## Evidence

- [abstract] mvTCR distinguishes T cell subpopulations binding to SARS-CoV-2 antigens from bystander cells: "we use mvTCR to distinguish T cell subpopulations binding to SARS-CoV-2 antigens from bystander cells"
- [results] IEDB query with MHCFlurry HLA-type filtering: [evidence span withheld — anchor status: not_found]
- [methods] Workflow step: IEDB query plus MHCFlurry donor-HLA filtering: [evidence span withheld — anchor status: not_found]
- [results] Validation criterion for specific vs bystander cluster reproduction: [evidence span withheld — anchor status: not_found]
