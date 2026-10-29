---
name: single-cell-batch-effect-correction
description: Use when scRNA-seq data from multiple scRNA-seq protocols or batches
  must be integrated for clustering, and batch mixing must be quantified (e.g., ASW)
  against baselines such as DESC, Harmony, MNN, scVI or Scanorama.
when_to_use_negative:
- The data come from a single protocol/batch so no batch merging is needed — plain
  clustering (e.g., scMGCA on a single dataset) suffices without the batch-correction
  comparison
- The input is raw counts from a single dataset where the goal is denoising rather
  than batch integration (DCA/ZINB autoencoders are the appropriate choice)
- Cell-type annotation is the goal on an already-integrated dataset (e.g., the PDAC
  workflow with marker-gene annotation), not batch-effect removal
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMGCA
  role: Core graph convolutional autoencoder that builds the cell-PPMI graph, learns
    the batch-merged embedding, and performs KL-divergence self-optimized clustering;
    installed from PyPI with Python scientific-stack dependencies
  repo: https://pypi.org/project/scMGCA
- name: SCANPY
  role: 'Preprocessing: normalization of the gene expression matrix and selection
    of highly variable genes before graph construction'
- name: DESC
  role: Baseline deep-embedding integration method whose embedding is compared against
    scMGCA's batch-mixing score (ASW)
  repo: https://github.com/eleozzr/desc
- name: TensorFlow
  role: Framework on which the scMGCA core model (two-layer GCN autoencoder) is built
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:29e6e122ecf189641c46ad7ebba28c405144d43afd97cf0cb8c1e4d691e3e2e2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESC
  - SCANPY
  - TensorFlow
  - scMGCA
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# single-cell-batch-effect-correction

## Summary

Removes technical batch effects from multi-protocol single-cell RNA-seq data (e.g., human pancreas profiled with CEL-seq, CEL-seq2, Fluidigm C1 and Smart-seq2) by learning a batch-merged latent embedding with scMGCA, a graph convolutional autoencoder that uses a cell-PPMI graph and KL-divergence self-optimized clustering, while retaining cell-type separation.

## When to use

Use when scRNA-seq data from multiple scRNA-seq protocols or batches must be integrated for clustering, and batch mixing must be quantified (e.g., ASW) against baselines such as DESC, Harmony, MNN, scVI or Scanorama.

## When NOT to use

- The data come from a single protocol/batch so no batch merging is needed — plain clustering (e.g., scMGCA on a single dataset) suffices without the batch-correction comparison
- The input is raw counts from a single dataset where the goal is denoising rather than batch integration (DCA/ZINB autoencoders are the appropriate choice)
- Cell-type annotation is the goal on an already-integrated dataset (e.g., the PDAC workflow with marker-gene annotation), not batch-effect removal

## Inputs

- Combined multi-protocol human pancreas scRNA-seq dataset (CEL-seq, CEL-seq2, Fluidigm C1, Smart-seq2) as an AnnData object
- Baseline integration embeddings (e.g., DESC) for comparison
- requirements.txt dependency list (numpy, scipy, tensorflow, scikit-learn, pandas, scanpy, anndata)

## Outputs

- Batch-merged cell-type cluster assignments per cell
- batch_correction_results.csv with cluster labels and batch-effect correction comparison metrics (e.g., ASW)
- scMGCA latent embedding used for silhouette-based batch-mixing evaluation

## How to apply

Load the combined multi-protocol dataset as an AnnData object, normalize the gene expression matrix and select highly variable genes with scanpy preprocessing. Build a cell graph with the random surfing algorithm on a KNN graph to compute the cell-PPMI matrix, then train the two-layer graph convolution autoencoder with a multinomial-based decoder on batched data (datasets with >25,000 cells are split into multiple batches). Initialize cluster centers with the best strategy from K-means and spectral clustering, then perform self-optimized clustering via Kullback-Leibler divergence on the latent representations to obtain batch-merged cell-type cluster assignments. Evaluate batch merging with silhouette-based metrics (ASW) on the embeddings and compare against baseline integration embeddings (DESC, Harmony, MNN, scVI, Scanorama), accepting the result when scMGCA's batch-mixing score matches or exceeds the baselines reported in the paper's Source Data while cell-type separation is retained.

## Related tools

- **scMGCA** (Core graph convolutional autoencoder that builds the cell-PPMI graph, learns the batch-merged embedding, and performs KL-divergence self-optimized clustering; installed from PyPI with Python scientific-stack dependencies) — https://pypi.org/project/scMGCA
- **SCANPY** (Preprocessing: normalization of the gene expression matrix and selection of highly variable genes before graph construction)
- **DESC** (Baseline deep-embedding integration method whose embedding is compared against scMGCA's batch-mixing score (ASW)) — https://github.com/eleozzr/desc
- **TensorFlow** (Framework on which the scMGCA core model (two-layer GCN autoencoder) is built)

## Examples

```
conda create -n scmgca python=3.6.8; pip install -r requirements.txt scMGCA; python train_scmgca.py --input pancreas_multisource.h5ad --batch-key protocol --output batch_correction_results.csv
```

## Evaluation signals

- ASW (silhouette-based batch-mixing score) computed on scMGCA embeddings matches or exceeds the baseline integration methods reported in the paper's Source Data
- Cell-type separation is retained in the UMAP/latent embedding despite batch merging (biological signal preserved)
- Output file batch_correction_results.csv is produced with cluster labels and comparison metrics against DESC embeddings
- scMGCA outperforms DESC, Harmony, MNN, scVI and Scanorama on the combined pancreas multi-protocol data
- AnnData object carries both the batch key (protocol: CEL-seq/CEL-seq2/Fluidigm C1/Smart-seq2) and the inferred cluster labels after correction

## Limitations

- No software version numbers are reported in the article (Python, TensorFlow, SCANPY unstated); the task workflow specifies Python 3.6.8 and dependencies from requirements.txt, so exact reproduction may require pinning
- Key hyperparameters (KNN neighbor count k, restart probability α, loss weights γ1/γ2/γ3, maximum iteration count, number of highly variable genes d) are not given numeric values in the article
- Datasets with >25,000 cells must be divided into multiple batches for learning, adding a decision point; datasets are described only as coming from public repositories without specific accessions
- The article states 'To reproduce the results, please find the Source Data file we provided.' — the Source Data file is referenced without a DOI or URL in that section

## Evidence

- [other] scMGCA can effectively merge the combined human pancreas datasets generated by different scRNA-seq protocols and remove the batch effect, outperforming DESC, Harmony, MNN, scVI and Scanorama.: [evidence span withheld — anchor status: not_found]
- [results] Workflow normalization and HVG selection: "We first normalized the scRNA-seq gene expression matrix and selected the highly variable genes"
- [results] Cell graph construction via random surfing: "we used a random surfing algorithm based on KNN graph to aggregate neighboring cells"
- [results] KL-divergence self-optimized clustering: "we perform the Kullback-Leibler (KL) divergence on the latent representations to self-optimize the clustering task"
- [results] Batching rule for large datasets: "scMGCA divides the data into multiple batches for learning when clustering datasets with >25,000 cells"
- [intro] scMGCA core framework description: "we propose a graph convolutional autoencoder framework, called scMGCA, to analyze scRNA-seq datasets"
- [discussion] TensorFlow and PyPI availability: "the core model is built on the Tensorflow framework that is publicly available at https://pypi.org/project/scMGCA"
- [discussion] Reproduction pointer to Source Data: "To reproduce the results, please find the Source Data file we provided."
