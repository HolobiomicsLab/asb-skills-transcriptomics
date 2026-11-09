---
name: single-cell-embedding-benchmarking
description: Use when comparing how well alternative low-dimensional embeddings of
  simulated scRNA-seq preserve cell-cell topological structure, evaluated as trajectory
  (pseudotime) recovery measured by Kendall correlation against a known ground truth.
when_to_use_negative:
- The input is empirical data with no simulated ground-truth trajectory ordering —
  Kendall-tau benchmarking against truth is undefined there
- The goal is clustering rather than trajectory recovery — the paper reports scGAE
  vs competitors differ in their clustering ranking, so tau on pseudotime would not
  answer the question
- A single method's embedding quality is already established and no competitor comparison
  is needed — building the full multi-method table adds no information
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scGAE
  role: primary graph autoencoder embedding method; builds a KNN cell graph and preserves
    topological and feature information
  repo: github.com/ZixiangLuo1161/scGAE
- name: scGNN
  role: graph-based competitor expected to perform comparably on trajectory inference
- name: scVI
  role: deep-learning competitor embedding method run with default settings
- name: DCA
  role: deep-learning competitor embedding method run with default settings
- name: PHATE
  role: competitor embedding method run with default settings
- name: SAUCIE
  role: competitor embedding method run with default settings
- name: ZINB-WaVE
  role: competitor embedding method run with default settings
- name: Ivis
  role: competitor embedding method run with default settings
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:c1d18caa3478578ee7b9253b9120e43b8a5170c3ffb40a83a0719a7d6e19de92
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DCA
  - Ivis
  - PHATE
  - SAUCIE
  - ZINB-WaVE
  - scGAE
  - scGNN
  - scVI
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-embedding-benchmarking

## Summary

Benchmarks dimensionality-reduction methods for scRNA-seq by embedding the same simulated lineage datasets with scGAE and competitors (DCA, scVI, PHATE, SAUCIE, scGNN, ZINB-WaVE, Ivis), running diffusion pseudotime on each embedding, and scoring trajectory recovery by Kendall-tau correlation against the simulated ground-truth ordering.

## When to use

Use when comparing how well alternative low-dimensional embeddings of simulated scRNA-seq preserve cell-cell topological structure, evaluated as trajectory (pseudotime) recovery measured by Kendall correlation against a known ground truth.

## When NOT to use

- The input is empirical data with no simulated ground-truth trajectory ordering — Kendall-tau benchmarking against truth is undefined there
- The goal is clustering rather than trajectory recovery — the paper reports scGAE vs competitors differ in their clustering ranking, so tau on pseudotime would not answer the question
- A single method's embedding quality is already established and no competitor comparison is needed — building the full multi-method table adds no information

## Inputs

- Simulated scRNA-seq lineage datasets data5 (scenario1) and data2 (scenario2) from the scGAE repository Run_scGAE.ipynb
- Ground-truth simulated trajectory cell orderings for both scenarios
- scGAE model files (preprocessing.py, scgae.py, layers.py, losses.py, utils.py)

## Outputs

- Low-dimensional embeddings per method per scenario
- Diffusion pseudotime (DPT) orderings per embedding
- Comparison table kendall_tau_comparison.csv with one Kendall-tau value per method per scenario in [-1, 1]

## How to apply

Load the simulated lineage datasets (scenario1 'data5' and scenario2 'data2' from the scGAE repository notebook Run_scGAE.ipynb) and embed each with scGAE — preprocessing with scTransform (Seurat), building a KNN cell graph with default K=35 via get_adj, and training the multitask-oriented graph autoencoder with model.train() before model.embedding(). Embed the identical datasets with each competitor (DCA, scVI, PHATE, SAUCIE, scGNN, ZINB-WaVE, Ivis) under default settings so the comparison isolates embedding quality. Run diffusion pseudotime (DPT) on every low-dimensional embedding and compute Kendall-tau between each inferred ordering and the ground-truth simulated ordering, excluding scDeepCluster as in the paper's Fig. 3a,b comparison. Assemble one row per method per scenario into kendall_tau_comparison.csv; the graph-based methods (scGAE, scGNN, scVI) are expected to recover trajectories better than feature-only embeddings, reflecting the paper's argument that preserving cell-cell relationships via a cell graph maintains developmental structure.

## Related tools

- **scGAE** (primary graph autoencoder embedding method; builds a KNN cell graph and preserves topological and feature information) — github.com/ZixiangLuo1161/scGAE
- **scGNN** (graph-based competitor expected to perform comparably on trajectory inference)
- **scVI** (deep-learning competitor embedding method run with default settings)
- **DCA** (deep-learning competitor embedding method run with default settings)
- **PHATE** (competitor embedding method run with default settings)
- **SAUCIE** (competitor embedding method run with default settings)
- **ZINB-WaVE** (competitor embedding method run with default settings)
- **Ivis** (competitor embedding method run with default settings)

## Examples

```
from scgae import SCGAE, get_adj; adj, X = get_adj(data); model = SCGAE(X, adj); model.train(); emb = model.embedding()
```

## Evaluation signals

- Every reported Kendall tau lies within [-1, 1]
- Relative ordering of methods in kendall_tau_comparison.csv matches Fig. 3a,b of the paper (scGAE, scGNN, scVI ahead of other competitors on both scenarios)
- Both scenarios (scenario1/data5 and scenario2/data2) are represented for every method, with no missing method-by-scenario rows
- All method embeddings were produced under default settings from identical input matrices, and DPT was run on each before computing tau

## Limitations

- The optimal K for KNN graph construction varies among datasets and across parts of a dataset; default K=35 may need adjustment
- The graph autoencoder does not directly produce a 2D embedding — an intermediate dimension plus tSNE is used for visualization, and a better structure-preserving 2D method is still needed
- No random seed or stochastic-variation control is reported, and no software/package versions are stated in the article (README pins tensorflow 2.3.0, scanpy 1.8.0, spektral 0.6.1, etc.)

## Evidence

- [intro] competitor list: "Recently developed competitive methods include DCA7, scVI8, scDeepCluster9, PHATE10, SAUCIE11, scGNN12, ZINB-WaVE13 and Ivis14"
- [abstract] graph-based advantage: "scGAE builds a cell graph and uses a multitask-oriented graph autoencoder to preserve topological structure information and feature information"
- [discussion] KNN default: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
- [discussion] scGAE vs scGNN tradeoff: "scGAE and scGNN have similar performance on the trajectory inference while scGAE has better performance on clustering."
- [readme] README quick start: "Compute the adjacency matrix using get_adj function and build the SCGAE model."
