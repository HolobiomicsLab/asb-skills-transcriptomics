---
name: variational-autoencoder-joint-embedding
description: Use when you have paired RNA + ADT (CITE-Seq/TEA-Seq) counts and want
  a joint low-dimensional embedding from which to cluster cells, or when benchmarking
  decision-level integrators (scTriangulate) against data-level VAE-based joint embedding.
when_to_use_negative:
- When the goal is decision-level integration that makes no assumption about the distributions
  of raw underlying values, or when mixing annotation sets from label transfer, multiple
  algorithms, and multiple resolutions — scTriangulate is designed for that, while
  TotalVI is limited to specific modalities (RNA + surface protein)
- When the data are single-modality only (e.g. ATAC alone); TotalVI requires RNA paired
  with surface protein expression
- When you need to reconcile conflicting pre-existing cluster labels rather than produce
  a joint embedding from counts
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: TotalVI
  role: Variational autoencoder that infers the joint latent space from RNA and surface
    protein expression; trained for 400 epochs on top-4000 variable genes with scvi-tools
    0.14.3
- name: scanpy
  role: Runs Leiden clustering on the TotalVI joint embedding to derive the baseline
    clustering solution
- name: scTriangulate
  role: Decision-level framework benchmarked against this VAE joint-embedding baseline
    on the TEA-Seq dataset
  repo: https://github.com/frankligy/scTriangulate
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:98559b503d0133616057fcf283e27da4c1d4eb5c9a6d47bcfa9cec24f1b1ab5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - ClusterEnsembles
  - TotalVI
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# variational-autoencoder-joint-embedding

## Summary

Data-level multimodal integration in which a variational autoencoder (TotalVI, scvi-tools 0.14.3) infers a joint latent space from RNA and surface-protein (ADT) expression, on which Leiden clustering is then run. In the scTriangulate paper it serves as the state-of-the-art joint-embedding baseline benchmarked against decision-level integration on TEA-Seq PBMC data.

## When to use

Use when you have paired RNA + ADT (CITE-Seq/TEA-Seq) counts and want a joint low-dimensional embedding from which to cluster cells, or when benchmarking decision-level integrators (scTriangulate) against data-level VAE-based joint embedding.

## When NOT to use

- When the goal is decision-level integration that makes no assumption about the distributions of raw underlying values, or when mixing annotation sets from label transfer, multiple algorithms, and multiple resolutions — scTriangulate is designed for that, while TotalVI is limited to specific modalities (RNA + surface protein)
- When the data are single-modality only (e.g. ATAC alone); TotalVI requires RNA paired with surface protein expression
- When you need to reconcile conflicting pre-existing cluster labels rather than produce a joint embedding from counts

## Inputs

- Paired RNA + ADT count matrices from a multimodal single-cell dataset (e.g. TEA-Seq RNA-ATAC combined h5 files and ADT count CSV from NCBI GEO GSM4949911)
- AnnData object with QC-filtered nuclei (~8,213 nuclei in the TEA-Seq benchmark)
- Highly variable gene set (top-4000 variable genes in the paper)
- External reference annotations for benchmarking (Azimuth human-pbmc level-2 labels)

## Outputs

- TotalVI joint latent embedding of RNA+ADT data
- Leiden clustering assignments computed on the TotalVI joint embedding
- Clustering performance metrics (Homogeneity, Completeness, V-Measure) against the Azimuth L2 reference

## How to apply

After QC (e.g. TEA-Seq filtering of min_gene >= 300, min_counts >= 500, pct_counts_mt <= 20%, min_atac_peaks >= 1000 to retain 8,213 nuclei), select the top-4000 highly variable genes from the RNA and ADT data, then train a TotalVI variational autoencoder using scvi-tools (version 0.14.3 in the paper) for 400 epochs to infer the joint latent space considering RNA and surface protein expression. Run Leiden clustering directly on the resulting TotalVI joint embedding to produce the clustering solution used as a baseline. Compare the outcome against alternatives (Seurat-WNN, HGPA/MCLA/HBGF, scTriangulate) using Homogeneity, Completeness, and V-Measure against an independent reference (e.g. Azimuth human-pbmc L2 annotations). Rationale: VAE-based integration operates at the data level via a joint latent space, unlike scTriangulate, which integrates at the decision level and makes no assumption about the distributions of raw underlying values — the paper found scTriangulate outperformed TotalVI across all resolutions on TEA-Seq.

## Related tools

- **TotalVI** (Variational autoencoder that infers the joint latent space from RNA and surface protein expression; trained for 400 epochs on top-4000 variable genes with scvi-tools 0.14.3)
- **scanpy** (Runs Leiden clustering on the TotalVI joint embedding to derive the baseline clustering solution)
- **scTriangulate** (Decision-level framework benchmarked against this VAE joint-embedding baseline on the TEA-Seq dataset) — https://github.com/frankligy/scTriangulate

## Evaluation signals

- TotalVI training completes 400 epochs on the top-4000 variable genes without error (scvi-tools 0.14.3)
- Leiden clustering applied to the TotalVI embedding yields per-cell population assignments that can be tabulated against the 9 unimodal Leiden solutions
- Homogeneity, Completeness, and V-Measure computed against the Azimuth human-pbmc L2 reference are lower than (or comparable to) scTriangulate's, as expected from the paper's finding
- Rare populations (NK CD56+ bright, B-memory, HSCP) are examined in the final annotation set to assess recovery

## Limitations

- TotalVI is limited to specific modalities (RNA + surface protein) and specific types of unsupervised clustering, and results depend on the software resolution selected
- Data-level joint embedding makes assumptions about the distributions of raw underlying values, unlike decision-level integration
- In the TEA-Seq tri-modal benchmark, TotalVI underperformed scTriangulate at all evaluated clustering resolutions

## Evidence

- [intro] TotalVI, which leverages a variational autoencoder to infer the joint latent space considering RNA and surface protein expression: "and TotalVI, which leverages a variational autoencoder to infer the joint latent space considering RNA and surface protein expression"
- [abstract] Unlike existing integration strategies that rely on modality-specific joint embedding or geometric graphs, scTriangulate makes no assumption about the distributions of raw underlying values.: "Unlike existing integration strategies that rely on modality-specific joint embedding or geometric graphs, scTriangulate makes no assumption"
- [intro] While other multimodal methods integrate at the data level (through either a low-dimensional latent space6 or through a geometric graph5), scTriangulate integrates results at a decision level: "While other multimodal methods integrate at the data level (through either a low-dimensional latent space6 or through a geometric graph5),"
- [results] scTriangulate showed improved performance over all evaluated clustering solutions including state-of-the-art multimodal integration.: [evidence span withheld — anchor status: not_found]
- [readme] Integrate labels from multi-modality single cell datasets (CITE-Seq, Multiome, TEA-Seq, ASAP-Seq, etc.).: "Integrate labels from multi-modality single cell datasets (CITE-Seq, Multiome, TEA-Seq, ASAP-Seq, etc.)."
