---
name: supplementary-table-comparison
description: 'Use when validating a clustering benchmark from a scRNA-seq paper: simulated
  or real datasets with ground-truth labels exist, and the goal is to reproduce the
  published NMI/ARI values in supplementary comparison tables for the proposed method
  vs. competitors.'
when_to_use_negative:
- The dataset has no ground-truth cell-type labels, since NMI and ARI cannot be computed
  against simulated true labels
- Very large-scale scRNA-seq datasets, where SSRE is known to be relatively time-consuming
  and reproduction would be prohibitively slow without parallel computing
- The goal is downstream tasks such as visualization or differential expression analysis
  rather than benchmark table reproduction
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MATLAB
  role: Runs the SSRE implementation used to produce the benchmark NMI/ARI scores
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: Alternative implementation of SSRE for reproducing the supplementary tables
  repo: https://github.com/CSUBioGroup/SSRE
- name: MPSSC
  role: Example competing spectral clustering framework whose MATLAB code computes
    NMI, Purity, and ARI against true labels
  repo: https://github.com/ishspsy/project
- name: SC3
  role: Competing unsupervised single-cell clustering tool from the compared methods
  repo: https://github.com/hemberg-lab/SC3
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:ffb242842ff6bc805e054f1c1738de8ada7eefac56f2950abac9f903aa6b9a6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - MPSSC
  - Python
  - SC3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# supplementary-table-comparison

## Summary

Rebuilds a paper's supplementary benchmark comparison tables (e.g., Tables S1 and S2 of the SSRE paper) by regenerating or re-running the underlying clustering experiments and assembling NMI/ARI scores for the proposed method against competing methods, then checking the reproduced numbers within numerical tolerance.

## When to use

Use when validating a clustering benchmark from a scRNA-seq paper: simulated or real datasets with ground-truth labels exist, and the goal is to reproduce the published NMI/ARI values in supplementary comparison tables for the proposed method vs. competitors.

## When NOT to use

- The dataset has no ground-truth cell-type labels, since NMI and ARI cannot be computed against simulated true labels
- Very large-scale scRNA-seq datasets, where SSRE is known to be relatively time-consuming and reproduction would be prohibitively slow without parallel computing
- The goal is downstream tasks such as visualization or differential expression analysis rather than benchmark table reproduction

## Inputs

- Five Splatter-simulated scRNA-seq count matrices (nCells=1000/1000/1000/500/1500; sparsity 0.61/0.8/0.94/0.94/0.94; group.prob=(0.65,0.25,0.1))
- Simulated true cell-type label vectors for each dataset
- SSRE MATLAB or Python implementation from CSUBioGroup/SSRE
- Implementations of the competing state-of-the-art single-cell clustering methods evaluated in the paper

## Outputs

- tables_s1_s2_reproduction.csv containing NMI and ARI entries for SSRE and every competing method across all five simulated datasets
- Per-dataset clustering assignments from each method for downstream metric computation

## How to apply

Regenerate the exact test datasets first — for the SSRE benchmark this means five Splatter-simulated scRNA-seq count matrices in R with group.prob=(0.65,0.25,0.1), cell counts nCells=1000/1000/1000/500/1500, and sparsity levels 0.61/0.8/0.94/0.94/0.94, retaining the simulated true cell-type labels. Run the target method (SSRE from CSUBioGroup/SSRE, MATLAB or Python) in its default configuration — all three pairwise similarities used for gene selection and similarity enhancement — and run each competing state-of-the-art single-cell clustering method on the same matrices. Compute NMI and ARI between each method's cluster assignments and the simulated true labels for every dataset. Assemble the scores into tables mirroring Tables S1 and S2 and save them as tables_s1_s2_reproduction.csv. Validate by confirming the CSV contains NMI and ARI entries for SSRE and every competitor across all five datasets and that values reproduce the published scores within numerical tolerance, with SSRE superior in the majority of cases as reported.

## Related tools

- **MATLAB** (Runs the SSRE implementation used to produce the benchmark NMI/ARI scores) — https://github.com/CSUBioGroup/SSRE
- **Python** (Alternative implementation of SSRE for reproducing the supplementary tables) — https://github.com/CSUBioGroup/SSRE
- **MPSSC** (Example competing spectral clustering framework whose MATLAB code computes NMI, Purity, and ARI against true labels) — https://github.com/ishspsy/project
- **SC3** (Competing unsupervised single-cell clustering tool from the compared methods) — https://github.com/hemberg-lab/SC3

## Examples

```
[P] = clus_fin_update(rho, lam, lam2, eta, c, in_X, true_labs); [NMI, ~, clus_labs, ~] = calc2_nmis(max(true_labs), double(P), true_labs); ARI = RandIndex(clus_labs, true_labs);
```

## Evaluation signals

- tables_s1_s2_reproduction.csv contains both NMI and ARI entries for SSRE and every competing method across all five simulated datasets
- Reproduced scores match the published supplementary Tables S1 and S2 values within numerical tolerance
- SSRE is superior in the majority of cases, consistent with the reported finding of good stability across dataset sizes and sparsity levels
- Dataset generation parameters match the paper: group.prob=(0.65,0.25,0.1), nCells=1000/1000/1000/500/1500, sparsity 0.61/0.8/0.94/0.94/0.94
- SSRE was run with default configuration, i.e., all three pairwise similarities used for gene selection and similarity enhancement

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is suggested as an acceleration strategy but was not implemented in the paper
- Exact numerical reproduction requires matching the paper's Splatter simulation seeds and per-dataset sparsity settings; otherwise scores may drift from Tables S1/S2
- Users may choose one or two of the three pairwise similarities, but the benchmark result corresponds to the default of using all three

## Evidence

- [abstract] Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several state-of-the-art single-cell clustering methods.: "Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several state-of-the-art single-cell clustering methods."
- [discussion] SSRE limitation on large-scale data: "One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework [60]."
- [discussion] Default use of all three pairwise similarities: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
- [readme] NMI computation example from competing method README: [evidence span withheld — anchor status: not_found]
- [supplementary] Repository availability of MATLAB and Python implementations: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
