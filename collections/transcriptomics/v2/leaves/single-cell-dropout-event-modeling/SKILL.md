---
name: single-cell-dropout-event-modeling
description: Use when reconstructing the decoder of the scMGCA graph convolutional
  autoencoder, or when a count-matrix generative loss must account for high-probability
  dropout events without assuming a zero-inflated negative binomial model.
when_to_use_negative:
- When the goal is explicit zero-inflated negative binomial modeling or zero-expansion
  imputation — the scMGCA decoder deliberately uses a multinomial distribution rather
  than a zero-inflated distribution (contrast DCA/scDeepCluster, which use ZINB losses)
- When the input is already log-normalized or non-count expression data — the multinomial
  likelihood requires a count matrix over the retained highly variable genes
- When genes expressed in zero cells have not been filtered out — the article states
  this filtering is the first step to reduce the impact of high dropout rates
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scMGCA
  role: Host framework whose graph convolutional autoencoder contains the multinomial-based
    decoder with Bernoulli dropout simulation
  repo: https://pypi.org/project/scMGCA
- name: TensorFlow
  role: Framework in which the core model (decoder, softmax activation, multinomial
    NLL loss) is built
- name: SCANPY
  role: Loads and preprocesses the count matrix (normalization, highly variable gene
    selection) that feeds the decoder
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:7d22139fbd2ad30e10a6ad3299f7eae3beef99c7bb0e87d045471114253ff09a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCANPY
  - TensorFlow
  - scMGCA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-dropout-event-modeling

## Summary

The scMGCA multinomial-based decoder models the scRNA-seq count matrix as following a multinomial distribution (rather than a zero-inflated distribution) and simulates dropout events with Bernoulli dropout variables on zero-count entries, so the autoencoder loss compensates for high dropout rates during clustering-oriented denoising.

## When to use

Use when reconstructing the decoder of the scMGCA graph convolutional autoencoder, or when a count-matrix generative loss must account for high-probability dropout events without assuming a zero-inflated negative binomial model.

## When NOT to use

- When the goal is explicit zero-inflated negative binomial modeling or zero-expansion imputation — the scMGCA decoder deliberately uses a multinomial distribution rather than a zero-inflated distribution (contrast DCA/scDeepCluster, which use ZINB losses)
- When the input is already log-normalized or non-count expression data — the multinomial likelihood requires a count matrix over the retained highly variable genes
- When genes expressed in zero cells have not been filtered out — the article states this filtering is the first step to reduce the impact of high dropout rates

## Inputs

- Preprocessed scRNA-seq count matrix (genes filtered as not expressed removed; top d highly variable genes retained), e.g. loaded via scanpy/anndata
- Latent representation Z produced by the scMGCA graph convolutional encoder (two GCN layers)
- Loss weight coefficients γ1, γ2, γ3 for balancing the total objective function

## Outputs

- Multinomial parameter matrix M over genes (softmax output of the 128→256→512 decoder)
- Estimated Bernoulli dropout probabilities for zero-count gene entries
- Reconstructed/denoised count estimates used by the multinomial NLL loss (Eqs. 14–15)
- Per-epoch loss values written to training_loss_log.csv; decoder module saved as multinomial_decoder.py

## How to apply

First filter out genes that are not expressed in cells to reduce the impact of the high dropout rates in scRNA-seq data, and keep the top d highly variable genes. In TensorFlow, build the three-layer fully connected decoder (128→256→512) with a softmax final activation that maps the latent representation Z from the two-layer GCN encoder to the multinomial parameter matrix M over genes. Implement the Bernoulli dropout-variable formulation of Eqs. (11)–(13): estimate dropout probabilities for zero-count entries and combine them with the softmax probabilities so the full probability of each gene entry accounts for dropout. Train the decoder with the multinomial negative log-likelihood loss of Eqs. (14)–(15) with dropout compensation, balancing it against the other losses via the weight coefficients γ1, γ2 and γ3, until the maximum number of iterations is achieved; log per-epoch loss values to training_loss_log.csv. Note that scMGCA divides data into multiple batches for learning when clustering datasets with >25,000 cells, so the decoder should be trained per batch on such inputs.

## Related tools

- **scMGCA** (Host framework whose graph convolutional autoencoder contains the multinomial-based decoder with Bernoulli dropout simulation) — https://pypi.org/project/scMGCA
- **TensorFlow** (Framework in which the core model (decoder, softmax activation, multinomial NLL loss) is built)
- **SCANPY** (Loads and preprocesses the count matrix (normalization, highly variable gene selection) that feeds the decoder)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('scMGCA_input.h5ad'); sc.pp.normalize_total(adata); sc.pp.highly_variable_genes(adata)
```

## Evaluation signals

- Each row of the decoder output M sums to 1, confirming the softmax final activation yields valid multinomial parameters over genes
- Per-epoch loss values in training_loss_log.csv decrease monotonically toward the maximum-iterations stopping condition
- Zero-count entries in the input matrix receive estimated dropout probabilities that are combined with the softmax probabilities per Eqs. (11)–(13)
- Reconstructed entries are nonnegative and consistent with multinomial sampling over the filtered count matrix
- The total objective respects the balance controlled by γ1, γ2 and γ3 across the autoencoder and clustering losses

## Limitations

- Numeric values for the loss weights γ1, γ2, γ3 are not provided in the article
- The maximum iteration count for the stopping condition is not specified
- The number of highly variable genes d is not given a numeric value, so the decoder input dimensionality must be chosen by the practitioner
- Datasets with >25,000 cells must be divided into multiple batches for learning, complicating decoder training on very large matrices

## Evidence

- [discussion] The first step is to filter out genes that are not expressed in cells, reducing the impact of the high dropout rates in scRNA-seq data.: "The first step is to filter out genes that are not expressed in cells, reducing the impact of the high dropout rates in scRNA-seq data."
- [discussion] scMGCA is implemented in Python, and the core model is built on the Tensorflow framework that is publicly available at https://pypi.org/project/scMGCA: "scMGCA is implemented in Python, and the core model is built on the Tensorflow framework that is publicly available at https://pypi.org/project/scMGCA"
- [results] scMGCA divides the data into multiple batches for learning when clustering datasets with >25,000 cells: "scMGCA divides the data into multiple batches for learning when clustering datasets with >25,000 cells"
- [readme] scMGCA is a Python package containing tools for clustering single-cell data based on a graph-embedding autoencoder that simultaneously learns cell–cell topology representation and: "scMGCA is a Python package containing tools for clustering single-cell data based on a graph-embedding autoencoder that simultaneously learns cell–cell topology representation and"
- [discussion] γ1, γ2 and γ3 are weight coefficients assigned to each loss to control the balance of the total objective function: "γ1, γ2 and γ3 are weight coefficients assigned to each loss to control the balance of the total objective function"
