---
name: latent-space-integration-benchmarking
description: Use when a multi-batch single-cell dataset has been integrated with a
  disentangling model (e.g., CODAL) and you need to judge whether the new latent space
  corrects technical batch effects while preserving or improving biological cell-type
  separation, relative to a baseline latent space.
when_to_use_negative:
- Single-batch datasets with no technical covariates to disentangle — there is no
  batch mixing to benchmark against a baseline.
- Datasets where cell states are not batch-confounded and standard scanpy PCA already
  separates cell types cleanly, so a disentangling model adds no benchmark value.
- Benchmarking against feature tables or precomputed embeddings that cannot be re-clustered;
  the comparison requires raw counts refit through both the model and the baseline
  pipeline.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: CODAL
  role: Variational autoencoder topic model with mutual information regularization
    that disentangles technical from biological effects, producing the corrected latent
    space to be benchmarked
  repo: https://mira-multiome.readthedocs.io
- name: MIRA
  role: Python package whose topic models use CODAL for inference; supplies ILR-transformed
    topic compositions and latent variables for UMAP and k-NN graph construction
  repo: https://github.com/cistrome/MIRA
- name: scanpy
  role: Standard preprocessing workflow (filtering, normalize_total target_sum=10000,
    log1p, HVG selection) and PCA baseline latent space used as the comparison reference
- name: UMAP-learn
  role: Computes UMAP embeddings from both the CODAL latent space (min_dist 0.1, negative_sample_rate
    3) and the PCA baseline for visual comparison
- name: Leiden algorithm
  role: Clustering in the accepted latent space to detect batch-confounded or perturbation-induced
    cell states (e.g., Tal1-/- HE-prog subclusters)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:8fb8df9ae7bf348076829be32d93ba4748bbddb4548ce8ea3b3687e76554f74c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CODAL
  - Leiden algorithm
  - MIRA
  - UMAP-learn
  - scanpy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# latent-space-integration-benchmarking

## Summary

Benchmarking a batch-corrected latent space (e.g., CODAL topic-model latent variables) against a baseline latent space (e.g., scanpy PCA) by visual inspection of UMAPs and comparison of batch mixing versus cell-type discrimination, as demonstrated on the Pijuan-Sala mouse gastrulation atlas with Tal1-/- chimeras.

## When to use

Use when a multi-batch single-cell dataset has been integrated with a disentangling model (e.g., CODAL) and you need to judge whether the new latent space corrects technical batch effects while preserving or improving biological cell-type separation, relative to a baseline latent space.

## When NOT to use

- Single-batch datasets with no technical covariates to disentangle — there is no batch mixing to benchmark against a baseline.
- Datasets where cell states are not batch-confounded and standard scanpy PCA already separates cell types cleanly, so a disentangling model adds no benchmark value.
- Benchmarking against feature tables or precomputed embeddings that cannot be re-clustered; the comparison requires raw counts refit through both the model and the baseline pipeline.

## Inputs

- Concatenated multi-batch single-cell RNA-seq count matrix (AnnData), e.g., mouse embryonic atlas E6.5-E8.5 batches (Pijuan-Sala et al.)
- Per-cell technical covariate annotations (sequencing batch, chimeric status, embryo mixing, collection day)
- Baseline scanpy-preprocessed PCA latent space with HVG selection (minimum dispersion threshold)
- Cell-type annotations for qualitative discrimination assessment

## Outputs

- UMAP embeddings from the ILR-transformed CODAL topic compositions (min_dist=0.1, negative_sample_rate=3)
- UMAP embeddings from the scanpy PCA baseline latent space
- Comparative assessment of batch mixing and cell-type discrimination between the two latent spaces
- Leiden cluster / subcluster assignments computed in the accepted latent space (e.g., HE-prog subclusters A-E)

## How to apply

