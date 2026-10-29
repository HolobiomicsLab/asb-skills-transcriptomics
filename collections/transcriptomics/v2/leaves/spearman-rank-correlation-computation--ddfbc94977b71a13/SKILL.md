---
name: spearman-rank-correlation-computation
description: Use when paired CITE-seq data (ADT protein + RNA counts) is available
  and you need a ground-truth-based readout of whether scRNA-seq denoising improves
  protein-mRNA co-expression, e.g. CD3E detection vs CD3 protein.
when_to_use_negative:
- No paired protein (ADT) measurements exist, so there is no external ground truth
  against which to rank-correlate the RNA signal
- The goal is gene-gene (rather than protein-mRNA) co-expression assessment, where
  rank correlation against an independent protein measurement is not applicable
- Data has already been size-factor normalized, since DCA in this benchmark runs on
  raw counts without normalization and the comparison must use the same counts basis
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DCA (deep count autoencoder network)
  role: Denoises the RNA counts with an NB loss; the denoised matrix is the NB mean
    prediction that feeds the Spearman correlation comparison
  repo: https://github.com/theislab/dca
- name: Scanpy
  role: Python framework in which DCA is exposed as a preprocessing 'dca' method,
    used to handle the count data alongside correlation computation
  repo: https://github.com/theislab/dca
- name: anndata
  role: Reads and stores the h5ad-formatted CITE-seq count matrix paired with ADT
    protein measurements
  repo: https://github.com/theislab/anndata
- name: Keras
  role: Implementation backend of the DCA autoencoder that produces the denoised NB
    mean matrix
- name: TensorFlow
  role: Backend for Keras during DCA training and NB-mean prediction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:03f9c2ff7ad885000c09b03305e09a29b51a1deb41ca7ab2dea9f405e5fd0c56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA (deep count autoencoder network)
  - Keras
  - Scanpy
  - TensorFlow
  - anndata
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# spearman-rank-correlation-computation

## Summary

Compute Spearman rank correlations between per-cell CITE-seq antibody-derived-tag (ADT) protein measurements and matching mRNA levels, before and after DCA denoising, to validate that denoising improves protein-RNA co-expression agreement (Fig. 6 of Eraslan et al. 2019).

## When to use

Use when paired CITE-seq data (ADT protein + RNA counts) is available and you need a ground-truth-based readout of whether scRNA-seq denoising improves protein-mRNA co-expression, e.g. CD3E detection vs CD3 protein.

## When NOT to use

- No paired protein (ADT) measurements exist, so there is no external ground truth against which to rank-correlate the RNA signal
- The goal is gene-gene (rather than protein-mRNA) co-expression assessment, where rank correlation against an independent protein measurement is not applicable
- Data has already been size-factor normalized, since DCA in this benchmark runs on raw counts without normalization and the comparison must use the same counts basis

## Inputs

- CITE-seq paired RNA count matrix and ADT protein measurements (Stoeckius et al. cord blood, GSE100866, 8,005 human cells after filtering)
- List of eight protein-mRNA pairs (CD3, CD4, CD8, CD127, etc.)
- DCA-denoised RNA matrix (NB mean values) and the original raw counts

## Outputs

- protein_mrna_correlations.csv — per-pair Spearman correlations for original vs denoised data across all 8,005 cells
- CD3E mRNA detection fractions (fraction of cells with non-zero counts) for original (~80%) vs denoised (~99.9%) data

## How to apply

Load the Stoeckius et al. cord blood CITE-seq count matrix (GSE100866) with anndata/Scanpy and subset to the 8,005 human cells by removing cells with less than 90% human UMI counts, pairing RNA counts with the ADT protein measurements. Run DCA on the RNA counts using the negative binomial (NB) noise model without size-factor normalization, generating the denoised matrix by replacing original counts with the predicted NB mean. Then compute Spearman rank correlations between protein and mRNA abundances for the eight protein-mRNA pairs (e.g., CD3, CD4, CD8, CD127) across all 8,005 cells, separately for original and DCA-denoised data, and additionally compute the fraction of cells with non-zero CD3E mRNA (expected ~80% original vs ~99.9% denoised). Write per-pair correlations and CD3E detection fractions into a single protein_mrna_correlations.csv results table.

## Related tools

- **DCA (deep count autoencoder network)** (Denoises the RNA counts with an NB loss; the denoised matrix is the NB mean prediction that feeds the Spearman correlation comparison) — https://github.com/theislab/dca
- **Scanpy** (Python framework in which DCA is exposed as a preprocessing 'dca' method, used to handle the count data alongside correlation computation) — https://github.com/theislab/dca
- **anndata** (Reads and stores the h5ad-formatted CITE-seq count matrix paired with ADT protein measurements) — https://github.com/theislab/anndata
- **Keras** (Implementation backend of the DCA autoencoder that produces the denoised NB mean matrix)
- **TensorFlow** (Backend for Keras during DCA training and NB-mean prediction)

## Examples

```
from dca.api import dca; dca(adata, ae_type='nb-conddisp')  # then Spearman-correlate ADT vs mRNA per pair
```

## Evaluation signals

- CD3E mRNA detection fraction rises from ~80% in original counts toward ~99.9% after DCA, consistent with the CD3 protein ground truth
- Median Spearman correlation across all eight protein-mRNA pairs is highest for DCA-denoised data relative to the original data
- Exactly 8,005 human cells retained after the 90% human UMI count filter
- Denoised values correspond to the negative binomial mean output of DCA (not imputed integer counts or ZINB output)
- Results table protein_mrna_correlations.csv contains per-pair correlation values for both original and denoised conditions

## Limitations

- Evaluation of denoising is difficult because the definition of a ground truth can be challenging for real data; here only eight protein-mRNA pairs provide ground truth
- In general, it may be difficult to determine when denoising improves scRNA-seq data
- Overimputation may occur with inadequate hyperparameter choices such as a too low-dimensional bottleneck layer; DCA hyperparameters were not fine-tuned for the manuscript analyses

## Evidence

- [discussion] Denoising by replacing original counts with the predicted negative binomial mean: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [discussion] Human-cell filter for CITE-seq dataset: "data were subset to 8,005 human cells by removing cells with less than 90% human UMI counts."
- [discussion] DCA implementation stack: "DCA is implemented in Python 3 using Keras53 and its TensorFlow54 backend."
- [readme] DCA purpose and loss model: "A deep count autoencoder network to denoise scRNA-seq data and remove the dropout effect"
- [intro] Scanpy integration: [evidence span withheld — anchor status: not_found]
