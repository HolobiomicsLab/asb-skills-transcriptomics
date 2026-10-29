---
name: single-cell-grn-construction
description: Use when building a lineage-specific GRN from a scRNA-seq expression
  matrix (e.g., AnnData with a developmental trajectory) and a prior gene interaction
  network edgelist, as input to driver regulator or gene module analyses.
when_to_use_negative:
- When multiple fate trajectories with bifurcations must be processed simultaneously
  — CEFCON can only handle one cell lineage at a time
- When only scRNA-seq data are available without a prior gene interaction network,
  since CEFCON requires a prior network as input
- When modeling extrinsic factors (cell–cell communication, spatial location) or epigenetic
  regulation — CEFCON focuses on intracellular RNA-level factors
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_2259
tools:
- name: CEFCON
  role: 'Main framework: graph attention neural network with contrastive learning
    that infers the cell-lineage-specific GRN'
  repo: https://github.com/WPZgithub/CEFCON
- name: PyTorch
  role: Deep learning backend for training the graph attention network (pytorch >=1.13.0,<2.0)
  repo: https://pytorch.org/get-started/locally/
- name: torch-geometric
  role: Graph neural network library implementing the graph attention layers (torch-geometric
    >=2.1.0)
  repo: https://pytorch-geometric.readthedocs.io/en/latest/notes/installation.html
- name: scanpy
  role: Handles the AnnData scRNA-seq expression input (scanpy >=1.8.2)
  repo: https://scanpy.readthedocs.io/en/stable/installation.html
- name: networkx
  role: Loads and manipulates the prior gene interaction network edgelist (networkx
    >=3.0)
- name: PRROC
  role: R package used to compute AUPRC when evaluating inferred GRNs against ground-truth
    edges
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:925c5ef6f67e217833ab5228c7ddbd27aa37663982e1655488fd8ccca0f039ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WPZgithub/CEFCON
  tools:
  - CEFCON
  - PRROC
  - PyTorch
  - Python
  - networkx
  - pandas
  - scanpy
  - scikit-learn
  - torch-geometric
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# single-cell-grn-construction

## Summary

Construct cell-lineage-specific gene regulatory networks (GRNs) from scRNA-seq data and a prior gene interaction network using CEFCON's graph attention neural network under a contrastive learning framework, as described in Wang et al. 2023 (Nature Communications).

## When to use

Use when building a lineage-specific GRN from a scRNA-seq expression matrix (e.g., AnnData with a developmental trajectory) and a prior gene interaction network edgelist, as input to driver regulator or gene module analyses.

## When NOT to use

- When multiple fate trajectories with bifurcations must be processed simultaneously — CEFCON can only handle one cell lineage at a time
- When only scRNA-seq data are available without a prior gene interaction network, since CEFCON requires a prior network as input
- When modeling extrinsic factors (cell–cell communication, spatial location) or epigenetic regulation — CEFCON focuses on intracellular RNA-level factors

## Inputs

- scRNA-seq expression matrix as AnnData object (e.g., hESC dataset, 1,000 highly variable genes, from Zenodo record 7564872)
- Prior gene interaction network edgelist (human or mouse), loadable with networkx (>=3.0)
- Cell lineage/trajectory assignment (e.g., from Palantir), one lineage per CEFCON run

## Outputs

- Cell-lineage-specific inferred GRN as edge list with confidence weights, thresholded via edge_threshold_avgDegree=8
- CEFCON results object (cefcon_results) supporting gene_influence_score(), driver_regulators(), and RGM_activity()
- GRN edge predictions evaluable against ground-truth regulatory edges via AUPRC (PRROC R package)

## How to apply

Prepare inputs with `cf.data_preparation(adata, prior_network)`, aligning the AnnData expression matrix (e.g., the hESC dataset with 1,000 highly variable genes) with the prior network's node set. Instantiate the model with `cf.NetModel(epochs=350, repeats=3, cuda='0')` and train on GPU using PyTorch (>=1.13.0,<2.0) and torch-geometric (>=2.1.0); the graph attention network under contrastive learning propagates regulatory signal from the prior network constrained by the expression data. Retrieve inferred GRNs with `cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)`, thresholding edges by average degree to obtain a sparse regulatory graph per cell lineage. Rationale: leveraging a prior network plus attention-based GNN yields more reliable lineage-specific GRNs than inference from scRNA-seq data alone, which the paper identifies as a key gap. The output GRNs feed downstream driver regulator identification, influence scoring, and RGM activity analysis.

## Related tools

- **CEFCON** (Main framework: graph attention neural network with contrastive learning that infers the cell-lineage-specific GRN) — https://github.com/WPZgithub/CEFCON
- **PyTorch** (Deep learning backend for training the graph attention network (pytorch >=1.13.0,<2.0)) — https://pytorch.org/get-started/locally/
- **torch-geometric** (Graph neural network library implementing the graph attention layers (torch-geometric >=2.1.0)) — https://pytorch-geometric.readthedocs.io/en/latest/notes/installation.html
- **scanpy** (Handles the AnnData scRNA-seq expression input (scanpy >=1.8.2)) — https://scanpy.readthedocs.io/en/stable/installation.html
- **networkx** (Loads and manipulates the prior gene interaction network edgelist (networkx >=3.0))
- **PRROC** (R package used to compute AUPRC when evaluating inferred GRNs against ground-truth edges)

## Examples

```
import cefcon as cf; data = cf.data_preparation(adata, prior_network); model = cf.NetModel(epochs=350, repeats=3, cuda='0'); results = model.get_cefcon_results(edge_threshold_avgDegree=8)
```

## Evaluation signals

- get_cefcon_results with edge_threshold_avgDegree=8 returns a non-empty thresholded GRN for each lineage, and perturbed_auprc_results.csv (in robustness studies) contains AUPRC columns per dataset
- Benchmark AUPRC against ground-truth regulatory edges should exceed baseline methods and remain within ~10% of baseline when 20% of prior network edges are randomly shuffled
- The inferred GRN's node set matches the prior network's node set and the genes in the AnnData object after cf.data_preparation
- Training completes for the specified repeats (repeats=3) and epochs (epochs=350) on the chosen CUDA device without convergence errors

## Limitations

- CEFCON can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously
- Only intracellular factors are modeled; cell–cell communication, spatial location, and epigenetics are not incorporated
- Limited to RNA-seq data; extension to spatial transcriptomics, chromatin accessibility, or multi-omics is future work
- Python version restricted to 3.9–3.10 and PyTorch to >=1.13.0,<2.0; GPU with CUDA recommended for practical runtimes

## Evidence

- [abstract] first uses a graph neural network with attention mechanism to infer a cell-lineage-specific gene regulatory network (GRN) from single-cell RNA-sequencing data: "first uses a graph neural network with attention mechanism to infer a cell-lineage-specific gene regulatory network (GRN) from single-cell RNA-sequencing data"
- [methods] data_preparation and NetModel invocation: "data = cf.data_preparation(adata, prior_network)"
- [methods] model instantiation: "cefcon_GRN_model = cf.NetModel(epochs=350, repeats=3, cuda='0')"
- [methods] results retrieval threshold: "cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)"
- [intro] gap motivating prior network use: "they rely heavily on the inferred GRNs, while current GRN construction methods only based on scRNA-seq data are still not accurate enough"
- [readme] dependency requirements: "pytorch(>=1.13.0,<2.0)"
- [discussion] one lineage limitation: "First, CEFCON can only handle one cell lineage at a time."
