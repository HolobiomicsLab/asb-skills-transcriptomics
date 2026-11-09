---
name: gradient-boosting-grn-edge-inference
description: Use when building a genome-scale directed GRN from a full single-cell
  transcriptome count matrix (e.g., snRNA-seq of a specific cell type like microglia),
  where sparse expression and cell-to-cell heterogeneity make standard GRN tools computationally
  intractable or inaccurate.
when_to_use_negative:
- The goal requires proven causal directionality — the paper states the inferred links
  between genes are not necessarily causal or directional
- Input is spatial transcriptomics and the analysis requires explicit use of spatial
  coordinates during network construction — SCING does not utilize spatial information
  when building GRNs
- Only a small pre-selected gene panel (thousands of genes) is needed; simpler benchmarked
  methods like GRNBOOST2, GENIE3, ppcor, or PIDC may suffice
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SCING
  role: 'Primary pipeline: supercell building (BuildNetwork), parallel GBR-based GRN
    construction on subsamples, network merging, and conditional mutual information
    edge pruning'
  repo: https://github.com/XiaYangLabOrg/SCING
- name: conda
  role: 'Environment setup: conda env create -n scing --file install/scing.environment.yml'
  repo: https://github.com/XiaYangLabOrg/SCING
- name: GRNBOOST2
  role: Benchmark comparison method using a gradient boosting approach for GRN inference
- name: GENIE3
  role: Benchmark comparison method using random forest for GRN inference
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:c6d28084b56cc769ff2a476df4ba882b85c6ff441335541885269a7eeea4da65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - GENIE3
  - GRNBOOST2
  - SCING
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gradient-boosting-grn-edge-inference

## Summary

Infer a robust, directed gene regulatory network from single-cell (scRNA-seq/snRNA-seq) or spatial transcriptomics data using SCING, which combines supercell construction, gradient boosting regressor (GBR) edge inference on gene-clustered subsamples, bagging-based consensus (edges retained in ≥20% of subsample networks), and conditional mutual information pruning of edges and cycles.

## When to use

Use when building a genome-scale directed GRN from a full single-cell transcriptome count matrix (e.g., snRNA-seq of a specific cell type like microglia), where sparse expression and cell-to-cell heterogeneity make standard GRN tools computationally intractable or inaccurate.

## When NOT to use

- The goal requires proven causal directionality — the paper states the inferred links between genes are not necessarily causal or directional
- Input is spatial transcriptomics and the analysis requires explicit use of spatial coordinates during network construction — SCING does not utilize spatial information when building GRNs
- Only a small pre-selected gene panel (thousands of genes) is needed; simpler benchmarked methods like GRNBOOST2, GENIE3, ppcor, or PIDC may suffice

## Inputs

- Single-cell or single-nucleus count matrix (e.g., human prefrontal cortex snRNAseq microglia counts)
- Supercell file produced by SCING's BuildNetwork module (aggregated transcriptionally similar cells)
- Cell-type selection or whole visium spatial data as input scope

## Outputs

- 100 bagged subsample GRNs (intermediate networks)
- Merged consensus GRN (edges present in ≥20% of subsample networks)
- Pruned directed GRN edge list, e.g., final_grn_edge_list.csv (reference: 10,159 genes, 63,056 edges)

## How to apply

First construct supercells by clustering cells with the leiden graph partitioning algorithm and merging subclusters, saving the supercell file (BuildNetwork module). Then build 100 GRNs in parallel from subsamples of the supercells; for each subsample, genes are clustered to restrict the search to likely regulatory edges, and candidate edges are scored with gradient boosting regressors (GBR). Merge the 100 bagged networks into a consensus GRN, retaining only edges appearing in at least 20% of the subsample networks. Finally prune edges and cycles using conditional mutual information and edge importance scoring to remove indirect associations, producing a directed edge list (e.g., final_grn_edge_list.csv; the reference microglia network has 10,159 genes and 63,056 edges). Set up the environment via conda (scing.environment.yml) and install pyitlib plus the package with pip install -e .

## Related tools

- **SCING** (Primary pipeline: supercell building (BuildNetwork), parallel GBR-based GRN construction on subsamples, network merging, and conditional mutual information edge pruning) — https://github.com/XiaYangLabOrg/SCING
- **conda** (Environment setup: conda env create -n scing --file install/scing.environment.yml) — https://github.com/XiaYangLabOrg/SCING
- **GRNBOOST2** (Benchmark comparison method using a gradient boosting approach for GRN inference)
- **GENIE3** (Benchmark comparison method using random forest for GRN inference)

## Examples

```
git clone https://github.com/XiaYangLabOrg/SCING.git && conda env create -n scing --file install/scing.environment.yml && conda activate scing && pip install pyitlib && pip install -e .
```

## Evaluation signals

- final_grn_edge_list.csv exists and parses as a directed edge table
- Gene count and edge count match reference values (e.g., 10,159 genes and 63,056 edges for the microglia network)
- Retained consensus edges appear in at least 20% of the 100 subsample networks
- Edges and cycles are pruned via conditional mutual information; network should be acyclic after pruning
- Held-out validation: genes predicted from parents in a train/test split show high cosine similarity to true expression; perturb-seq AUROC/TPR-at-FPR-0.05 benchmarks outperform GRNBOOST2, ppcor, and PIDC

## Limitations

- Inferred GRN links are not necessarily causal or directional
- Many hyperparameters require selection and optimization can be dataset dependent, although the method is robust across a range of hyperparameters
- Spatial information in spatial transcriptomics is not explicitly utilized during GRN construction despite resulting GRNs showing spatial patterns

## Evidence

- [abstract] gradient boosting and mutual information-based approach for identifying robust GRNs: "we present Single Cell INtegrative Gene regulatory network inference (SCING), a gradient boosting and mutual information-based approach for identifying robust GRNs from scRNA-seq, snRNA-seq, and"
- [readme] consensus threshold and CMI pruning: "We find the consensus as edges that show up in 20% of the networks. We then prune edges and cycles using conditional mutual information"
- [readme] workflow decomposition into three scripts: "First, build supercells and save the file. Second, for 100 networks (in parallel) read in the supercell and build GRNs. Lastly, read in the supercells, and the 100 networks, and merge the networks."
- [readme] bagging and gene clustering rationale: "We utilize bagging through subsamples of supercells to keep robust edges in the final GRN. For each subsample, the genes are clustered to only operate on likely regulatory edges."
- [discussion] hyperparameter sensitivity: "there are many hyperparameters that require selection, and while we show robustness across a range of hyperparameters, selection can be dataset dependent"
- [methods] conda environment setup: "conda env create -n scing --file install/scing.environment.yml"
