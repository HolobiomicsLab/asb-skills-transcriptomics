---
name: dropout-robustness-evaluation-across-simulated-scenarios
description: Use when a scRNA-seq clustering/dimensionality-reduction method's robustness
  to dropout (zero-inflation) must be quantified, i.e. simulated count matrices at
  multiple dropout rates with ground-truth cell labels are available and NMI versus
  true labels is the target metric.
when_to_use_negative:
- Input is a single empirical dataset without ground-truth labels or controlled dropout
  levels — NMI versus true labels cannot be computed and dropout robustness cannot
  be swept
- Goal is 2D visualization rather than robustness quantification — the paper notes
  the graph autoencoder cannot directly embed into 2D and requires an intermediate
  dimension plus tSNE, a different workflow
- The competing method is scGNN-style trajectory inference — the evaluation target
  here is clustering NMI, where the paper reports scGAE, not trajectory reconstruction,
  as the differentiator
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0102
tools:
- name: scGAE
  role: 'Method under test: builds a KNN cell graph (default K=35) and trains a multitask-oriented
    graph autoencoder to produce embeddings for Louvain clustering and NMI scoring'
  repo: github.com/ZixiangLuo1161/scGAE
- name: Spektral
  role: Python package used to implement the scGAE graph autoencoder model (version
    0.6.1 per README)
- name: scanpy
  role: Python package (version 1.8.0 per README) usable for single-cell data preprocessing
    before model building
- name: Seurat
  role: R package used for scRNA-seq preprocessing (scTransform) prior to scGAE graph
    construction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:bc126912c0c642115335aac63611d5aa503d75d6195b2ba3678f0d042f137a1f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat
  - Spektral
  - scGAE
  - scanpy
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# dropout-robustness-evaluation-across-simulated-scenarios

## Summary

Quantifies how robust a scRNA-seq dimensionality-reduction-and-clustering pipeline is to zero-inflation by recomputing clustering NMI across simulated scenario4 datasets spanning four dropout rates (12%, 17%, 23%, 30%) and showing the metric stays near-constant (0.61–0.65 for scGAE).

## When to use

Use when a scRNA-seq clustering/dimensionality-reduction method's robustness to dropout (zero-inflation) must be quantified, i.e. simulated count matrices at multiple dropout rates with ground-truth cell labels are available and NMI versus true labels is the target metric.

## When NOT to use

- Input is a single empirical dataset without ground-truth labels or controlled dropout levels — NMI versus true labels cannot be computed and dropout robustness cannot be swept
- Goal is 2D visualization rather than robustness quantification — the paper notes the graph autoencoder cannot directly embed into 2D and requires an intermediate dimension plus tSNE, a different workflow
- The competing method is scGNN-style trajectory inference — the evaluation target here is clustering NMI, where the paper reports scGAE, not trajectory reconstruction, as the differentiator

## Inputs

- Four simulated scenario4 scRNA-seq count matrices with dropout rates of 12%, 17%, 23%, and 30% (as referenced by Run_scGAE.ipynb in the scGAE repository)
- Ground-truth cell-type labels for each simulated dataset

## Outputs

- 4-row CSV table with one row per dropout rate containing the dropout rate and the scGAE clustering NMI (expected ≈ 0.62, 0.62, 0.65, 0.61)
- Latent embeddings and predicted cluster labels per dataset (from model.embedding() and Louvain clustering)

## How to apply

Load the simulated count matrices spanning the dropout range (scenario4 variants at 12±0.3%, 17±0.4%, 23±0.5%, 30±0.6% dropout), preprocess them (e.g. scTransform via Seurat), and run the method under identical settings at each dropout level. For scGAE, build a KNN cell graph with the default K=35 using get_adj, train the multitask-oriented graph autoencoder (implemented with the Spektral package) with model.train(), obtain initial centroids by Louvain clustering with get_centers_louvain, optionally refine with model.clustering_train(), and extract the latent embedding with model.embedding(). Compute normalized mutual information (NMI) between Louvain-predicted clusters and ground-truth labels for each dropout rate. Rationale: if the method truly preserves topological/feature structure under zero-inflation, NMI should stay within a narrow band across the four levels rather than degrade monotonically with dropout. Emit a 4-row CSV (dropout rate, NMI) with expected values approximately 0.62, 0.62, 0.65, 0.61 and validate each within a 0.55–0.70 tolerance band.

## Related tools

- **scGAE** (Method under test: builds a KNN cell graph (default K=35) and trains a multitask-oriented graph autoencoder to produce embeddings for Louvain clustering and NMI scoring) — github.com/ZixiangLuo1161/scGAE
- **Spektral** (Python package used to implement the scGAE graph autoencoder model (version 0.6.1 per README))
- **scanpy** (Python package (version 1.8.0 per README) usable for single-cell data preprocessing before model building)
- **Seurat** (R package used for scRNA-seq preprocessing (scTransform) prior to scGAE graph construction)

## Examples

```
from scgae import SCGAE, get_adj, get_centers_louvain; adj = get_adj(data); model = SCGAE(adj); model.train(); centers = get_centers_louvain(model.embedding()); model.clustering_train(centers); emb = model.embedding()
```

## Evaluation signals

- Each per-dropout NMI falls within the tolerance band 0.55–0.70 around the reported values (0.62, 0.62, 0.65, 0.61)
- NMI varies by no more than ~0.04 across the 12%→30% dropout sweep, demonstrating zero-inflation robustness
- Output CSV has exactly 4 rows pairing each dropout rate (12%, 17%, 23%, 30%) with its NMI
- Simulated dataset dropout rates match the reported ranges 12±0.3%, 17±0.4%, 23±0.5%, 30±0.6%
- The pipeline follows the README sequence: get_adj → SCGAE model → model.train() → get_centers_louvain → model.clustering_train() → model.embedding()

## Limitations

- No random seed or stochastic-variation control is reported; Louvain-initialized clustering on pre-trained embeddings introduces run-to-run variability in NMI
- The optimal K for KNN graph construction varies among datasets and dataset regions; the default K=35 may not be optimal for every dropout level
- The paper specifies no software/package versions (README lists pinned versions, e.g. tensorflow 2.3.0, spektral 0.6.1), so exact reproduction depends on environment reconstruction

## Evidence

- [abstract] scGAE accurately reconstructs developmental trajectories and separates discrete cell clusters under different simulated scenarios: "Analyses of simulated data showed that scGAE accurately reconstructs developmental trajectory and separates discrete cell clusters under different scenarios, outperforming recently developed deep"
- [discussion] KNN graph default K=35: "The default K is predened as 35 in this study and adjusted according to the datasets in our experiments."
- [discussion] Louvain initial centroids from pre-trained embeddings: "In experiments, we use the pre-trained network to generate the initial embedded data which are clustered to obtain the initial centroids by Louvain37."
- [discussion] Optimal K unresolved across datasets: "the optimal K varies among different datasets and different parts of a dataset."
- [readme] README clustering-training workflow: "After getting the clustering centers using get_centers_louvain function, perform the clustering training using model.clustering_train() function."
