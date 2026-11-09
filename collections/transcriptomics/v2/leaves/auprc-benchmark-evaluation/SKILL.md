---
name: auprc-benchmark-evaluation
description: Use when benchmarking the predictive accuracy or robustness of inferred
  cell-lineage-specific GRNs against ground-truth regulatory edges, e.g., testing
  AUPRC stability after randomly perturbing the prior network's edges.
when_to_use_negative:
- No ground-truth regulatory edge set exists for the cell type or lineage — the article
  notes that label datasets serving as ground-truths are often lacking, so AUPRC cannot
  be computed.
- The goal is driver regulator identification or regulon-like gene module (RGM) activity
  scoring, not GRN prediction-accuracy benchmarking.
- The input is already a computed AUPRC results table (e.g., an existing perturbed_auprc_results.csv)
  that only needs comparison or plotting rather than re-inference.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CEFCON
  role: Trains the graph attention network (cf.NetModel, epochs=350, repeats=3) whose
    inferred GRNs are scored by AUPRC under both the original and edge-shuffled prior
    networks
  repo: https://github.com/WPZgithub/CEFCON
- name: networkx
  role: Loads the prior gene interaction network edgelist and randomly shuffles 20%
    of its edges while keeping the same node set, with a fixed random seed
- name: PRROC
  role: R package used to compute the area under the precision-recall curve (AUPRC)
    of inferred GRN edges against ground-truth regulatory edges
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:d93e9bcf5bbb6f6ce483c3a49b73c089d9a9bce0ad5690658cb5e8fbebded2f8
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# auprc-benchmark-evaluation

## Summary

Scores CEFCON-inferred cell-lineage-specific GRNs against ground-truth regulatory edges using the area under the precision-recall curve (AUPRC), computed with the PRROC R package, and tests robustness by re-scoring GRNs built from a prior gene interaction network with 20% of its edges randomly shuffled (reported ~10% average AUPRC drop).

## When to use

Use when benchmarking the predictive accuracy or robustness of inferred cell-lineage-specific GRNs against ground-truth regulatory edges, e.g., testing AUPRC stability after randomly perturbing the prior network's edges.

## When NOT to use

- No ground-truth regulatory edge set exists for the cell type or lineage — the article notes that label datasets serving as ground-truths are often lacking, so AUPRC cannot be computed.
- The goal is driver regulator identification or regulon-like gene module (RGM) activity scoring, not GRN prediction-accuracy benchmarking.
- The input is already a computed AUPRC results table (e.g., an existing perturbed_auprc_results.csv) that only needs comparison or plotting rather than re-inference.

## Inputs

- Prior gene interaction network edgelist (human or mouse) from Zenodo record 7564872
- Benchmark scRNA-seq data (e.g., hESC dataset with 1,000 highly variable genes)
- Ground-truth regulatory edge sets for each benchmark dataset

## Outputs

- perturbed_auprc_results.csv with AUPRC columns for each dataset
- Percentage AUPRC drop per dataset relative to the original-network baseline
- Average AUPRC drop across benchmark datasets (~10% expected)

## How to apply

Download the benchmark inputs (e.g., the hESC dataset with 1,000 highly variable genes and the human/mouse prior gene interaction network edgelists) from Zenodo record 7564872. Load the prior network with networkx and randomly shuffle 20% of its edges while keeping the same node set, using a fixed random seed for reproducibility. Prepare the scRNA-seq data and perturbed prior via cf.data_preparation(adata, prior_network), then re-run GRN construction per cell lineage with cf.NetModel(epochs=350, repeats=3, cuda='0') under PyTorch (>=1.13.0,<2.0) and torch-geometric (>=2.1.0) on GPU. Retrieve inferred GRNs with get_cefcon_results(edge_threshold_avgDegree=8) and compute AUPRC against the ground-truth regulatory edges using the PRROC R package. For each benchmark dataset, compute the percentage AUPRC drop of the edge-shuffled run relative to the original-network baseline and average across datasets.

## Related tools

- **CEFCON** (Trains the graph attention network (cf.NetModel, epochs=350, repeats=3) whose inferred GRNs are scored by AUPRC under both the original and edge-shuffled prior networks) — https://github.com/WPZgithub/CEFCON
- **networkx** (Loads the prior gene interaction network edgelist and randomly shuffles 20% of its edges while keeping the same node set, with a fixed random seed)
- **PRROC** (R package used to compute the area under the precision-recall curve (AUPRC) of inferred GRN edges against ground-truth regulatory edges)

## Examples

```
cefcon_GRN_model = cf.NetModel(epochs=350, repeats=3, cuda='0'); cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)
```

## Evaluation signals

- perturbed_auprc_results.csv exists and contains AUPRC columns for each benchmark dataset
- Average AUPRC drop across benchmark datasets is approximately 10%, consistent with the reported robustness result in Supplementary Fig. 3
- Edge shuffling preserves the same node set as the original prior network and uses a fixed random seed so runs are reproducible
- GRN inference parameters match the baseline (epochs=350, repeats=3, cuda='0') and GRN edges are thresholded identically (edge_threshold_avgDegree=8) before AUPRC computation
- AUPRC is computed with the PRROC R package against the same ground-truth regulatory edges used for the baseline evaluation

## Limitations

- CEFCON can only handle one cell lineage at a time, so robustness must be evaluated separately for each lineage and cannot cover multiple fate trajectories with bifurcations simultaneously.
- Ground-truth label datasets for supervised evaluation are often lacking, especially for specific cell types or lineages.
- The reported ~10% average AUPRC drop depends on the specific settings used (20% edge shuffle, edge_threshold_avgDegree=8, fixed random seed); other perturbation rates or thresholds may yield different results.

## Evidence

- [methods] GRN inference parameters: "cefcon_GRN_model = cf.NetModel(epochs=350, repeats=3, cuda='0')"
- [methods] GRN retrieval threshold: "cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)"
- [other] AUPRC computation against ground truth: [evidence span withheld — anchor status: not_found]
- [other] Expected robustness result: [evidence span withheld — anchor status: not_found]
- [readme] GRN construction context: "CEFCON initially employs the graph attention neural networks under a contrastive learning framework to construct reliable GRNs"
