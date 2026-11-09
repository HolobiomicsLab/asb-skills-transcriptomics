---
name: metagene-clustering-from-gene-vectors
description: Use when you have a trained GeneVector embedding (genes.vec) from scRNA-seq
  AnnData and need to discover co-regulated gene modules (metagenes) and test whether
  their per-cell module scores change across conditions or treatment timepoints.
when_to_use_negative:
- The goal is distinguishing activation from repression or identifying negative regulation
  — GeneVector's MI target cannot discern the direction of statistical dependency.
- Batches differ strongly in cell type composition and you intend to interpret correction
  vectors as transcriptional signatures without manual verification against cell type
  signatures.
- The input is not single-cell expression data (e.g. bulk RNA-seq or a precomputed
  feature table); the embedding requires per-cell expression sparsity structure in
  AnnData .X.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GeneVector
  role: Provides GeneEmbedding (vector='average'), get_adata(resolution=40) clustering,
    and get_metagenes for metagene discovery from the trained embedding
  repo: https://github.com/nceglia/genevector
- name: Scanpy
  role: Loads the AnnData .h5ad used for module scoring and downstream visualization
- name: PyTorch
  role: Backend for the trained embedding consumed by GeneEmbedding
- name: SciPy
  role: Statistics for per-metagene linear regression of module scores on treatment
    status
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39985-2
    title: Identification of transcriptional programs using dense vector representations
      defined by mutual information with GeneVector
  source_content_hash: sha256:7cb59c60c383cc19affcc872242c99bd3a7f185ab1ff82ccd9050e4a3c031b3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nceglia/genevector
  tools:
  - AnnData
  - GeneVector
  - NumPy
  - PyTorch
  - Python
  - Scanpy
  - SciPy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# metagene-clustering-from-gene-vectors

## Summary

Cluster trained GeneVector gene embeddings into metagenes (unsupervised transcriptional programs) and score them per cell to detect context- or treatment-specific programs, e.g. the TGF-beta metagene 24 in cisplatin-treated TNBC PDXs.

## When to use

Use when you have a trained GeneVector embedding (genes.vec) from scRNA-seq AnnData and need to discover co-regulated gene modules (metagenes) and test whether their per-cell module scores change across conditions or treatment timepoints.

## When NOT to use

- The goal is distinguishing activation from repression or identifying negative regulation — GeneVector's MI target cannot discern the direction of statistical dependency.
- Batches differ strongly in cell type composition and you intend to interpret correction vectors as transcriptional signatures without manual verification against cell type signatures.
- The input is not single-cell expression data (e.g. bulk RNA-seq or a precomputed feature table); the embedding requires per-cell expression sparsity structure in AnnData .X.

## Inputs

- Trained GeneVector embedding file (genes.vec) plus the GeneVectorDataset it was trained on
- Scanpy AnnData (.h5ad) with raw counts in .X and condition/timepoint metadata (e.g. treatment status, timepoints X1–X4)
- Per-cell covariate table mapping cells to treated/untreated status for module-score regression

## Outputs

- GeneEmbedding object with clustered gene adata and metagene gene-set dictionary
- Per-cell metagene module scores
- CSV of per-metagene regression coefficients (beta_treated, beta_untreated), Bonferroni-corrected p-values, and treatment-specificity calls (e.g. tnbc_metagene_coefficients.csv)
- List of treatment-specific metagenes with annotated pathway enrichment (e.g. metagene 24 = TGF-beta signaling: ID1–ID4, EPCAM, FOXP1)

## How to apply

Load the trained gene vectors with GeneEmbedding('genes.vec', dataset, vector='average') — averaging input and output weights is recommended for best results. Cluster the gene vectors via gene_embed.get_adata(resolution=40) and extract metagenes with get_metagenes(gene_adata), yielding functionally related gene sets. Compute metagene module scores per cell, then fit a per-metagene linear regression of module score on treatment status (beta_treated vs beta_untreated) across timepoints (e.g. X1–X4 in the TNBC PDX cisplatin series). Call treatment-specific metagenes using beta_treated > 0.1, Bonferroni-corrected p = 0.001, and beta_treated > beta_untreated; in the source task this recovered metagene 24, enriched for TGF-beta signaling with ID1–ID4, EPCAM, and FOXP1 among its top genes. Write per-metagene coefficients, Bonferroni p-values, and specificity calls to a CSV (e.g. tnbc_metagene_coefficients.csv) for validation.

## Related tools

- **GeneVector** (Provides GeneEmbedding (vector='average'), get_adata(resolution=40) clustering, and get_metagenes for metagene discovery from the trained embedding) — https://github.com/nceglia/genevector
- **Scanpy** (Loads the AnnData .h5ad used for module scoring and downstream visualization)
- **PyTorch** (Backend for the trained embedding consumed by GeneEmbedding)
- **SciPy** (Statistics for per-metagene linear regression of module scores on treatment status)

## Examples

```
gene_embed = GeneEmbedding("genes.vec", dataset, vector="average"); gene_adata = gene_embed.get_adata(resolution=40); metagenes = gene_embed.get_metagenes(gene_adata)
```

## Evaluation signals

- Treatment-specific metagenes meeting all three criteria (beta_treated > 0.1, Bonferroni p = 0.001, beta_treated > beta_untreated) are recovered from the module-score regressions.
- Metagene 24 is identified as TGF-beta-signaling enriched, with ID1, ID2, ID3, ID4, EPCAM, and FOXP1 among its top genes and overexpressed in chemotherapy-resistant samples.
- Per-metagene coefficients, Bonferroni p-values, and specificity calls are present in the output CSV (tnbc_metagene_coefficients.csv) with one row per metagene.
- Clustering gene vectors at resolution=40 yields a gene adata whose clusters map to interpretable, functionally enriched metagenes (spot-check top genes for known pathways).
- The GeneEmbedding was loaded with vector='average' (averaging input and output weights), not a single weight file.

## Limitations

- GeneVector is unable to discern repression from activation, as it builds MI that is agnostic to the direction of the statistical dependency; metagenes cannot distinguish activation from repression.
- Empirical MI estimation is limited by available number of cells, fidelity of counts, and the discretization strategy (default 10 quantile-based bins, zero expression in bin 0).
- MI calculation is computationally expensive (~26 hours legacy for 2,000 genes × 5,000 cells; use numba/rust/gpu backends to reduce to minutes).
- Batch correction via correction vectors is unreliable when batches differ in cell type composition and must be manually verified against cell type signatures.

## Evidence

- [discussion] The resulting vectors can be used to generate a gene co-expression graph, and can be clustered to predict transcriptional programs, or metagenes, in an unsupervised fashion: "The resulting vectors can be used to generate a gene co-expression graph, and can be clustered to predict transcriptional programs, or metagenes, in an unsupervised fashion"
- [readme] gene_adata = gene_embed.get_adata(resolution=40); metagenes = gene_embed.get_metagenes(gene_adata): "gene_adata = gene_embed.get_adata(resolution=40)
metagenes = gene_embed.get_metagenes(gene_adata)"
- [readme] Training produces two vector files (input and output weights). Using the average of both weights is recommended for best results.: [evidence span withheld — anchor status: external_doc]
- [discussion] In cisplatin treated TNBC PDXs, GeneVector uncovers transcriptional signatures active in drug resistance: "In cisplatin treated TNBC PDXs, GeneVector uncovers transcriptional signatures active in drug resistance"
- [discussion] GeneVector is unable to discern repression from activation, as it builds MI that is agnostic to the direction of the statistical dependency: "GeneVector is unable to discern repression from activation, as it builds MI that is agnostic to the direction of the statistical dependency"
