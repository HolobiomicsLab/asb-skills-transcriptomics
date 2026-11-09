---
name: decoder-weight-variance-gene-ranking
description: Use when a trained scMGCA (GCN autoencoder) latent embedding is available
  and the goal is functional interpretation — identifying the pivotal genes the decoder
  weights encode so they can be tested for pathway/GO enrichment.
when_to_use_negative:
- You want marker genes per cell cluster (p_val_adj <= 0.05, avg_logFC >= 0.5) rather
  than pivotal genes extracted from autoencoder weight matrices
- The model is not a GCN-based graph autoencoder (e.g. DCA or scVI without decoder
  weight matrices) — no W1/W2 weight structure exists to rank
- You need differential expression across conditions instead of functional interpretation
  of the compressed transcriptomic space
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scMGCA
  role: Trained graph-convolutional autoencoder whose W1/W2 weight matrices supply
    the standard deviation and weights used to rank and select the top 200 genes
  repo: https://pypi.org/project/scMGCA
- name: TensorFlow
  role: Framework in which the scMGCA core model is built, producing the weight matrices
    analyzed
- name: SCANPY
  role: Preprocesses the input gene expression matrix (normalize, filter genes not
    expressed, select highly variable genes) before training
- name: ClusterProfiler
  role: Runs KEGG and GO over-representation analysis on the extracted top 200 pivotal
    genes (pvalueCutoff=0.01)
- name: scikit-learn
  role: Provides K-means (vs spectral clustering) initialization of clustering centers
    used during scMGCA training
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:cf46496aa7795bd4722a28c1e123f623e7f32b51ab6069ada01df83f0772fb3d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClusterProfiler
  - SCANPY
  - TensorFlow
  - scMGCA
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Decoder-weight-variance gene ranking from a graph autoencoder

## Summary

Ranks genes by the standard deviation and weight of a trained scMGCA graph-convolutional autoencoder's weight matrices (W1/W2) to extract the top 200 pivotal genes from the compressed transcription space, which are then used for KEGG/GO over-representation analysis.

## When to use

Use when a trained scMGCA (GCN autoencoder) latent embedding is available and the goal is functional interpretation — identifying the pivotal genes the decoder weights encode so they can be tested for pathway/GO enrichment.

## When NOT to use

- You want marker genes per cell cluster (p_val_adj <= 0.05, avg_logFC >= 0.5) rather than pivotal genes extracted from autoencoder weight matrices
- The model is not a GCN-based graph autoencoder (e.g. DCA or scVI without decoder weight matrices) — no W1/W2 weight structure exists to rank
- You need differential expression across conditions instead of functional interpretation of the compressed transcriptomic space

## Inputs

- Trained scMGCA model weight matrices W1 and W2 (two GCN layers, 128 latent dimensions)
- Preprocessed scRNA-seq count matrix (normalized, genes not expressed in cells removed, highly variable genes selected in SCANPY)
- Latent embedded representation from the graph-embedding autoencoder

## Outputs

- Ranked gene list (r2-based ranking)
- Top 200 pivotal genes (highly variable genes from r2)
- KEGG/GO over-representation results (ClusterProfiler, pvalueCutoff=0.01) written to pivotal_genes_enrichment.csv

## How to apply

After training scMGCA in TensorFlow (two GCN layers, multinomial-based decoder), compute the standard deviation across the 128 latent dimensions of the second-layer weight matrix W2 and retain the top 50% (128 x 50% = 64) highest-variance dimensions; use these to filter the first-layer weight matrix W1. Rank genes by the standard deviation and weight of the latent embedded representation (r2) and extract the top 200 highly variable genes from r2. These 200 pivotal genes then feed KEGG/GO over-representation analysis in ClusterProfiler with pvalueCutoff=0.01; write the ranked gene list and enrichment table to pivotal_genes_enrichment.csv. The rationale is that high variance in decoder weights marks the latent dimensions carrying the strongest gene-encoding signal in the compressed transcriptomic space.

## Related tools

- **scMGCA** (Trained graph-convolutional autoencoder whose W1/W2 weight matrices supply the standard deviation and weights used to rank and select the top 200 genes) — https://pypi.org/project/scMGCA
- **TensorFlow** (Framework in which the scMGCA core model is built, producing the weight matrices analyzed)
- **SCANPY** (Preprocesses the input gene expression matrix (normalize, filter genes not expressed, select highly variable genes) before training)
- **ClusterProfiler** (Runs KEGG and GO over-representation analysis on the extracted top 200 pivotal genes (pvalueCutoff=0.01))
- **scikit-learn** (Provides K-means (vs spectral clustering) initialization of clustering centers used during scMGCA training)

## Examples

```
import numpy as np; W2_sd = W2.std(axis=0); keep = np.argsort(-W2_sd)[:64]; r2 = np.abs(W1[:, keep]).sum(axis=1); top200 = np.argsort(-r2)[:200]
```

## Evaluation signals

- Exactly 64 dimensions retained from W2 (128 x 50% = 64 highest-variance dimensions)
- Gene list contains exactly the top 200 highly variable genes extracted from r2
- Enrichment output contains significant KEGG/GO terms at adjusted p-value <= 0.01 (pvalueCutoff=0.01)
- On the 'Qx Limb Muscle' dataset, the top 200 genes should reproduce the reported enrichment in chemotaxis and migration of neutrophils (GO biological process)
- Ranked gene list and enrichment table written to pivotal_genes_enrichment.csv

## Limitations

- No software version numbers are reported (Python, TensorFlow, SCANPY all unstated), so exact reproduction may be environment-sensitive
- The number of highly variable genes d is not given a numeric value in the methods; only the 64-dimension and 200-gene cutoffs are specified
- The ranking depends on the trained weight matrices, so random seed / initialization of clustering centers (K-means vs spectral clustering) can shift which genes are selected; dataset accessions for the benchmark data are not provided in the text

## Evidence

- [results] Select top 200 highly expressed genes using standard deviation and weight of latent representation: "we selected the top 200 highly expressed genes using the standard deviation and weight of the latent embedded representation"
- [discussion] Retain top 50% (64) of highest-variance dimensions from W2 standard deviation: "we select the top 50% (128 × 50% = 64) of highest variance from"
- [discussion] Extract top 200 highly variable genes from r2: "we extract the top 200 highly variable genes from"
- [readme] scMGCA package description from README: "scMGCA is a Python package containing tools for clustering single-cell data based on a graph-embedding autoencoder"
