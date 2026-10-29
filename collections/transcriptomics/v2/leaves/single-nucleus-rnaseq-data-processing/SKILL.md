---
name: single-nucleus-rnaseq-data-processing
description: Use when you have a snRNA-seq (or scRNA-seq/spatial transcriptomics)
  count matrix for a specific cell type — e.g. human prefrontal cortex microglia —
  and need a directed GRN covering the full transcriptome rather than a small pre-selected
  gene set.
when_to_use_negative:
- You need causal, directional regulatory mechanisms — the article's discussion notes
  'the links between genes are not necessarily causal or directional'.
- You want spatial information explicitly used in network construction — SCING 'does
  not currently utilize the spatial information during network construction' even
  for spatial transcriptomics inputs.
- Your dataset is small or you only have a pre-selected set of a few thousand genes
  and cells; SCING is designed for full transcriptomes of large datasets, where its
  supercell and bagging strategy pays off.
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: SCING
  role: 'Primary pipeline: supercell construction (BuildNetwork), GBR-based edge inference
    on 100 subsamples, network merging, and conditional mutual information edge pruning'
  repo: https://github.com/XiaYangLabOrg/SCING
- name: conda
  role: 'Environment provisioning: conda env create -n scing --file install/scing.environment.yml'
- name: pyscenic
  role: Optional AUCell implementation for graph-based dimensionality reduction on
    GRN modules
- name: enrichr
  role: Optional R-based pathway enrichment analysis of genes in each leiden-derived
    GRN subnetwork
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:9e7e776c2d69aab9917274db7a28b10506e4a8984e04b3d61fb2b02e7f1d0f9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - SCING
  - conda
  - enrichr
  - pyscenic
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-nucleus-rnaseq-data-processing

## Summary

Process single-nucleus RNA-seq (snRNA-seq) count matrices into robust gene regulatory networks using SCING, which combines supercell construction, bagging of 100 subsample GRNs via gradient boosting regressors, and conditional mutual information edge pruning to handle sparsity and cell-to-cell heterogeneity.

## When to use

Use when you have a snRNA-seq (or scRNA-seq/spatial transcriptomics) count matrix for a specific cell type — e.g. human prefrontal cortex microglia — and need a directed GRN covering the full transcriptome rather than a small pre-selected gene set.

## When NOT to use

- You need causal, directional regulatory mechanisms — the article's discussion notes 'the links between genes are not necessarily causal or directional'.
- You want spatial information explicitly used in network construction — SCING 'does not currently utilize the spatial information during network construction' even for spatial transcriptomics inputs.
- Your dataset is small or you only have a pre-selected set of a few thousand genes and cells; SCING is designed for full transcriptomes of large datasets, where its supercell and bagging strategy pays off.

## Inputs

- snRNA-seq count matrix for a selected cell type (e.g. human prefrontal cortex microglia from AD and Control patients)
- Cell type annotation or selection of cells/visium spots to include
- SCING environment (conda env 'scing' from install/scing.environment.yml, plus pyitlib and pip install -e .)

## Outputs

- Supercell file (aggregated transcriptionally similar cells, saved for reuse)
- 100 bagged subsample GRNs from gradient boosting regressor inference
- Merged consensus GRN (edges present in >=20% of subsample networks)
- Directed pruned GRN edge list, e.g. final_grn_edge_list.csv (microglia reference: 10,159 genes, 63,056 edges)
- Optional leiden subnetworks/modules with AUCell module scores per cell

## How to apply

Load the snRNA-seq count matrix (e.g. human prefrontal cortex microglia from AD and Control patients) and construct supercells by clustering cells with the leiden graph partitioning algorithm and merging transcriptionally similar subclusters, saving the supercell file for reuse. Build 100 GRNs in parallel from subsamples of the supercells: for each subsample, cluster genes to restrict inference to likely regulatory edges, then fit a gradient boosting regressor (GBR) per gene to infer edges. Merge the 100 bagged networks into a consensus GRN, retaining only edges that appear in at least 20% of the subsample networks to keep robust edges. Prune the merged network's edges and cycles using conditional mutual information and edge importance scoring to remove indirect associations, yielding a directed edge list (e.g. final_grn_edge_list.csv with 10,159 genes and 63,056 edges for microglia). Optionally partition the GRN into subnetworks with leiden partitioning and combine with AUCell to compute module-specific expression per cell for clustering or pathway enrichment with enrichr.

## Related tools

- **SCING** (Primary pipeline: supercell construction (BuildNetwork), GBR-based edge inference on 100 subsamples, network merging, and conditional mutual information edge pruning) — https://github.com/XiaYangLabOrg/SCING
- **conda** (Environment provisioning: conda env create -n scing --file install/scing.environment.yml)
- **pyscenic** (Optional AUCell implementation for graph-based dimensionality reduction on GRN modules)
- **enrichr** (Optional R-based pathway enrichment analysis of genes in each leiden-derived GRN subnetwork)

## Examples

```
git clone https://github.com/XiaYangLabOrg/SCING.git && cd SCING && conda env create -n scing --file install/scing.environment.yml && conda activate scing && pip install pyitlib && pip install -e .
```

## Evaluation signals

- final_grn_edge_list.csv exists and parses as a directed edge table (regulator -> target columns).
- Gene count and edge count match the reported reference: 10,159 genes and 63,056 edges for the Morabito microglia snRNA-seq network.
- Every retained consensus edge appears in at least 20% of the 100 subsample networks (bagging threshold respected).
- Supercell file was saved before parallel GRN construction, and the 100 networks were built from that file rather than raw cells.
- Post-pruning, indirect associations are reduced — edge pruning used conditional mutual information (pyitlib) rather than raw GBR importance alone.

## Limitations

- Inferred GRN links are not necessarily causal or directional in a mechanistic sense.
- Many hyperparameters require selection, and while robust across a range, selection can be dataset dependent.
- Spatial information of spatial transcriptomics data is not explicitly utilized during GRN construction, despite resulting GRNs showing spatial patterns; leveraging it is deferred to future work.
- Additional omics modalities (scATAC-seq, scHi-C, ChIP-seq, cis/trans eQTL) are not yet incorporated, which could improve GRN accuracy.

## Evidence

- [readme] First, we select a specific cell type, or use whole visium data. We then cluster the cells using the leiden graph partitioning algorithm and merge subclusters into supercells.: "First, we select a specific cell type, or use whole visium data. We then cluster the cells using the leiden graph partitioning algorithm and merge subclusters into supercells."
- [readme] We find the consensus as edges that show up in 20% of the networks. We then prune edges and cycles using conditional mutual information: "We find the consensus as edges that show up in 20% of the networks. We then prune edges and cycles using conditional mutual information"
- [methods] We apply SCING to human prefrontal cortex snRNAseq data with AD and Control patients: "We apply SCING to human prefrontal cortex snRNAseq data with AD and Control patients"
- [methods] conda env create -n scing --file install/scing.environment.yml: "conda env create -n scing --file install/scing.environment.yml"
- [readme] BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges.: "BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges."
