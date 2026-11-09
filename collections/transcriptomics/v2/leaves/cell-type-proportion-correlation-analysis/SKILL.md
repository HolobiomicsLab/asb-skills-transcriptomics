---
name: cell-type-proportion-correlation-analysis
description: Use when evaluating SC-ST mapping outputs (SC-ST / ST-SC mapping matrices)
  against ground-truth ST cell-type labels, to quantify whether each method preserves
  cell-type composition of spots rather than just coordinate accuracy.
when_to_use_negative:
- Real datasets without ground-truth spot cell-type labels (the paper's PCC metric
  is computed on semi-simulation E1z2 data where ground truth is known)
- When the analysis goal is coordinate-level accuracy alone — use MAE between predicted
  and ground-truth spatial coordinates instead
- When the goal is neighborhood reconstruction fidelity — use hit-number@K (K=200)
  against ground-truth adjacency instead
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: STEM
  role: Produces the netst2sc / netsc2st mapping matrices whose implied cell-type
    proportions are scored; it is the reference method being benchmarked against CellTrek,
    scSpace, Seurat, Spaotsc, Tangram
  repo: https://github.com/WhirlFirst/STEM
- name: CellTrek
  role: Baseline whose mapping output is unified into predicted coordinates and SC-ST
    mapping matrices for PCC comparison
- name: scSpace
  role: Baseline whose mapping output is unified for per-cell-type PCC comparison
- name: Seurat
  role: Integration baseline constructing SC-ST integrated graphs, unified into mapping
    matrices for PCC comparison
- name: Spaotsc
  role: Optimal-transport baseline whose SC-ST mapping matrix feeds the per-cell-type
    PCC metric
- name: Tangram
  role: Baseline learning an SC-to-ST mapping matrix that is scored with per-cell-type
    PCC
- name: scanpy
  role: Preprocessing (normalize_total, log1p, calculate_qc_metrics) and AnnData handling
    of SC/ST inputs before mapping
- name: PyTorch
  role: Trains the STEM SOmodel and softmaxes embedding dot products to build the
    mapping matrices scored by PCC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:3edb1ddec58fe327d0a185e9c7c2624050dcfe9ffce31f335091fdd15b0a1bce
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cell-type-proportion-correlation-analysis

## Summary

Benchmarks SC-ST mapping methods by comparing per-cell-type proportions predicted from a mapping matrix against ground-truth spot cell-type labels using per-cell-type Pearson correlation coefficient (PCC), as done in the STEM paper's semi-simulation E1z2 benchmark alongside MAE and hit-number@K.

## When to use

Use when evaluating SC-ST mapping outputs (SC-ST / ST-SC mapping matrices) against ground-truth ST cell-type labels, to quantify whether each method preserves cell-type composition of spots rather than just coordinate accuracy.

## When NOT to use

