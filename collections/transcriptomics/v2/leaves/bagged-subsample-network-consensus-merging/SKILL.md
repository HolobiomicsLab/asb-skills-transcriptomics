---
name: bagged-subsample-network-consensus-merging
description: Use when a robust GRN is needed from scRNA-seq/snRNA-seq data and many
  subsample networks have been built from supercells, so edges recurring across networks
  must be merged into a consensus before edge pruning.
when_to_use_negative:
- The analysis requires explicitly leveraging spatial coordinates during network construction
  — SCING does not utilize spatial information when building GRNs from spatial transcriptomics
- The goal is causal or strictly directional regulatory claims — inferred GRN links
  are not necessarily causal or directional
- The input is bulk transcriptomics or a pre-computed network rather than single-cell/snRNA-seq
  data from which supercells and subsample networks can be built
tools:
- name: SCING
  role: Provides BuildNetwork module for supercell construction, gradient boosting
    regressor edge inference on subsamples, network merging, and conditional mutual
    information edge pruning
  repo: github.com/XiaYangLabOrg/SCING
- name: conda
  role: Environment manager used to install the SCING environment (scing.environment.yml)
    that runs the supercell build, parallel GRN construction, and merge scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:cccd88f1ffb5a137e544ed8f47a979b23b2438d6aa478bbbfbea9dfe9077abe2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCING
  - conda
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bagged-subsample-network-consensus-merging

## Summary

A bagging-based consensus step of the SCING pipeline in which many GRNs (100 in the reference workflow) are built in parallel from subsamples of supercell-aggregated single-cell data and merged so that only edges recurring across subsample networks (20% consensus threshold) survive into the final network, which is then pruned with conditional mutual information.

## When to use

Use when a robust GRN is needed from scRNA-seq/snRNA-seq data and many subsample networks have been built from supercells, so edges recurring across networks must be merged into a consensus before edge pruning.

## When NOT to use

- The analysis requires explicitly leveraging spatial coordinates during network construction — SCING does not utilize spatial information when building GRNs from spatial transcriptomics
- The goal is causal or strictly directional regulatory claims — inferred GRN links are not necessarily causal or directional
- The input is bulk transcriptomics or a pre-computed network rather than single-cell/snRNA-seq data from which supercells and subsample networks can be built

## Inputs

- Preprocessed single-cell/single-nucleus count matrix for a selected cell type (e.g., human prefrontal cortex microglia snRNA-seq)
- Saved supercell file produced by SCING's BuildNetwork module
- 100 GRN networks built in parallel from subsamples of the supercell data

## Outputs

- Merged consensus network containing only edges present in >=20% of the subsample networks
- Pruned directed GRN edge list (final_grn_edge_list.csv), e.g., the microglia reference network with 10,159 genes and 63,056 edges

## How to apply

First build supercells by clustering cells with the leiden graph partitioning algorithm and merging transcriptionally similar cells into aggregated supercells, saving the supercell file for reuse (SCING's BuildNetwork module). Then build 100 GRNs in parallel from subsamples of the supercell data using SCING's gradient boosting regressor (GBR) edge inference, with genes clustered in each subsample to restrict the search to likely regulatory edges. Merge the 100 bagged networks into a single network, retaining as the consensus only edges that appear in at least 20% of the subsample networks. Finally, prune edges and cycles from the merged network using conditional mutual information and edge importance scoring to remove indirect associations, yielding a directed edge list (e.g., final_grn_edge_list.csv). Rationale: bagging through subsamples of supercells keeps robust edges and suppresses noise-driven edges that appear only in individual subsamples.

## Related tools

- **SCING** (Provides BuildNetwork module for supercell construction, gradient boosting regressor edge inference on subsamples, network merging, and conditional mutual information edge pruning) — github.com/XiaYangLabOrg/SCING
- **conda** (Environment manager used to install the SCING environment (scing.environment.yml) that runs the supercell build, parallel GRN construction, and merge scripts)

## Examples

```
conda env create -n scing --file install/scing.environment.yml; conda activate scing; pip install pyitlib; pip install -e .
```

## Evaluation signals

- The merge retains only edges that appear in >=20% of the 100 subsample networks (the 20% consensus threshold).
- Exactly 100 parallel subsample GRNs are built from the supercell data before merging, matching the reported bagging design.
- The final output is a directed edge list (final_grn_edge_list.csv) that parses as a directed edge table.
- For the Morabito microglia snRNA-seq reference, gene and edge counts match the reported values of 10,159 genes and 63,056 edges.
- Post-merge pruning with conditional mutual information removes indirect associations and cycles before the edge list is written.

## Limitations

- Many hyperparameters require selection, and while SCING shows robustness across a range of hyperparameters, selection can be dataset dependent.
- Inferred GRN links are not necessarily causal or directional.
- For spatial transcriptomics, spatial information is not explicitly utilized during network construction, which SCING authors identify as an important future direction.

## Evidence

- [readme] We utilize bagging through subsamples of supercells to keep robust edges in the final GRN.: "We utilize bagging through subsamples of supercells to keep robust edges in the final GRN."
- [readme] We find the consensus as edges that show up in 20% of the networks.: "We find the consensus as edges that show up in 20% of the networks."
- [methods] BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges.: "BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges."
- [intro] Here, we present Single Cell INtegrative gene regulatory network inference (SCING), a gradient boosting based approach to efficiently identify GRNs in full single cell transcriptomes.: "Here, we present Single Cell INtegrative gene regulatory network inference (SCING), a gradient boosting based approach to efficiently identify GRNs in full single cell transcriptomes."
- [discussion] there are many hyperparameters that require selection, and while we show robustness across a range of hyperparameters, selection can be dataset dependent: "there are many hyperparameters that require selection, and while we show robustness across a range of hyperparameters, selection can be dataset dependent and optimization of hyperparameters is"
