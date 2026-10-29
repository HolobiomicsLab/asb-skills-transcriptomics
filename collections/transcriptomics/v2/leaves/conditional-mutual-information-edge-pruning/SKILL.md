---
name: conditional-mutual-information-edge-pruning
description: Use when a merged bagged GRN (e.g., consensus of 100 subsample networks
  at a 20% consensus threshold) contains indirect or spurious edges that must be pruned
  before downstream module analysis or disease subnetwork modeling.
when_to_use_negative:
- Input is a raw count matrix that has not yet been through supercell construction
  and bagged GBR-based network inference — pruning presupposes an already-built merged
  network
- The goal is to leverage spatial information explicitly during network construction
  — SCING does not currently utilize spatial information of spatial transcriptomics
  during GRN construction
- Strict causal/directional regulatory claims are required — pruned SCING links are
  not necessarily causal or directional
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: SCING
  role: Provides the BuildNetwork module that merges bagged subsample GRNs and prunes
    edges and cycles using conditional mutual information and edge importance scoring
  repo: https://github.com/XiaYangLabOrg/SCING
- name: conda
  role: Environment manager used to create the scing environment (scing.environment.yml)
    in which the pruning step runs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:21486138603686419ff1c448f4dd9f4d7b66a5320bc62ba917df7fe250110e09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - SCING
  - conda
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# conditional-mutual-information-edge-pruning

## Summary

A SCING edge-filtering step that prunes edges and cycles from a merged, bagged consensus GRN using conditional mutual information and edge importance scoring, removing indirect associations to yield a directed, interpretable gene regulatory network edge list.

## When to use

Use when a merged bagged GRN (e.g., consensus of 100 subsample networks at a 20% consensus threshold) contains indirect or spurious edges that must be pruned before downstream module analysis or disease subnetwork modeling.

## When NOT to use

- Input is a raw count matrix that has not yet been through supercell construction and bagged GBR-based network inference — pruning presupposes an already-built merged network
- The goal is to leverage spatial information explicitly during network construction — SCING does not currently utilize spatial information of spatial transcriptomics during GRN construction
- Strict causal/directional regulatory claims are required — pruned SCING links are not necessarily causal or directional

## Inputs

- Merged consensus GRN from ~100 bagged subsample networks built on supercells (edges retained at >=20% consensus)
- Supercell file produced by SCING BuildNetwork from a snRNA-seq count matrix (e.g., human prefrontal cortex microglia)
- Python environment with pyitlib installed for conditional mutual information computation

## Outputs

- Directed GRN edge list (final_grn_edge_list.csv) with pruned edges and cycles removed (e.g., 10,159 genes, 63,056 edges for the microglia network)
- Pruned network suitable for leiden graph partitioning into modules and AUCell module scoring

## How to apply

After building ~100 GRNs from subsamples of supercells (aggregated transcriptionally similar cells) with gradient boosting regressor edge inference, merge the networks keeping only edges present in at least 20% of subsample networks. Then apply conditional mutual information-based pruning with edge importance scoring to remove indirect associations and cycles from the merged network. The rationale is that gradient boosting alone captures co-expression-like dependencies, while conditional mutual information tests whether a regulatory edge carries information beyond already-connected paths, retaining direct regulatory relationships. The final output is a directed edge list (e.g., the microglia network with 10,159 genes and 63,056 edges) written as final_grn_edge_list.csv. Note that in the SCING workflow (BuildNetwork.ipynb) this pruning is fused with the merge step, but for cluster parallelizability it can be split into a separate third script that reads the supercells and the 100 networks, merges, and prunes.

## Related tools

- **SCING** (Provides the BuildNetwork module that merges bagged subsample GRNs and prunes edges and cycles using conditional mutual information and edge importance scoring) — https://github.com/XiaYangLabOrg/SCING
- **conda** (Environment manager used to create the scing environment (scing.environment.yml) in which the pruning step runs)

## Examples

```
conda env create -n scing --file install/scing.environment.yml; conda activate scing; pip install pyitlib; pip install -e .
```

## Evaluation signals

- The pruned output final_grn_edge_list.csv exists, parses as a directed edge table (regulator-target pairs)
- Gene count and edge count of the pruned network match reference values reported for the dataset (10,159 genes and 63,056 edges for the microglia network)
- No edges below the 20% consensus threshold across the 100 bagged subsample networks survive into the merged input to pruning
- Downstream validity: the pruned network supports leiden graph partitioning into subnetworks and AUCell module scoring without dangling genes
- Hyperparameter robustness: pruning behavior should be stable across a range of hyperparameter settings, per the paper's robustness analysis

## Limitations

- Inferred GRN links after pruning are not necessarily causal or directional despite being represented as a directed edge list
- Many hyperparameters require selection, and while the authors show robustness across a range of hyperparameters, selection can be dataset dependent
- Additional omics information (scATAC-seq, scHi-C, ChIP-seq, or cis/trans eQTL) is not incorporated, which could otherwise improve GRN accuracy after pruning

## Evidence

- [readme] We then prune edges and cycles using conditional mutual information: "We find the consensus as edges that show up in 20% of the networks. We then prune edges and cycles using conditional mutual information"
- [methods] Merge GRNs to a final GRN and prune edges: "BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges"
- [intro] Bagging-based GRN consensus and edge directing/pruning via edge importance and conditional mutual information: [evidence span withheld — anchor status: not_found]
- [discussion] the links between genes are not necessarily causal or directional: "the links between genes are not necessarily causal or directional"
