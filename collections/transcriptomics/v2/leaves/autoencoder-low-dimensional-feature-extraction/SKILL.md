---
name: autoencoder-low-dimensional-feature-extraction
description: Use when gene-by-cell scRNA-seq count matrices are noisy or high-dimensional
  (many cells, high dropout) and a compressed cell embedding is needed before clustering,
  visualization, cell classification, or pseudo-time inference.
when_to_use_negative:
- Input is already a low-dimensional embedding or compressed feature table — the first
  module's gene/component filtering based on encoder weight distribution assumes a
  raw gene-by-cell expression matrix.
- No labeled training set exists for the target cells — scDHA.class requires train
  data matrix, train data label and new data matrix, so use the unsupervised path
  (scDHA + result$cluster) instead.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scDHA
  role: 'Primary package: the scDHA main function runs the non-negative kernel autoencoder
    plus stacked Bayesian self-learning network to produce the low-dimensional embedding
    and clusters'
  repo: github.com/duct317/scDHA
- name: scDHA.class
  role: Classifies new data using the embedding, with inputs consisting of train data
    matrix, train data label and new data matrix
  repo: github.com/duct317/scDHA
- name: H2O
  role: Hosts the XGBoost, Random Forest (RF), Deep Learning (DL), and Gradient Boosting
    Machine (GBM) classifiers benchmarked on the same 20 pancreas train-test pairs
- name: mclust
  role: adjustedRandIndex(cluster, label) quantifies agreement of result$cluster with
    ground-truth cell labels
- name: torch
  role: libtorch backend for the autoencoder framework, checked at library(scDHA)
    load time and installed via torch::install_torch()
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:6f1543f0f0cbeeffd2113dba88ea2846c1ba2110923693d8ea5654f3ec73c1ac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Gradient Boosting Machine (GBM)
  - H2O
  - Random Forest (RF)
  - mclust
  - scDHA
  - scDHA.class
  - torch
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# autoencoder-low-dimensional-feature-extraction

## Summary

Extracts compact, denoised low-dimensional representations of scRNA-seq data with scDHA's hierarchical autoencoder: a non-negative kernel autoencoder first removes insignificant genes/components, then a stacked Bayesian self-learning network compresses cells for clustering, visualization, classification, and pseudo-time inference.

## When to use

Use when gene-by-cell scRNA-seq count matrices are noisy or high-dimensional (many cells, high dropout) and a compressed cell embedding is needed before clustering, visualization, cell classification, or pseudo-time inference.

## When NOT to use

- Input is already a low-dimensional embedding or compressed feature table — the first module's gene/component filtering based on encoder weight distribution assumes a raw gene-by-cell expression matrix.
- No labeled training set exists for the target cells — scDHA.class requires train data matrix, train data label and new data matrix, so use the unsupervised path (scDHA + result$cluster) instead.

## Inputs

- Gene-by-cell scRNA-seq expression matrix with rows as samples and columns as genes (e.g., data('Goolam') from the package, or the Baron, Segerstolpe, Muraro, Xin, and Wang pancreas count tables downloaded via their GSE accessions)
- Cell-type labels for a training set when running scDHA.class (train data matrix, train data label, new data matrix)

## Outputs

- Low-dimensional (compressed) cell embedding from the stacked Bayesian autoencoder
- Cluster assignments (result$cluster) for unsupervised cell segregation
- Predicted cell-type labels for new/test cells from scDHA.class
- Visualization coordinates (scDHA.vis) and pseudo-time values (scDHA.pt) derived from the embedding
- Per-pair classification accuracy table (e.g., classification_accuracy_table.csv) when benchmarking the 20 pancreas train-test pairs

## How to apply

