---
name: adjusted-rand-index-benchmarking
description: Use when comparing a predicted scRNA-seq clustering against known cell-type
  labels to quantify accuracy — e.g., benchmarking DESC, scVI, or other methods on
  datasets with reference annotations like the GSE115746 macaque retina bipolar-cell
  data.
when_to_use_negative:
- No reference cell-type labels exist for the dataset — ARI cannot be computed against
  ground truth
- The goal is batch-mixing assessment rather than clustering accuracy — use regional
  KL divergence of batch mixing (K nearest neighbors, K = 5 × number of batches, 200
  iterations) instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DESC
  role: Clustering method being benchmarked; produces cluster assignments evaluated
    by ARI
  repo: https://github.com/eleozzr/desc
- name: Scanpy
  role: Preprocessing (sc.pp.log1p, sc.pp.filter_genes_dispersion, sc.pp.scale) and
    ARI-based evaluation of clustering performance
  repo: https://github.com/theislab/scanpy
- name: Keras
  role: Backend used to train the DESC neural network whose clusterings are scored
    by ARI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:17f28332f4309983d84384501b0a9a70eb50953673547ab93048bb4b02dd5436
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - DESC
  - Keras
  - Scanpy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# adjusted-rand-index-benchmarking

## Summary

Benchmark clustering accuracy of scRNA-seq methods by computing the adjusted Rand index (ARI) between predicted cluster assignments and reference cell-type labels, as used in the DESC paper to show DESC reaches ARI 0.920 on macaque retina bipolar cells even without batch information.

## When to use

Use when comparing a predicted scRNA-seq clustering against known cell-type labels to quantify accuracy — e.g., benchmarking DESC, scVI, or other methods on datasets with reference annotations like the GSE115746 macaque retina bipolar-cell data.

## When NOT to use

- No reference cell-type labels exist for the dataset — ARI cannot be computed against ground truth
- The goal is batch-mixing assessment rather than clustering accuracy — use regional KL divergence of batch mixing (K nearest neighbors, K = 5 × number of batches, 200 iterations) instead

## Inputs

- Predicted cluster assignments per cell (e.g., from DESC or scVI)
- Reference cell-type labels per cell (macaque retina bipolar-cell type annotations)
- Preprocessed expression matrix (GSE115746 count matrix: 21,017 foveal + 9,285 peripheral cells) used to generate clusterings

## Outputs

- ARI value(s) comparing predicted clustering to reference labels per condition (batch definition)
- ari_results.csv tabulating ARI values across batch-information conditions

## How to apply

Run the clustering method under each condition of interest — in the DESC paper, desc.train was repeated four times on the macaque retina data with batch labels supplied as macaque id, region id, sample id, or no batch information (dims=[n_genes,64,32], tol=0.005, n_neighbors=10, batch_size=256, louvain_resolution=[0.8,1.0], use_GPU=False, num_Cores=1 for reproducibility). Compute the ARI of each resulting clustering against the reference bipolar-cell type labels using scikit-learn/Scanpy. Larger ARI values indicate higher clustering accuracy; the paper validates by checking that ARI with any batch definition falls within 0.919–0.970 and ~0.920 without batch information, versus scVI dropping to 0.242. Write all ARI values to an output file such as ari_results.csv for comparison.

## Related tools

- **DESC** (Clustering method being benchmarked; produces cluster assignments evaluated by ARI) — https://github.com/eleozzr/desc
- **Scanpy** (Preprocessing (sc.pp.log1p, sc.pp.filter_genes_dispersion, sc.pp.scale) and ARI-based evaluation of clustering performance) — https://github.com/theislab/scanpy
- **Keras** (Backend used to train the DESC neural network whose clusterings are scored by ARI)

## Examples

```
from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(reference_labels, desc_clusters); print(ari)
```

## Evaluation signals

- ARI with any batch definition (macaque id, region id, sample id) falls within 0.919–0.970
- ARI without batch information is ~0.920, matching the paper's reported value
- ARI is computed against the same reference bipolar-cell type labels across all compared methods (e.g., DESC vs scVI ARI 0.242)
- All ARI values are written to ari_results.csv with one entry per batch condition

## Limitations

- ARI requires trusted reference labels; accuracy claims are only valid where high-quality cell-type annotations exist
- DESC's batch-effect removal depends on technical differences across batches being smaller than true biological variations — ARI may degrade when this assumption fails
- Reproducibility of reported ARI requires matching software versions: tensorflow < 2.0 with desc(2.0.3), python 3.5.x/3.6.x, and fixed settings like num_Cores=1

## Evidence

- [discussion] ARI against reference labels is the benchmark metric: "we use ARI to compare the performance of different clustering algorithms. Larger values of ARI indicate higher accuracy in clustering."
- [results] Reported macaque retina ARI result: [evidence span withheld — anchor status: not_found]
- [methods] DESC training parameters used in the benchmark: [evidence span withheld — anchor status: not_found]
- [abstract] DESC does not require batch information for batch effect removal: "DESC offers a proper balance of clustering accuracy and stability, has a small footprint on memory, does not explicitly require batch information for batch effect removal"
- [readme] Version pinning needed for reproducible ARI results: "you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper."