Fit the integrative topic model (CODAL, extending MIRA) on the concatenated batches, taking the ILR-transformed topic compositions as the corrected latent representation, and compute UMAPs from this space with min_dist 0.1 and negative_sample_rate 3 (k-NN graphs use Manhattan distance between transformed latent variables). In parallel, compute the baseline latent space from the standard scanpy workflow (filter cells <400 counts and genes <30 counts, normalize_total target_sum=10000, log1p, highly variable genes by minimum dispersion threshold) and its PCA-derived UMAP. Side-by-side compare the two UMAPs for (1) intermixing of cells across technical covariates (sequencing batch, chimeric status, embryo mixing, collection day) and (2) separation/discrimination of annotated cell types. Accept the integration when batch intermixing and cell-type discrimination in the CODAL UMAP are comparable to or better than the PCA baseline; downstream uses include Leiden clustering in the latent space to detect batch-confounded subpopulations (e.g., Tal1-/- hemato-endothelial progenitor subclusters A-E).

## Related tools

- **CODAL** (Variational autoencoder topic model with mutual information regularization that disentangles technical from biological effects, producing the corrected latent space to be benchmarked) — https://mira-multiome.readthedocs.io
- **MIRA** (Python package whose topic models use CODAL for inference; supplies ILR-transformed topic compositions and latent variables for UMAP and k-NN graph construction) — https://github.com/cistrome/MIRA
- **scanpy** (Standard preprocessing workflow (filtering, normalize_total target_sum=10000, log1p, HVG selection) and PCA baseline latent space used as the comparison reference)
- **UMAP-learn** (Computes UMAP embeddings from both the CODAL latent space (min_dist 0.1, negative_sample_rate 3) and the PCA baseline for visual comparison)
- **Leiden algorithm** (Clustering in the accepted latent space to detect batch-confounded or perturbation-induced cell states (e.g., Tal1-/- HE-prog subclusters))

## Examples

```
conda create --name mira-env -c conda-forge -c pytorch -c bioconda scanpy jupyter leidenalg; conda activate mira-env; pip3 install mira-multiome
```

## Evaluation signals

- CODAL UMAP shows batch intermixing comparable to or better than the scanpy PCA baseline UMAP.
- Cell-type discrimination in the CODAL UMAP is comparable to or better than the PCA baseline.
- Leiden clustering in the CODAL latent space recovers biologically coherent subclusters whose marker-expression profiles match published annotations (e.g., HE-prog subclusters A-E matching Pijuan-Sala et al. mesodermal annotations).
- Hyperparameter tuning (60 Bayesian iterations over N_topics 35-80) converged to a stable final model (63 topics) whose latent compositions are consistent across the four technical covariates.

## Limitations

- Results showing batch-confounded or perturbation-induced cell states must be validated using orthogonal biological evidence to rule out under-correction of technical effects.
- CODAL does not remove the need for good experimental design; experimentalists must include appropriate controls, e.g., a control group of cells unaffected by the perturbation in both treatment and control batches.
- Further improvements would require better characterization of technical sources of bias in conjunction with the CODAL model.

## Evidence

- [methods] CODAL UMAP validation criterion: [evidence span withheld — anchor status: not_found]
- [methods] ILR-transformed topic compositions and k-NN graph: "We then calculate distances between transformed cell latent variables using the Manhattan distance to create a k-nearest neighbors (k-NN) graph"
- [abstract] CODAL purpose: "uses a mutual information regularization technique to explicitly disentangle factors related to technical and biological effects"
- [abstract] Benchmark on embryonic atlases with knockouts: "We demonstrate CODAL's capacity for batch-confounded cell type discovery when applied to simulated datasets and embryonic development atlases with gene knockouts."
- [discussion] Validation caveat: "the existence of new batch-confounded or perturbation-induced cell states should be validated using orthogonal biological evidence to rule out the under-correction of technical effects"
- [readme] README: CODAL as batch correction in MIRA: [evidence span withheld — anchor status: external_doc]
