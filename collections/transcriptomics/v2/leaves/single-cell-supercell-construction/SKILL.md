---
name: single-cell-supercell-construction
description: Use when building a GRN from large single-cell transcriptomics data (e.g.
  snRNA-seq count matrices from a specific cell type like microglia or whole visium
  data) where full-transcriptome GRN inference is computationally intractable and
  per-cell sparsity must be reduced.
when_to_use_negative:
- The input is already aggregated (e.g. bulk RNA-seq or a precomputed pseudobulk table)
  and has no per-cell heterogeneity to consolidate
- Spatial transcriptomics data where the analysis goal requires explicitly using spatial
  coordinates during network construction — SCING does not utilize spatial information
  in GRN construction
- The goal is a GRN over a small pre-selected panel of a few thousand genes where
  direct per-cell inference is already tractable
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SCING
  role: Provides the BuildNetwork module and BuildNetwork.ipynb used to cluster cells
    with leiden graph partitioning and merge subclusters into supercells before GRN
    inference
  repo: https://github.com/XiaYangLabOrg/SCING
- name: conda
  role: Environment manager used to install the scing environment (install/scing.environment.yml)
    required to run supercell construction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:5351b886899ba22151cbf2eb7c34fd39e6a5165e28a085242fe672791cfa8c58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - SCING
  - conda
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-supercell-construction

## Summary

Aggregates transcriptionally similar single cells into supercells as the first stage of the SCING gene regulatory network inference pipeline, reducing sparsity and noise before GRN construction from snRNA-seq or spatial transcriptomics data.

## When to use

Use when building a GRN from large single-cell transcriptomics data (e.g. snRNA-seq count matrices from a specific cell type like microglia or whole visium data) where full-transcriptome GRN inference is computationally intractable and per-cell sparsity must be reduced.

## When NOT to use

- The input is already aggregated (e.g. bulk RNA-seq or a precomputed pseudobulk table) and has no per-cell heterogeneity to consolidate
- Spatial transcriptomics data where the analysis goal requires explicitly using spatial coordinates during network construction — SCING does not utilize spatial information in GRN construction
- The goal is a GRN over a small pre-selected panel of a few thousand genes where direct per-cell inference is already tractable

## Inputs

- Single-cell or single-nucleus count matrix (e.g. human prefrontal cortex snRNAseq microglia count matrix)
- Cell type selection or whole visium spatial transcriptomics data
- Leiden clustering resolution parameters

## Outputs

- Saved supercell file (aggregated expression per supercell) for reuse in parallel GRN construction

## How to apply

Select a specific cell type (e.g. human prefrontal cortex snRNAseq microglia) or use whole visium data, then cluster cells with the leiden graph partitioning algorithm and merge subclusters into supercells using SCING's BuildNetwork module. Save the resulting supercell file for reuse so downstream steps can break the pipeline into separate scripts. The rationale is that supercells reduce cell-to-cell sparsity and heterogeneity, making gradient boosting regressor-based edge inference feasible on full transcriptomes. Downstream, bagging over subsamples of the supercells (100 GRNs built in parallel, merging edges appearing in at least 20% of networks) keeps only robust edges in the final GRN.

## Related tools

- **SCING** (Provides the BuildNetwork module and BuildNetwork.ipynb used to cluster cells with leiden graph partitioning and merge subclusters into supercells before GRN inference) — https://github.com/XiaYangLabOrg/SCING
- **conda** (Environment manager used to install the scing environment (install/scing.environment.yml) required to run supercell construction)

## Examples

```
conda env create -n scing --file install/scing.environment.yml; conda activate scing; pip install pyitlib; pip install -e .; jupyter notebook tutorials/BuildNetwork.ipynb
```

## Evaluation signals

- The supercell file is saved and can be reloaded by the subsequent parallel GRN construction scripts (the README specifies saving the supercell file so steps can be broken into three separate scripts)
- Each supercell corresponds to a merge of leiden subclusters of transcriptionally similar cells of the selected cell type
- The 100 downstream GRNs built from subsamples of the supercells converge: merged edges retained at the 20% consensus threshold, and the final microglia network contains 10,159 genes and 63,056 edges
- The environment is correctly set up: conda env create -n scing succeeds, and pyitlib plus pip install -e . complete without error

## Limitations

- Hyperparameter selection (including leiden clustering granularity used to merge subclusters) can be dataset dependent; robustness is shown across a range but optimization is required
- SCING does not explicitly utilize spatial information from spatial transcriptomics during supercell/GRN construction, despite resulting GRNs showing spatial patterns
- Inferred GRN links derived downstream of supercells are not necessarily causal or directional in a mechanistic sense

## Evidence

- [readme] Supercell construction via leiden merge: "First, we select a specific cell type, or use whole visium data. We then cluster the cells using the leiden graph partitioning algorithm and merge subclusters into supercells."
- [readme] Bagging rationale over supercells: "We utilize bagging through subsamples of supercells to keep robust edges in the final GRN."
- [methods] Pipeline structure and script split: "BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges"
- [readme] Consensus threshold: "We find the consensus as edges that show up in 20% of the networks."
- [intro] Motivation for supercells on full transcriptomes: "Existing GRN tools utilize scRNA-seq data with thousands of pre-select genes and cells, because GRNs from full transcriptomes in large scRNA-seq datasets are often computationally intensive and"
