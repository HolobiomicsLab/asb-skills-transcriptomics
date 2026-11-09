---
name: baseline-method-benchmarking-comparison
description: Use when multiple SC-to-ST spatial mapping methods must be compared on
  the same preprocessed SC/ST inputs where ground-truth spatial coordinates and cell-type
  labels exist, so coordinate error (MAE), neighbor-recovery (hit-number@K), and cell-type
  proportion fidelity (PCC) can be tabulated per.
when_to_use_negative:
- Ground-truth single-cell spatial coordinates or ground-truth SC adjacency are unavailable
  — MAE and hit-number@K cannot be computed, so the comparison collapses
- Only one method's output exists with no baselines run on the same preprocessed data
  — a comparative benchmark requires all methods on identical inputs
- The task is deconvolution of spots into cell-type mixtures rather than single-cell-level
  SC-ST mapping — the article notes deconvolution treats a spot as a mixture of a
  fixed number of cell types and cannot link SC and ST at the single-cell level
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: STEM
  role: Primary method under benchmark; trained as SOmodel (400 epochs, lr 0.002,
    sigma 3, alpha 0.8, dp ~0.584) to produce embeddings and st2sc/sc2st mapping matrices
  repo: https://github.com/WhirlFirst/STEM
- name: CellTrek
  role: Baseline; multivariate random forest model mapping cells to spatial locations,
    outputs coerced to predicted coordinates
- name: scSpace
  role: Baseline; MLP predicting absolute spatial coordinates from gene expression
- name: Seurat
  role: Baseline; not designed for spatial mapping but constructs SC-ST integrated
    graphs used to derive coordinates
- name: Spaotsc
  role: Baseline; optimal transport with spatial constraints to learn the SC-ST mapping
    matrix
- name: Tangram
  role: Baseline; learns an SC-to-ST mapping matrix optimized by minimizing cosine
    similarity
- name: scanpy
  role: 'Preprocessing shared across all methods: normalize_total, log1p, calculate_qc_metrics
    (median n_genes_by_counts for dropout rate)'
- name: PyTorch
  role: Backend for training STEM SOmodel on GPU (cuda:0) and computing softmax embedding
    dot-product mapping matrices
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:c7f280f80ebc5d2e847c56f22f7e16c36e874cf8c7c69098e33fd44ed1d62c2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - CellTrek
  - PyTorch
  - STEM
  - Seurat
  - Spaotsc
  - Tangram
  - scSpace
  - scanpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# baseline-method-benchmarking-comparison

## Summary

Benchmarks STEM against five single-cell spatial mapping baselines (CellTrek, scSpace, Seurat, Spaotsc, Tangram) on the semi-simulation pseudo-SC/pseudo-ST E1z2 mouse embryo data, unifying each method's outputs into predicted coordinates, SC-SC adjacency, and SC-ST mapping matrices for head-to-head evaluation with MAE, hit-number@K, and per-cell-type PCC.

## When to use

Use when multiple SC-to-ST spatial mapping methods must be compared on the same preprocessed SC/ST inputs where ground-truth spatial coordinates and cell-type labels exist, so coordinate error (MAE), neighbor-recovery (hit-number@K), and cell-type proportion fidelity (PCC) can be tabulated per method.

## When NOT to use

- Ground-truth single-cell spatial coordinates or ground-truth SC adjacency are unavailable — MAE and hit-number@K cannot be computed, so the comparison collapses
- Only one method's output exists with no baselines run on the same preprocessed data — a comparative benchmark requires all methods on identical inputs
- The task is deconvolution of spots into cell-type mixtures rather than single-cell-level SC-ST mapping — the article notes deconvolution treats a spot as a mixture of a fixed number of cell types and cannot link SC and ST at the single-cell level

## Inputs

- E1z2 simulated ST count matrix (simu_st_counts.csv) and spatial metadata (simu_st_metadata.csv)
- Raw SeqFISH SC count matrix (simu_sc_counts.csv) with SC metadata (metadata.csv, x_global/y_global)
- Ground-truth ST cell types (simu_st_celltype.csv) and ground-truth SC spatial adjacency
- Trained STEM SOmodel checkpoints / processed data from Figshare DOI 10.6084/m9.figshare.24452812
- Baseline method outputs (CellTrek, scSpace, Seurat, Spaotsc, Tangram) run on the same normalized data

## Outputs

- benchmark_metrics.csv: per-method MAE, hit-number@K (K=200), and per-cell-type PCC table
- Unified per-method predicted single-cell spatial coordinates (adata.obsm['spatialDA'])
- Unified per-method SC-SC adjacency and SC-ST mapping matrices (netst2sc, netsc2st)

## How to apply

