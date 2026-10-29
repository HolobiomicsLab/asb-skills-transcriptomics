---
name: splatter-single-cell-rnaseq-trajectory-simulation
description: Use when benchmarking feature selection or trajectory inference methods
  on scRNA-seq data where ground-truth cluster labels, DE genes, and latent step pseudotime
  must be known and noise must be systematically varied via BCV.
when_to_use_negative:
- Evaluating methods on real experimental scRNA-seq or imaging data, where no simulation
  ground truth (DE genes, latent step time) exists
- Benchmarking supervised feature selection that requires ground-truth annotations
  at training time if the simulation labels would leak into the selection step
- Studying branched or complex trajectories — this setup generates linear Splatter
  paths only
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Splatter
  role: R simulation package (v1.18.2) whose paths model generates the ground-truth
    linear-trajectory scRNA-seq data
  repo: https://github.com/jranek/delve_benchmark
- name: scprep (SplatSimulate)
  role: Python wrapper used to call the Splatter paths simulation and return the simulated
    matrix with ground truth labels
- name: delve_benchmark
  role: Python package whose fs class runs feature selection (select()) and evaluation
    (evaluate_select()) on the simulated datasets
  repo: https://github.com/jranek/delve_benchmark
- name: Scanpy
  role: Performs diffusion pseudotime (dpt) inference on a k = 10 k-NN graph for the
    Kendall-tau trajectory metric
- name: scikit-learn
  role: Trains the k = 3 k-nearest neighbor classifier on selected features for cell
    type classification accuracy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:357c8a64bca7e078c7bd5a50861365f9c722a903c0442551eaeec1e3f7576c9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve_benchmark
  tools:
  - Python
  - Scanpy
  - Splatter
  - delve_benchmark
  - scikit-learn
  - scprep (SplatSimulate)
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# splatter-single-cell-rnaseq-trajectory-simulation

## Summary

Generate simulated single-cell RNA-seq datasets with known ground truth (cell type labels, differentially expressed genes, latent step time) using the Splatter 'paths' model via the scprep SplatSimulate wrapper, to benchmark unsupervised feature selection methods for trajectory preservation.

## When to use

Use when benchmarking feature selection or trajectory inference methods on scRNA-seq data where ground-truth cluster labels, DE genes, and latent step pseudotime must be known and noise must be systematically varied via BCV.

## When NOT to use

- Evaluating methods on real experimental scRNA-seq or imaging data, where no simulation ground truth (DE genes, latent step time) exists
- Benchmarking supervised feature selection that requires ground-truth annotations at training time if the simulation labels would leak into the selection step
- Studying branched or complex trajectories — this setup generates linear Splatter paths only

## Inputs

- Splatter simulation parameters (mean_rate, mean_shape, lib_loc, lib_scale, bcv_df)
- Noise-level grid (BCV = 0.1, 0.25, 0.5)
- Simulation design parameters: n_cells = 1500, n_genes = 500, 6 clusters, linear (paths) trajectory, 30 replicate datasets per condition

## Outputs

- Simulated scRNA-seq count matrices (cells x genes) per noise condition
- Ground-truth cluster labels, ground-truth differentially expressed gene sets, and latent step vector per dataset
- Aggregated median benchmark metric table (precision@k, k-NN classification accuracy, Kendall tau per method) written to CSV

## How to apply

Generate datasets with the splatter paths function through the scprep SplatSimulate wrapper using the paper's estimated parameters (mean_rate = 0.0173, mean_shape = 0.54, lib_loc = 12.6, lib_scale = 0.423, bcv_df = 90.2, no dropout), producing 1500 cells, 500 genes, and 6 clusters per dataset. Vary the biological coefficient of variation across BCV = 0.1, 0.25, and 0.5 to create high, medium, and low signal-to-noise conditions (30 replicate linear-trajectory datasets per condition). Retain the ground-truth cluster labels and latent step vector from the simulation as evaluation references. Feed the simulated matrices into each feature selection strategy (e.g., DELVE, Laplacian Score, hotspot) within 3-fold cross-validation splits, select the top 100 ranked features, then score performance via precision@k against Splatter ground-truth DE genes, k-NN (k = 3) cell type classification accuracy, and Kendall rank correlation between Scanpy diffusion pseudotime and the ground truth latent step vector.

## Related tools

- **Splatter** (R simulation package (v1.18.2) whose paths model generates the ground-truth linear-trajectory scRNA-seq data) — https://github.com/jranek/delve_benchmark
- **scprep (SplatSimulate)** (Python wrapper used to call the Splatter paths simulation and return the simulated matrix with ground truth labels)
- **delve_benchmark** (Python package whose fs class runs feature selection (select()) and evaluation (evaluate_select()) on the simulated datasets) — https://github.com/jranek/delve_benchmark
- **Scanpy** (Performs diffusion pseudotime (dpt) inference on a k = 10 k-NN graph for the Kendall-tau trajectory metric)
- **scikit-learn** (Trains the k = 3 k-nearest neighbor classifier on selected features for cell type classification accuracy)

## Examples

```
import scprep; data = scprep.simulate.SplatSimulate(method='paths', n_cells=1500, n_genes=500, n_groups=6, lib_loc=12.6, lib_scale=0.423, bcv_common=0.25)
```

## Evaluation signals

- Each simulated dataset must contain exactly 1500 cells, 500 genes, and 6 clusters with matching-length ground-truth label and latent step vectors
- Precision@k of selected features against Splatter ground-truth DE genes is computable for every method because ground-truth DE labels are retained from the simulation
- Kendall rank correlation between estimated diffusion pseudotime and the ground truth latent step vector should be well above 0 for trajectory-preserving methods (DELVE, Laplacian Score, Hotspot)
- k-NN (k = 3) classification accuracy should degrade as BCV increases from 0.1 to 0.5; DELVE's expected medians are 0.937, 0.882, and 0.734 respectively
- Median metric scores aggregated across replicates per noise condition should reproduce the reported ordering of the 12 methods when written to the results CSV

## Limitations

- Simulated linear trajectories may not reflect branched or real biological trajectories; feature selection can greatly bias interpretation of the underlying cellular trajectory, requiring careful consideration
- All dynamic modules were leveraged equally when constructing the approximate cellular trajectory; excluding unwanted dynamic modules is possible but not implemented
- Requires the R environment with splatter (v1.18.2) and BiocManager installed in addition to the Python benchmark package

## Evidence

- [readme] Delve benchmark reproducibility: [evidence span withheld — anchor status: external_doc]
- [readme] Splatter install requirement: [evidence span withheld — anchor status: external_doc]
- [abstract] Simulation-based demonstration in abstract: "Using simulations, single-cell RNA sequencing, and iterative immunofluorescence imaging data in the context of cell cycle and cellular differentiation, we demonstrate how DELVE selects features that"
- [discussion] Similarity-based methods top on simulated data: "similarity-based feature selection methods (e.g., DELVE, Laplacian score, and Hotspot) achieved the highest recovery of differentially expressed genes within a cell type or along a cellular lineage"
- [discussion] Benchmark scope of twelve methods: "we benchmarked twelve feature selection methods29,49,53,55–58 on their ability to identify biologically relevant features for trajectory analysis from single-cell RNA sequencing data and proteomic"
