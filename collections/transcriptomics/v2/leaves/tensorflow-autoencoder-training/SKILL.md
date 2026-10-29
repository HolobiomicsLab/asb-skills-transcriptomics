---
name: tensorflow-autoencoder-training
description: Use when clustering scRNA-seq count matrices (preprocessed with scanpy/anndata,
  HVG-selected) where dropout events are high and cell-cell topology should be preserved,
  or when reproducing scMGCA's multinomial decoder training.
when_to_use_negative:
- If the goal is denoising with a zero-inflated negative binomial model (e.g., DCA-style
  ZINB loss) rather than the multinomial-with-dropout formulation scMGCA uses
- If the input is already a reduced feature/PCA table rather than a raw count matrix,
  since the pipeline expects a normalized expression matrix with HVG selection and
  cell-graph construction
- If cell-cell topological structure is irrelevant to the analysis, e.g., simple marker-gene-based
  annotation on already-clustered data
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scMGCA
  role: The graph-embedding autoencoder implementation whose GCN encoder, multinomial
    decoder and KL clustering are trained
  repo: https://pypi.org/project/scMGCA
- name: TensorFlow
  role: Deep learning framework on which the scMGCA core model (GCN encoder + multinomial
    decoder) is built and trained
- name: SCANPY
  role: Loading and preprocessing the scRNA-seq count matrix (normalization, HVG selection)
    before autoencoder training
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:d736d25b93d13472060567fd01065feb7b36b21dc5b86d1b57b3ee8f0f1d8bed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCANPY
  - TensorFlow
  - scMGCA
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# tensorflow-autoencoder-training

## Summary

Train the scMGCA graph convolutional autoencoder in TensorFlow: two GCN encoder layers learn cell-cell topology from a cell-PPMI graph, a multinomial-based decoder with Bernoulli dropout simulation reconstructs the scRNA-seq count distribution, and KL-divergence self-optimization refines cluster assignments.

## When to use

Use when clustering scRNA-seq count matrices (preprocessed with scanpy/anndata, HVG-selected) where dropout events are high and cell-cell topology should be preserved, or when reproducing scMGCA's multinomial decoder training.

## When NOT to use

- If the goal is denoising with a zero-inflated negative binomial model (e.g., DCA-style ZINB loss) rather than the multinomial-with-dropout formulation scMGCA uses
- If the input is already a reduced feature/PCA table rather than a raw count matrix, since the pipeline expects a normalized expression matrix with HVG selection and cell-graph construction
- If cell-cell topological structure is irrelevant to the analysis, e.g., simple marker-gene-based annotation on already-clustered data

## Inputs

- Preprocessed scRNA-seq count matrix (scanpy/anndata) with highly variable genes selected
- Cell-PPMI matrix from random surfing on the KNN cell graph
- Latent representation Z from the GCN encoder
- Cluster number / initialization strategy (K-means or spectral clustering)

## Outputs

- Trained graph convolutional autoencoder (e.g., saved as multinomial_decoder.py)
- Latent embedding with self-optimized cluster assignments
- Per-epoch loss log (e.g., training_loss_log.csv)

## How to apply

Normalize the scRNA-seq gene expression matrix and select highly variable genes, then build the cell graph with a random surfing algorithm on a KNN graph to compute the cell-PPMI matrix. Train an autoencoder with two GCN encoder layers, a multinomial-based decoder (softmax over genes) and an inner product decoder; the multinomial decoder models the scRNA-seq distribution and simulates dropout via Bernoulli dropout variables rather than a zero-inflated distribution. Jointly optimize the autoencoder losses with KL divergence on the latent representations to self-optimize clustering, initializing cluster centers with the best of K-means and spectral clustering; loss weights gamma1, gamma2, gamma3 balance the total objective. Train until the stopping condition (maximum number of iterations) is reached; for datasets with >25,000 cells, split into multiple batches. The core model is TensorFlow-based and installable from PyPI as scMGCA.

## Related tools

- **scMGCA** (The graph-embedding autoencoder implementation whose GCN encoder, multinomial decoder and KL clustering are trained) — https://pypi.org/project/scMGCA
- **TensorFlow** (Deep learning framework on which the scMGCA core model (GCN encoder + multinomial decoder) is built and trained)
- **SCANPY** (Loading and preprocessing the scRNA-seq count matrix (normalization, HVG selection) before autoencoder training)

## Examples

```
pip install scMGCA
```

## Evaluation signals

- Training loss (multinomial negative log-likelihood plus dropout-compensation terms) decreases monotonically per epoch and is logged to training_loss_log.csv
- Cluster assignments converge under KL self-optimization without large label flips in late epochs
- Latent embedding quality holds up on benchmarks: scMGCA achieves highest ASW on 'QS Trachea' and 'QS Heart' datasets and outperforms state-of-the-art methods on rare-cell-type datasets ('QS heart', 'Muraro', 'Qx limb Muscle', 'Adam')
- Decoder output M is a valid probability distribution over genes (softmax row sums = 1) and dropout-corrected probabilities explain zero-count entries
- Reproduction against the provided Source Data file matches published clustering metrics

## Limitations

- Loss weights gamma1, gamma2, gamma3 are not numerically specified in the article, so training balance requires tuning
- Maximum iteration count for the stopping condition, KNN neighbor count k, and restart probability (1 - alpha) are not numerically specified
- Number of highly variable genes d is left symbolic; datasets with >25,000 cells must be split into multiple batches for learning
- No software version numbers (Python, TensorFlow, scanpy) are reported, which may complicate reproduction

## Evidence

- [intro] we propose a graph convolutional autoencoder framework: "we propose a graph convolutional autoencoder framework, called scMGCA, to analyze scRNA-seq datasets"
- [results] scMGCA uses two layers of GCN to effectively extract the gene expression information and cell-cell topological structure: "scMGCA uses two layers of GCN to effectively extract the gene expression information and cell-cell topological structure"
- [results] KL divergence self-optimization of clustering: "we perform the Kullback-Leibler (KL) divergence on the latent representations to self-optimize the clustering task"
- [discussion] TensorFlow implementation availability: "the core model is built on the Tensorflow framework that is publicly available at https://pypi.org/project/scMGCA"
- [readme] scMGCA Python package purpose: "scMGCA is a Python package containing tools for clustering single-cell data based on a graph-embedding autoencoder that simultaneously learns cell–cell topology representation and"
