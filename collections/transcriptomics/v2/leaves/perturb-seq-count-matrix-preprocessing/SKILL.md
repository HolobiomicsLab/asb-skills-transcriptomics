---
name: perturb-seq-count-matrix-preprocessing
description: Use when raw Perturb-seq count matrices (e.g., GEO accessions GSE90063
  or GSE153056) must be converted into preprocessed inputs for GRN inference, or when
  benchmarking methods like SCING, GRNBOOST2, PIDC, or ppcor on perturbation-target
  prediction.
when_to_use_negative:
- The input is already preprocessed for GRN inference (e.g., a saved SCING supercell
  object) — rebuilding supercells would duplicate the leiden-clustering step
- The analysis goal is perturbation-target prediction scoring (AUROC / TPR at FPR
  0.05) rather than data preparation — that is a separate downstream evaluation step
- The dataset is not a Perturb-seq / scRNA-seq count matrix (e.g., spatial transcriptomics
  visium data uses a different SCING workflow branch)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCING
  role: Primary GRN inference pipeline that consumes the preprocessed Perturb-seq
    matrices; its BuildNetwork.ipynb builds supercells and the final pruned GRN
  repo: https://github.com/XiaYangLabOrg/SCING
- name: GRNBOOST2
  role: Baseline gradient boosting GRN inference method run on the identically preprocessed
    Perturb-seq matrices for benchmarking
- name: PIDC
  role: Baseline mutual-information GRN inference method run on the preprocessed matrices
    for benchmarking
- name: ppcor
  role: Baseline partial correlation GRN inference method run on the preprocessed
    matrices for benchmarking
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:3edc503d330fd9b2ac1de24e508ebe14cb9b42e18f29a3402364a96bd1a0cf34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - GRNBOOST2
  - PIDC
  - SCING
  - ppcor
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# perturb-seq-count-matrix-preprocessing

## Summary

Download and preprocess Perturb-seq count matrices (e.g., Dixit et al. GSE90063 and Papalexi et al. GSE153056 from GEO) into a form suitable for gene regulatory network (GRN) inference, including supercell construction as used by SCING. This makes raw perturbation-screen data usable for benchmarking GRN methods against downstream perturbation-target prediction.

## When to use

Use when raw Perturb-seq count matrices (e.g., GEO accessions GSE90063 or GSE153056) must be converted into preprocessed inputs for GRN inference, or when benchmarking methods like SCING, GRNBOOST2, PIDC, or ppcor on perturbation-target prediction.

## When NOT to use

- The input is already preprocessed for GRN inference (e.g., a saved SCING supercell object) — rebuilding supercells would duplicate the leiden-clustering step
- The analysis goal is perturbation-target prediction scoring (AUROC / TPR at FPR 0.05) rather than data preparation — that is a separate downstream evaluation step
- The dataset is not a Perturb-seq / scRNA-seq count matrix (e.g., spatial transcriptomics visium data uses a different SCING workflow branch)

## Inputs

- Perturb-seq count matrices from GEO (Dixit et al., GSE90063; Papalexi et al., GSE153056)
- Guide/gene perturbation annotations for downstream perturbed-gene labels
- Cell-level metadata for cell-type selection or filtering

## Outputs

- Preprocessed expression matrix ready for GRN inference
- SCING supercell object (saved file for reuse in the 100-network bagging step)
- identically preprocessed inputs for GRNBOOST2, ppcor, and PIDC baselines

## How to apply

Download the Perturb-seq count matrices from GEO (Dixit et al., GSE90063; Papalexi et al., GSE153056) and preprocess them for GRN inference as described in the SCING workflow. For SCING specifically, select a cell type or use the whole dataset, cluster cells using the leiden graph partitioning algorithm, and merge subclusters into supercells to reduce sparsity before network building. Save the supercell object so it can be reused: run 100 GRNs in parallel on subsamples of the supercells (bagging to keep robust edges, consensus at edges appearing in 20% of networks), then merge and prune edges using conditional mutual information. Decision point: sparsity adjustment matters — the benchmark finding is that SCING outperforms all other GRN inference approaches when sparsity is adjusted, and outperforms select methods when all cells are used, so record which cell-aggregation setting was applied. The same preprocessed matrices should be fed identically to baseline methods (GRNBOOST2, ppcor, PIDC) so downstream AUROC and TPR-at-FPR-0.05 comparisons are fair.

## Related tools

- **SCING** (Primary GRN inference pipeline that consumes the preprocessed Perturb-seq matrices; its BuildNetwork.ipynb builds supercells and the final pruned GRN) — https://github.com/XiaYangLabOrg/SCING
- **GRNBOOST2** (Baseline gradient boosting GRN inference method run on the identically preprocessed Perturb-seq matrices for benchmarking)
- **PIDC** (Baseline mutual-information GRN inference method run on the preprocessed matrices for benchmarking)
- **ppcor** (Baseline partial correlation GRN inference method run on the preprocessed matrices for benchmarking)

## Examples

```
git clone https://github.com/XiaYangLabOrg/SCING.git; cd SCING; conda env create -n scing --file install/scing.environment.yml; conda activate scing; pip install pyitlib; pip install -e .
```

## Evaluation signals

- The supercell object is saved and loadable so the three-step parallel workflow (build supercells → 100 parallel GRNs → merge) can consume it
- Each of the four methods (SCING, GRNBOOST2, PIDC, ppcor) receives the same preprocessed gene-by-cell matrix per dataset
- The final benchmark table (perturbseq_benchmark_results.csv) contains one AUROC and one TPR-at-FPR-0.05 entry per method per dataset, with SCING directed evaluation reported separately
- Sparsity-adjustment setting (supercells vs. all cells) is explicitly recorded, since performance conclusions differ between the two regimes

## Limitations

- SCING has many hyperparameters that require selection, and while the authors show robustness across a range, selection can be dataset dependent and optimization is needed per dataset
- Inferred GRN links from the preprocessed data are not necessarily causal or directional
- Edge directing/pruning via conditional mutual information prunes edges and cycles, so the preprocessed-to-GRN output is a consensus network, not a fully directed causal model

## Evidence

- [other] Source card workflow step: [evidence span withheld — anchor status: not_found]
- [readme] README supercell construction: "We then cluster the cells using the leiden graph partitioning algorithm and merge subclusters into supercells. We utilize bagging through subsamples of supercells to keep robust edges in the final"
- [readme] README consensus threshold: "We find the consensus as edges that show up in 20% of the networks. We then prune edges and cycles using conditional mutual information"
- [discussion] Article hyperparameter caveat: "there are many hyperparameters that require selection, and while we show robustness across a range of hyperparameters, selection can be dataset dependent"
- [intro] Article benchmark objective: "We evaluate our approach against GRNBOOST2, ppcor, and PIDC through perturbation target prediction in perturb-seq data, goodness of fit, network characteristic metrics, and disease modeling accuracy."
