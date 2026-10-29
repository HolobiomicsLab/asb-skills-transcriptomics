---
name: simulated-toy-dataset-generation
description: 'Use when validating or benchmarking a clustering algorithm (e.g. KMD
  linkage) against known structure: you need reproducible synthetic datasets with
  controllable cluster shapes before testing on real biological data.'
when_to_use_negative:
- The goal is benchmarking on real experimental biological data with known ground
  truth (e.g. mass cytometry datasets Levine15_13, Levine15_23, Samusik16 or scRNA-seq
  datasets) — simulated generators will not capture that structure
- The implementation under test is already trusted and only needs to be fit to an
  existing dataset matrix (use KMDClustering.fit directly instead)
- You need to evaluate scalability to million-object datasets, where the paper's sampling-based
  extension on real data is the appropriate workflow
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn
  role: Random sample generators used to create the simulated toy datasets with 1000
    objects and seed = 1
- name: SciPy
  role: cluster.hierarchy.linkage provides reference dendrograms (single linkage at
    k=1, average linkage at large k) for cophenetic-correlation comparison
- name: KMDHierarchicalClustering
  role: Implementation under test that consumes the toy datasets and produces KMD-linkage
    dendrograms; ships the Toy_dataset_example.ipynb demo
  repo: github.com/KaplanLab/KMDHierarchicalClustering
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:d70edf8affc93dbe2be8fa0e3bd04c3d29c77c127f854bd1ca7b82fb398789da
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KMDHierarchicalClustering
  - Python
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# simulated-toy-dataset-generation

## Summary

Generates standard simulated toy datasets with scikit-learn random sample generators (1000 objects, seed = 1) to serve as ground-truth benchmarks for verifying clustering implementations such as KMD linkage.

## When to use

Use when validating or benchmarking a clustering algorithm (e.g. KMD linkage) against known structure: you need reproducible synthetic datasets with controllable cluster shapes before testing on real biological data.

## When NOT to use

- The goal is benchmarking on real experimental biological data with known ground truth (e.g. mass cytometry datasets Levine15_13, Levine15_23, Samusik16 or scRNA-seq datasets) — simulated generators will not capture that structure
- The implementation under test is already trusted and only needs to be fit to an existing dataset matrix (use KMDClustering.fit directly instead)
- You need to evaluate scalability to million-object datasets, where the paper's sampling-based extension on real data is the appropriate workflow

## Inputs

- scikit-learn generator parameters (n_samples=1000, seed=1) for standard toy datasets
- Reference linkage specification (SciPy cluster.hierarchy.linkage single linkage for k=1, average linkage for large k)

## Outputs

- Simulated feature matrices X with 1000 objects per toy dataset (seed = 1, reproducible)
- Dendrogram comparison results (cophenetic correlations vs SciPy single and average linkage)
- Per-dataset timing and peak-memory log
- kmd_linkage_verification_report.md

## How to apply

Generate the standard simulated toy datasets using scikit-learn random sample generators with 1000 objects and seed = 1, matching the paper's benchmarking setup so results are comparable to published KMD clustering runs (see Toy_dataset_example.ipynb, expected run time ~8 minutes). Use these synthetic matrices as input to the implementation under test (e.g. KMDHierarchicalClustering) and to reference linkages such as SciPy's cluster.hierarchy.linkage. Judge correctness by comparing dendrograms via cophenetic correlation: at k=1 against single linkage and at a large k against average linkage, accepting the implementation when both correlations are >= 0.99. Record per-dataset timing and peak-memory measurements for each k setting during the runs, then write results plus the timing/memory log to kmd_linkage_verification_report.md.

## Related tools

- **scikit-learn** (Random sample generators used to create the simulated toy datasets with 1000 objects and seed = 1)
- **SciPy** (cluster.hierarchy.linkage provides reference dendrograms (single linkage at k=1, average linkage at large k) for cophenetic-correlation comparison)
- **KMDHierarchicalClustering** (Implementation under test that consumes the toy datasets and produces KMD-linkage dendrograms; ships the Toy_dataset_example.ipynb demo) — github.com/KaplanLab/KMDHierarchicalClustering

## Examples

```
from sklearn import datasets; X, y = datasets.make_blobs(n_samples=1000, centers=3, random_state=1)
```

## Evaluation signals

- Cophenetic correlation between the k=1 KMD dendrogram and SciPy single linkage is >= 0.99 on the simulated datasets
- Cophenetic correlation between the large-k KMD dendrogram and SciPy average linkage is >= 0.99 on the simulated datasets
- Each toy dataset contains exactly 1000 objects and regeneration with seed = 1 reproduces identical data
- Per-dataset timing and peak-memory measurements are recorded for each k setting during the runs
- Results plus the timing/memory log are written to kmd_linkage_verification_report.md

## Limitations

- Automatic k selection over the scan range requires running clustering many times, consuming more computational resources on the toy benchmarks
- The python/scipy implementation runs longer than widely used clustering algorithms with optimized non-interpreted implementations
- Simulated toy data cannot represent the biological structure of real datasets (mass cytometry, scRNA-seq) where KMD's performance claims were additionally validated

## Evidence

- [discussion] All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1.: "All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1."
- [readme] Toy_dataset_example.ipynb - Evaluates performance on a standard set of simulated toy datasets - expected run time: ~8 minutes: "Toy_dataset_example.ipynb - Evaluates performance on a standard set of simulated toy datasets - expected run time: ~8 minutes"
- [intro] we then apply our method to a range of simulated and experimental biological datasets in which ground truth is known: [evidence span withheld — anchor status: not_found]