- Real datasets without ground-truth spot cell-type labels (the paper's PCC metric is computed on semi-simulation E1z2 data where ground truth is known)
- When the analysis goal is coordinate-level accuracy alone — use MAE between predicted and ground-truth spatial coordinates instead
- When the goal is neighborhood reconstruction fidelity — use hit-number@K (K=200) against ground-truth adjacency instead

## Inputs

- Normalized, log1p-transformed SC count matrix (simu_sc_counts.csv from the Figshare deposit DOI 10.6084/m9.figshare.24452812) as scanpy AnnData
- Normalized, log1p-transformed simulated ST count matrix (simu_st_counts.csv)
- Ground-truth ST cell-type labels (simu_st_celltype.csv)
- ST spatial metadata / coordinates (simu_st_metadata.csv)
- SC-ST and ST-SC mapping matrices (netsc2st / netst2sc) from STEM or one of the five baselines

## Outputs

- Per-cell-type PCC of cell-type proportions for each method
- Rows of the unified benchmark_metrics.csv table alongside MAE and hit-number@K

## How to apply

After preprocessing SC and ST data with sc.pp.normalize_total and sc.pp.log1p, obtain a SC-ST mapping matrix (e.g. netsc2st from softmaxing the dot product of SC and ST embeddings in STEM, or the equivalent unified output from CellTrek, scSpace, Seurat, Spaotsc, or Tangram). Convert the mapping matrix into predicted cell-type proportions per spot (each softmax row sums to 1, so spot-level probabilities over single cells can be aggregated by ground-truth cell type from simu_st_celltype.csv). For every cell type, compute the Pearson correlation coefficient (PCC) between predicted and ground-truth spot proportions using the E1z2 ground-truth ST cell types. Assemble the per-cell-type PCC values together with MAE of predicted spatial coordinates and hit-number@K (K=200, using ground-truth adjacency) into a single per-method metrics table written as benchmark_metrics.csv. The rationale is that a correct mapping should reconstruct both cell-type composition and spatial adjacency, so PCC complements coordinate-level (MAE) and neighborhood-level (hit-number@K) metrics.

## Related tools

- **STEM** (Produces the netst2sc / netsc2st mapping matrices whose implied cell-type proportions are scored; it is the reference method being benchmarked against CellTrek, scSpace, Seurat, Spaotsc, Tangram) — https://github.com/WhirlFirst/STEM
- **CellTrek** (Baseline whose mapping output is unified into predicted coordinates and SC-ST mapping matrices for PCC comparison)
- **scSpace** (Baseline whose mapping output is unified for per-cell-type PCC comparison)
- **Seurat** (Integration baseline constructing SC-ST integrated graphs, unified into mapping matrices for PCC comparison)
- **Spaotsc** (Optimal-transport baseline whose SC-ST mapping matrix feeds the per-cell-type PCC metric)
- **Tangram** (Baseline learning an SC-to-ST mapping matrix that is scored with per-cell-type PCC)
- **scanpy** (Preprocessing (normalize_total, log1p, calculate_qc_metrics) and AnnData handling of SC/ST inputs before mapping)
- **PyTorch** (Trains the STEM SOmodel and softmaxes embedding dot products to build the mapping matrices scored by PCC)

## Examples

```
netst2sc = F.softmax(stembedding.mm(scembedding.t()),dim=1).detach().cpu().numpy(); props = pd.DataFrame(netst2sc, index=stadata.obs_names, columns=adata.obs_names); from scipy.stats import pearsonr; pcc = pearsonr(props.sum(axis=0), gt_props)
```

## Evaluation signals

- Mapping matrices must be stochastic: rows of the softmaxed ST-SC matrix sum to 1 per spot (the probability of one spot to other cells summarizes to 1) and vice versa for netsc2st
- Per-cell-type PCC values must fall in [-1, 1]; the best-performing method in the paper's E1z2 benchmark showed consistently strongest PCC among CellTrek, scSpace, Seurat, Spaotsc, Tangram
- PCC ranking should be reported together with MAE and hit-number@K (K=200) in benchmark_metrics.csv for cross-metric consistency checks
- Predicted per-spot cell-type proportions must be derived from the same preprocessed (normalize_total + log1p) data for all methods so PCC values are comparable

## Limitations

- The PCC metric is only meaningful on semi-simulation data with ground-truth ST cell types (E1z2); on real datasets like hSCC, MTG, or mouse liver there is no such ground truth for this metric
- PCC evaluates cell-type composition only and does not capture single-cell spatial coordinate error or SC-SC adjacency fidelity, which require MAE and hit-number@K respectively
- Applying STEM to SC and ST data from different donors in the same disease state remains untested future work, so PCC-based conclusions may not transfer across donors

## Evidence

- [readme] Tangram-style mapping matrices are built from spatially aware embeddings: "The matrix `netst2sc` and `netsc2st` are ST-SC and SC-ST mapping matrices, respectively. In the ST-SC mapping matrix, the probability of one spot to other cells summarizes to 1."
- [intro] Tangram learns a cosine-similarity-optimized mapping matrix; related baselines are defined analogously: "Tangram23 learns a mapping matrix to convert SC to ST, and the matrix is optimized by minimizing the cosine similarity"
- [readme] Standard preprocessing underpins proportion comparisons: "First, we load the simulated ST data as ST data and the raw SeqFISH data as SC data. Then we normalized and log-scaled these data."
- [abstract] Benchmark relies on semi-simulation with ground truth: "Semi-simulation and real data experiments verify that the embeddings preserved spatial information and eliminated technical biases between SC and ST data."
