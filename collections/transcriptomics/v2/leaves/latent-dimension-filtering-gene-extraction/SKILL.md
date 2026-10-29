---
name: latent-dimension-filtering-gene-extraction
description: Use when you have trained scMGCA (two GCN layers, 128-dim latent space)
  on a scRNA-seq dataset and need a biological interpretation of the compressed transcription
  space via gene ranking and enrichment.
when_to_use_negative:
- You need cell-type annotation or clustering (use scMGCA's clustering output directly,
  not gene extraction)
- The model is not an autoencoder with interpretable W1/W2 GCN weight matrices — dimension
  filtering assumes the 128-dim latent structure
- You only need differential expression between annotated clusters, e.g. marker genes
  with p_val_adj ≤ 0.05 and avg_logFC ≥ 0.5
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scMGCA
  role: Trained graph-embedding autoencoder whose W1/W2 weight matrices and latent
    representation r2 are the raw material for gene ranking
  repo: https://pypi.org/project/scMGCA
- name: SCANPY
  role: Normalizes the gene expression matrix and selects highly variable genes before
    model training and gene extraction
- name: TensorFlow
  role: Framework in which the GCN autoencoder is built and from which weight matrices
    are extracted
- name: ClusterProfiler
  role: Runs KEGG and GO over-representation analysis on the top 200 pivotal genes
    with pvalueCutoff=0.01
- name: scikit-learn
  role: Provides K-means initialization of clustering centers for scMGCA training
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36134-7
    title: Topological identification and interpretation for single-cell gene regulation
      elucidation across multiple platforms using scMGCA
  source_content_hash: sha256:f3ed9d4bbbf1684dad43c7b4226e1cbfc5b5748439f145be04fd9249123dc872
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ClusterProfiler
  - SCANPY
  - TensorFlow
  - scMGCA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# latent-dimension-filtering-gene-extraction

## Summary

Extract interpretable 'pivotal' genes from the scMGCA graph-convolutional autoencoder by ranking latent-space weight matrices (filtering W2 to the top 50% highest-variance dimensions, i.e. 64 of 128) and selecting the top 200 highly variable genes from the latent representation r2, then interpreting them with KEGG/GO enrichment.

## When to use

Use when you have trained scMGCA (two GCN layers, 128-dim latent space) on a scRNA-seq dataset and need a biological interpretation of the compressed transcription space via gene ranking and enrichment.

## When NOT to use

- You need cell-type annotation or clustering (use scMGCA's clustering output directly, not gene extraction)
- The model is not an autoencoder with interpretable W1/W2 GCN weight matrices — dimension filtering assumes the 128-dim latent structure
- You only need differential expression between annotated clusters, e.g. marker genes with p_val_adj ≤ 0.05 and avg_logFC ≥ 0.5

## Inputs

- Trained scMGCA model weight matrices W1 and W2 from the two-layer GCN encoder (TensorFlow)
- Latent embedded representation r2 of the autoencoder's compressed transcription space
- Preprocessed scRNA-seq count matrix (normalized, highly variable genes selected, zero-expression genes filtered out in SCANPY)

## Outputs

- Ranked list of top 200 highly variable pivotal genes
- KEGG/GO over-representation results table at pvalueCutoff=0.01 (ClusterProfiler)
- pivotal_genes_enrichment.csv (ranked gene list + enrichment results)

## How to apply

After training scMGCA in TensorFlow (GCN encoder with two layers, multinomial-based decoder, KL-divergence self-optimized clustering initialized by the better of K-means and spectral clustering), compute the standard deviation across the 128 latent dimensions of W2 and retain the top 50% (128 × 50% = 64) highest-variance dimensions to filter W1. Rank genes using the standard deviation and weight of the latent embedded representation, and extract the top 200 highly variable genes from r2. Run KEGG and GO over-representation analysis on these 200 pivotal genes with ClusterProfiler using pvalueCutoff=0.01. The rationale is that high-variance latent dimensions and high gene weights indicate which genes drive the cell–cell topology representation learned by the graph-embedding autoencoder. Write the ranked gene list and enrichment results to pivotal_genes_enrichment.csv.

## Related tools

- **scMGCA** (Trained graph-embedding autoencoder whose W1/W2 weight matrices and latent representation r2 are the raw material for gene ranking) — https://pypi.org/project/scMGCA
- **SCANPY** (Normalizes the gene expression matrix and selects highly variable genes before model training and gene extraction)
- **TensorFlow** (Framework in which the GCN autoencoder is built and from which weight matrices are extracted)
- **ClusterProfiler** (Runs KEGG and GO over-representation analysis on the top 200 pivotal genes with pvalueCutoff=0.01)
- **scikit-learn** (Provides K-means initialization of clustering centers for scMGCA training)

## Examples

```
import numpy as np; dims = np.argsort(np.std(W2, axis=0))[::-1][:64]; W1f = W1[dims]; genes = np.argsort(r2)[::-1][:200]; write_csv('pivotal_genes_enrichment.csv', genes)
```

## Evaluation signals

- Exactly the top 50% (64 of 128) latent dimensions are retained when filtering W2 by standard deviation
- Exactly 200 genes are extracted from the ranked list of r2
- Enrichment output contains significant KEGG/GO terms at adjusted p-value ≤ 0.01
- On 'Qx Limb Muscle', enriched biological processes should reproduce reported hits such as chemotaxis and migration of neutrophils
- pivotal_genes_enrichment.csv contains both a ranked gene list and the enrichment results table

## Limitations

- Software version numbers (Python, TensorFlow, SCANPY, R packages) are not reported, making exact reproduction of the environment uncertain
- The number of latent dimensions (128) and the 50% retention cutoff are hard-coded to scMGCA's architecture; other latent sizes need a proportional choice of d
- The article does not state numeric loss weights (γ1, γ2, γ3), KNN neighbor count k, restart probability α, or maximum iteration count, so the trained embedding — and hence the extracted genes — may vary between runs

## Evidence

- [discussion] dimension filtering: "we select the top 50% (128 × 50% = 64) of highest variance from"
- [discussion] gene extraction: "we extract the top 200 highly variable genes from"
- [results] gene selection rationale: "we selected the top 200 highly expressed genes using the standard deviation and weight of the latent embedded representation"
- [abstract] interpretation objective: "we perform genomic interpretation on the key compressed transcriptomic space of the graph-embedding autoencoder"
