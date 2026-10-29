---
name: synthetic-moons-dataset-simulation
description: Use when a clustering algorithm or intrinsic clustering-quality measure
  needs benchmarking on non-globular (interleaved crescent) clusters where ground
  truth is known and noise level can be controlled.
when_to_use_negative:
- The evaluation target is real experimental data (e.g. scRNA-seq or mass cytometry
  datasets like Levine15_13, Zeisel15, Lawlor17) rather than a controlled simulation
  with known ground truth
- The clustering method under test is specifically designed for globular clusters,
  since silhouette-style evaluation is biased towards globular shapes and the two-moon
  shape is deliberately non-globular
- Very small clusters are the subject of study, since sampling-based extensions may
  miss very small clusters that are not sampled well
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: scikit-learn
  role: Provides the random sample generators used to create the simulated two-moon
    datasets (1000 objects, controlled noise and seed)
- name: KMDHierarchicalClustering
  role: Clustering method evaluated on the simulated moons; fitted with n_clusters=2,
    k='compute', k_scan_range=(1,100), y_true and plot_scores=True
  repo: github.com/KaplanLab/KMDHierarchicalClustering
- name: SciPy
  role: Used to compute Pearson correlations between KMD silhouette and clustering
    performance across k values
- name: Python
  role: Implementation language for the simulation and evaluation pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-05480-z
    title: 'KMD clustering: robust general-purpose clustering of biological data'
  source_content_hash: sha256:c8d7beabbe5abee6cd1d0a8da52fbd82b785618db9fc8462887114efce52f3a0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KMDHierarchicalClustering
  - Python
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# synthetic-moons-dataset-simulation

## Summary

Generates simulated two-moon datasets (1000 objects) with tunable noise using scikit-learn random sample generators, retaining ground-truth labels, so that clustering methods and intrinsic quality measures such as the KMD silhouette can be benchmarked against known truth.

## When to use

Use when a clustering algorithm or intrinsic clustering-quality measure needs benchmarking on non-globular (interleaved crescent) clusters where ground truth is known and noise level can be controlled.

## When NOT to use

- The evaluation target is real experimental data (e.g. scRNA-seq or mass cytometry datasets like Levine15_13, Zeisel15, Lawlor17) rather than a controlled simulation with known ground truth
- The clustering method under test is specifically designed for globular clusters, since silhouette-style evaluation is biased towards globular shapes and the two-moon shape is deliberately non-globular
- Very small clusters are the subject of study, since sampling-based extensions may miss very small clusters that are not sampled well

## Inputs

- Specification of sample size (1000 objects), noise level (e.g. 0.24), number of instances (e.g. 50), and random seed (seed=3 for the Fig. 2 instance; seed=1 default elsewhere in the paper)
- Ground-truth cluster labels returned together with the simulated feature matrix

## Outputs

- Simulated two-moon feature matrix (1000 x 2) with ground-truth labels per instance
- Per-k clustering accuracy and KMD silhouette scores (e.g. written to kmd_halfmoons_scores.csv)
- Per-instance Pearson correlations between intrinsic score and accuracy/NMI/ARI

## How to apply

Generate the two-moon dataset with scikit-learn's random sample generators (make_moons-style) at 1000 objects, setting noise to the desired level (e.g. noise=0.24 for the high-noise benchmark, seed=3 for the Fig. 2 instance) and retaining the ground-truth labels for evaluation. Produce multiple instances (e.g. 50 draws) so correlation statistics between predicted quality and true clustering accuracy can be computed across instances. Fit the clustering method under test — here KMDClustering with n_clusters=2, k='compute', k_scan_range=(1,100), y_true set to the ground-truth labels, and plot_scores=True — on the dataset matrix and record accuracy and the KMD silhouette score for every k value in the scan range. Compute Pearson correlations between the intrinsic score and clustering performance metrics (accuracy, NMI, ARI) across k values per instance; on high-noise half moons the median correlations reported are ~0.89 (accuracy), ~0.85 (NMI), ~0.87 (ARI). The article notes all simulated datasets used 1000 objects, and seed=1 is the default in the generators while seed=3 reproduces the Fig. 2 instance.

## Related tools

- **scikit-learn** (Provides the random sample generators used to create the simulated two-moon datasets (1000 objects, controlled noise and seed))
- **KMDHierarchicalClustering** (Clustering method evaluated on the simulated moons; fitted with n_clusters=2, k='compute', k_scan_range=(1,100), y_true and plot_scores=True) — github.com/KaplanLab/KMDHierarchicalClustering
- **SciPy** (Used to compute Pearson correlations between KMD silhouette and clustering performance across k values)
- **Python** (Implementation language for the simulation and evaluation pipeline)

## Examples

```
kmd_cluster = KMDClustering(n_clusters=2, k='compute', k_scan_range=(1,100), y_true=y_true, plot_scores=True); kmd_cluster.fit(X)  # X from sklearn make_moons(n_samples=1000, noise=0.24, random_state=3)
```

## Evaluation signals

- Median across the 50 instances of Pearson correlation between KMD silhouette and clustering accuracy is ~0.89 (NMI ~0.85, ARI ~0.87) at noise=0.24, matching the reported benchmark
- Each simulated instance contains exactly 1000 objects with two interleaved moon clusters and retained ground-truth labels
- Seed reproducibility: seed=3 regenerates the Fig. 2 instance; other simulations use seed=1
- Per-k accuracy and KMD silhouette scores are recorded for every k in the scan range (1,100) and persisted (e.g. kmd_halfmoons_scores.csv)

## Limitations

- Simulated two-moon data is non-globular, so standard silhouette scores are biased towards globular cluster shapes and are not adequate as an intrinsic evaluation function on this data
- Automatic k selection requires running clustering over a large range of k values, requiring more computational resources; a single KMD run is also slower because it maintains lists of k minimal distances per pair of clusters
- If a sampling-based extension is later applied to the simulated large datasets, it may miss very small clusters that are not sampled well

## Evidence

- [discussion] All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1.: "All simulated datasets were generated using scikit-learn random sample generators with 1000 objects and seed = 1."
- [methods] kmd silhouette predictiveness workflow (task card): [evidence span withheld — anchor status: not_found]
- [results] On 50 instances of the high-noise half moons dataset, the KMD silhouette score is predictive of all three clustering performance metrics, with median Pearson correlations of 0.89 (accuracy), 0.85 (NMI) and 0.87 (ARI).: [evidence span withheld — anchor status: not_found]
- [discussion] The silhouette score is often used to evaluate and compare clustering solutions, yet it is biased towards globular cluster shapes and is thus not adequate.: "The silhouette score is often used to evaluate and compare clustering solutions, yet it is biased towards globular cluster shapes and is thus not adequate."
- [readme] KMDClustering parameters from README: "KMD = KMDClustering(k='compute', n_clusters = 2, min_cluster_size = 'compute', affinity = 'euclidean', certainty = 0.5 ,k_scan_range = (1,100,3), y_true = [], plot_scores=False, path=False)"
