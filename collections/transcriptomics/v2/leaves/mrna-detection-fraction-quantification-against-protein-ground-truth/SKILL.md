---
name: mrna-detection-fraction-quantification-against-protein-ground-truth
description: Use when validating scRNA-seq denoising/imputation against an orthogonal
  per-cell protein measurement (CITE-seq ADT), by comparing mRNA detection rates against
  protein-positivity as ground truth.
when_to_use_negative:
- No orthogonal per-cell protein measurements (e.g., CITE-seq ADT tags) are available
  to serve as ground truth for the mRNA
- The goal is bulk RNA-seq or full-length scRNA-seq where dropout-driven false zeros
  are not the dominant detection limitation
- The dataset lacks a defined positive cell population (e.g., T cells for CD3E), making
  the protein-based ground-truth comparison uninterpretable
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: DCA
  role: Denoises the RNA counts using an NB (or ZINB) loss autoencoder; denoised matrix
    is the NB mean prediction used for detection-fraction comparison
  repo: https://github.com/theislab/dca
- name: Scanpy
  role: Python framework in which the 'dca' preprocessing method is directly available
    for denoising within the analysis workflow
  repo: https://github.com/theislab/dca
- name: anndata
  role: Loads and stores the h5ad-formatted CITE-seq count matrix with paired RNA
    and ADT protein layers
  repo: https://github.com/theislab/anndata
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:a1108477d63a066215229ba1fa0837399591d58480114cfe05559addd52be50e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - Keras
  - Scanpy
  - TensorFlow
  - anndata
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# mRNA detection fraction quantification against protein ground truth

## Summary

Quantifies the fraction of cells in which an mRNA (e.g., CD3E) is detected (non-zero counts) before and after DCA denoising, using CITE-seq antibody-derived-tag (ADT) protein measurements (e.g., CD3) as ground truth to validate that denoising recovers biologically true expression.

## When to use

Use when validating scRNA-seq denoising/imputation against an orthogonal per-cell protein measurement (CITE-seq ADT), by comparing mRNA detection rates against protein-positivity as ground truth.

## When NOT to use

- No orthogonal per-cell protein measurements (e.g., CITE-seq ADT tags) are available to serve as ground truth for the mRNA
- The goal is bulk RNA-seq or full-length scRNA-seq where dropout-driven false zeros are not the dominant detection limitation
- The dataset lacks a defined positive cell population (e.g., T cells for CD3E), making the protein-based ground-truth comparison uninterpretable

## Inputs

- Stoeckius et al. CITE-seq cord blood count matrix (GSE100866) with RNA counts and ADT protein measurements
- Cell annotation of human cells (subset to 8,005 cells after removing cells with <90% human UMI counts)
- DCA-denoised RNA count matrix (NB mean predictions)

## Outputs

- CD3E mRNA detection fraction (percent of cells with non-zero counts) for original and denoised data (expected ~80% vs ~99.9%)
- Per-pair Spearman rank correlations between protein and mRNA abundances for the eight protein-mRNA pairs
- Results table saved as protein_mrna_correlations.csv

## How to apply

Load the Stoeckius et al. CITE-seq cord blood count matrix (GSE100866) with anndata/Scanpy and subset to the 8,005 human cells by removing cells with less than 90% human UMI counts, pairing RNA counts with the ADT protein measurements. Run DCA on the RNA counts with the negative binomial (NB) noise model without size-factor normalization, generating the denoised matrix by replacing original counts with the predicted NB mean. For a protein-RNA pair with known co-expression (e.g., CD3 protein and CD3E mRNA in T cells), compute the fraction of cells with non-zero mRNA counts in the original and DCA-denoised matrices. Acceptance is that denoising raises the detection fraction from ~80% (original) toward ~99.9%, consistent with the CD3 protein ground truth. Optionally accompany this with per-pair Spearman correlations between protein and mRNA abundances across all 8,005 cells for the eight available protein-mRNA pairs (e.g., CD3, CD4, CD8, CD127).

## Related tools

- **DCA** (Denoises the RNA counts using an NB (or ZINB) loss autoencoder; denoised matrix is the NB mean prediction used for detection-fraction comparison) — https://github.com/theislab/dca
- **Scanpy** (Python framework in which the 'dca' preprocessing method is directly available for denoising within the analysis workflow) — https://github.com/theislab/dca
- **anndata** (Loads and stores the h5ad-formatted CITE-seq count matrix with paired RNA and ADT protein layers) — https://github.com/theislab/anndata

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('cite_cord_blood.h5ad'); sc.pp.dca(adata, mode='denoise'); frac = (adata[:, 'CD3E'].X > 0).mean()
```

## Evaluation signals

- CD3E detection fraction after DCA is ~99.9% in cells versus ~80% in original data, matching the CD3 protein ground truth
- Median Spearman correlation between protein and mRNA across all eight protein-mRNA pairs (CD3, CD4, CD8, CD127, etc.) is higher after DCA than on original counts
- Cell subset equals exactly 8,005 human cells after removing cells with less than 90% human UMI counts
- protein_mrna_correlations.csv contains both original and denoised values for every protein-mRNA pair
- Denoised values are the predicted NB means (DCA run without size-factor normalization), not scaled raw counts

## Limitations

- Evaluation of denoising is difficult because defining ground truth for real data is challenging; protein measurements only cover the eight available protein-mRNA pairs
- Overimputation may occur with inadequate hyperparameter choices such as too low-dimensional a bottleneck layer, artificially inflating detection fractions
- Hyperparameters were not fine-tuned for any of the analyses described in the manuscript; general overimputation/overfitting is a not-yet-extensively-treated issue of imputation methods

## Evidence

- [results] CD3E detection in 99.9% of T cells after DCA vs 80% originally: [evidence span withheld — anchor status: not_found]
- [discussion] Denoising by NB mean replacement: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [discussion] Human cell subsetting threshold: "data were subset to 8,005 human cells by removing cells with less than 90% human UMI counts."
- [intro] Scanpy integration of DCA: [evidence span withheld — anchor status: not_found]
- [discussion] Overimputation failure mode: "this may also lead to overimputation in case of inadequate hyperparameter choices such as too low-dimensional bottleneck layer and hence data manifold"
