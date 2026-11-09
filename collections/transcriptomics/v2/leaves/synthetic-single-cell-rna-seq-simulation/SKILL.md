---
name: synthetic-single-cell-rna-seq-simulation
description: Use when a decomposition choice (e.g., ICA vs NMF) must be validated
  for program recovery on single-cell expression data, or when designing simulation-based
  benchmarks with known embedded transcriptional signatures.
when_to_use_negative:
- Real tumor scRNA-seq cohorts with no ground-truth embedded GEPs — recovery scores
  cannot be computed without known programs
- Cross-dataset integration/label-transfer tasks (e.g., Harmony, scVI, scArches joint
  embeddings) rather than unsupervised decomposition recovery benchmarks
- Data where components are expected to be statistically dependent, violating the
  independence criterion that favors ICA
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scsim
  role: Simulates 20 synthetic single-cell count matrices with 14 pre-embedded gene
    expression programs as the ground truth for recovery benchmarking
- name: MetaTiME
  role: Provides the ICA-based pipeline that decomposes pooled single-dataset expression
    matrices into independent meta-components for recovery scoring
  repo: github.com/yi-zhang/MetaTiME
- name: scikit-learn
  role: Supplies the NMF implementation used as the comparator decomposition in the
    recovery benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-38333-8
    title: MetaTiME integrates single-cell gene expression to characterize the meta-components
      of the tumor immune microenvironment
  source_content_hash: sha256:93c588e8ac6f2dfef14a07bea713f56b252c3dd7ff129068a596b0573614198b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaTiME
  - scikit-learn
  - scsim
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# synthetic-single-cell-rna-seq-simulation

## Summary

Benchmark workflow that simulates multi-cohort scRNA-seq data with pre-embedded gene expression programs (GEPs) using scsim, then measures how well ICA-based versus NMF-based decomposition recovers the true programs, justifying MetaTiME's choice of ICA over NMF.

## When to use

Use when a decomposition choice (e.g., ICA vs NMF) must be validated for program recovery on single-cell expression data, or when designing simulation-based benchmarks with known embedded transcriptional signatures.

## When NOT to use

- Real tumor scRNA-seq cohorts with no ground-truth embedded GEPs — recovery scores cannot be computed without known programs
- Cross-dataset integration/label-transfer tasks (e.g., Harmony, scVI, scArches joint embeddings) rather than unsupervised decomposition recovery benchmarks
- Data where components are expected to be statistically dependent, violating the independence criterion that favors ICA

## Inputs

- Simulator configuration for scsim specifying number of synthetic cohorts (20), genes, cells, and 14 embedded GEPs (13 cell-type-specific + 1 signaling)
- Pooled synthetic count matrices at cohort sizes 5, 10, and 20
- Ground-truth list of the 14 embedded gene expression programs for matching

## Outputs

- Per-program recovery scores for each decomposition method (ICA, NMF) and each cohort-size setting (5, 10, 20)
- Comparison table simulation_benchmark_results.csv aggregating ICA vs NMF recovery scores by cohort size

## How to apply

Configure the scsim simulator to generate 20 synthetic single-cell count matrices, each embedding 14 known gene expression programs (13 cell-type-specific programs plus 1 signaling program). For each cohort-size setting — 5, 10, and 20 datasets — pool the corresponding synthetic datasets and run the ICA-based recovery pipeline (MetaTiME's ICA decomposition, chosen to 'optimize information independence among components') to decompose pooled expression into independent components. Repeat the identical 5/10/20 cohort pooling and recovery with an NMF-based pipeline (e.g., scikit-learn NMF) as the comparator. Match recovered components back to the 14 known GEPs per simulation and compute per-program recovery scores at each cohort size. Rationale: MetaTiME learns reproducible meta-components reflecting independent components of gene expression variation across cohorts, so the benchmark tests whether the factorization recovers ground-truth independent signals as pooled data volume grows.

## Related tools

- **scsim** (Simulates 20 synthetic single-cell count matrices with 14 pre-embedded gene expression programs as the ground truth for recovery benchmarking)
- **MetaTiME** (Provides the ICA-based pipeline that decomposes pooled single-dataset expression matrices into independent meta-components for recovery scoring) — github.com/yi-zhang/MetaTiME
- **scikit-learn** (Supplies the NMF implementation used as the comparator decomposition in the recovery benchmark)

## Examples

```
pip install metatime
```

## Evaluation signals

- ICA recovery scores for the true GEPs are slightly better than NMF scores at matched cohort sizes
- Recovery scores improve monotonically as pooled cohort count increases from 5 to 10 to 20
- Recovered components are successfully matched to all 14 known GEPs (13 cell-type-specific + 1 signaling) in each simulation
- simulation_benchmark_results.csv contains rows covering the full 2 methods x 3 cohort-size grid with per-program scores
- Identical simulation seeds/configurations are reused across the ICA and NMF pipelines so differences reflect the decomposition, not the data

## Limitations

- Conclusion that ICA outperforms NMF is based on simulations with pre-embedded signatures; real TME data may not contain cleanly separable programs
- Only two decompositions (ICA, NMF) at three cohort sizes (5, 10, 20) were compared; other methods or settings are untested
- Recovery depends on component-to-GEP matching, which may be ambiguous when recovered components are similar

## Evidence

- [discussion] performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components51: "performed a large-scale and pan-cancer integration of tumor single cell datasets using ICA to optimize information independence among components"
- [intro] MetaTiME identifies reproducible low-dimensional meta-components that reflect independent components of gene expression variation across cohorts and cancer types.: [evidence span withheld — anchor status: not_found]
- [readme] MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data.: "MetaTiME learns data-driven, interpretable, and reproducible gene programs by integrating millions of single cells from hundreds of tumor scRNA-seq data."
