---
name: cite-seq-protein-rna-integration
description: Use when CITE-seq data pairs surface-protein abundance (e.g. CD16/CD56
  ADTs) with RNA counts for the same genes and the goal is to test whether RNA denoising
  recovers protein-defined sub-population structure.
when_to_use_negative:
- Data lacks paired surface-protein (ADT) measurements, so no protein-derived ground-truth
  sub-population labels can be defined
- Goal is de novo clustering of cell types rather than validating denoising against
  existing marker-based labels
- Dataset has not been filtered (cells/genes with zero expression retained, no human-UMI
  fraction filter), which violates DCA's input requirements
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DCA (deep count autoencoder)
  role: Denoises the raw RNA counts by replacing original count values with the mean
    of the negative binomial component, producing the denoised NCAM1/FCGR3A matrix
    used for silhouette comparison
  repo: https://github.com/theislab/dca
- name: Scanpy
  role: Python API entry point for DCA via the 'dca' method in the preprocessing package;
    handles loading and filtering of the anndata/h5ad CITE-seq object
  repo: https://github.com/theislab/dca
- name: mclust
  role: R package used to fit the two-component Gaussian mixture model on CD16/CD56
    protein levels to derive the reference sub-population labels
- name: anndata
  role: h5ad-formatted HDF5 container for the count matrix and ADT layer used as DCA
    input
  repo: https://github.com/theislab/anndata
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:12a838ff894e91a313d8aaffd8352214e5dbc69dfe713896aa4e965d6a462f31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA (deep count autoencoder)
  - Scanpy
  - anndata
  - mclust
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cite-seq-protein-rna-integration

## Summary

A CITE-seq evaluation skill that compares how well CD16/CD56 antibody-derived tag protein levels versus original and DCA-denoised NCAM1/FCGR3A RNA expression recover the same CD56bright/CD56dim sub-population labels, quantified by average silhouette widths.

## When to use

Use when CITE-seq data pairs surface-protein abundance (e.g. CD16/CD56 ADTs) with RNA counts for the same genes and the goal is to test whether RNA denoising recovers protein-defined sub-population structure.

## When NOT to use

- Data lacks paired surface-protein (ADT) measurements, so no protein-derived ground-truth sub-population labels can be defined
- Goal is de novo clustering of cell types rather than validating denoising against existing marker-based labels
- Dataset has not been filtered (cells/genes with zero expression retained, no human-UMI fraction filter), which violates DCA's input requirements

## Inputs

- Stoeckius et al. CITE-seq cord blood mononuclear cell dataset (GSE100866) with RNA count matrix and antibody-derived tag (ADT) protein measurements
- Cell annotation identifying the NK cell cluster (906 cells of 8,005 human cells)
- DCA-denoised NCAM1/FCGR3A RNA expression matrix (NB means from the deep count autoencoder)

## Outputs

- Protein-derived CD56bright/CD56dim cluster labels from the two-component mclust Gaussian mixture model
- Three average silhouette widths (protein space, original RNA, DCA-denoised RNA)
- Comparison table written to silhouette_scores.csv

## How to apply

Load the Stoeckius et al. CITE-seq cord blood mononuclear cell dataset (GSE100866), remove cells with less than 90% human UMI counts to keep 8,005 human cells, and subset the annotated NK cell population (906 cells). Fit a two-component Gaussian mixture model with R mclust on the CD16 (FCGR3A) and CD56 (NCAM1) antibody-derived tag protein levels to obtain protein-derived cluster labels. Run DCA on the raw RNA counts — denoising replaces original counts with the predicted negative binomial mean — then compute average silhouette widths of the protein labels in three spaces: protein (expected ~0.47), original NCAM1/FCGR3A RNA (expected ~0.17), and DCA-denoised RNA (expected ~0.58), e.g. with cluster::silhouette. Validation is the reproduced ordering silhouette_denoised > silhouette_protein > silhouette_raw.

## Related tools

- **DCA (deep count autoencoder)** (Denoises the raw RNA counts by replacing original count values with the mean of the negative binomial component, producing the denoised NCAM1/FCGR3A matrix used for silhouette comparison) — https://github.com/theislab/dca
- **Scanpy** (Python API entry point for DCA via the 'dca' method in the preprocessing package; handles loading and filtering of the anndata/h5ad CITE-seq object) — https://github.com/theislab/dca
- **mclust** (R package used to fit the two-component Gaussian mixture model on CD16/CD56 protein levels to derive the reference sub-population labels)
- **anndata** (h5ad-formatted HDF5 container for the count matrix and ADT layer used as DCA input) — https://github.com/theislab/anndata

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('cord_blood_citeseq.h5ad'); sc.pp.dca(adata); # then silhouette on adata.X vs adata.obsm['protein']
```

## Evaluation signals

- Average silhouette width of protein labels in DCA-denoised NCAM1/FCGR3A space is ~0.58, higher than both protein space (~0.47) and original RNA (~0.17)
- mclust mixture model on CD16/CD56 ADTs converges to exactly two components over the 906-cell NK population
- Denoised matrix equals the negative binomial mean values, not raw counts or a generic smoothed matrix
- Expected ordering silhouette_denoised (0.58) > silhouette_protein (0.47) > silhouette_raw (0.17) reproduced and written to silhouette_scores.csv
- Cell count after filtering equals 8,005 human cells (>=90% human UMI counts) with the NK subset of 906 cells

## Limitations

- It may be difficult to determine when denoising improves scRNA-seq data; the definition of a ground truth can be challenging for real data
- Overimputation can occur with inadequate hyperparameter choices such as a too low-dimensional bottleneck layer
- Hyperparameters were not fine-tuned for any of the analyses described in the manuscript, so results may be sensitive to default settings

## Evidence

- [discussion] data were subset to 8,005 human cells by removing cells with less than 90% human UMI counts.: "data were subset to 8,005 human cells by removing cells with less than 90% human UMI counts."
- [discussion] Denoising by NB mean replacement: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [intro] Scanpy DCA integration: [evidence span withheld — anchor status: not_found]
- [readme] README method description: "A deep count autoencoder network to denoise scRNA-seq data and remove the dropout effect"