Preprocess all methods on identical inputs: load the E1z2 simulated ST counts, raw SeqFISH SC counts, ground-truth ST cell types, and spatial metadata from the Figshare deposit (DOI 10.6084/m9.figshare.24452812), then apply sc.pp.normalize_total and sc.pp.log1p to both AnnData objects. Train the STEM SOmodel on GPU for 400 epochs with lr=0.002, sigma=3 (chosen as half the ST spot adjacent distance), alpha (MMD weight)=0.8, mmdbatch=1000, and dropout dp = 1 - median(SC n_genes_by_counts)/median(ST n_genes_by_counts) (~0.584), deriving st2sc/sc2st matrices by softmaxing the dot product of SC and ST embeddings. Run CellTrek, scSpace, Seurat, Spaotsc, and Tangram on the same normalized data and coerce each method's heterogeneous output into a common schema: predicted per-cell spatial coordinates, SC-SC adjacency, and an SC-ST mapping matrix. Compute (1) MAE between predicted and ground-truth coordinates, (2) hit-number@K counting correct SC neighbors among K=200 using ground-truth adjacency, and (3) per-cell-type PCC of cell-type proportions, then assemble all three metrics per method into benchmark_metrics.csv. The decision point is the comparative ranking: STEM was expected to achieve the lowest MAE and roughly two-fold the hit number of the second-best method.

## Related tools

- **STEM** (Primary method under benchmark; trained as SOmodel (400 epochs, lr 0.002, sigma 3, alpha 0.8, dp ~0.584) to produce embeddings and st2sc/sc2st mapping matrices) — https://github.com/WhirlFirst/STEM
- **CellTrek** (Baseline; multivariate random forest model mapping cells to spatial locations, outputs coerced to predicted coordinates)
- **scSpace** (Baseline; MLP predicting absolute spatial coordinates from gene expression)
- **Seurat** (Baseline; not designed for spatial mapping but constructs SC-ST integrated graphs used to derive coordinates)
- **Spaotsc** (Baseline; optimal transport with spatial constraints to learn the SC-ST mapping matrix)
- **Tangram** (Baseline; learns an SC-to-ST mapping matrix optimized by minimizing cosine similarity)
- **scanpy** (Preprocessing shared across all methods: normalize_total, log1p, calculate_qc_metrics (median n_genes_by_counts for dropout rate))
- **PyTorch** (Backend for training STEM SOmodel on GPU (cuda:0) and computing softmax embedding dot-product mapping matrices)

## Examples

```
testmodel.modeleval(); scembedding = testmodel.netE(torch.tensor(scdata.values,dtype=torch.float32).cuda()); stembedding = testmodel.netE(torch.tensor(stdata.values,dtype=torch.float32).cuda()); netsc2st = F.softmax(scembedding.mm(stembedding.t()),dim=1).detach().cpu().numpy()
```

## Evaluation signals

- benchmark_metrics.csv contains one row per method (STEM, CellTrek, scSpace, Seurat, Spaotsc, Tangram) with all three metrics populated (MAE, hit-number@200, per-cell-type PCC)
- STEM's MAE is the lowest among all methods and its hit number is approximately two-fold that of the second-best method, matching the reported finding
- All methods consumed the same sc.pp.normalize_total + sc.pp.log1p-transformed E1z2 inputs, so metric differences are attributable to methods, not preprocessing
- Per-method outputs were successfully unified into the common schema: predicted coordinates, SC-SC adjacency, and SC-ST mapping matrix (softmax rows sum to 1)
- Derived dropout rate dp ≈ 0.584 (1 - median SC n_genes_by_counts / median ST n_genes_by_counts) matches the README value 0.5836734693877551

## Limitations

- Baseline performance depends on unifying heterogeneous outputs (random forest, MLP, integrated graphs, optimal transport mappings) into a common coordinate/adjacency schema; coercing outputs may not reflect each baseline's native best-case behavior
- Sigma was set empirically as half the ST spot adjacent distance — performance is sensitive to this dataset-specific spatial parameter
- The benchmark uses semi-simulation data; applying STEM to SC and ST data from different donors in the same disease state remains untested future work
- Dropout rate dp is derived from median n_genes_by_counts and is dataset-specific (0.584 for E1z2), requiring recomputation for other datasets

## Evidence

- [abstract] abstract: "Semi-simulation and real data experiments verify that the embeddings preserved spatial information and eliminated technical biases between SC and ST data."
- [intro] intro: "CellTrek25 uses a multivariate random forest model to map cells to spatial locations."
- [intro] intro2: "Tangram23 learns a mapping matrix to convert SC to ST, and the matrix is optimized by minimizing the cosine similarity"
- [readme] readme1: "We first config the STEM model and then train it. Empirically we found by setting the sigma as half of the ST spot adjacent distance, STEM achieves the best performance."
- [readme] readme2: "Compared with other methods, STEM is the only method that preserves the original topology structure of all single cells."
- [methods] methods: "The processed data and trained models used for reproducing the results are deposited in [Figshare](https://doi.org/10.6084/m9.figshare.24452812)"
