---
name: robustness-sensitivity-analysis-of-prior-knowledge-networks
description: Use when a GRN-inference or network-based method consumes a prior gene
  interaction network and you must quantify how much method performance degrades when
  that prior is partially randomized or noisy.
when_to_use_negative:
- The prior network is not required by the method, or method performance cannot be
  reduced to a scalar metric comparable across perturbed and unperturbed runs
- The goal is to test sensitivity to input scRNA-seq data or hyperparameters rather
  than to perturbations of the prior interaction network itself
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CEFCON
  role: GRN-inference method whose AUPRC robustness to prior-network edge shuffling
    is being quantified
  repo: https://github.com/WPZgithub/CEFCON
- name: networkx
  role: Loads the prior gene interaction network edgelist and performs random edge
    shuffling with a fixed seed
- name: PyTorch
  role: Backend for running cf.NetModel(epochs=350, repeats=3, cuda='0') on GPU with
    perturbed priors
- name: torch-geometric
  role: Graph neural network dependency for CEFCON's graph attention-based GRN construction
- name: PRROC
  role: R package used to compute AUPRC of inferred GRNs against ground-truth edges
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:104bebbd92489b289818b0133078c0c84181a0e0327fab232272cec6368e9ba2
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# robustness-sensitivity-analysis-of-prior-knowledge-networks

## Summary

Quantifies how sensitive a downstream method's performance (e.g., CEFCON's GRN AUPRC) is to perturbations of its input prior gene interaction network, by randomly shuffling a fixed fraction (e.g., 20%) of edges and comparing performance against the unperturbed baseline.

## When to use

Use when a GRN-inference or network-based method consumes a prior gene interaction network and you must quantify how much method performance degrades when that prior is partially randomized or noisy.

## When NOT to use

- The prior network is not required by the method, or method performance cannot be reduced to a scalar metric comparable across perturbed and unperturbed runs
- The goal is to test sensitivity to input scRNA-seq data or hyperparameters rather than to perturbations of the prior interaction network itself

## Inputs

- Prior gene interaction network edgelist (human/mouse) from Zenodo record 7564872
- Benchmark scRNA-seq dataset (e.g., hESC dataset with 1,000 highly variable genes)
- Ground-truth regulatory edge sets for AUPRC evaluation
- Perturbation specification (fraction of edges to shuffle, e.g., 20%, and random seed)

## Outputs

- perturbed_auprc_results.csv with AUPRC columns for each dataset
- Per-dataset and average percentage AUPRC drop versus the unperturbed baseline
- Robustness conclusion comparable to Supplementary Fig. 3

## How to apply

Download benchmark inputs (e.g., hESC dataset with 1,000 highly variable genes and human/mouse prior gene interaction networks) from Zenodo record 7564872. Load the prior network edgelist with networkx and randomly shuffle 20% of its edges while keeping the same node set, using a fixed random seed for reproducibility. Re-run the method on the perturbed prior — for CEFCON, instantiate cf.NetModel(epochs=350, repeats=3, cuda='0') with PyTorch/torch-geometric on GPU per cell lineage, then retrieve GRNs via get_cefcon_results(edge_threshold_avgDegree=8). Evaluate predicted edges against ground truth using AUPRC computed with the PRROC R package, and compute the percentage AUPRC drop per dataset relative to the original-network baseline, averaging across benchmark datasets. The rationale is that a small average drop (~10% reported in CEFCON's Supplementary Fig. 3) demonstrates that conclusions are not artifacts of prior-network inaccuracies.

## Related tools

- **CEFCON** (GRN-inference method whose AUPRC robustness to prior-network edge shuffling is being quantified) — https://github.com/WPZgithub/CEFCON
- **networkx** (Loads the prior gene interaction network edgelist and performs random edge shuffling with a fixed seed)
- **PyTorch** (Backend for running cf.NetModel(epochs=350, repeats=3, cuda='0') on GPU with perturbed priors)
- **torch-geometric** (Graph neural network dependency for CEFCON's graph attention-based GRN construction)
- **PRROC** (R package used to compute AUPRC of inferred GRNs against ground-truth edges)

## Examples

```
cefcon_GRN_model = cf.NetModel(epochs=350, repeats=3, cuda='0'); cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)
```

## Evaluation signals

- The average AUPRC drop across benchmark datasets after shuffling 20% of prior-network edges is approximately 10%, consistent with the reported result in Supplementary Fig. 3
- perturbed_auprc_results.csv exists and contains AUPRC columns for each benchmark dataset
- Perturbed and baseline runs use identical node sets and the same edge count (only edge endpoints are shuffled)
- Fixed random seeds make repeated shuffling and re-runs reproducible
- AUPRC for each run is computed against the same ground-truth edge set with the PRROC R package

## Limitations

- Only random edge shuffling is tested; biologically structured prior-network errors (e.g., false edges from specific databases) are not modeled
- CEFCON can only handle one cell lineage at a time, so robustness must be assessed per lineage rather than across multi-fate trajectories with bifurcations simultaneously
- Ground-truth label datasets for validating inferred edges are often lacking, especially for specific cell types or lineages

## Evidence

- [methods] All the input data mentioned in the paper can be downloaded from [here](https://zenodo.org/record/7564872): "All the input data mentioned in the paper can be downloaded from [here](https://zenodo.org/record/7564872)"
- [methods] NetModel invocation: "# Construct cell-lineage-specific GRN
cefcon_GRN_model = cf.NetModel(epochs=350, repeats=3, cuda='0')"
- [methods] edge threshold: "cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)"
- [readme] README environment: "CEFCON was originally tested on Ubuntu 20.04 with Python (3.9~3.10). We recommend running CEFCON on CUDA if possible."
- [discussion] genelink discussion: "the label datasets serving as ground-truths are often lacking, especially for specific cell types or lineages"
