---
name: synthetic-dataset-generation
description: Use when benchmarking a clustering method against ground truth and no
  labeled real data is available; controlled synthetic problems with tunable noise
  and cluster shape are needed to expose failure modes of general-purpose algorithms.
when_to_use_negative:
- The analysis target is a real experimental dataset (e.g. mass cytometry or scRNA-seq)
  with externally assigned labels — synthetic generators should not replace ground-truthed
  biological data
- The goal is evaluating clustering-quality metrics themselves rather than algorithms,
  since these generators are tuned to shapes where standard metrics like the silhouette
  score are biased towards globular cluster shapes
tools:
- name: scikit-learn
  role: random sample generators (make_circles, make_moons, make_blobs) used to produce
    simulated datasets with 1000 objects and fixed seeds
- name: Python
  role: implementation language for the data generation and benchmarking scripts
- name: KMDHierarchicalClustering
  role: consumer of the synthetic datasets in demo notebooks (Toy_dataset_example.ipynb)
    that evaluate clustering against ground truth
  repo: github.com/KaplanLab/KMDHierarchicalClustering
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:b8b0b5a8f4b570f9360fba4be2c54ce992f4f508556bad6518b6571e91930af3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bluster
  - FlowSOM
  - KMDHierarchicalClustering
  - PhenoGraph
  - Python
  - SCCAF
  - Scanpy
  - SciPy
  - Seurat
  - hdbscan
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# synthetic-dataset-generation

## Summary

Generates labeled simulated clustering benchmarks with scikit-learn random sample generators (nested circles, two moons, Gaussian blobs, anisotropic blobs) at controlled noise levels so that a candidate algorithm like KMD clustering can be scored against known ground truth.

## When to use

Use when benchmarking a clustering method against ground truth and no labeled real data is available; controlled synthetic problems with tunable noise and cluster shape are needed to expose failure modes of general-purpose algorithms.

## When NOT to use

- The analysis target is a real experimental dataset (e.g. mass cytometry or scRNA-seq) with externally assigned labels — synthetic generators should not replace ground-truthed biological data
- The goal is evaluating clustering-quality metrics themselves rather than algorithms, since these generators are tuned to shapes where standard metrics like the silhouette score are biased towards globular cluster shapes

## Inputs

- scikit-learn random sample generator settings (make_circles factor=0.3, make_moons, make_blobs with cluster std vector, anisotropic transformation matrix)
- sample count (1000 objects) and seed (seed=1 low-noise; seed=185 high-noise)
- noise parameters (noise=0.05 low-noise; noise=0.14/0.24 high-noise)

## Outputs

- Feature matrix X and ground-truth label vector y_true for each simulated dataset
- Four low-noise and four high-noise simulated datasets suitable for clustering benchmark tables (e.g. toy_dataset_clustering_scores.csv)

## How to apply

Generate each dataset with scikit-learn random sample generators using 1000 objects and a fixed seed (seed = 1 for the standard set) so results are reproducible. For the low-noise suite: concentric circles (factor=0.3, noise=0.05), two moons (noise=0.05), three Gaussian blobs with cluster std=[1.0, 2.5, 0.5] and random_state=170, and an anisotropically transformed blob set, always retaining the generator's ground-truth labels y_true. For the high-noise suite, regenerate the same four problems with noise=0.14/0.24, std=[2.0, 2.0, 2.0], and seed=185 to stress-test the algorithm's outlier-aware behavior. The rationale is that these shapes cover both non-globular structures (circles, moons, where plain silhouette scoring is biased) and globular/anisotropic structures, letting you separate shape sensitivity from noise sensitivity. Feed each (X, y_true) pair into the clustering pipeline and score predicted labels against ground truth with accuracy, NMI and ARI, e.g. KMD must attain accuracy/NMI/ARI = 1 on nested circles under the low-noise settings.

## Related tools

- **scikit-learn** (random sample generators (make_circles, make_moons, make_blobs) used to produce simulated datasets with 1000 objects and fixed seeds)
- **Python** (implementation language for the data generation and benchmarking scripts)
- **KMDHierarchicalClustering** (consumer of the synthetic datasets in demo notebooks (Toy_dataset_example.ipynb) that evaluate clustering against ground truth) — github.com/KaplanLab/KMDHierarchicalClustering

## Examples

```
from sklearn import datasets; X, y_true = datasets.make_circles(n_samples=1000, factor=0.3, noise=0.05, random_state=1)
```

## Evaluation signals

- Dataset shapes match the specified generator settings: concentric circles factor=0.3, moons noise=0.05, three blobs with std=[1.0, 2.5, 0.5] at random_state=170
- Every generated dataset carries its ground-truth labels from the generator (y_true retained)
- Reproducibility check: regenerating with the same 1000-object count and seed reproduces identical matrices
- Downstream sanity check: a competent algorithm (KMD clustering) attains accuracy/NMI/ARI = 1 on the low-noise nested-circles dataset
- High-noise variants (noise=0.14/0.24, std=[2.0,2.0,2.0]) yield non-trivial scores (KMD: 0.989, 0.933, 0.909, 0.992) rather than perfect separation

## Limitations

- Simulated datasets with only 1000 objects and small noise do not capture the scale and sparsity of real biological data (e.g. million-object scRNA-seq)
- Anisotropically transformed blob sets and fixed seeds restrict the diversity of benchmark shapes; performance on other geometries is untested here

## Evidence

- [discussion] All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1.: "All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1."
- [readme] Toy_dataset_example.ipynb evaluates simulated toy datasets: "Toy_dataset_example.ipynb - Evaluates performance on a standard set of simulated toy datasets - expected run time: ~8 minutes"
- [readme] DBCV README moon generation example: [evidence span withheld — anchor status: external_doc]
- [readme] silhouette bias motivates non-globular synthetic shapes: [evidence span withheld — anchor status: external_doc]
