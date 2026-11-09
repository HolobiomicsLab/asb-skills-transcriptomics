---
name: cross-cell-type-benchmarking-of-grn-methods
description: Use when multiple GRN inference methods must be compared on identical
  single-cell datasets and the goal is to rank them by biological validity, e.g. disease
  subnetwork modeling accuracy across tissues or cell types.
when_to_use_negative:
- When the goal is perturbation-target prediction accuracy — use the Perturb-seq benchmark
  instead of disease subnetwork modeling
- When comparing methods on datasets without disease gene set overlap — degree-preserved
  nulls and DisGeNET random walks require gene-set membership in the GRN
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3407
tools:
- name: SCING
  role: Candidate GRN method; builds supercells, 100 subsampled GRNs merged to a final
    pruned network per cell type
  repo: https://github.com/XiaYangLabOrg/SCING
- name: GRNBOOST2
  role: Benchmark GRN method using gradient boosting
- name: ppcor
  role: Benchmark GRN method using partial correlation
- name: PIDC
  role: Benchmark GRN method using conditional mutual information
- name: conda
  role: Environment setup for the SCING pipeline used to build the networks being
    benchmarked
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:5ad8a31eac418347f26bf72813f5f43f400b75d0de71e04535b362dfcf6be5ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - GRNBOOST2
  - PIDC
  - SCING
  - conda
  - ppcor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Cross-cell-type benchmarking of GRN methods

## Summary

Benchmarks SCING against GRNBOOST2, ppcor, and PIDC by building GRNs on the same scRNA-seq cells and scoring each network's ability to model DisGeNET disease subnetworks (Immune, Metabolic, Neuronal) via the Huang et al. random walk framework, reporting performance gain over degree-preserved random GRNs across mouse cell atlas tissues and cell types.

## When to use

Use when multiple GRN inference methods must be compared on identical single-cell datasets and the goal is to rank them by biological validity, e.g. disease subnetwork modeling accuracy across tissues or cell types.

## When NOT to use

- When the goal is perturbation-target prediction accuracy — use the Perturb-seq benchmark instead of disease subnetwork modeling
- When comparing methods on datasets without disease gene set overlap — degree-preserved nulls and DisGeNET random walks require gene-set membership in the GRN

## Inputs

- Mouse cell atlas scRNA-seq matrices (bone marrow, brain, liver, and full 33 tissues/106 cell types)
- DisGeNET disease gene sets: Immune, Metabolic, Neuronal categories
- GRNs inferred per method: SCING, GRNBOOST2, ppcor, PIDC

## Outputs

- Per-cell-type, per-disease-category random-walk performance-gain values (real GRN vs degree-preserved random GRNs)
- Aggregated cell-type-by-disease performance-gain matrix (disease_subnetwork_performance_gain.csv)

## How to apply

Build GRNs on the same cells with each method: SCING (via BuildNetwork.ipynb: supercells, 100 subsampled networks merged and pruned), GRNBOOST2 (gradient boosting), ppcor (partial correlation), and PIDC (conditional mutual information). For each inferred GRN, run the Huang et al. random walk framework against Immune, Metabolic, and Neuronal DisGeNET disease gene sets, generating degree-preserved random GRNs with similar node attributes as nulls and computing the real GRN's performance gain over these nulls. Apply SCING across the mouse cell atlas (33 tissues, 106 cell types, e.g. bone marrow, brain, liver) to produce a cell-type-by-disease performance-gain matrix, saved as disease_subnetwork_performance_gain.csv. The method with the largest gain over random networks best models disease subnetworks.

## Related tools

- **SCING** (Candidate GRN method; builds supercells, 100 subsampled GRNs merged to a final pruned network per cell type) — https://github.com/XiaYangLabOrg/SCING
- **GRNBOOST2** (Benchmark GRN method using gradient boosting)
- **ppcor** (Benchmark GRN method using partial correlation)
- **PIDC** (Benchmark GRN method using conditional mutual information)
- **conda** (Environment setup for the SCING pipeline used to build the networks being benchmarked)

## Examples

```
conda env create -n scing --file install/scing.environment.yml; conda activate scing; python BuildNetwork.ipynb
```

## Evaluation signals

- SCING achieves higher random-walk performance gain than GRNBOOST2, ppcor, and PIDC across Immune, Metabolic, and Neuronal disease categories in every tissue (bone marrow, brain, liver)
- All gains are computed against degree-preserved random GRNs with similar node attributes — gains should be positive if the GRN models the disease subnetwork better than chance
- The aggregated output is a complete cell-type-by-disease matrix over 33 tissues and 106 cell types saved as disease_subnetwork_performance_gain.csv
- Each benchmarked method runs on the identical cells/supercells used for SCING to ensure a fair comparison

## Limitations

- Random-walk performance gain measures disease subnetwork modeling, not causal directionality — inferred GRN links are not necessarily causal or directional
- SCING hyperparameters require selection and selection can be dataset dependent, which may affect cross-cell-type comparability
- Benchmarking via DisGeNET coverage depends on disease gene sets being represented in the GRN's gene universe

## Evidence

- [abstract] Performance evaluation using Perturb-seq datasets, held-out data, and the mouse cell atlas combined with the DisGeNET database demonstrates the improved accuracy and biological interpretability: "Performance evaluation using Perturb-seq datasets, held-out data, and the mouse cell atlas combined with the DisGeNET database demonstrates the improved accuracy and biological interpretability"
- [intro] We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics, and disease modeling accuracy.: "We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics, and disease modeling accuracy."
- [methods] We utilize a random walk framework from Huang et al. to determine the increase in performance of a GRN to model disease subnetworks versus random GRNs: "We utilize a random walk framework from Huang et al. to determine the increase in performance of a GRN to model disease subnetworks versus random GRNs"
- [readme] BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges.: "BuildNetwork.ipynb: Build Supercells, build many GRNs based on subsamples of the data, merge many GRNs to a final GRN plus prune edges."
