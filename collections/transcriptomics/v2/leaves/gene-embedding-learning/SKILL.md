---
name: gene-embedding-learning
description: Use when constructing cell-lineage-specific gene embeddings from scRNA-seq
  data and a prior gene interaction network, e.g. inputs to downstream driver-regulator
  or RGM analysis.
when_to_use_negative:
- The goal is driver-regulator identification or RGM activity scoring alone, without
  learning new embeddings from a prior network (use cf.driver_regulators / cf.RGM_activity
  directly on existing results)
- The data involve multiple fate trajectories with bifurcations or extrinsic factors
  such as cell–cell communication, which CEFCON explicitly does not handle
- Ground-truth labels are available and a supervised framework such as GENELink is
  feasible
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CEFCON
  role: Provides cf.NetModel (graph attention network trained with DGI contrastive
    learning) and get_cefcon_results to produce gene embeddings
  repo: https://github.com/WPZgithub/CEFCON
- name: PyTorch
  role: Deep-learning backend for the graph attention model
  repo: https://pytorch.org/get-started/locally/
- name: torch-geometric
  role: Implements the graph attention layers used to learn node representations
  repo: https://pytorch-geometric.readthedocs.io/en/latest/notes/installation.html
- name: scanpy
  role: Handles the scRNA-seq expression matrix (.h5ad) input for data preparation
  repo: https://scanpy.readthedocs.io/en/stable/installation.html
- name: networkx
  role: Handles the prior gene interaction network in edgelist format
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:e6bc20fef74eafa6d6145a7233bd37f9ec114e87667297c06eb49758b28f6273
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WPZgithub/CEFCON
  tools:
  - CEFCON
  - PyTorch
  - Python
  - networkx
  - scanpy
  - torch-geometric
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# gene-embedding-learning

## Summary

Learns gene representations by training CEFCON's graph attention network under a deep graph infomax (DGI) contrastive objective on a cell-lineage prior network with scRNA-seq expression and differential-expression encodings; the resulting node embeddings accompany the cell-lineage-specific GRN.

## When to use

Use when constructing cell-lineage-specific gene embeddings from scRNA-seq data and a prior gene interaction network, e.g. inputs to downstream driver-regulator or RGM analysis.

## When NOT to use

- The goal is driver-regulator identification or RGM activity scoring alone, without learning new embeddings from a prior network (use cf.driver_regulators / cf.RGM_activity directly on existing results)
- The data involve multiple fate trajectories with bifurcations or extrinsic factors such as cell–cell communication, which CEFCON explicitly does not handle
- Ground-truth labels are available and a supervised framework such as GENELink is feasible

## Inputs

- scRNA-seq expression matrix for the target cell lineage (.csv or .h5ad)
- prior gene interaction network in edgelist format
- gene log fold-change table (differential-expression encoding)

## Outputs

- gene_embs.csv with learned node (gene) representations
- cell_lineage_GRN.csv, the cell-lineage-specific GRN edge list derived from degree-scaled attention coefficients

## How to apply

Prepare the inputs with cf.data_preparation by combining the scRNA-seq expression matrix (.csv or .h5ad), the prior gene interaction network in edgelist format, and the gene log fold-change table for the target cell lineage. Instantiate cf.NetModel(epochs=350, repeats=3, cuda='0') with a cosine attention variant (attention={COS,AD,SD}) that incorporates the gene differential-expression encoding into the attention computation. Train the two-layer graph attention network with the configured hidden_dim/output_dim/heads under the DGI contrastive objective, repeating training the configured number of times (repeats=3) to stabilize the learned representations. After training, obtain results via get_cefcon_results(edge_threshold_avgDegree=8), which writes both the degree-scaled attention-derived GRN edges (cell_lineage_GRN.csv) and the learned node embeddings (gene_embs.csv) to the output directory.

## Related tools

- **CEFCON** (Provides cf.NetModel (graph attention network trained with DGI contrastive learning) and get_cefcon_results to produce gene embeddings) — https://github.com/WPZgithub/CEFCON
- **PyTorch** (Deep-learning backend for the graph attention model) — https://pytorch.org/get-started/locally/
- **torch-geometric** (Implements the graph attention layers used to learn node representations) — https://pytorch-geometric.readthedocs.io/en/latest/notes/installation.html
- **scanpy** (Handles the scRNA-seq expression matrix (.h5ad) input for data preparation) — https://scanpy.readthedocs.io/en/stable/installation.html
- **networkx** (Handles the prior gene interaction network in edgelist format)

## Examples

```
cefcon_GRN_model = cf.NetModel(epochs=350, repeats=3, cuda='0'); cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)
```

## Evaluation signals

- gene_embs.csv exists in the output directory and contains an embedding vector for each gene in the input prior network
- cell_lineage_GRN.csv is a valid edge-list CSV derived from the degree-scaled attention coefficients
- The inferred GRN's average node degree is consistent with the configured edge_threshold_avgDegree (8)
- Training was repeated the configured number of times (repeats=3) and attention-derived edge weights averaged across repeats
- Embedding dimensionality matches the model's hidden_dim/output_dim/heads configuration

## Limitations

- CEFCON can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously
- Only intracellular factors are modeled; cell–cell communication and spatial location are not incorporated
- Epigenetic information is not incorporated, and extension to spatial transcriptome or chromatin accessibility data is future work

## Evidence

- [intro] DGI contrastive learning framing: "CEFCON first employs a graph attention neural network under a contrastive learning framework to construct a cell-lineage-specific GRN."
- [methods] model instantiation and result extraction: "cefcon_GRN_model = cf.NetModel(epochs=350, repeats=3, cuda='0')"
- [other] attention coefficients scaled by node degree: [evidence span withheld — anchor status: not_found]
- [readme] tool purpose from README: "CEFCON initially employs the graph attention neural networks under a contrastive learning framework to construct reliable GRNs 
for specific developmental cell lineages"
