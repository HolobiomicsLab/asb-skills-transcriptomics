---
name: ablation-study-comparison-analysis
description: Use when you need to isolate the contribution of one method component
  (here, the similarity-enhancement step of SSRE) by comparing a full pipeline against
  its ablated baseline on the same labeled benchmark datasets using NMI and ARI.
when_to_use_negative:
- The goal is benchmarking SSRE against external competing methods (e.g., SC3 or SIMLR)
  rather than isolating the similarity-enhancement component within SSRE
- Datasets lack ground-truth cell type labels, so NMI and ARI cannot be computed
- Data are very large-scale, where SSRE is relatively time-consuming and acceleration
  (parallel computing) would be needed first
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MATLAB
  role: Runs the SSR baseline and full SSRE pipeline (gene selection, similarity enhancement,
    spectral clustering) and computes NMI/ARI in the ablation comparison
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: Alternative implementation for running SSR and SSRE pipelines to compute the
    ablation metrics
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:8bb12031db8001153a2a384b45cc3352c42cade9c209a89158009f2e5c27d66b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - Python
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# ablation-study-comparison-analysis

## Summary

Reproduces the SSRE vs. SSR ablation by running the sparse subspace representation (SSR) baseline and the full SSRE pipeline (adaptive gene selection + similarity enhancement + spectral clustering) side by side and comparing NMI/ARI on ten real scRNA-seq datasets with ground-truth cell type labels.

## When to use

Use when you need to isolate the contribution of one method component (here, the similarity-enhancement step of SSRE) by comparing a full pipeline against its ablated baseline on the same labeled benchmark datasets using NMI and ARI.

## When NOT to use

- The goal is benchmarking SSRE against external competing methods (e.g., SC3 or SIMLR) rather than isolating the similarity-enhancement component within SSRE
- Datasets lack ground-truth cell type labels, so NMI and ARI cannot be computed
- Data are very large-scale, where SSRE is relatively time-consuming and acceleration (parallel computing) would be needed first

## Inputs

- Ten real scRNA-seq gene expression matrices (cells x genes) from the SSRE project data repository
- Ground-truth cell type label vectors for each dataset
- MATLAB or Python implementations of SSRE and the SSR baseline from https://github.com/CSUBioGroup/SSRE

## Outputs

- Per-dataset table of NMI and ARI for SSR (baseline) vs. SSRE (full pipeline)
- Count of datasets on which SSRE outperforms SSR (expected: 9 of 10 for both NMI and ARI)
- Flagged list of any datasets where the reproduced outcome diverges from the paper's ablation result

## How to apply

Load the ten real scRNA-seq datasets together with ground-truth cell type labels from the project data repository. Run the SSR baseline — sparse subspace representation cell-to-cell similarity followed by spectral clustering — and then the full SSRE pipeline, which adds adaptive gene selection and similarity enhancement (combining three classical pairwise similarities with sparse structure embedding) before spectral clustering. Compute NMI and ARI against the provided labels for both variants on each dataset, then tabulate SSR vs. SSRE per dataset and count the datasets where SSRE wins. Note that by default all three pairwise similarities are used for gene selection and similarity enhancement, and users may choose only one or two. Validation: SSRE should outperform SSR on nine of the ten datasets in both NMI and ARI; flag any dataset whose outcome diverges from the paper's reported ablation.

## Related tools

- **MATLAB** (Runs the SSR baseline and full SSRE pipeline (gene selection, similarity enhancement, spectral clustering) and computes NMI/ARI in the ablation comparison) — https://github.com/CSUBioGroup/SSRE
- **Python** (Alternative implementation for running SSR and SSRE pipelines to compute the ablation metrics) — https://github.com/CSUBioGroup/SSRE

## Examples

```
[NMI, ~,clus_labs,~]=calc2_nmis(max(true_labs), double(P),true_labs); ARI=RandIndex(clus_labs,true_labs)
```

## Evaluation signals

- SSRE outperforms SSR on nine of the ten real scRNA-seq datasets in both NMI and ARI, matching the paper's reported ablation result
- Any dataset where SSRE does not win is explicitly flagged as a divergence from the paper
- All NMI and ARI values fall within [0, 1] and are computed against the same ground-truth labels for both SSR and SSRE
- Both pipeline variants run on identical datasets with the same cell type labels, so differences are attributable to the enhancement strategy
- The comparison table contains an NMI and ARI entry for both methods on every one of the ten datasets (no missing cells)

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is suggested as a possible acceleration strategy but was not implemented
- The enhancement strategy can be configured with one or two of the three pairwise similarities instead of all three (default uses all three), which changes the ablation comparison
- The ablation claim is tied to the paper's ten real scRNA-seq datasets; outcomes may differ on other data

## Evidence

- [results] SSRE outperforms SSR on 9/10 datasets: "Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several state-of-the-art single-cell clustering methods."
- [supplementary] Implementations availability: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [discussion] Default similarity configuration: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
- [discussion] Runtime limitation: "One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework [60]."
- [readme] Metric computation pattern in code: [evidence span withheld — anchor status: not_found]
