---
name: chipseq-ground-truth-network-evaluation
description: Use when an inferred gene regulatory network from scRNA-seq data must
  be quantitatively validated against a ChIP-seq ground-truth network using AUPRC
  and EPR on BEELINE lineage benchmarks.
when_to_use_negative:
- No ChIP-seq ground-truth network exists for the cell type or lineage of interest
  — the article notes supervised/validated alternatives are limited because 'the label
  datasets serving as ground-truths are often lacking'
- The goal is driver regulator or regulon-like gene module (RGM) identification rather
  than GRN construction performance benchmarking
- Evaluating a network inferred by a supervised framework such as GENELink trained
  directly on ground-truth labels, where AUPRC/EPR benchmarking against the same labels
  would be circular
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0749
tools:
- name: CEFCON
  role: Infers the cell-lineage-specific GRN under evaluation; provides data_preparation,
    NetModel and get_cefcon_results APIs used in the benchmark workflow
  repo: https://github.com/WPZgithub/CEFCON
- name: Python
  role: Implementation language for running CEFCON and computing AUPRC/EPR metrics
    against ground-truth networks
- name: PyTorch
  role: Deep learning backend (>=1.13.0,<2.0) for training the CEFCON graph attention
    network (epochs=350, repeats=3)
- name: torch-geometric
  role: Graph neural network library (>=2.1.0) underlying CEFCON's graph attention
    model on prior networks
- name: scanpy
  role: Loads and preprocesses the BEELINE scRNA-seq adata objects passed to cf.data_preparation
    (>=1.8.2)
- name: networkx
  role: Handles prior/ground-truth graph structures (>=3.0) during data preparation
    and metric computation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:eee32901c7f20e5cb722d6fab9a29820903c5a441a9c0c3d741802c9c3a54b20
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# chipseq-ground-truth-network-evaluation

## Summary

Evaluates inferred cell-lineage-specific GRNs against ChIP-seq-derived ground-truth networks on BEELINE benchmark scRNA-seq datasets using AUPRC and EPR (early precision) metrics, as done to benchmark CEFCON against SCINET, NetREX, CellOracle, GRNBoost2, DeepSEM and Random_NicheNet.

## When to use

Use when an inferred gene regulatory network from scRNA-seq data must be quantitatively validated against a ChIP-seq ground-truth network using AUPRC and EPR on BEELINE lineage benchmarks.

## When NOT to use

- No ChIP-seq ground-truth network exists for the cell type or lineage of interest — the article notes supervised/validated alternatives are limited because 'the label datasets serving as ground-truths are often lacking'
- The goal is driver regulator or regulon-like gene module (RGM) identification rather than GRN construction performance benchmarking
- Evaluating a network inferred by a supervised framework such as GENELink trained directly on ground-truth labels, where AUPRC/EPR benchmarking against the same labels would be circular

## Inputs

- BEELINE lineage scRNA-seq datasets (seven lineages)
- ChIP-seq prior gene interaction networks
- ChIP-seq ground-truth networks per lineage
- Inferred cell-lineage-specific GRN (cell_lineage_GRN.csv) from CEFCON's get_cefcon_results(edge_threshold_avgDegree=8)

## Outputs

- AUPRC value per lineage dataset
- EPR (early precision ratio) value per lineage dataset
- auprc_epr_benchmark.csv comparing CEFCON vs SCINET, NetREX, CellOracle, GRNBoost2, DeepSEM, Random_NicheNet

## How to apply

Download the seven BEELINE lineage scRNA-seq datasets, ChIP-seq prior interaction networks, and ground-truth networks from Zenodo record 7564872, and load each lineage via cf.data_preparation(adata, prior_network). Train the CEFCON graph attention network with cf.NetModel(epochs=350, repeats=3, cuda='0') built on PyTorch and torch-geometric, then extract the inferred GRN with get_cefcon_results(edge_threshold_avgDegree=8) and save it as cell_lineage_GRN.csv. For each lineage, compute AUPRC and EPR of the inferred network against the corresponding ChIP-seq ground-truth network. Assemble all metrics into auprc_epr_benchmark.csv and compare CEFCON's AUPRC/EPR values with the published results of the six baseline methods. Correct application is judged by CEFCON reproducing superior AUPRC on all benchmark datasets (reported ~64% improvement over DeepSEM, ~92% over NetREX) and best or tied-best EPR.

## Related tools

- **CEFCON** (Infers the cell-lineage-specific GRN under evaluation; provides data_preparation, NetModel and get_cefcon_results APIs used in the benchmark workflow) — https://github.com/WPZgithub/CEFCON
- **Python** (Implementation language for running CEFCON and computing AUPRC/EPR metrics against ground-truth networks)
- **PyTorch** (Deep learning backend (>=1.13.0,<2.0) for training the CEFCON graph attention network (epochs=350, repeats=3))
- **torch-geometric** (Graph neural network library (>=2.1.0) underlying CEFCON's graph attention model on prior networks)
- **scanpy** (Loads and preprocesses the BEELINE scRNA-seq adata objects passed to cf.data_preparation (>=1.8.2))
- **networkx** (Handles prior/ground-truth graph structures (>=3.0) during data preparation and metric computation)

## Examples

```
cefcon_GRN_model = cf.NetModel(epochs=350, repeats=3, cuda='0'); cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)
```

## Evaluation signals

- CEFCON achieves superior AUPRC on all five BEELINE benchmark GRN construction datasets, improving ~64% over the second-best method (DeepSEM) and ~92% over NetREX
- CEFCON achieves best or tied-best EPR on the tested datasets
- Each inferred GRN is exported as cell_lineage_GRN.csv with edges thresholded via edge_threshold_avgDegree=8 before metric computation
- All per-lineage AUPRC/EPR values assemble into a complete auprc_epr_benchmark.csv covering CEFCON and the six baselines
- Input lineage data loads cleanly via cf.data_preparation(adata, prior_network) with matching gene identifiers between adata, prior network, and ground truth

## Limitations

- ChIP-seq ground-truth networks are lacking for many specific cell types or lineages, so AUPRC/EPR validation is only possible on benchmarked lineages
- CEFCON handles one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously, constraining benchmark scope
- Results depend on edge thresholding (edge_threshold_avgDegree=8) of the inferred network, which affects AUPRC/EPR values

## Evidence

- [results] CEFCON achieved superior AUPRC performance on all benchmark GRN construction datasets, improving 64% over the second-best method (DeepSEM) and 92% over NetREX, and achieved the best or tied-best EPR on the tested datasets.: [evidence span withheld — anchor status: not_found]
- [methods] data preparation snippet: "data = cf.data_preparation(adata, prior_network)"
- [methods] thresholded result extraction: "cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)"
- [methods] zenodo data availability: "All the input data mentioned in the paper can be downloaded from [here](https://zenodo.org/record/7564872)"
- [discussion] ground-truth scarcity caveat: "the label datasets serving as ground-truths are often lacking, especially for specific cell types or lineages"