Preprocess the matrix (rows as samples/cells, columns as genes) with gene filtering, library-size normalization, and log2(x + 1) transformation, as in the README's Goolam example and the paper's pancreas pipeline. Call scDHA(data, seed = 1): the first module, a non-negative kernel autoencoder, removes genes or components with insignificant contributions based on the encoder weight distribution, yielding a non-negative, part-based representation. The second module, a stacked Bayesian autoencoder (Stacked Bayesian Self-learning Network built on a Variational Autoencoder), projects cells onto the low-dimensional space, repeatedly perturbing the compressed space to learn a more generalized representation — hence set the seed for reproducibility. Consume the embedding by downstream goal: result$cluster for unsupervised cell segregation (scored with mclust::adjustedRandIndex against labels), scDHA.vis for the transcriptome landscape, scDHA.pt for pseudo-time, and scDHA.class (train data matrix, train data label, new data matrix) for cross-dataset classification, e.g., the 20 train-test pairs over the Baron, Segerstolpe, Muraro, Xin, and Wang pancreas datasets where scDHA.class averaged 0.96 accuracy.

## Related tools

- **scDHA** (Primary package: the scDHA main function runs the non-negative kernel autoencoder plus stacked Bayesian self-learning network to produce the low-dimensional embedding and clusters) — github.com/duct317/scDHA
- **scDHA.class** (Classifies new data using the embedding, with inputs consisting of train data matrix, train data label and new data matrix) — github.com/duct317/scDHA
- **H2O** (Hosts the XGBoost, Random Forest (RF), Deep Learning (DL), and Gradient Boosting Machine (GBM) classifiers benchmarked on the same 20 pancreas train-test pairs)
- **mclust** (adjustedRandIndex(cluster, label) quantifies agreement of result$cluster with ground-truth cell labels)
- **torch** (libtorch backend for the autoencoder framework, checked at library(scDHA) load time and installed via torch::install_torch())

## Examples

```
library(scDHA); data('Goolam'); data <- t(Goolam$data); data <- log2(data + 1); result <- scDHA(data, seed = 1); cluster <- result$cluster
```

## Evaluation signals

- mclust::adjustedRandIndex(cluster, label) on result$cluster shows high agreement with known cell labels, consistent with scDHA outperforming state-of-the-art in cell segregation.
- Re-running scDHA with the same seed reproduces identical clusters/embeddings — README: 'The result is reproducible by setting seed for these functions.'
- Cross-dataset benchmark on the 20 pancreas train-test pairs yields scDHA.class average accuracy ≈ 0.96 vs 0.77 (XGB), 0.69 (RF), 0.43 (DL), 0.72 (GBM), significant at p = 2.1 × 10−8 (one-sided Wilcoxon test).
- After the first module the representation is non-negative and part-based, and genes/components with insignificant encoder-weight contributions are removed before projection.
- The embedding is substantially compressed relative to the input gene count, supporting downstream analyses with mostly linear or lower time complexity.

## Limitations

- The current release runs on Torch rather than TensorFlow: library(scDHA) checks for libtorch and may require torch::install_torch(); Python environments are no longer used, so older TensorFlow-based reproductions will not match.
- The discussion states accuracy advantages as expectations ('expected to be more accurate than existing autoencoders') and defers the scVI comparison to Supplementary Section 6 and Fig. 25–32.
- Generality claims to other domains ('from cancer to obesity to aging') are not supported by cited experiments in the discussion.
- Data must be log-transformed (log2(data + 1)) before projection; passing raw counts deviates from the documented workflow and can degrade results.

## Evidence

- [abstract] The second module is a stacked Bayesian autoencoder that projects the data onto a low-dimensional space (compressed).: "The second module is a stacked Bayesian autoencoder that projects the data onto a low-dimensional space (compressed)."
- [abstract] we repeatedly perturb the compressed space to learn a more generalized representation of the data: "we repeatedly perturb the compressed space to learn a more generalized representation of the data"
- [intro] Based on the weight distribution of the encoder, scDHA removes genes or components that have insignificant contributions to the representation: "Based on the weight distribution of the encoder, scDHA removes genes or components that have insignificant contributions to the representation"
- [intro] The first module is a non-negative kernel autoencoder that provides a non-negative, part-based representation of the data.: "The first module is a non-negative kernel autoencoder that provides a non-negative, part-based representation of the data."
- [readme] scDHA.class: classification new data using available one. The inputs consist of train data matrix, train data label and new data matrix.: [evidence span withheld — anchor status: external_doc]
- [readme] The result is reproducible by setting seed for these functions.: [evidence span withheld — anchor status: external_doc]
